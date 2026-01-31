---
title: 'Hadoop3自动部署脚本'
date: 2025-09-26T22:00:03+08:00
draft: false
Tags: ['Hadoop', 'Script']
---

# Hadoop自动化部署shell脚本

## 仓库地址

https://github.com/foryyz/HadoopDeploymentScript

## 运行方式

### 0 虚拟机安装Ubuntu 24 & 设置ip段

**Ubuntu24LTS 清华镜像下载地址：** https://mirrors.tuna.tsinghua.edu.cn/ubuntu-releases/noble/ubuntu-24.04.3-desktop-amd64.iso

用户名使用hadoop，主机名使用node

### 1 把三个文件拷贝到同目录
- `cluster.conf`
- `set-static-ip.sh`
- `install-hadoop-ubuntu24.sh`
### 2 给文件执行权限
```sh
chmod +x sc_1.sh sc_2.sh
```
### 3 完整克隆后打开各虚拟机

### 4 按顺序执行文件

master
```sh
sudo ./sc_1.sh master
```
worker1
```sh
sudo ./sc_1.sh worker1
```
worker2
```sh
sudo ./sc_1.sh worker2
```
master
```sh
sudo ./sc_2.sh
```

### 5 启动集群

```sh
su - hadoop
start-dfs.sh
start-yarn.sh
mapred --daemon start historyserver
jps
```

## 代码

##### sc_1.sh

