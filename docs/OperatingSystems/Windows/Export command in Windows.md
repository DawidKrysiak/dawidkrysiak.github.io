---
layout: post
title: How to implement export command from Linux in Windows
tagline: When WSL is just a bit too much for your needs
description: Tutorial on how to implement Linux export command in Windows PowerShell
url: itisoktoask.me
date: 2023-11-04T15:42:00Z
author: Dawid Krysiak
tags:
  - Windows
  - Linux
draft: false
share: "true"
---

## check if you have a PowerShell profile set up.

In PowerShell (not the classic CLI!)
```
Test-Path $PROFILE
```
The expected result is `False`, if it returns `True` you already have a profile set up!
## Create a profile (-Force recreates the profile if one already exists)
```
New-Item –Path $Profile –Type File –Force
```
## Create a module
The above command should have printed out the path to your PowerShell profile
**NOTE: for some reason, different versions of powershell might create a different folder name. I know of WindowsPowerShell and Powershell, so be vigilant and adjust the following commands accordingly.**
```
Directory: C:\Users\<user>\Documents\WindowsPowerShell

Microsoft.PowerShell_profile.ps1
```
##  create modules folder
```
cd C:\Users\<user>\Documents\Powershell
mkdir modules
cd modules
mkdir export
cd export
notepad export.psm1
```

paste the below code to the file and save

```
Function export{
    Param ($linuxExport)
        try {
            $CharArray =$linuxExport.Split("=")
            [Environment]::SetEnvironmentVariable( $CharArray[0], $CharArray[1])
            $var = $CharArray[0]
            "Environment Variable Set - $var"   | write-host -fore green; 
          }
          catch {
          "Expected: export variable=value"  | write-host -fore red; 
         }
    }
```
## test
Exit the current PowerShell session and open another one.
execute export command

```
export
```
the result should be

```
Expected: export variable=value
```