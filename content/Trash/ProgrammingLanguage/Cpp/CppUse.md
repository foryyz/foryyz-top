---
title: 'C++ - 一些操作'
date: 2024-11-14T23:43:17+08:00
draft: false
tags: ['C++']
---

# for C++ Use

## 1 (注册表方式)使用Windows头文件修改环境变量

​	**(AutoDeployer_Cpp)常用使用方式: **[AddWindowsEnvironment.h foryyz](https://github.com/foryyz/Programming-Basic-Projects/blob/main/Cpp_USE_Function/AddWindowsEnvironment.h)

### 1.1 添加环境变量 ENV: KEY - VALUE

#### 1.1.1 用户级别

```c++
#include <iostream>
#include <windows.h>
#include <string>

// 修改用户级别
bool setUserEnvironmentVariable(const std::wstring& name, const std::wstring& value) {
    HKEY hKey;
    // 打开用户环境变量的注册表键
    if (RegOpenKeyExW(HKEY_CURRENT_USER, L"Environment", 0, KEY_SET_VALUE, &hKey) != ERROR_SUCCESS) {
        std::wcerr << L"Failed to open registry key for user environment variables." << std::endl;
        return false;
    }

    // 设置环境变量的值
    if (RegSetValueExW(hKey, name.c_str(), 0, REG_EXPAND_SZ, reinterpret_cast<const BYTE*>(value.c_str()), (value.size() + 1) * sizeof(wchar_t)) != ERROR_SUCCESS) {
        std::wcerr << L"Failed to set registry value for: " << name << std::endl;
        RegCloseKey(hKey);
        return false;
    }

    RegCloseKey(hKey);
    // 通知系统更新环境变量
    SendMessageTimeoutW(HWND_BROADCAST, WM_SETTINGCHANGE, 0, reinterpret_cast<LPARAM>(L"Environment"), SMTO_ABORTIFHUNG, 5000, nullptr);
    return true;
}
```

#### 1.1.2 系统级别

```c++
// 修改系统级别环境变量
bool setPermanentEnvironmentVariable(const std::wstring& name, const std::wstring& value) {
    HKEY hKey;
    // 打开注册表键：SYSTEM 环境变量位于 HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Session Manager\Environment
    if (RegOpenKeyExW(HKEY_LOCAL_MACHINE, L"SYSTEM\\CurrentControlSet\\Control\\Session Manager\\Environment", 0, KEY_SET_VALUE, &hKey) != ERROR_SUCCESS) {
        std::wcerr << L"Failed to open registry key for environment variables." << std::endl;
        return false;
    }
    // 设置环境变量的值
    if (RegSetValueExW(hKey, name.c_str(), 0, REG_EXPAND_SZ, reinterpret_cast<const BYTE*>(value.c_str()), (value.size() + 1) * sizeof(wchar_t)) != ERROR_SUCCESS) {
        std::wcerr << L"Failed to set registry value for: " << name << std::endl;
        RegCloseKey(hKey);
        return false;
    }
    
    RegCloseKey(hKey);
    // 通知系统更新环境变量
    SendMessageTimeoutW(HWND_BROADCAST, WM_SETTINGCHANGE, 0, reinterpret_cast<LPARAM>(L"Environment"), SMTO_ABORTIFHUNG, 5000, nullptr);
    return true;
}
```

#### 1.1.3 主函数

```c++
// 主函数
int main() {
    setUserEnvironmentVariable(L"JAVA_HOME", L"C:\\Path\\To\\Javahome"); // 以用户级别举例
    system("pause");
    return 0;
}
```

### 1.2 获取环境变量的值

#### 1.2.1 获取变量name的值

```c++
std::wstring getEnvironmentVariable(const std::wstring& name) {
    // 获取环境变量的值
    wchar_t buffer[32767];
    DWORD size = GetEnvironmentVariableW(name.c_str(), buffer, 32767);
    if (size == 0) {
        return L"";
    }
    return std::wstring(buffer, size);
}

// 获取现有的PATH环境变量
std::wstring currentPath = getEnvironmentVariable(L"PATH");
std::wcout << currentPath;
```

#### 1.2.2 获取PATH的值

##### (推荐)方式1. 通过注册表读取(用户级)

```c++
std::wstring getUserPathFromRegistry() {
    HKEY hKey;
    wchar_t buffer[32767];  // 用于存储读取的路径
    DWORD bufferSize = sizeof(buffer);

    // 打开 HKEY_CURRENT_USER\Environment 注册表键
    if (RegOpenKeyExW(HKEY_CURRENT_USER, L"Environment", 0, KEY_READ, &hKey) != ERROR_SUCCESS) {
        std::wcerr << L"Failed to open registry key for user environment variables." << std::endl;
        return L"";
    }

    // 从注册表中获取 PATH 变量的值
    if (RegQueryValueExW(hKey, L"PATH", nullptr, nullptr, reinterpret_cast<BYTE*>(buffer), &bufferSize) != ERROR_SUCCESS) {
        std::wcerr << L"Failed to read PATH from registry." << std::endl;
        RegCloseKey(hKey);
        return L"";
    }

    RegCloseKey(hKey);
    return std::wstring(buffer);
}

std::wstring userPath = getUserPathFromRegistry();
```

##### 方式2. 通过方法获取(进程级)

​	注意：这种方式获得的PATH值=用户级+系统级

```c++
std::wstring getCurrentPath() {
    wchar_t buffer[32767];  // 读取当前 PATH 的缓冲区
    DWORD size = GetEnvironmentVariableW(L"PATH", buffer, sizeof(buffer) / sizeof(wchar_t));
    if (size == 0) {
        std::wcerr << L"Failed to get current PATH." << std::endl;
        return L"";
    }
    return std::wstring(buffer, size);
}

std::wstring currentPath = getCurrentPath();
```

### 1.3 添加路径至PATH

​	注：此处使用了[通过注册表读取PATH值的方法](#推荐方式1-通过注册表读取用户级)，需要导入

```c++
// 添加路径至 PATH
bool addUserPathEnvironment(const std::wstring& add_path) {
    // 获取用户级别的 PATH 环境变量
    std::wstring currentPath = getUserPathFromRegistry();
    if (!currentPath.empty()) {
        //std::wcout << L"User PATH: " << currentPath << std::endl;
        std::wcout << L"--Tip: User PATH Load SUCCESS! :) " << std::endl;
    } else {
        std::wcerr << L"--Tip: Failed to retrieve user PATH. XX " << std::endl;
    }
    std::wstring env_path = add_path + L"\\bin";
    std::wstring newPath = env_path + L";" + currentPath;

    // 更新 PATH 环境变量
    if (setUserEnvironmentVariable(L"PATH", newPath)) {
        std::wcout << L"--Tip: Add [" << add_path << "] to PATH SUCCESS! :) " << std::endl;

    } else {
        std::wcerr << L"--Tip: Failed to add [" << add_path << "] to PATH. XX " << std::endl;
        return false;
    }
    return true;
}

```

## 2 (Setx)C++ & Command修改环境变量

[setx | Microsoft Learn](https://learn.microsoft.com/zh-cn/windows-server/administration/windows-commands/setx)

​	**pass**

