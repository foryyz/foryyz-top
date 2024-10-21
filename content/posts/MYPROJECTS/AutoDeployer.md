---
title: 'AutoDeployer - Windows环境部署工具'
date: 2024-10-21T21:37:19+08:00
draft: false
---

> 项目开始时间: 2024/10/08
>
> 项目成员: yyz
>
> 最新更新时间: 2024/10/11
>
> 版本号: a0.123

# AutoDeployer

**a0.123**

**项目目的：实现一键的全自动Windows环境部署**

**当前版本支持部署模式：环境变量式 - env_key**



## 项目结构

### run.py - RUN Install - 执行环境安装

### runUninstall.py - RUN Uninstall - 执行环境卸载

#### envManager.py - 目前的主要运行类

- **EnvLoader**
- **EnvChecker**
- **EnvInstaller**
- **EnvUninstaller**

#### logManager.py - 读取used.log文件

- **LogLoader** - 书写使用程序安装过的环境

#### config.yaml - 主要的环境安装配置文件

#### used.log - 保存使用该程序安装过的环境



## 更新日志

```
# - 表示新增加的功能
# / 表示发现的BUG
# ~~*~~ 表示删除线
a0.123	yyz	2024/10/11	*Push
	- 实现 可以为环境添加任意个环境变量,并支持多种方式识别config.yaml中的env_var
	- 优化 控制台输出美化
	
a0.122	yyz	2024/10/11	*Push
	- 修复 卸载环境时单Path添加路径的逻辑错误
	- 注意 在config.yaml文件中添加bin_paths时一定要在末尾加 ;
	- 优化 run.py和runUninstall.py的部分逻辑错误
	# 测试 成功在2台win11电脑正常完成测试任务 Maven, JDK
	# 已知问题：无法为一个环境添加多个变量; 控制台输出需美化
	
a0.121	yyz	2024/10/11
	- 修复 check无法检测log日志文件内的错误
	- 优化 runUninstall.py中的检测是否安装过的算法
	- 修复 检测是否使用该软件安装过环境的逻辑错误
	
a0.120	yyz	2024/10/11
	- 实现 多PATH路径环境的安装和卸载
		→ 重构 安装与卸载部分的核心算法的实现方式
	- 优化 EnvManager各类的各方法返回方式
	- 优化 run.py和runUninstall.py的结构逻辑
	- 优化 EnvInstaller和EnvUninstaller对象的运行方式，由类内返回运行结果

a0.112	yyz	2024/10/11
	- 优化 config.yaml 层级结构，添加变量注释，更改部分变量昵称
		→ check_way 改为 env_key,表示意义为环境变量关键值
			→ env_var-变量名;bin_paths-添加到PATH中的值
		→ download_path 改为 zip_path，此后不同类型的压缩包类型使用不同变量名
		→ install_path 改为 ENVS_PATH，表示软件的默认环境安装路径
		
	- 移除 暂时停止对default_path默认路径安装的检测开发
	
a0.111	yyz	2024/10/10	*Push
	- 优化 如果已经下载过环境压缩包就不再下载
	- 优化 config.yaml文件内路径格式
	- 解决 安装环境时在PATH变量中添加重复值
		# 最后还是没明白第一种算法为什么失败，可能是subprocess的特性问题
	- 解决 Check的$env:?检测问题
	- 解决 powershell中变量不会主动刷新的问题
	→ 以安装JDK为测试，目前可以(完全)实现安装和卸载
	
a0.110	yyz
	- 实现 软件安装后的卸载功能→runUninstall.py
	- 卸载 功能正常,成功删除了[env_var]和在[PATH]中追加的内容
		/会在PATH中遗留重复的内容
	- 优化 对应日志生成
	/ ~~[env_var] EnvCheck系统已存在变量的检测存在问题~~
	/ 添加环境变量的算法存在重复添加原本值的问题;
		/-问题原因 powershell不会主动刷新$env:PATH的值
		/-解决方法 直接对通过powershell对底层进行调用 刷新环境变量$env:?的值
		
a0.100	yyz
	- 实现 JDK环境的安装测试
	- 实现 对使用程序的系统进行环境是否已安装的检测
	- 实现 对已安装的程序进行日志生成
	- 实现 对不同环境的不同安装管理→config.yaml
```



## Python环境

version=3.12.2

- tqdm
- pyyaml



## 附录：

### 1 subprocess库

```python
env_var = "JAVA_HOME"
result = subprocess.run(["powershell", "-Command",f'echo $env:{env_var}'], capture_output=True, text=True, shell=True)
value = result.stdout
#此时value的结尾一定会有一个\n 可以使用[:-1]去掉

#对于subprocess.run的参数：
#	- capture_output=True/False 表示是否将返回输出保存到 stdout,如果设置否则直接输出到控制台
#	- text=True/False 表示返回值为文本
#	- shell=True 允许你在 shell 中执行命令，支持使用 shell 的功能

```



### 2 相关PowerShell指令

```powershell
#查找名为JAVA_HOME的环境变量(当前用户)
Get-ChildItem Env:JAVA_HOME

#显示变量PATH的值(当前用户)
$env:PATH

#查看系统环境变量 - 版本低无法使用
#Get-ChildItem Env: -Scope PATH

```

#### 修改环境变量的值

```powershell
#其中第三个参数[System.EnvironmentVariableTarget]::Target指定了作用范围
#Machine-系统级别,所有用户 ; User-当前用户级别 ; Process-当前进程级
#设置 环境变量[VariableName] 的 值[Value]
[System.Environment]::SetEnvironmentVariable("VariableName", "Value", [System.EnvironmentVariableTarget]::Target)

#例↓
#[System.Environment]::SetEnvironmentVariable('Path', $env:PATH+';C:\AutoDeployToolsEnvs\Envs\jdk-23\bin', [System.EnvironmentVariableTarget]::User)
```

#### 查找环境变量的值

```powershell
[System.Environment]::GetEnvironmentVariable("VariableName",[System.EnvironmentVariableTarget]::User)
#此为查找用户级别的环境变量值
```

#### 手动更新/刷新环境变量

```powershell
$env:Path = [System.Environment]::GetEnvironmentVariable("Path",[System.EnvironmentVariableTarget]::User)
```