```sh
#!/usr/bin/env bash
set -Eeuo pipefail

# ============================================================
# sc_1.sh - Ubuntu 24 Hadoop Cluster Prerequisites (per-node)
# - Install SSH/tools
# - Fix clone identity (machine-id + ssh host keys) (optional)
# - Set hostname
# - Configure static IP via netplan
# - Write /etc/hosts for all nodes
# - Prepare SSH keypair for HADOOP_USER
# ============================================================

SCRIPT_NAME="$(basename "$0")"
WORKDIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
LOG_FILE="/var/log/hadoop-deploy-sc_1.log"

# --------------------- Logging ---------------------
log() { echo "[$(date '+%F %T')] $*" | tee -a "$LOG_FILE" >&2; }
die() { log "ERROR: $*"; exit 1; }

on_err() {
  local ec=$?
  log "ERROR: command failed (exit=$ec) at line ${BASH_LINENO[0]}: ${BASH_COMMAND}"
  exit "$ec"
}
trap on_err ERR

# --------------------- Defaults (can be overridden by cluster.conf) ---------------------
CONF_PATH=""
ROLE=""
STATIC_IP=""
OVERRIDE_HOSTNAME=""
OVERRIDE_NET_PREFIX=""
OVERRIDE_NET_CIDR=""
OVERRIDE_GATEWAY=""
OVERRIDE_DNS=""

# --------------------- Helpers ---------------------
require_root() {
  if [[ "${EUID}" -ne 0 ]]; then
    die "请用 root 或 sudo 运行：sudo ./${SCRIPT_NAME} ..."
  fi
}

command_exists() { command -v "$1" >/dev/null 2>&1; }

trim() {
  local s="$1"
  # shellcheck disable=SC2001
  echo "$(echo "$s" | sed 's/^[[:space:]]\+//; s/[[:space:]]\+$//')"
}

# Convert "a,b,c" => array
split_csv_to_array() {
  local csv="$1"
  local -n out_arr="$2"
  IFS=',' read -r -a out_arr <<< "$csv"
}

# --------------------- Config ---------------------
load_config() {
  # default: cluster.conf in script dir
  if [[ -z "${CONF_PATH}" ]]; then
    if [[ -f "${WORKDIR}/cluster.conf" ]]; then
      CONF_PATH="${WORKDIR}/cluster.conf"
    else
      die "找不到配置文件 cluster.conf。请使用 --conf 指定路径，或把 cluster.conf 放到脚本同目录。"
    fi
  fi

  # shellcheck disable=SC1090
  source "${CONF_PATH}"

  # Basic required fields
  : "${HADOOP_USER:?cluster.conf 缺少 HADOOP_USER}"
  : "${MASTER_HOSTNAME:?cluster.conf 缺少 MASTER_HOSTNAME}"
  : "${WORKER1_HOSTNAME:?cluster.conf 缺少 WORKER1_HOSTNAME}"
  : "${WORKER2_HOSTNAME:?cluster.conf 缺少 WORKER2_HOSTNAME}"
  : "${DEFAULT_IP_MASTER:?cluster.conf 缺少 DEFAULT_IP_MASTER}"
  : "${DEFAULT_IP_WORKER1:?cluster.conf 缺少 DEFAULT_IP_WORKER1}"
  : "${DEFAULT_IP_WORKER2:?cluster.conf 缺少 DEFAULT_IP_WORKER2}"
  : "${NET_CIDR:?cluster.conf 缺少 NET_CIDR}"
  : "${GATEWAY:?cluster.conf 缺少 GATEWAY}"
  : "${NETPLAN_RENDERER:?cluster.conf 缺少 NETPLAN_RENDERER}"
  : "${FIX_CLONE_IDENTITY:?cluster.conf 缺少 FIX_CLONE_IDENTITY}"
  : "${CLUSTER_HOSTNAMES:?cluster.conf 缺少 CLUSTER_HOSTNAMES}"
  : "${CLUSTER_IPS:?cluster.conf 缺少 CLUSTER_IPS}"
}

# --------------------- Args ---------------------
usage() {
  cat >&2 <<EOF
用法:
  sudo ./${SCRIPT_NAME} <master|worker1|worker2> --ip <IPv4> [options]

必选:
  role                   节点角色: master / worker1 / worker2
  --ip <IPv4>            本机静态IP

可选:
  --conf <path>          配置文件路径 (默认: 脚本同目录 cluster.conf)
  --hostname <name>      覆盖配置文件中的主机名（不建议，除非你确实要改）
  --cidr <n>             覆盖 NET_CIDR（默认来自 cluster.conf）
  --gateway <IPv4>       覆盖网关（默认来自 cluster.conf）
  --dns <a,b,c>          覆盖 DNS_SERVERS（默认来自 cluster.conf）
  -h, --help             帮助

示例:
  sudo ./${SCRIPT_NAME} master  --ip 192.168.120.10
  sudo ./${SCRIPT_NAME} worker1 --ip 192.168.120.11 --conf /path/cluster.conf
EOF
}

parse_args() {
  if [[ $# -lt 1 ]]; then usage; exit 1; fi

  ROLE="$(trim "$1")"
  shift || true

  while [[ $# -gt 0 ]]; do
    case "$1" in
      --conf) CONF_PATH="$2"; shift 2;;
      --ip) STATIC_IP="$2"; shift 2;;
      --hostname) OVERRIDE_HOSTNAME="$2"; shift 2;;
      --cidr) OVERRIDE_NET_CIDR="$2"; shift 2;;
      --gateway) OVERRIDE_GATEWAY="$2"; shift 2;;
      --dns) OVERRIDE_DNS="$2"; shift 2;;
      -h|--help) usage; exit 0;;
      *) die "未知参数: $1 (用 -h 查看帮助)";;
    esac
  done

  case "${ROLE}" in
    master|worker1|worker2) ;;
    *) die "role 必须是 master/worker1/worker2，你输入的是: ${ROLE}";;
  esac

  [[ -n "${STATIC_IP}" ]] || die "必须提供 --ip <IPv4>"
}

validate_ipv4() {
  local ip="$1"
  if [[ ! "$ip" =~ ^([0-9]{1,3}\.){3}[0-9]{1,3}$ ]]; then
    return 1
  fi
  # check each octet 0-255
  IFS='.' read -r o1 o2 o3 o4 <<< "$ip"
  for o in "$o1" "$o2" "$o3" "$o4"; do
    [[ "$o" -ge 0 && "$o" -le 255 ]] || return 1
  done
  return 0
}

# --------------------- NIC detect ---------------------
detect_primary_nic() {
  # Use default route interface
  local nic
  nic="$(ip route | awk '/default/ {print $5; exit}')"
  [[ -n "${nic}" ]] || die "无法检测默认路由网卡。请确认网络已连接。"
  echo "${nic}"
}

# --------------------- Package install ---------------------
apt_install() {
  local pkgs=("$@")
  export DEBIAN_FRONTEND=noninteractive

  log "apt update..."
  apt-get update -y >/dev/null

  log "apt install: ${pkgs[*]} ..."
  apt-get install -y "${pkgs[@]}" >/dev/null
}

install_base_packages() {
  local pkgs=(openssh-server openssh-client rsync curl wget tar net-tools)
  apt_install "${pkgs[@]}"

  systemctl enable --now ssh >/dev/null 2>&1 || true
  systemctl restart ssh >/dev/null 2>&1 || true
  log "SSH 服务已启用。"
}

# --------------------- Fix clone identity ---------------------
fix_clone_identity() {
  # VMware 完整克隆很可能复制 machine-id 与 ssh host keys
  if [[ "${FIX_CLONE_IDENTITY}" != "true" ]]; then
    log "FIX_CLONE_IDENTITY=false，跳过克隆身份修复。"
    return 0
  fi

  log "修复克隆身份：重建 machine-id + 重新生成 SSH host keys ..."

  # machine-id
  if [[ -f /etc/machine-id ]]; then
    rm -f /etc/machine-id
  fi
  if [[ -f /var/lib/dbus/machine-id ]]; then
    rm -f /var/lib/dbus/machine-id
  fi
  systemd-machine-id-setup >/dev/null

  # ssh host keys
  rm -f /etc/ssh/ssh_host_* || true
  dpkg-reconfigure openssh-server >/dev/null 2>&1 || true
  ssh-keygen -A >/dev/null 2>&1 || true
  systemctl restart ssh >/dev/null 2>&1 || true

  log "克隆身份修复完成。"
}

# --------------------- Hostname ---------------------
desired_hostname_for_role() {
  case "${ROLE}" in
    master) echo "${MASTER_HOSTNAME}" ;;
    worker1) echo "${WORKER1_HOSTNAME}" ;;
    worker2) echo "${WORKER2_HOSTNAME}" ;;
  esac
}

set_hostname() {
  local desired
  if [[ -n "${OVERRIDE_HOSTNAME}" ]]; then
    desired="${OVERRIDE_HOSTNAME}"
  else
    desired="$(desired_hostname_for_role)"
  fi

  local current
  current="$(hostnamectl --static 2>/dev/null || hostname)"

  if [[ "${current}" == "${desired}" ]]; then
    log "主机名已是 ${desired}，跳过。"
    return 0
  fi

  log "设置主机名: ${current} -> ${desired}"
  hostnamectl set-hostname "${desired}"
}

# --------------------- Netplan ---------------------
configure_netplan() {
  local nic="$1"
  local cidr="${NET_CIDR}"
  local gw="${GATEWAY}"
  local -a dns_servers=("${DNS_SERVERS[@]}")

  if [[ -n "${OVERRIDE_NET_CIDR}" ]]; then cidr="${OVERRIDE_NET_CIDR}"; fi
  if [[ -n "${OVERRIDE_GATEWAY}" ]]; then gw="${OVERRIDE_GATEWAY}"; fi
  if [[ -n "${OVERRIDE_DNS}" ]]; then
    dns_servers=()
    split_csv_to_array "${OVERRIDE_DNS}" dns_servers
  fi

  validate_ipv4 "${STATIC_IP}" || die "--ip 不是合法 IPv4：${STATIC_IP}"
  validate_ipv4 "${gw}" || die "GATEWAY 不是合法 IPv4：${gw}"

  for d in "${dns_servers[@]}"; do
    d="$(trim "$d")"
    [[ -n "$d" ]] || continue
    validate_ipv4 "$d" || die "DNS 不是合法 IPv4：$d"
  done

  local file="/etc/netplan/01-hadoop-cluster.yaml"

  log "写入 netplan: ${file} (nic=${nic}, ip=${STATIC_IP}/${cidr})"

  cat > "${file}" <<EOF
network:
  version: 2
  renderer: ${NETPLAN_RENDERER}
  ethernets:
    ${nic}:
      dhcp4: no
      addresses:
        - ${STATIC_IP}/${cidr}
      routes:
        - to: default
          via: ${gw}
      nameservers:
        addresses: [$(printf '%s,' "${dns_servers[@]}" | sed 's/,$//')]
EOF

  chmod 600 "${file}"

  log "netplan apply..."
  netplan apply

  # quick verify
  local newip
  newip="$(ip -4 addr show "${nic}" | awk '/inet / {print $2}' | head -n1 || true)"
  log "当前网卡 ${nic} IPv4: ${newip:-unknown}"
}

# --------------------- /etc/hosts ---------------------
write_etc_hosts() {
  # Create an idempotent block
  local hosts_file="/etc/hosts"
  local begin="# BEGIN HADOOP_CLUSTER_HOSTS"
  local end="# END HADOOP_CLUSTER_HOSTS"

  log "更新 ${hosts_file}（幂等区块）..."

  # Build block content from arrays
  local block="${begin}\n"
  local i
  for i in "${!CLUSTER_HOSTNAMES[@]}"; do
    local hn="${CLUSTER_HOSTNAMES[$i]}"
    local ip="${CLUSTER_IPS[$i]}"
    block+="${ip}\t${hn}\n"
  done
  block+="${end}\n"

  # Remove old block if exists
  if grep -qF "${begin}" "${hosts_file}"; then
    # delete from begin to end inclusive
    sed -i "/${begin}/,/${end}/d" "${hosts_file}"
  fi

  # Ensure localhost lines exist (do not remove existing; just ensure common ones)
  if ! grep -qE '^127\.0\.0\.1\s+localhost' "${hosts_file}"; then
    echo -e "127.0.0.1\tlocalhost" >> "${hosts_file}"
  fi
  if ! grep -qE '^::1\s+localhost' "${hosts_file}"; then
    echo -e "::1\tlocalhost ip6-localhost ip6-loopback" >> "${hosts_file}"
  fi

  # Append block
  echo -e "${block}" >> "${hosts_file}"

  log "/etc/hosts 已写入集群映射："
  echo -e "${block}" | tee -a "$LOG_FILE" >/dev/null
}

# --------------------- SSH keys for HADOOP_USER ---------------------
ensure_hadoop_user_exists() {
  if id -u "${HADOOP_USER}" >/dev/null 2>&1; then
    return 0
  fi

  log "用户 ${HADOOP_USER} 不存在，创建中..."
  useradd -m -s /bin/bash "${HADOOP_USER}"
  # 这里不设置密码（教学环境可自行设置），后续免密通过 ssh-copy-id 推送
  log "已创建用户 ${HADOOP_USER}。"
}

prepare_ssh_keys_for_user() {
  ensure_hadoop_user_exists

  local uhome
  uhome="$(eval echo "~${HADOOP_USER}")"
  local ssh_dir="${uhome}/.ssh"
  local key_file="${ssh_dir}/id_rsa"

  log "为用户 ${HADOOP_USER} 准备 SSH 密钥..."

  mkdir -p "${ssh_dir}"
  chown "${HADOOP_USER}:${HADOOP_USER}" "${ssh_dir}"
  chmod 700 "${ssh_dir}"

  if [[ ! -f "${key_file}" ]]; then
    sudo -u "${HADOOP_USER}" ssh-keygen -t rsa -b 4096 -N "" -f "${key_file}" >/dev/null
    log "已生成 ${key_file}"
  else
    log "密钥已存在，跳过生成：${key_file}"
  fi

  # ensure authorized_keys exists with proper perms
  touch "${ssh_dir}/authorized_keys"
  chown "${HADOOP_USER}:${HADOOP_USER}" "${ssh_dir}/authorized_keys"
  chmod 600 "${ssh_dir}/authorized_keys"
}

# --------------------- Checks ---------------------
system_checks_summary() {
  local nic
  nic="$(detect_primary_nic)"

  log "========== SUMMARY =========="
  log "role            : ${ROLE}"
  log "hostname        : $(hostnamectl --static 2>/dev/null || hostname)"
  log "primary nic     : ${nic}"
  log "expected ip     : ${STATIC_IP}/${NET_CIDR}"
  log "current ip      : $(ip -4 addr show "${nic}" | awk '/inet / {print $2}' | head -n1 || echo 'unknown')"
  log "ssh status      : $(systemctl is-active ssh 2>/dev/null || echo 'unknown')"
  log "hosts resolve   :"
  for h in "${CLUSTER_HOSTNAMES[@]}"; do
    log "  - ${h} => $(getent hosts "${h}" | awk '{print $1}' | head -n1 || echo 'unresolved')"
  done
  log "log file        : ${LOG_FILE}"
  log "============================="
}

# --------------------- Main ---------------------
main() {
  require_root
  parse_args "$@"
  load_config

  # role -> if user didn't set cluster.conf CLUSTER_IPS to match, still okay:
  # /etc/hosts will reflect cluster.conf. The local static IP uses --ip.
  # If you want cluster.conf to track overrides, user can edit cluster.conf manually.

  log "========== ${SCRIPT_NAME} START =========="
  log "使用配置文件: ${CONF_PATH}"
  log "角色: ${ROLE}, 设置静态IP: ${STATIC_IP}"

  install_base_packages
  fix_clone_identity
  set_hostname

  local nic
  nic="$(detect_primary_nic)"
  configure_netplan "${nic}"

  write_etc_hosts
  prepare_ssh_keys_for_user

  system_checks_summary
  log "========== ${SCRIPT_NAME} DONE =========="
}

main "$@"

```

