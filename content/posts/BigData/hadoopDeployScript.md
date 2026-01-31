---
title: '(更新中)Hadoop自动部署脚本'
date: 2025-09-26T22:00:03+08:00
draft: false
Tags: ['Hadoop', 'Script']
---

# Hadoop自动化部署shell脚本

## 仓库地址

https://github.com/foryyz/HadoopDeploymentScript

## 运行方式
#### 1 把三个文件拷贝到同目录
- `cluster.conf`
- `set-static-ip.sh`
- `install-hadoop-ubuntu24.sh`
#### 2 给文件执行权限
```sh
chmod +x set-static-ip.sh install-hadoop-ubuntu24.sh
```
#### 3 按顺序执行文件
master
```sh
sudo ./set-static-ip.sh master
```
worker1
```sh
sudo ./set-static-ip.sh worker1
```
worker2
```sh
sudo ./set-static-ip.sh worker2
```
master
```sh
sudo ./install-hadoop-ubuntu24.sh master
```

## 操作系统与环境
Ubuntu 24
Hadoop 3.4.2

## 集群规划
Master
- HDFS: NameNode
- YARN: ResourceManager
- MapReduce: JobHistoryServer
## IP 规划
IP段 - 192.168.120.x
Master - 192.168.120.10
Worker1 - 192.168.120.11
Worker2 - 192.168.120.12

## 代码

##### install-hadoop-ubuntu24.sh

