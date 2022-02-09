---
layout: post
title: SSM agent inexplicably failing to connect
description: SSM fails to report to inventory or fleet mananger
date: 2022-01-26T20:10:00Z
author: Dawid Krysiak
tags: AWS EC2 SSM
draft: false
---

# SSM agent's role.

SSM agent facilitates connection and control over the OS to the AWS APIs.
The most noticable benefits - ability to open SSM terminal sessions. AWS Config tools (Inventory, Patch Manager) also use SSM agent.

I found an issue recently, where SSM is not able to connect to the outside world.

# Underlying issue.

AWS API don't like if your machine's system time is out of sync with the atomic clocks syncing the Internet.

# Solution

## make sure your SSM agent is up to date:

### Download: 

`` wget https://s3.amazonaws.com/ec2-downloads-windows/SSMAgent/latest/linux_amd64/amazon-ssm-agent.rpm ``

### Install:

`` rpm --upgrade --oldpackage --verbose amazon-ssm-agent.rpm ``

### Sychronise the time.

`` sudo date -s "$(wget -S  "http://www.google.com/" 2>&1 | grep -E '^[[:space:]]*[dD]ate:' | sed 's/^[[:space:]]*[dD]ate:[[:space:]]*//' | h" ``

SSM should be able to report in a few minutes time

(c) Dawid Krysiak https://itisoktoask.me/ http://www.krysiak.biz/