##### sc_2.sh

```sh
#!/usr/bin/env bash
set -Eeuo pipefail

SCRIPT_NAME="$(basename "$0")"
WORKDIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
LOG_FILE="/var/log/hadoop-deploy-sc_2.log"

log() { echo "[$(date '+%F %T')] $*" | tee -a "$LOG_FILE" >&2; }
die() { log "ERROR: $*"; exit 1; }

trap 'ec=$?; log "ERROR(exit=$ec) line ${BASH_LINENO[0]}: ${BASH_COMMAND}"; exit $ec' ERR

CONF_PATH=""
FORCE_REINSTALL="false"

usage() {
  cat >&2 <<EOF
用法:
  sudo ./${SCRIPT_NAME} [--conf /path/cluster.conf] [--force]

说明:
  - 仅在 master 上执行
  - 负责 JDK/Hadoop 下载、安装、生成配置、分发到 worker（远程 root 执行）
  - 不执行 format/start/health_check（将由单独脚本负责）

可选:
  --conf <path>   指定配置文件（默认: 脚本同目录 cluster.conf）
  --force         强制覆盖安装目录（谨慎）
EOF
}

require_root() {
  [[ "${EUID}" -eq 0 ]] || die "请用 root 或 sudo 运行：sudo ./${SCRIPT_NAME} ..."
}

parse_args() {
  while [[ $# -gt 0 ]]; do
    case "$1" in
      --conf) CONF_PATH="$2"; shift 2;;
      --force) FORCE_REINSTALL="true"; shift 1;;
      -h|--help) usage; exit 0;;
      *) die "未知参数: $1";;
    esac
  done
}

apt_install() {
  local pkgs=("$@")
  export DEBIAN_FRONTEND=noninteractive
  log "apt update..."
  apt-get update -y >/dev/null
  log "apt install: ${pkgs[*]} ..."
  apt-get install -y "${pkgs[@]}" >/dev/null
}

load_config() {
  if [[ -z "${CONF_PATH}" ]]; then
    [[ -f "${WORKDIR}/cluster.conf" ]] || die "找不到 cluster.conf，请用 --conf 指定"
    CONF_PATH="${WORKDIR}/cluster.conf"
  fi
  # shellcheck disable=SC1090
  source "${CONF_PATH}"

  : "${HADOOP_USER:?}"
  : "${MASTER_HOSTNAME:?}"
  : "${WORKER1_HOSTNAME:?}"
  : "${WORKER2_HOSTNAME:?}"
  : "${CLUSTER_HOSTNAMES:?}"
  : "${CLUSTER_IPS:?}"

  : "${JDK_DOWNLOAD_LINK:?}"
  : "${HADOOP_DOWNLOAD_LINK:?}"

  : "${INSTALL_BASE:?}"
  : "${JAVA_DIR:?}"
  : "${HADOOP_DIR:?}"
  : "${HADOOP_SYMLINK:?}"

  : "${HADOOP_DATA_DIR:?}"
  : "${HDFS_NAME_DIR:?}"
  : "${HDFS_DATA_DIR:?}"

  : "${FS_DEFAULT_PORT:?}"
  : "${HDFS_REPLICATION:?}"

  : "${SECONDARY_NAMENODE_HOSTNAME:?}"
  : "${JOBHISTORYSERVER_HOSTNAME:?}"
  : "${MAPREDUCE_JOBHISTORY_ADDRESS_PORT:?}"
  : "${MAPREDUCE_JOBHISTORY_WEBAPP_PORT:?}"

  # 新增：root 推送免密模式（推荐 sshpass）
  : "${SSH_PUSH_MODE:?}"          # copy-id | sshpass
  : "${SSH_DEFAULT_PASSWORD:?}"   # 仅 sshpass 时需要（用于克隆默认密码）
}

require_master() {
  local hn
  hn="$(hostnamectl --static 2>/dev/null || hostname)"
  [[ "${hn}" == "${MASTER_HOSTNAME}" ]] || die "只能在 master 执行：当前=${hn} 期望=${MASTER_HOSTNAME}"
}

get_workers() { echo "${WORKER1_HOSTNAME} ${WORKER2_HOSTNAME}"; }

ensure_hadoop_user_exists_local() {
  if id -u "${HADOOP_USER}" >/dev/null 2>&1; then return 0; fi
  useradd -m -s /bin/bash "${HADOOP_USER}"
}

assert_hosts_ready() {
  local h
  for h in "${CLUSTER_HOSTNAMES[@]}"; do
    getent hosts "${h}" >/dev/null 2>&1 || die "无法解析 ${h}，请确认三台已跑 sc_1.sh 并写好 /etc/hosts"
  done
}

# ---- SSH (root -> workers) ----
prepare_root_known_hosts() {
  mkdir -p /root/.ssh
  chmod 700 /root/.ssh
  touch /root/.ssh/known_hosts
  chmod 600 /root/.ssh/known_hosts

  local w
  for w in $(get_workers); do
    ssh-keygen -R "${w}" >/dev/null 2>&1 || true
    ssh-keyscan -H "${w}" >> /root/.ssh/known_hosts 2>/dev/null || true
  done
}

push_root_key_to_workers() {
  # root keypair
  if [[ ! -f /root/.ssh/id_rsa ]]; then
    ssh-keygen -t rsa -b 4096 -N "" -f /root/.ssh/id_rsa >/dev/null
  fi

  local w
  for w in $(get_workers); do
    if [[ "${SSH_PUSH_MODE}" == "copy-id" ]]; then
      log "ssh-copy-id root@${w}（需要输入 root 密码）"
      ssh-copy-id -o StrictHostKeyChecking=yes "root@${w}"
    elif [[ "${SSH_PUSH_MODE}" == "sshpass" ]]; then
      apt_install sshpass >/dev/null
      log "sshpass 推送 root 公钥到 root@${w}"
      sshpass -p "${SSH_DEFAULT_PASSWORD}" ssh-copy-id -o StrictHostKeyChecking=yes "root@${w}"
    else
      die "未知 SSH_PUSH_MODE=${SSH_PUSH_MODE}（只支持 copy-id/sshpass）"
    fi
  done
}

# ---- Download ----
fname() { echo "${1##*/}"; }

download_artifacts() {
  mkdir -p "${INSTALL_BASE}/src"
  local jdk_tar="${INSTALL_BASE}/src/$(fname "${JDK_DOWNLOAD_LINK}")"
  local hdp_tar="${INSTALL_BASE}/src/$(fname "${HADOOP_DOWNLOAD_LINK}")"

  if [[ ! -f "${jdk_tar}" ]]; then
    log "下载 JDK..."
    curl -L --fail -o "${jdk_tar}" "${JDK_DOWNLOAD_LINK}"
  else
    log "JDK 包已存在：${jdk_tar}"
  fi
  if [[ ! -f "${hdp_tar}" ]]; then
    log "下载 Hadoop..."
    curl -L --fail -o "${hdp_tar}" "${HADOOP_DOWNLOAD_LINK}"
  else
    log "Hadoop 包已存在：${hdp_tar}"
  fi

  tar -tzf "${jdk_tar}" >/dev/null
  tar -tzf "${hdp_tar}" >/dev/null

  echo "${jdk_tar}|${hdp_tar}"
}

install_jdk_local() {
  local jdk_tar="$1"
  if [[ -d "${JAVA_DIR}" && "${FORCE_REINSTALL}" != "true" ]]; then
    log "JAVA_DIR 已存在，跳过：${JAVA_DIR}"
  else
    log "安装 JDK 到 ${JAVA_DIR}"
    rm -rf "${JAVA_DIR}"
    mkdir -p "${INSTALL_BASE}/.tmp_jdk"
    rm -rf "${INSTALL_BASE}/.tmp_jdk/*" || true
    tar -xzf "${jdk_tar}" -C "${INSTALL_BASE}/.tmp_jdk"
    local top
    top="$(find "${INSTALL_BASE}/.tmp_jdk" -mindepth 1 -maxdepth 1 -type d | head -n1)"
    [[ -n "${top}" ]] || die "JDK 包结构异常"
    mv "${top}" "${JAVA_DIR}"
    rm -rf "${INSTALL_BASE}/.tmp_jdk"
  fi

  cat > /etc/profile.d/java.sh <<EOF
export JAVA_HOME="${JAVA_DIR}"
export PATH="\$JAVA_HOME/bin:\$PATH"
EOF
  chmod 644 /etc/profile.d/java.sh

  "${JAVA_DIR}/bin/java" -version >/dev/null 2>&1 || die "JDK 验证失败"
  log "JDK 安装完成。"
}

install_hadoop_local() {
  local hdp_tar="$1"
  mkdir -p "${INSTALL_BASE}/.tmp_hadoop"
  rm -rf "${INSTALL_BASE}/.tmp_hadoop/*" || true
  tar -xzf "${hdp_tar}" -C "${INSTALL_BASE}/.tmp_hadoop"
  local top
  top="$(find "${INSTALL_BASE}/.tmp_hadoop" -mindepth 1 -maxdepth 1 -type d | head -n1)"
  [[ -n "${top}" ]] || die "Hadoop 包结构异常"
  local version_dir="${HADOOP_DIR}-$(basename "${top}" | sed 's/^hadoop-//')"

  if [[ -d "${version_dir}" && "${FORCE_REINSTALL}" != "true" ]]; then
    log "Hadoop 已存在，跳过：${version_dir}"
    rm -rf "${INSTALL_BASE}/.tmp_hadoop"
  else
    log "安装 Hadoop 到 ${version_dir}"
    rm -rf "${version_dir}"
    mv "${top}" "${version_dir}"
    rm -rf "${INSTALL_BASE}/.tmp_hadoop"
  fi

  rm -f "${HADOOP_SYMLINK}"
  ln -s "${version_dir}" "${HADOOP_SYMLINK}"

  cat > /etc/profile.d/hadoop.sh <<EOF
export HADOOP_HOME="${HADOOP_SYMLINK}"
export PATH="\$HADOOP_HOME/bin:\$HADOOP_HOME/sbin:\$PATH"
EOF
  chmod 644 /etc/profile.d/hadoop.sh

  chown -R "${HADOOP_USER}:${HADOOP_USER}" "${version_dir}" || true
  log "Hadoop 安装完成：${HADOOP_SYMLINK} -> ${version_dir}"
}

generate_hadoop_configs() {
  local etc_dir="${HADOOP_SYMLINK}/etc/hadoop"
  [[ -d "${etc_dir}" ]] || die "找不到 ${etc_dir}"

  local secondary_port="9868"  # Hadoop3 SecondaryNameNode web default
  cat > "${etc_dir}/core-site.xml" <<EOF
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
  <property>
    <name>fs.defaultFS</name>
    <value>hdfs://${MASTER_HOSTNAME}:${FS_DEFAULT_PORT}</value>
  </property>
  <property>
    <name>hadoop.tmp.dir</name>
    <value>${HADOOP_DATA_DIR}/tmp</value>
  </property>
</configuration>
EOF

  cat > "${etc_dir}/hdfs-site.xml" <<EOF
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
  <property>
    <name>dfs.replication</name>
    <value>${HDFS_REPLICATION}</value>
  </property>
  <property>
    <name>dfs.namenode.name.dir</name>
    <value>file://${HDFS_NAME_DIR}</value>
  </property>
  <property>
    <name>dfs.datanode.data.dir</name>
    <value>file://${HDFS_DATA_DIR}</value>
  </property>
  <property>
    <name>dfs.permissions.enabled</name>
    <value>false</value>
  </property>
  <property>
    <name>dfs.namenode.datanode.registration.ip-hostname-check</name>
    <value>false</value>
  </property>
  <property>
    <name>dfs.namenode.secondary.http-address</name>
    <value>${SECONDARY_NAMENODE_HOSTNAME}:${secondary_port}</value>
  </property>
</configuration>
EOF

  cat > "${etc_dir}/yarn-site.xml" <<EOF
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
  <property>
    <name>yarn.resourcemanager.hostname</name>
    <value>${MASTER_HOSTNAME}</value>
  </property>
  <property>
    <name>yarn.nodemanager.aux-services</name>
    <value>mapreduce_shuffle</value>
  </property>
</configuration>
EOF

  cat > "${etc_dir}/mapred-site.xml" <<EOF
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
  <property>
    <name>mapreduce.framework.name</name>
    <value>yarn</value>
  </property>
  <property>
    <name>mapreduce.jobhistory.address</name>
    <value>${JOBHISTORYSERVER_HOSTNAME}:${MAPREDUCE_JOBHISTORY_ADDRESS_PORT}</value>
  </property>
  <property>
    <name>mapreduce.jobhistory.webapp.address</name>
    <value>${JOBHISTORYSERVER_HOSTNAME}:${MAPREDUCE_JOBHISTORY_WEBAPP_PORT}</value>
  </property>
</configuration>
EOF

  cat > "${etc_dir}/workers" <<EOF
${WORKER1_HOSTNAME}
${WORKER2_HOSTNAME}
EOF

  # JAVA_HOME into hadoop-env.sh (idempotent marker)
  local env_file="${etc_dir}/hadoop-env.sh"
  sed -i '/# BEGIN HADOOP_CLUSTER_JAVA_HOME/,/# END HADOOP_CLUSTER_JAVA_HOME/d' "${env_file}" || true
  cat >> "${env_file}" <<EOF

# BEGIN HADOOP_CLUSTER_JAVA_HOME
export JAVA_HOME="${JAVA_DIR}"
# END HADOOP_CLUSTER_JAVA_HOME
EOF

  chown -R "${HADOOP_USER}:${HADOOP_USER}" "${etc_dir}" || true
  log "Hadoop 配置生成完成。"
}

# ---- Distribute to workers (root) ----
distribute_to_workers_root() {
  local version_dir
  version_dir="$(readlink -f "${HADOOP_SYMLINK}")"

  local w
  for w in $(get_workers); do
    log "=== 分发到 ${w}（root）==="

    # ensure base dirs
    ssh -o BatchMode=yes "root@${w}" "mkdir -p '${INSTALL_BASE}' '${HADOOP_DATA_DIR}' '${HDFS_NAME_DIR}' '${HDFS_DATA_DIR}'"

    # rsync Java + Hadoop version dir
    rsync -az --delete "${JAVA_DIR}/" "root@${w}:${JAVA_DIR}/"
    rsync -az --delete "${version_dir}/" "root@${w}:${version_dir}/"

    # symlink
    ssh -o BatchMode=yes "root@${w}" "rm -f '${HADOOP_SYMLINK}' && ln -s '${version_dir}' '${HADOOP_SYMLINK}'"

    # profile.d
    rsync -az /etc/profile.d/java.sh "root@${w}:/etc/profile.d/java.sh"
    rsync -az /etc/profile.d/hadoop.sh "root@${w}:/etc/profile.d/hadoop.sh"
    ssh -o BatchMode=yes "root@${w}" "chmod 644 /etc/profile.d/java.sh /etc/profile.d/hadoop.sh"

    # ensure hadoop user owns needed dirs
    ssh -o BatchMode=yes "root@${w}" "id -u '${HADOOP_USER}' >/dev/null 2>&1 || useradd -m -s /bin/bash '${HADOOP_USER}'"
    ssh -o BatchMode=yes "root@${w}" "chown -R '${HADOOP_USER}:${HADOOP_USER}' '${HADOOP_DATA_DIR}' '${version_dir}'"

    log "${w} 分发完成。"
  done
}

main() {
  require_root
  parse_args "$@"
  load_config
  require_master

  log "========== ${SCRIPT_NAME} START =========="
  log "conf: ${CONF_PATH}"

  apt_install openssh-client rsync curl wget tar ca-certificates openssh-server
  systemctl enable --now ssh >/dev/null 2>&1 || true

  assert_hosts_ready
  ensure_hadoop_user_exists_local

  prepare_root_known_hosts
  push_root_key_to_workers

  local files
  files="$(download_artifacts)"
  local jdk_tar="${files%%|*}"
  local hdp_tar="${files##*|}"

  install_jdk_local "${jdk_tar}"
  install_hadoop_local "${hdp_tar}"

  # create data dirs local
  mkdir -p "${HADOOP_DATA_DIR}" "${HDFS_NAME_DIR}" "${HDFS_DATA_DIR}"
  chown -R "${HADOOP_USER}:${HADOOP_USER}" "${HADOOP_DATA_DIR}" || true

  generate_hadoop_configs
  distribute_to_workers_root

  log "DONE: sc_2 完成安装+配置+分发（未 format/start/health_check）"
  log "========== ${SCRIPT_NAME} DONE =========="
}

main "$@"

```

