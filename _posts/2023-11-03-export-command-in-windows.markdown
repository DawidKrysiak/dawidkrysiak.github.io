---
layout: post
title: "How to implement export command from Linux in Windows"
tagline: "When WSL is just a bit too much for your needs"
description: Tutorial on how to implement Linux export command in Windows PowerShell
url: itisoktoask.me
date: 2023-11-04T15:42:00Z
author: Dawid Krysiak
tags: Windows Linux
draft: false
---


# `export` command in PowerShell

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

```
Directory: C:\Users\user\Documents\WindowsPowerShell

Microsoft.PowerShell_profile.ps1
```


## edit the profile

```
notepad $Profile
```


```

# Powershell function to convert the guardian user 
# credential "export" format into powershell environment variables

# How to import into your powershell
# 1. Open the Modules folder and create a folder in there named 'export'
#     C:\Users\USER_NAME\Documents\WindowsPowerShell\Modules\export
# 2. copy this file into the modules folder 
#     C:\Users\USER_NAME\Documents\WindowsPowerShell\Modules\export\export.psm1
# 3. Start a new powershell session and enjoy

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

