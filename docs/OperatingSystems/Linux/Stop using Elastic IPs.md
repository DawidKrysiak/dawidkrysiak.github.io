---
layout: post
title: Stop using Elastic IPs
description: Elastic IPs are useful but will cost you if not attached to a running instance. Use dynamically updated Route53 records instead.
date: 2021-12-16T06:42:00Z
author: Dawid Krysiak
tags:
  - AWS
draft: false
share: "true"
---

## Elastic IP is useful but... (gotcha!)

If your instance is not up and running 24/7, EIP will incur cost.
But what if you wish to attach a domain name without fiddling with ALB... Which without an instance behind it... (you guest it!) will incur cost!.
The solution is to make sure that your instance upon every boot updates Route53 record with its new external IP.
Let's do it!

### Create IAM role to access Route53 from an instance.
Your instance will have to be able to modify Route53 resources (I added some read permissions useful during learning).


Create the below and attach to the IAM role associated with your instance.

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": [
                "route53:GetHostedZone",
                "route53:ListResourceRecordSets",
                "route53:ChangeResourceRecordSets"
            ],
            "Resource": "arn:aws:route53:::hostedzone/<zone id from details>"
        },
        {
            "Sid": "VisualEditor1",
            "Effect": "Allow",
            "Action": "route53:ListHostedZones",
            "Resource": "*"
        }
    ]
}

```

### Check if you can read the configuration of your R53 resources.
```
aws route53 list-resource-record-sets --hosted-zone-id <zone id from details>

```
It should return a JSON response with the zone details.

### Check if you can modify your R53 resources.
Let's create/update a record.
Create a JSON file and fill in the details.

( I suggest a looong session with [official documentation](https://docs.aws.amazon.com/cli/latest/reference/route53/change-resource-record-sets.html#examples) )

```
{
  "Comment": "testing",
  "Changes": [
    {
      "Action": "UPSERT",
      "ResourceRecordSet": {
        "Name": "host.domain.com",
        "Type": "A",
        "TTL": 60,
        "ResourceRecords": [
          {
            "Value": "8.8.8.8"
          }
        ]
      }
    }
  ]
}
```

And let's test it.

```
aws route53 change-resource-record-sets --hosted-zone-id <zone id> --change-batch file://call.json
```

If everything goes well, you should see a confirmation:

```
{
    "ChangeInfo": {
        "Status": "PENDING",
        "Comment": "testing",
        "SubmittedAt": "2021-12-16T23:07:54.128Z",
        "Id": "/change/C04585772S8P7ZILBKLBP"
    }
}
```

### Let's try to figure out what's our external IP
Some people try to use whatsmyip.com and similar - in case of AWS it's not actually necessary.
`curl -s http://169.254.169.254/latest/meta-data/public-ipv4`
should return the public IP allocated to the machine.

### So let us put it all together, shall we?
* prepare a file `/root/template.json` and place the JSON payload content used earlier
* in root's crontab set up a line which will curl the instance metadata system to obtain the IP, then using sed it will replace preexisting 8.8.8.8  from above example with the current IP *(note: this means the template.json needs to have one (and one only) IP in the "ResourceRecords")*

```
@reboot cd /root ; ip=`curl -s http://169.254.169.254/latest/meta-data/public-ipv4` ;sed -r 's/(\b[0-9]{1,3}\.){3}[0-9]{1,3}\b'/"$ip"/ template.json > payload.json ; aws route53 change-resource-record-sets --hosted-zone-id <zone id> --change-batch file://payload.json >> /root/eip_history.log
```

### Test it.
Stop/start your machine to see if you loose your external (public) IP. If your instance comes up with a new IP, give it a minute or two and check if Route53 record is updated.




(c) Dawid Krysiak https://itisoktoask.me/ 