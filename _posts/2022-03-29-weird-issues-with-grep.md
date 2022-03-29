 ---
layout: post
title: "Weird issues with grep when searching through logs"
date: 2021-11-30T19:05:00Z
draft: false
---

# Weird issue I've never seen before

## Symptoms
```
cat /var/log/logfile.log | grep "magic"
Binary file (standard input) matches
```
## So let's check the file datatype:
```
file /var/log/logifle.log
data
```
## What is happening?
So, somehow, a text based log file is identified as binary?
It's apparently caused by a NULL character spilled to the log

## Solution
Solution is to filter out the NULL character first.

```
cat /var/log/logfile.log |tr -d '\000'| grep "magic"
magic
```
