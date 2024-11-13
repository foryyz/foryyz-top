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
