---
layout: post
title: AWS Config generating cost while 'disabled'
description: why AWS Config costs money when not running
date: 2021-12-23 21:31:46 +0000
author: Dawid Krysiak
categories: AWS
tags:
  - AWS
  - aws
  - AWSConfig
share: "true"
---


## AWS is like a drug dealer
They will lure you in by free tier, never warn you about hidden cost and as they say - 'easy to get in, hard to get out'. They literally advertise '1-click setup'. But of course - there is no '1-click to delete' button :)

## The hidden cost of AWS Config
Reading the official documentation of AWS Config one could think that all they need to do to stop incurring cost is simply disable the collector/recorder. So that's what they do, and they walk away. And it takes a while to notice a small charge in the billing dashboard regarding 'config'.

## What they don't tell you

Even if the recorder is disabled, it still incurs cost - in my case it was $0.24/mo, so negligible (but still annoying).
And removing that recorder through the interface or disabling AWS Config through the interface is not possible ('easy to get in', remember?)

## Solution 1: if you actually use AWS Config
Enable retention policy. By default, AWS config retains the collection results **indefinitely**. By introducing lifecycle policy, you can delete collections older than x number of days.

## Solution 2: if you don't need AWS Config
You need an IAM user with sufficient permissions to modify `configservice`, configure AWS CLI in your system so you can make an API call.

### List available recorders

```
aws configservice describe-configuration-recorders
```
### Delete recorder

```
aws configservice delete-configuration-recorder --configuration-recorder-name <name - usually `default`>
```
Just to make sure it doesn't come back, remove the AWSConfig role from IAM Roles.






(c) Dawid Krysiak https://itisoktoask.me/ 