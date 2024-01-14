---
layout: post
title: SSM agent update fails due to AWS Config
description: How to work around issues with AWS Config blocking update of SSM Agent
date: 2022-01-28T20:10:00Z
author: Dawid Krysiak
tags:
  - AWS
  - EC2
  - SSM
  - AWSConfig
  - windows
draft: false
share: "true"
---
# Symptoms
If you try to update SSM agent on an older instance with AWSConfig installed, SSM Agent will refuse to update, reporting issues caused by AWSConfig

# Remediation steps

## Backup your AWSConfig customisation
On Windows, config file can be found in C:\Program Files\Amazon\EC2Config\settings\config.xml
Technically the installer WILL do the backup. But I prefer doing my own backups :)

## Download the installer
If you are updating AWSConfig from version 3.x to 4.x you are probably ok, but if you are updating from 2.2.12 or earlier, please refer to the AWS documentation as there are issues with backwards compatibility which you have to go around by updating .NET and two stage installation

https://docs.aws.amazon.com/AWSEC2/latest/WindowsGuide/UsingConfig_Install.html

Download and unzip ``` https://s3.amazonaws.com/ec2-downloads-windows/EC2Config/EC2Install.zip ```

## Install AWSConfig through PowerShell

```EC2Install.exe /norestart```

## revert back the config.xml from backup

## restart EC2Config service through msconfig.msc

## Check the logs in C:\\Program Files\\Amazon\\EC2Config\\logs\\
If you see WMI error, that might not be a big problem (I've seen it all over the place), but important part is the report of pushing out reports.

## Install the latest SSM agent
``` https://docs.aws.amazon.com/systems-manager/latest/userguide/sysman-install-ssm-win.html ```

(c) Dawid Krysiak https://itisoktoask.me/