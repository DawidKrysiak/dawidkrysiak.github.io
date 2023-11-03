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

