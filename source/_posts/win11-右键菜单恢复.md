---
title: win11 右键菜单恢复
date: 2022-03-15 13:16:45
tags: windows
categories: 操作系统
---

命令行下输入如下命令
![](01.png)
### 1.恢复到win10菜单
```
reg.exe add "HKCU\Software\Classes\CLSID\{86ca1aa0-34aa-4e8b-a509-50c905bae2a2}\InprocServer32" /f /ve
```

### 2.恢复到win11菜单
```
reg.exe delete "HKCU\Software\Classes\CLSID\{86ca1aa0-34aa-4e8b-a509-50c905bae2a2}\InprocServer32" /va /f
```

> 操作完需要重启