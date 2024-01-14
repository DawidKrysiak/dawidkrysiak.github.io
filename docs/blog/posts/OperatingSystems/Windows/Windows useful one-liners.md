---
layout: post
title: Windows related useful oneliners
description: copy-paste one-liners for windows
date: 2021-11-12T19:05:00Z
tags:
  - windows
  - microsoft
  - tips
draft: false
share: "true"
---

### Windows reboot if there is no access to the graphical interface
```
c:\windows\system32\shutdown -t 0 -r -f
```

### Windows equivalent of diff
```
FC /B file1 file 2
```
### Windows directory content comparison
```
comp dir1 dir2
```
### Problems with VPN software in Windows?
VPN applications usually create a virtual network interface (normally not visible in the interface but can be un-hidden).  Problem is - there is (was?) a limit to how many network interfaces (net filters) you can have in Windows. If apps refuse to install/start with network interfaces related errors, try to increase the number of net filters > 10
```
HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Network\MaxNumFilters****
```

### Detect Windows version through PowerShell (useful if you only have access to SSM, no credentals to the RDP)
```
(Get-WmiObject Win32_OperatingSystem).Caption
```




(c) Dawid Krysiak https://itisoktoask.me/ 