---
layout: post
title: "AWS CloudWatch Insights - logs parsing"
tagline: "Use cloudwatch insights instead of athena"
description: if you have a log that contains valuable information stored in identifiable sections (e.g. square brackets), you can use insights query to do some stats
url: itisoktoask.me
date: 2022-06-04T22:42:00Z
author: Dawid Krysiak
tags: AWS EC2 CloudWatch security
draft: false
---


# CloudWatch Insights to find most commonly used API calls
Recently I had a case where a customer tried to figure out what api calls were used in their system. Fortunately, 3rd-party application's logs contained references to the APIs in square brackets like

```
2023-05-10 14:27:53,794 INFO  168372887373144 connector:30 -  Start : [loginPage]

```


```
fields @message, @timestamp
| filter @message like /(Start)/
| parse "[*]" as matchedValue
| stats count() as requestCount by matchedValue
| sort requestCount desc
```

result:
loginPage: 100000
