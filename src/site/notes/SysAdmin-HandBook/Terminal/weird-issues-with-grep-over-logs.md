---
{"dg-publish":true,"permalink":"/sys-admin-hand-book/terminal/weird-issues-with-grep-over-logs/","title":"Weird issues with grep over logs","tags":["AWS","EC2","Linux","windows","Bash","terminal"]}
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