---
layout: post
title: "General software and service related useful oneliners"
date: 2021-11-30T19:05:00Z
draft: false
---

### MySQL tunnel
Set up a tunnel through SSH and then use MySQL Workbench on your desktop to connect to 127.0.0.1:3336 rather than directly to the MySQL server.

```
ssh -i .\user.pem -L 3336:127.0.0.1:3306 ec2-user@<IP of the instance>

```
Of course, that would also work if the instance was inside of a network.

```
 ssh -i .\user.pem -L 3333:<internal ip>:3306 ec2-user@<gateway / bastion IP>
```

### Corrupted MySQL (MyISAM tables ONLY!!!)
```
stop mysql(d) process using appropriate command for your OS

myisamchk -r /var/lib/mysql/*/*.MYI
start mysql(d) process using appropriate command for your OS
```

### MySQL - load data from csv
```
use db-name;
LOAD DATA LOCAL INFILE '/report.csv' 
INTO TABLE tickets
FIELDS TERMINATED BY ',' 
ENCLOSED BY ''
LINES TERMINATED BY '\n'
IGNORE 1 ROWS;
```

### AWS RDS MySQL, load data from csv
Due to managed nature of RDS, 'local infile' will not work. If you use MySQL Workbench, set the system variable (in Advanced connection settings **OPT_LOCAL_INFILE=1**

```
use reports;
LOAD DATA INFILE 'report.csv' 
INTO TABLE tickets
FIELDS TERMINATED BY ',' 
ENCLOSED BY ''
LINES TERMINATED BY '\n'
IGNORE 1 ROWS;
```

### Cloudwatch Insights - count the unique IP addresses in the apache/nginx/flow logs

```
fields @timestamp, @message
  | parse @message /(?<@ip>([0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3})[.]*)/
  | stats count() as requests by @ip
  | sort requests desc

```
(c) Dawid Krysiak https://itisoktoask.me/ http://www.krysiak.biz/