```sh
#!/usr/bin/env bash
set -euo pipefail

ROLE="${1:-}"
shift || true

CONF_PATH="./cluster.conf"

usage() {
  echo "Usage:"
  echo "  sudo $0 master [--conf /path/to/cluster.conf]"
  echo "  sudo $0 worker [--conf /path/to/cluster.conf]"
  exit 1
}

log() { echo -e "[INFO] $*"; }
err() { echo -e "[ERROR] $*" >&2; exit 1; }

need_root() {
  if [[ "${EUID:-$(id -u)}" -ne 0 ]]; then
    err "Please run as root (sudo)."
  fi
}

load_conf() {
  if [[ -f "${CONF_PATH}" ]]; then
    # shellcheck disable=SC1090
    source "${CONF_PATH}"
  else
    err "cluster.conf not found: ${CONF_PATH}"
  fi
}

# parse args
[[ "${ROLE}" == "master" || "${ROLE}" == "worker" ]] || usage
while [[ $# -gt 0 ]]; do
  case "$1" in
    --conf) CONF_PATH="${2:-}"; shift 2 ;;
    *) err "Unknown arg: $1" ;;
  esac
done

ensure_packages() {
  log "Installing base packages..."
  apt-get update -y
  apt-get install -y openssh-server openssh-client rsync curl wget tar net-tools
}

disable_ufw() {
  if command -v ufw >/dev/null 2>&1; then
    log "Disabling ufw (if enabled)..."
    ufw disable || true
  fi
}

ensure_user() {
  if id "${HADOOP_USER}" >/dev/null 2>&1; then
    log "User ${HADOOP_USER} exists."
  else
    log "Creating user ${HADOOP_USER}..."
    useradd -m -s /bin/bash "${HADOOP_USER}"
    usermod -aG sudo "${HADOOP_USER}" || true
    echo "${HADOOP_USER} ALL=(ALL) NOPASSWD:ALL" >/etc/sudoers.d/99-hadoop-nopasswd
    chmod 440 /etc/sudoers.d/99-hadoop-nopasswd
  fi
}

write_hosts() {
  log "Writing /etc/hosts from cluster.conf..."
  sed -i '/#HADOOP_CLUSTER_BEGIN/,/#HADOOP_CLUSTER_END/d' /etc/hosts || true
  {
    echo "#HADOOP_CLUSTER_BEGIN"
    echo -e "${MASTER_IP}\t${MASTER_HOST}"
    echo -e "${WORKER1_IP}\t${WORKER1_HOST}"
    echo -e "${WORKER2_IP}\t${WORKER2_HOST}"
    echo "#HADOOP_CLUSTER_END"
  } >> /etc/hosts
}

download_and_install_jdk() {
  if [[ -d "${JDK_DIR}" && -x "${JDK_DIR}/bin/java" ]]; then
    log "JDK already installed at ${JDK_DIR}"
    return
  fi
  log "Downloading JDK..."
  mkdir -p /tmp/hadoop_setup
  cd /tmp/hadoop_setup
  wget -O jdk8.tar.gz "${JDK_DOWNLOAD_LINK}"
  log "Installing JDK to ${JDK_DIR}..."
  rm -rf "${JDK_DIR}"
  mkdir -p "${INSTALL_BASE}"
  tar -xzf jdk8.tar.gz -C "${INSTALL_BASE}"
  local top
  top="$(tar -tzf jdk8.tar.gz | head -1 | cut -d/ -f1)"
  rm -rf "${JDK_DIR}"
  mv "${INSTALL_BASE}/${top}" "${JDK_DIR}"
}

download_and_install_hadoop() {
  if [[ -d "${HADOOP_DIR}" && -x "${HADOOP_DIR}/bin/hdfs" ]]; then
    log "Hadoop already installed at ${HADOOP_DIR}"
    return
  fi
  log "Downloading Hadoop..."
  mkdir -p /tmp/hadoop_setup
  cd /tmp/hadoop_setup
  wget -O hadoop.tar.gz "${HADOOP_DOWNLOAD_LINK}"
  log "Installing Hadoop to ${HADOOP_DIR}..."
  rm -rf "${HADOOP_DIR}"
  mkdir -p "${INSTALL_BASE}"
  tar -xzf hadoop.tar.gz -C "${INSTALL_BASE}"
  local top
  top="$(tar -tzf hadoop.tar.gz | head -1 | cut -d/ -f1)"
  mv "${INSTALL_BASE}/${top}" "${HADOOP_DIR}"
}

write_env() {
  local env_sh="/etc/profile.d/hadoop_env.sh"
  log "Writing env to ${env_sh}..."
  cat > "${env_sh}" <<EOF
export JAVA_HOME=${JDK_DIR}
export HADOOP_HOME=${HADOOP_DIR}
export HADOOP_CONF_DIR=\$HADOOP_HOME/etc/hadoop
export PATH=\$PATH:\$JAVA_HOME/bin:\$HADOOP_HOME/bin:\$HADOOP_HOME/sbin
EOF
  chmod 644 "${env_sh}"
}

apply_hadoop_java_home() {
  local envfile="${HADOOP_DIR}/etc/hadoop/hadoop-env.sh"
  log "Setting JAVA_HOME in ${envfile}..."
  grep -q "^export JAVA_HOME=" "${envfile}" \
    && sed -i "s|^export JAVA_HOME=.*|export JAVA_HOME=${JDK_DIR}|" "${envfile}" \
    || echo "export JAVA_HOME=${JDK_DIR}" >> "${envfile}"
}

make_data_dirs() {
  log "Creating data dirs under ${DATA_BASE}..."
  mkdir -p "${DATA_BASE}/namenode" "${DATA_BASE}/datanode" "${DATA_BASE}/tmp"
  chown -R "${HADOOP_USER}:${HADOOP_USER}" "${DATA_BASE}"
}

write_hadoop_configs_master() {
  log "Writing Hadoop configs (3 nodes, <=4GB, Secondary on worker1, HistoryServer on master)..."
  local conf="${HADOOP_DIR}/etc/hadoop"

  cat > "${conf}/core-site.xml" <<EOF
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
  <property>
    <name>fs.defaultFS</name>
    <value>hdfs://${MASTER_IP}:9000</value>
  </property>
  <property>
    <name>hadoop.tmp.dir</name>
    <value>${DATA_BASE}/tmp</value>
  </property>
</configuration>
EOF

  cat > "${conf}/hdfs-site.xml" <<EOF
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
  <property>
    <name>dfs.replication</name>
    <value>${DFS_REPLICATION}</value>
  </property>

  <property>
    <name>dfs.namenode.name.dir</name>
    <value>file:${DATA_BASE}/namenode</value>
  </property>

  <property>
    <name>dfs.datanode.data.dir</name>
    <value>file:${DATA_BASE}/datanode</value>
  </property>

  <property>
    <name>dfs.permissions.enabled</name>
    <value>false</value>
  </property>

  <property>
    <name>dfs.namenode.datanode.registration.ip-hostname-check</name>
    <value>false</value>
  </property>

  <!-- SecondaryNameNode on worker1 -->
  <property>
    <name>dfs.namenode.secondary.http-address</name>
    <value>${SECONDARY_IP}:9868</value>
  </property>
</configuration>
EOF

  cat > "${conf}/mapred-site.xml" <<EOF
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
  <property>
    <name>mapreduce.framework.name</name>
    <value>yarn</value>
  </property>

  <!-- JobHistoryServer on master -->
  <property>
    <name>mapreduce.jobhistory.address</name>
    <value>${HISTORYSERVER_IP}:10020</value>
  </property>
  <property>
    <name>mapreduce.jobhistory.webapp.address</name>
    <value>${HISTORYSERVER_IP}:19888</value>
  </property>

  <!-- <=4GB VM: conservative -->
  <property>
    <name>mapreduce.map.memory.mb</name>
    <value>${MR_MAP_MB}</value>
  </property>
  <property>
    <name>mapreduce.reduce.memory.mb</name>
    <value>${MR_REDUCE_MB}</value>
  </property>
  <property>
    <name>mapreduce.map.java.opts</name>
    <value>-Xmx${MR_JAVA_XMX}</value>
  </property>
  <property>
    <name>mapreduce.reduce.java.opts</name>
    <value>-Xmx${MR_JAVA_XMX}</value>
  </property>
  <property>
    <name>yarn.app.mapreduce.am.resource.mb</name>
    <value>${MR_AM_MB}</value>
  </property>
  <property>
    <name>yarn.app.mapreduce.am.command-opts</name>
    <value>-Xmx${MR_JAVA_XMX}</value>
  </property>
</configuration>
EOF

  cat > "${conf}/yarn-site.xml" <<EOF
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
  <property>
    <name>yarn.resourcemanager.hostname</name>
    <value>${MASTER_IP}</value>
  </property>

  <property>
    <name>yarn.nodemanager.aux-services</name>
    <value>mapreduce_shuffle</value>
  </property>

  <property>
    <name>yarn.nodemanager.resource.memory-mb</name>
    <value>${YARN_NM_MEMORY_MB}</value>
  </property>
  <property>
    <name>yarn.scheduler.maximum-allocation-mb</name>
    <value>${YARN_MAX_ALLOC_MB}</value>
  </property>
  <property>
    <name>yarn.scheduler.minimum-allocation-mb</name>
    <value>${YARN_MIN_ALLOC_MB}</value>
  </property>

  <property>
    <name>yarn.nodemanager.resource.cpu-vcores</name>
    <value>${YARN_NM_VCORES}</value>
  </property>
  <property>
    <name>yarn.scheduler.maximum-allocation-vcores</name>
    <value>${YARN_NM_VCORES}</value>
  </property>
</configuration>
EOF

  : > "${conf}/workers"
  for ip in ${WORKERS_IPS}; do
    echo "${ip}" >> "${conf}/workers"
  done

  chown -R "${HADOOP_USER}:${HADOOP_USER}" "${HADOOP_DIR}"
}

setup_ssh_keys_master() {
  log "Setting up passwordless SSH for ${HADOOP_USER} on master..."
  sudo -u "${HADOOP_USER}" mkdir -p "/home/${HADOOP_USER}/.ssh"
  sudo -u "${HADOOP_USER}" chmod 700 "/home/${HADOOP_USER}/.ssh"
  if [[ ! -f "/home/${HADOOP_USER}/.ssh/id_rsa" ]]; then
    sudo -u "${HADOOP_USER}" ssh-keygen -t rsa -b 4096 -N "" -f "/home/${HADOOP_USER}/.ssh/id_rsa"
  fi
  sudo -u "${HADOOP_USER}" bash -c 'cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys'
  sudo -u "${HADOOP_USER}" chmod 600 "/home/${HADOOP_USER}/.ssh/authorized_keys"
}

distribute_to_workers_master() {
  log "Distributing JDK/Hadoop/config to workers (via root ssh/scp)..."
  local tarball="/tmp/hadoop_dist.tar.gz"
  tar -czf "${tarball}" -C / "${JDK_DIR#/}" "${HADOOP_DIR#/}" "etc/profile.d/hadoop_env.sh"

  for ip in ${WORKERS_IPS}; do
    log "==> Sending to ${ip}"
    scp -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null "${0}" "root@${ip}:/root/install-hadoop-ubuntu24.sh"
    scp -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null "${CONF_PATH}" "root@${ip}:/root/cluster.conf"
    scp -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null "${tarball}" "root@${ip}:/tmp/hadoop_dist.tar.gz"
    ssh -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null "root@${ip}" \
      "bash /root/install-hadoop-ubuntu24.sh worker --conf /root/cluster.conf"
  done

  rm -f "${tarball}"
}

unpack_dist_worker() {
  if [[ -f /tmp/hadoop_dist.tar.gz ]]; then
    log "Unpacking distributed package..."
    tar -xzf /tmp/hadoop_dist.tar.gz -C /
    rm -f /tmp/hadoop_dist.tar.gz
  fi
}

format_namenode_master() {
  log "Formatting NameNode (only once)..."
  # shellcheck disable=SC1091
  source /etc/profile.d/hadoop_env.sh
  sudo -u "${HADOOP_USER}" "${HADOOP_DIR}/bin/hdfs" namenode -format -force
}

main() {
  need_root
  load_conf

  ensure_packages
  disable_ufw
  ensure_user
  write_hosts

  if [[ "${ROLE}" == "master" ]]; then
    download_and_install_jdk
    download_and_install_hadoop
    write_env
    apply_hadoop_java_home
    make_data_dirs
    write_hadoop_configs_master
    setup_ssh_keys_master

    log "NOTE: first distribution may ask for worker root passwords during ssh/scp."
    distribute_to_workers_master
    format_namenode_master

    log "Master done. Next on master:"
    log "  su - ${HADOOP_USER}"
    log "  start-dfs.sh"
    log "  start-yarn.sh"
    log "  mapred --daemon start historyserver"
    log "  jps"
  else
    unpack_dist_worker
    # fallback download if dist missing
    download_and_install_jdk || true
    download_and_install_hadoop || true
    write_env
    apply_hadoop_java_home
    make_data_dirs
    chown -R "${HADOOP_USER}:${HADOOP_USER}" "${HADOOP_DIR}" || true
    log "Worker done."
  fi
}

main

```

