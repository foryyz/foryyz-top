---
title: 'Vcpkg - C++包管理工具的基本配置'
date: 2024-10-25T17:08:47+08:00
draft: false
---

# Vcpkg - C++包管理工具的基本配置

## 1 安装

### 1.1 下载包文件

```shell
git clone https://github.com/microsoft/vcpkg.git
```

### 1.2 执行安装文件

```shell
cd .\vcpkg\
.\bootstrap-vcpkg.bat
# Linux执行bootstrap-vcpkg.sh文件
```

此时，在vcpkg文件夹下会生成**vcpkg.exe**文件

### 1.3 配置环境变量

`VCPKG_ROOT` - `"C:\ENVIRONMENT\CPP\vcpkg"`

`PATH` - 添加 `%VCPKG_ROOT%`



## 2 使用

### 2.1 常用命令

```shell
查找可用的库：vcpkg search <library_name>
查看支持的架构：vcpkg help triplet
指定编译某种架构的程序库：vcpkg install xxxx:x64-windows（x86-windows）
卸载已安装库：vcpkg remove xxxx
指定卸载平台：vcpkg remove xxxx:x64-windows
移除所有旧版本库：vcpkg remove --outdated
查看已经安装的库：vcpkg list
更新已经安装的库：vcpkg update xxx
导出已经安装的库：vcpkg export xxxx --7zip（–7zip –raw –nuget –ifw –zip）
```

### 2.2 Visual Studio 2022 集成

**方式1.**单独为项目添加依赖

```shell
# 创建清单文件 - 生成文件vcpkg.json
vcpkg new --application

#为项目添加依赖 - 此时vcpkg.json会包含fmt
vcpkg add port fmt
```

**方式2.集成进vs，自动导入包**

```shell
集成到全局：vcpkg integrate install
移除全局：vcpkg integrate remove
集成到工程：vcpkg integrate project（在“\scripts\buildsystems”目录下，生成nuget配置文件）
```

### 2.3 (推荐) CMake集成

创建**CMakeLists.txt**文件

```
#CMakeLists.txt文件内容

cmake_minimum_required(VERSION 3.10)
project(HelloWorld)
find_package(fmt CONFIG REQUIRED)
add_executable(HelloWorld helloworld.cpp)
target_link_libraries(HelloWorld PRIVATE fmt::fmt)
```

- `cmake_minimum_required(VERSION 3.10)`：指定生成项目所需的 CMake 最低版本为 3.10。 如果系统上安装的 CMake 版本低于此版本，则将生成错误。
- `project(HelloWorld)`：将项目的名称设置为 "HelloWorld."。
- `find_package(fmt CONFIG REQUIRED)`：使用 `fmt` 库的 CMake 配置文件查找该库。 `REQUIRED` 关键字确保在找不到包时生成错误。
- `add_executable(HelloWorld helloworld.cpp)`：添加从源文件 `helloworld.cpp` 生成的名为 "HelloWorld," 的可执行目标。
- `target_link_libraries(HelloWorld PRIVATE fmt::fmt)`：指定 `HelloWorld` 可执行文件应链接到 `fmt` 库。 `PRIVATE` 关键字表明 `fmt` 仅在生成 `HelloWorld` 时需要，不应传播到其他依赖项目。

这样在使用CMake编译时只需要添加参数`-DCMAKE_TOOLCHAIN_FILE="[path to vcpkg]/scripts/buildsystems/vcpkg.cmake"`

```shell
#例
cmake -B build -DCMAKE_TOOLCHAIN_FILE="C:\ENVIRONMENT\CPP\vcpkg\scripts\buildsystems\vcpkg.cmake"
```

