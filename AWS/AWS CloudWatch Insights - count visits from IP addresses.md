---
layout: post
title: AWS CloudWatch Insights - count visits from IP addresses
tagline: Use cloudwatch insights instead of athena
description: if you have a log that contains the IP addresses of your visitors and you wish to calculate that, you can use CloudWatch Insights
url: itisoktoask.me
date: 2022-06-04T22:42:00Z
author: Dawid Krysiak
tags:
  - AWS
  - EC2
  - CloudWatch
  - security
draft: false
share: "true"
---


# CloudWatch Insights to find an abuser
Recently I had a case where a customer tried to figure out who is abusing their systems, probably by invoking bots.
Fortunately, their application was recording registrations in the main application logs, so the solution was quite simple:

```
fields @message
| filter @message like /POST \"\/registration/
| parse @message /(?<@ip>([0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3})[.]*)/
| stats count() as requestCount by @ip
| sort requestCount desc
| filter requestCount >5
```

## let's break it down
* first line selects the fields (in this case, just the body of the message
* second line filters out the lines that are relevant in this case - containing `POST "/registration` string
* third line filters the remaining lines by looking for an regex matching the IP address format
* fourth line is the equivalent of 'count()'
* fifth line orders the results in descending order
* sixth line filters out only IPs that occurred more than 5 times
* 