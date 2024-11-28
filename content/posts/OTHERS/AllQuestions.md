---
title: 'AllQuestions - 解决问题汇总'
date: 2024-11-03T23:57:02+08:00
draft: false
---

## 1 Microsoft Store 不走代理

原因：UWP沙箱环境默认不与本地网络连通，需要针对应用开启 loopback 豁免

```powershell
# powershell管理员执行 即可解决问题
foreach ($n in (get-appxpackage).packagefamilyname) {checknetisolation loopbackexempt -a -n="$n"}
```

## 2 Windows Terminal 快捷键

1. **桌面**新建**快捷方式**，地址填：
   - `	%LocalAppData%\Microsoft\WindowsApps\wt.exe`
2. 右键属性填写快捷键Ctrl+Alt+T

## 3 Win11右键菜单管理

取消折叠：

```powershell
reg.exe add "HKCU\Software\Classes\CLSID\{86ca1aa0-34aa-4e8b-a509-50c905bae2a2}\InprocServer32" /f /ve
# 重启资源管理器↓ 或者 重启即可
taskkill /f /im explorer.exe & start explorer.exe
```

恢复：

```powershell
reg.exe delete "HKCU\Software\Classes\CLSID\{86ca1aa0-34aa-4e8b-a509-50c905bae2a2}\InprocServer32" /va /f
```

