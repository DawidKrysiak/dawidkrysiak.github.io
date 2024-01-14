---
layout: post
title: Amazon Linux 2023 what every sysadmin should know
tagline: AL2023 is awesome, but might have some surprises
description: Amazon Linux 2023 sysadmin cheatsheet
url: itisoktoask.me
date: 2023-11-04T15:42:00Z
author: Dawid Krysiak
tags:
  - Windows
  - Linux
  - AmazonLinux2023
draft: false
share: "true"
---
# Things to know when migrating from Amazon Linux 1 or Amazon Linux 2 to Amazon Linux 2023 

AmazonLinux 2023 is based on Fedora Linux which might be a good thing or a bad thing (let's not go there :) ), which gives a lot of head start for many sysadmins.
I am positively surprised with this edition (which builds up on the experience of AL2022)  - the system is completely debloated, with no pre-installed languages, libraries, services etc... That approach has two outcomes:
* **Positive**: the system is noticeably 'faster' (more precisely: more responsive) and you don't pay for running stuff (both storage and compute) you'll never use (or in the worst case scenario will collide with your software)
* **Negative**: DIY all the way. Do you want something running? Figure out what you need, find the packages, install them, and configure the system the way YOU want it.

## /tmp handling
Amazon Linux 2023 comes equipped with /tmp directory mounted in memory.
It's visible in `df` as a separate device

```
/dev/xvda1      8.0G  4.2G  3.9G  52% /
tmpfs           986M     0  986M   0% /tmp
```

Rather than a fixed entry in /etc/fstab, it uses a service to remount the /tmp after a reboot

```
/usr/lib/systemd/system/tmp.mount
```

### Benefits
Tempfs is fast. Absurdly fast for something that looks like 'a storage'. The service definition mentioned above automatically allocates 50% of RAM as possible storage.

### Drawbacks
If your software depends on a large volume of temporary files (you know who you are :)), you have to carefully measure the performance -  if you fill in the /tmp with your files, and the demand for RAM from the application rises, the system will start using swap and that will have a massive impact on performance

### How to disable it
As mentioned earlier, it's mounted after each reboot via a service, so it is as simple as stopping the service and then disabling it.

```
sudo systemctl disable tmp.mount
sudo systemctl stop tmp.mount
```
But now the /tmp is useless, so let's recreate it.

```
sudo rm -rf /tmp
sudo mkdir /tmp
sudo chmod 1777 /tmp
```

test:  `df -h` should show you no /tmp as a separate mount point.

## System Logging
Sysadmins got used to the fact that their OSes come preinstalled with some sort of system-level logging. Some applications just automatically assume you have syslog or similar up and running and try to dump their output to /var/log/messages.
AmazonLinux 2023 comes with an unprecedented level of respect to the sysadmins and does not come preconfigured (well..there is journald as part of SystemD, but that's not what we are talking about here). So... When you set up your system, remember to include syslog, rsyslog etc... into your deployment process

## Python
Python is a dependency on many system-level tools (e.g. yum) so it needs to come preinstalled for the tools to work. But... If you type `python` into your console, no results will be found - python is not added to your profile's paths. So you have two choices - add python to the path (through `export PATH=$PATH;...`) or - as per new recommendations -set up a virtual environment (https://docs.python.org/3/library/venv.html) or even set up docker containers with the required versions of Python for your code -that way you won't accidentally mess up the system-level settings, breaking your system tools in the process. Separation of workflows is always a good idea in my book.

## SystemV / legacy support
You can still set up services in sysvinit (/etc/init.d/) but the system will let you know it's not ecstatic about it and it will be quite verbose.

```
SysV service '/etc/rc.d/init.d/cfn-hup' lacks a native systemd unit file. Automatically generating a unit file for compatibility. Pleaseupdate package to include a native systemd unit file, in order to make it more safe and robust.
```