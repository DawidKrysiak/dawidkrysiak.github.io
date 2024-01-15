---
layout: post
title: Automate Linux updates
date: 2021-12-11T19:05:00Z
author: Dawid Krysiak
tags:
  - Linux
  - Bash
draft: false
share: "true"
---

# How to ensure your Linux-based EC2 instance is updated
## (without invoking external tools)


AWS users are tempted to grab a new shiny thing for every possible task. But in the majority of cases, built-in tools will suffice.

Goal:
We need to make sure our system is patched to avoid vulnerabilities. We need to make sure it's patched regardless of the instance uptime - e.g. 
machine running 24/7 or onlyu 8h a day.


## Find the right tool for the job
There is a number of package managers in Linux world so make sure you pick the right one for your system. 

How to figure out which system it is? If you are in the console, try:

* `lsb_release -a`
* if doesn't work, try:
    `cat /etc/os-release`
    
* if ubuntu/debian = apt
* if Amazon/RedHat = yum
* if newer RedHat / Fedora = dnf

## where is the app?

```
which yum

/usr/bin/yum
```

## how to schedule the update? CRON to the rescue!
Highly recommend the below website which nicely explains crontab architecture and allows for easy experimentation without consequences.

```
https://crontab.guru/#5_0_*_*_*
``` 

## How to make sure it's always triggered?
If we would only implement rigid schedule (e.g. 00:05) if that machine would be stopped overnight, update would never triggered. So w need two entries in the crontab -one to cover 24/7 scenario, one to cover shutdown/reboot scenario.

```
# trigger update 5 minutes past midnight every day
5 0 * * * /usr/bin/yum update -y

# trigger update at system bootup
@reboot /usr/bin/yum update -y
```