##### set-static-ip.sh

```sh
#!/usr/bin/env bash
set -euo pipefail

# Usage:
#   sudo ./set-static-ip.sh master
#   sudo ./set-static-ip.sh worker1
#   sudo ./set-static-ip.sh worker2
#
# Optional:
#   sudo ./set-static-ip.sh master --conf /opt/cluster/cluster.conf

ROLE="${1:-}"
shift || true

CONF_PATH="./cluster.conf"

usage() {
  echo "Usage: sudo $0 <master|worker1|worker2> [--conf /path/to/cluster.conf]"
  exit 1
}

need_root() {
  if [[ "${EUID:-$(id -u)}" -ne 0 ]]; then
    echo "[ERROR] Please run as root (sudo)." >&2
    exit 1
  fi
}

valid_ipv4() {
  local ip="$1"
  [[ "$ip" =~ ^([0-9]{1,3}\.){3}[0-9]{1,3}$ ]] || return 1
  local o1 o2 o3 o4
  IFS='.' read -r o1 o2 o3 o4 <<<"$ip"
  for o in "$o1" "$o2" "$o3" "$o4"; do
    [[ "$o" -ge 0 && "$o" -le 255 ]] || return 1
  done
  return 0
}

detect_iface() {
  local iface
  iface="$(ip -4 route show default 2>/dev/null | awk '{for(i=1;i<=NF;i++) if($i=="dev"){print $(i+1); exit}}')"
  [[ -n "${iface}" ]] && { echo "${iface}"; return 0; }

  iface="$(ip -o -4 addr show | awk '$2!="lo"{print $2; exit}')"
  [[ -n "${iface}" ]] && { echo "${iface}"; return 0; }

  echo ""
  return 1
}

detect_gateway() {
  ip -4 route show default 2>/dev/null | awk '{print $3; exit}'
}

backup_netplan() {
  mkdir -p /root/netplan-backup
  local ts
  ts="$(date +%Y%m%d-%H%M%S)"
  cp -a /etc/netplan "/root/netplan-backup/netplan-${ts}" 2>/dev/null || true
  echo "[INFO] Backup: /root/netplan-backup/netplan-${ts}"
}

set_hostname() {
  local newname="$1"
  echo "[INFO] Setting hostname: ${newname}"
  hostnamectl set-hostname "${newname}"

  if grep -qE '^\s*127\.0\.1\.1\s+' /etc/hosts; then
    sed -i "s/^\s*127\.0\.1\.1\s\+.*/127.0.1.1\t${newname}/" /etc/hosts
  else
    echo -e "127.0.1.1\t${newname}" >> /etc/hosts
  fi
}

write_netplan() {
  local iface="$1"
  local ip="$2"
  local cidr="$3"
  local gw="$4"
  local dns_csv="$5"

  local outfile="/etc/netplan/99-static-ip.yaml"
  local dns_yaml="[]"

  if [[ -n "${dns_csv}" ]]; then
    IFS=',' read -ra dns_arr <<<"${dns_csv}"
    local joined=""
    for d in "${dns_arr[@]}"; do
      d="$(echo "$d" | xargs)"
      [[ -z "$d" ]] && continue
      valid_ipv4 "$d" || { echo "[ERROR] Invalid DNS: $d" >&2; exit 1; }
      if [[ -z "$joined" ]]; then
        joined="\"$d\""
      else
        joined="${joined}, \"$d\""
      fi
    done
    dns_yaml="[${joined}]"
  fi

  cat > "${outfile}" <<EOF
# Generated by set-static-ip.sh (cluster.conf)
network:
  version: 2
  renderer: networkd
  ethernets:
    ${iface}:
      dhcp4: no
      addresses:
        - ${ip}/${cidr}
      routes:
        - to: default
          via: ${gw}
      nameservers:
        addresses: ${dns_yaml}
EOF

  chmod 600 "${outfile}"
  echo "[INFO] Wrote: ${outfile}"
}

apply_netplan() {
  echo "[WARN] Applying netplan (may disconnect SSH)..."
  netplan generate
  netplan apply
  echo "[INFO] Netplan applied."
}

load_conf() {
  if [[ -f "${CONF_PATH}" ]]; then
    # shellcheck disable=SC1090
    source "${CONF_PATH}"
  else
    echo "[ERROR] cluster.conf not found: ${CONF_PATH}" >&2
    exit 1
  fi
}

# parse args
[[ "${ROLE}" == "master" || "${ROLE}" == "worker1" || "${ROLE}" == "worker2" ]] || usage
while [[ $# -gt 0 ]]; do
  case "$1" in
    --conf) CONF_PATH="${2:-}"; shift 2 ;;
    *) echo "[ERROR] Unknown arg: $1" >&2; usage ;;
  esac
done

main() {
  need_root
  load_conf

  local host="" ip="" cidr="${CIDR:-24}" dns="${DNS:-8.8.8.8,114.114.114.114}"
  case "${ROLE}" in
    master) host="${MASTER_HOST}"; ip="${MASTER_IP}" ;;
    worker1) host="${WORKER1_HOST}"; ip="${WORKER1_IP}" ;;
    worker2) host="${WORKER2_HOST}"; ip="${WORKER2_IP}" ;;
  esac

  [[ -n "${host}" && -n "${ip}" ]] || { echo "[ERROR] Missing host/ip mapping in cluster.conf" >&2; exit 1; }
  valid_ipv4 "${ip}" || { echo "[ERROR] Invalid IP in conf: ${ip}" >&2; exit 1; }

  local iface
  iface="$(detect_iface)" || true
  [[ -n "${iface}" ]] || { echo "[ERROR] Cannot detect interface. Ensure network is up." >&2; exit 1; }

  local gw="${GATEWAY:-}"
  if [[ -z "${gw}" ]]; then
    gw="$(detect_gateway)"
  fi
  if [[ -z "${gw}" ]]; then
    # fallback: prefix.2
    [[ -n "${NET_PREFIX:-}" ]] || { echo "[ERROR] NET_PREFIX missing in conf and gateway not detected" >&2; exit 1; }
    gw="${NET_PREFIX}.2"
  fi
  valid_ipv4 "${gw}" || { echo "[ERROR] Invalid gateway: ${gw}" >&2; exit 1; }

  echo "[INFO] Using conf: ${CONF_PATH}"
  echo "[INFO] Role     : ${ROLE}"
  echo "[INFO] Hostname : ${host}"
  echo "[INFO] IFACE    : ${iface}"
  echo "[INFO] IP/CIDR  : ${ip}/${cidr}"
  echo "[INFO] GW       : ${gw}"
  echo "[INFO] DNS      : ${dns}"

  backup_netplan
  set_hostname "${host}"
  write_netplan "${iface}" "${ip}" "${cidr}" "${gw}" "${dns}"
  apply_netplan

  echo "[INFO] Current IP:"
  ip -4 addr show "${iface}" || true
  echo "[INFO] Default route:"
  ip -4 route show default || true
  echo "[INFO] Done."
}

main

```