##### ctrl_hadoop.sh

```shell
chmod +x ctrl_hadoop.sh

# 首次启动（若未格式化会自动 format）
sudo ./ctrl_hadoop.sh start

# 只格式化（谨慎，会清空元数据）
sudo ./ctrl_hadoop.sh format

# 只做健康检查
sudo ./ctrl_hadoop.sh health

# 停止
sudo ./ctrl_hadoop.sh stop

# 查看状态（jps + yarn node list）
sudo ./ctrl_hadoop.sh status

# 重启
sudo ./ctrl_hadoop.sh restart

# 指定配置文件
sudo ./ctrl_hadoop.sh start --conf /path/to/cluster.conf

```

##### cluster.conf

```conf
# ============================================================
# Hadoop 3.x 完全分布式集群配置文件（Ubuntu 24 + VMware）
# 适配脚本：
#   sc_1.sh  - 系统前置（网络 / SSH / 主机名 / 克隆修复）
#   sc_2.sh  - Hadoop 安装 + 配置 + 分发（不含 format/start）
# ============================================================


# =========================
# 基础用户配置
# =========================
HADOOP_USER="hadoop"
HADOOP_GROUP="hadoop"
TIMEZONE="Asia/Shanghai"


# =========================
# 网络配置（VMware）
# =========================
# VMware 虚拟网络编辑器中设置的网段
NET_PREFIX="192.168.120"
NET_CIDR="24"

# 网关地址（以 VMware NAT/Host-only 实际值为准）
GATEWAY="192.168.120.2"

# DNS（可多个）
DNS_SERVERS=("114.114.114.114" "8.8.8.8")


# =========================
# IP 规划（默认）
# 可在 sc_1.sh 执行时用 --ip 覆盖
# =========================
DEFAULT_IP_MASTER="192.168.120.10"
DEFAULT_IP_WORKER1="192.168.120.11"
DEFAULT_IP_WORKER2="192.168.120.12"


# =========================
# 主机名规划（可随时修改）
# =========================
MASTER_HOSTNAME="master"
WORKER1_HOSTNAME="worker1"
WORKER2_HOSTNAME="worker2"


# =========================
# 集群角色规划
# =========================
# master:
#   - NameNode
#   - ResourceManager
#   - JobHistoryServer
#
# worker1:
#   - DataNode
#   - NodeManager
#   - SecondaryNameNode
#
# worker2:
#   - DataNode
#   - NodeManager

ENABLE_SECONDARY_NAMENODE="true"
SECONDARY_NAMENODE_HOSTNAME="${WORKER1_HOSTNAME}"

ENABLE_JOBHISTORYSERVER="true"
JOBHISTORYSERVER_HOSTNAME="${MASTER_HOSTNAME}"


# =========================
# Hadoop / JDK 下载源
# =========================
JDK_DOWNLOAD_LINK="https://mirrors.tuna.tsinghua.edu.cn/Adoptium/8/jdk/x64/linux/OpenJDK8U-jdk_x64_linux_hotspot_8u472b08.tar.gz"
HADOOP_DOWNLOAD_LINK="https://mirrors.tuna.tsinghua.edu.cn/apache/hadoop/common/hadoop-3.4.2/hadoop-3.4.2.tar.gz"


# =========================
# 安装路径规划
# =========================
# 推荐：程序与数据分离
INSTALL_BASE="/opt"

# Java 安装目录
JAVA_DIR="${INSTALL_BASE}/jdk8"

# Hadoop 版本目录（实际安装目录）
HADOOP_DIR="${INSTALL_BASE}/hadoop"

# Hadoop 当前版本软链接（方便升级）
HADOOP_SYMLINK="${INSTALL_BASE}/hadoop-current"


# =========================
# Hadoop 数据目录
# =========================
HADOOP_DATA_DIR="/data/hadoop"

HDFS_NAME_DIR="${HADOOP_DATA_DIR}/namenode"
HDFS_DATA_DIR="${HADOOP_DATA_DIR}/datanode"


# =========================
# Hadoop 核心参数
# =========================
FS_DEFAULT_PORT="9000"

# 只有 2 个 DataNode，必须是 2
HDFS_REPLICATION="2"


# =========================
# MapReduce JobHistoryServer
# =========================
MAPREDUCE_JOBHISTORY_ADDRESS_PORT="10020"
MAPREDUCE_JOBHISTORY_WEBAPP_PORT="19888"


# =========================
# SSH / 远程分发策略
# =========================
# sc_2.sh 使用 root@worker 分发文件
# copy-id  : 执行时手动输入 root 密码（推荐）
# sshpass  : 全自动（教学环境可用）
SSH_PUSH_MODE="copy-id"

# 仅当 SSH_PUSH_MODE=sshpass 时需要
SSH_DEFAULT_PASSWORD="hadoop"


# =========================
# Root SSH 支持（关键）
# =========================
# Ubuntu 默认 root 无密码，且 SSH 禁止 root 登录
# sc_1.sh 会根据以下配置自动处理

ENABLE_ROOT_SSH="true"

# true  = 允许 root 使用密码登录（用于首次推 key）
# false = 只允许 root 使用密钥登录
ROOT_SSH_PASSWORD_AUTH="true"

# root 默认密码（Ubuntu 必须设置，否则无法 ssh-copy-id）
# ⚠ 教学/实验环境可用，生产环境不推荐
ROOT_DEFAULT_PASSWORD="hadoop"


# =========================
# VMware 克隆修复
# =========================
# 修复 machine-id 与 SSH host key 冲突
FIX_CLONE_IDENTITY="true"


# =========================
# Netplan / Ubuntu 网络
# =========================
NETPLAN_RENDERER="networkd"


# =========================
# 集群节点清单（自动生成 hosts / workers）
# =========================
CLUSTER_HOSTNAMES=(
  "${MASTER_HOSTNAME}"
  "${WORKER1_HOSTNAME}"
  "${WORKER2_HOSTNAME}"
)

CLUSTER_IPS=(
  "${DEFAULT_IP_MASTER}"
  "${DEFAULT_IP_WORKER1}"
  "${DEFAULT_IP_WORKER2}"
)

```

## 操作系统与环境

Ubuntu 24 LTS
Hadoop 3.4.2

## IP 规划

IP段 - 192.168.120.x
Master - 192.168.120.10
Worker1 - 192.168.120.11
Worker2 - 192.168.120.12

## 集群规划

Master

- HDFS: NameNode
- YARN: ResourceManager
- MapReduce: JobHistoryServer

## 参考文献

2020年, [hadoop环境部署自动化shell脚本（伪分布式、完全分布式集群搭建）](https://blog.csdn.net/weixin_42011520/article/details/107571965)

2018年, [使用Shell脚本一键部署Hadoop](https://blog.csdn.net/u010993514/article/details/83349846)
