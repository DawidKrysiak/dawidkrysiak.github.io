---
layout: post
title: Lower the cost of AWS EC2
description: uptime control of EC2 and RDS instances to save cost. Python based AWS Lambda function
date: 2021-12-23 21:31:46 +0000
categories: AWS
tags:
  - aws
  - ec2
  - rds
share: "true"
---


## Preface
Probably most of the people who started their journey with AWS, had their 'ups!' moments when they looked at the Billing dashboard.
Learning curve should be steep, but why so expensive? AWS is fogging the view by offering 'free tier' for many of their services in the first year of your account activity (and sometimes permanently!) but understanding of all the 'gotcha!'s escapes even more experienced users and even certified AWS engineers.
One of the methods of staying on top of your bill is to set up clear rules in both your billing (e.g. setting up a billing alert if it approaches predefined budget limit) and your learning process. If your account is not set up for production purposes, I bet you don't need EC2 and RDS instances running 24/7? But sometimes it escapes your mind that you should shut down the systems at the end of the day. On top of that, even if you do stop it, RDS will not stay 'stopped' - it will quietly start back up after 7 days, so if you don't have any CloudWatch alarms set up for that instance, you will miss the fact that it's up and running and burning cash. I know, I've been there. Hence this tool.

## Purpose
[aws-instance-stop](https://github.com/Grendel-DMK/aws-instance-stop) Deployed as Lambda function, script stops EC2 and RDS instances based on the tags.

## Usage
- Add to your EC2/RDS instance(s) tag 'uptime' with appropriate value (in this example 'daytime').
- Deploy this script as Lambda
- If you need the systems to stop/start automatically, create another lambda from the code, just change `stop_instances` and `stop_db_instances` to `start_instances` and `start_db_instances` respectively 
- Add a trigger(s) 'EventBridge (CloudWatch Events)' (e.g. start lambda triggered at 8AM, stop lambda at 8PM)
- Create a new rule with appropriate Schedule expression.
- create new role with policy: 

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": [
                "rds:StopDBInstance",
                "rds:DescribeDBInstances",
                "logs:CreateLogStream",
                "ec2:DescribeInstances",
                "ec2:StopInstances",
                "logs:CreateLogGroup",
                "logs:PutLogEvents"
            ],
            "Resource": "*"
        }
    ]
}
```
Attach it to Lambda as execution policy



(c) Dawid Krysiak https://itisoktoask.me/ 