##### cluster.conf

```conf
# =========================
# cluster.conf (Ubuntu 24 + VMware)
# =========================

# --- Network ---
NET_PREFIX="192.168.120"
CIDR="24"
# Gateway: 如果留空，脚本会从当前默认路由自动检测；检测不到则 fallback 为 ${NET_PREFIX}.2
GATEWAY=""
DNS="8.8.8.8,114.114.114.114"

# --- Nodes (hostname must match role argument) ---
MASTER_HOST="master"
MASTER_IP="192.168.120.10"

WORKER1_HOST="worker1"
WORKER1_IP="192.168.120.11"

WORKER2_HOST="worker2"
WORKER2_IP="192.168.120.12"

# Workers list (space-separated IPs)
WORKERS_IPS="192.168.120.11 192.168.120.12"

# SecondaryNameNode placement
SECONDARY_HOST="worker1"
SECONDARY_IP="192.168.120.11"

# --- Hadoop/JDK download links ---
JDK_DOWNLOAD_LINK="https://mirrors.tuna.tsinghua.edu.cn/Adoptium/8/jdk/x64/linux/OpenJDK8U-jdk_x64_linux_hotspot_8u472b08.tar.gz"
HADOOP_DOWNLOAD_LINK="https://mirrors.tuna.tsinghua.edu.cn/apache/hadoop/common/hadoop-3.4.2/hadoop-3.4.2.tar.gz"

# --- Install paths ---
HADOOP_USER="hadoop"
INSTALL_BASE="/opt"
JDK_DIR="/opt/jdk8"
HADOOP_DIR="/opt/hadoop"
DATA_BASE="/data/hadoop"

# --- Cluster tuning for <=4GB VMs ---
DFS_REPLICATION="2"

# NodeManager resources
YARN_NM_MEMORY_MB="3072"
YARN_NM_VCORES="2"
YARN_MIN_ALLOC_MB="512"
YARN_MAX_ALLOC_MB="3072"

# MapReduce default container sizes
MR_MAP_MB="1024"
MR_REDUCE_MB="1024"
MR_AM_MB="1024"
MR_JAVA_XMX="768m"

# JobHistoryServer on master
HISTORYSERVER_HOST="master"
HISTORYSERVER_IP="192.168.120.10"

```



## 参考文献

2020年, [hadoop环境部署自动化shell脚本（伪分布式、完全分布式集群搭建）](https://blog.csdn.net/weixin_42011520/article/details/107571965)

2018年, [使用Shell脚本一键部署Hadoop](https://blog.csdn.net/u010993514/article/details/83349846)
