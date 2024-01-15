---
layout: post
title: Amazon Linux 2022, 2023 tempfs configuration
date: 2021-12-20T06:42:00Z
draft: false
tags:
  - aws
  - ec2
  - Linux
  - AmazonLinux2023
  - AmazonLinux2022
share: "true"
---

## Things to know when migrating from Amazon Linux 1 or Amazon Linux 2 to Amazon Linux 2022 - tempfs


Amazon Linux 2022 comes equipped with /tmp directory mounted in memory.
It's visible in `df` as a separate device

```
/dev/xvda1      8.0G  4.2G  3.9G  52% /
tmpfs           986M     0  986M   0% /tmp
```

Rather than a fixed entry in /etc/fstab, it uses a service to remount the /tmp after a reboot

```
/usr/lib/systemd/system/tmp.mount
```

## Benefits

Tempfs is fast. Absurdly fast for something that looks like 'a storage'. The service definition mentioned above automatically allocates 50% of RAM as possible storage.

## Drawbacks

If your software depends on a large volume of temporary files, you have to carefully measure the performance -  if you fill in the /tmp with your files, and the demand for RAM from application raises, system will start using swap and that will obviously have a massive impact on performance

## How to disable it

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

test:  `df -h` should show you no /tmp as a separate mountpoint.