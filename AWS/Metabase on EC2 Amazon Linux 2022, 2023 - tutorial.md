---
layout: post
title: Metabase on EC2 Amazon Linux 2022 - tutorial
tagline: Metabase installation on AWS Amazon Linux 2
description: Metabase installation and configuration on AWS Amazon Linux 2 EC2 instance. Step by step tutorial
url: itisoktoask.me
author: Dawid 'Grendel' Krysiak
date: 2022-07-27T20:06:00Z
draft: false
tags:
  - metabase
  - ec2
  - aws
  - tutorial
share: "true"
---
## This tutorial covers deployment steps of Metabase system using Amazon Linux 2022,2023.

### Preface: 
It's a follow-up to the previous tutorial based on Amazon Linux 2 (now deprecated as AL2 has very little time left, so let's all migrate to AL2023, shall we?).

Amazon Linux 2023 officially admits its roots in Fedora 38. From a 'console user'-perspective, that doesn't change much (apart from an inclination to use `dnf` rather than `yum` for package management). There are some interesting configuration choices as well - for example /tmp is mounted in RAM-disk which makes it absurdly fast but comes at a penalty of limited space. So if your software rely on thousands of files in /tmp (you know who you are :)) then you might want to consider remounting /tmp and disabling the service that keeps mounting it. More on that in another article soon.

Now to the fun part - deploy Metabase on AmazonLinux 2022. The below is 90% copy of the previous article with appropriate changes for the new release. Enjoy.

I started this project with 'best practices as understood by Amazon' in mind - decoupled, scalable, fault resistant... And then realized, that I know a few really heavy production systems that don't have any of that and... function very well. With dozens of users working simultaneously. So as much as it's fun to deploy complicated infrastructure, it only benefits Amazon. Metabase database is tiny (less than 10MB?) my source of information - a single table with tens of thousands of rows - 20MB (in reality you will query other database sources that exist for their own purpose so your needs will be even smaller!) - why would I create a separate RDS (or two!) for that? With snapshots? Multi-AZ? Why would I bother with ElasticBeanstalk, and cloudforming it? The ONLY benefit is that Jeff has more money to pay his alimony (and for his space trips). Need this to be portable/redeployable? Just make an AMI after initial deployment (before any data sources are added to metabase), share it with regions/accounts you wish, deploy there (remember to chage the passwords :) ) - happy days. Cheap and easy solution. Spend your money on keeping regular snapshots, AMIs (e.g. properly set up AWS Backup!) and coffee, not on infastructure that serves no purpose other than some imaginary brownie points. If you insist of having a fancy infra, follow Metabase's own tutorial [Metabase on ElasticBeanstalk](https://www.metabase.com/docs/latest/operations-guide/running-metabase-on-elastic-beanstalk.html) (which at the time of writing this tutorial, didn't actually worked for me).

One thing that absolutely needs to happen is - the backbone database (which metabase uses to store user accounts, system settings, 'questions', graphs settings and so on) need to be moved away from the default file-based h2 format to a real database. **H2 is not fit for any production purpose**. And the developers of Metabase will agree with me - repeatedly, all over the place. **DO NOT use the default H2 backbone for production**. A crash (e.g. forceful kill of a process by `kill -9 pid` of the app (jar) while it's doing any updates to the tables **WILL** corrupt the data, and to my experience so far, **NONE** of the available tutorials work. **EVER** (yeah, ask me how I know :( ). Yes, H2 has 100% failure rate in my experience. The only solution is to start over or (if you are smart enough to have EBS snapshots every night :) ) to revert back the whole system.

Having the above in mind, the project comprise of:

<ul>
<li>a single EC2 instance</li>
<li> MySQL server installed on the instance itself, not as RDS. But if you insist - RDS works just fine, MySQL/MariaDB and PostgreSQL, that's how I started, before I realized I'm burning cash. RDS in my specific case could be of benefit (as mentioned I have my own 'data source' that needs to be queried), but in usual situations - you will be querying other RDS systems ANYWAY.</li>
<li>Metabase running as a SystemD service</li>
</ul>

### Instance.
Amazon Linux 2022 preview is available only in Oregon, so if you wish to deploy it in any other region, you have to copy the AMI. Instance type depends on your needs. In my case, **t2.micro** was sufficient during development, but even then, machine was left with 60MB of available RAM, so 2GB of RAM is absolute minimum if you have more than one user working on it (it will work with 1GB RAM + a bit of swap, but it will be horribly slow in regenerating graphs). I would say that **t2.small** is a good compromise, **t2.medium** should be plenty, unless you have dozens of users working at the same time. If you wish to save money on the project even further, consider if the system has to work 24/7 - maybe you have fixed working hours beyond which you can set up the maintenance window for updates/patching and then shut down the system overnight? Consider using Lambda functions for start/stop [GitHub](https://github.com/Grendel-DMK/aws-instance-stop)

While you are preparing the instance, you might just as well set up the SecurityGroup at the same time.
<ul>
<li>22 (for ssh access)</li>
<li>3306 (if you wish to access MySQL remotely rather than working on the instance itself all the time. In this case, I suggest locking down the access to a single IP rather than leaving it for 0/0. Or better yet, don't open 3306 to the world at all, and use SSH tunnel for your connection to MySQL [SSH tunnel for MySQL](https://itisoktoask.me/linux-useful-oneliners/#mysql-tunnel). Less stable, but surely more secure).  </li>
<li>3000 (that's the default Metabase port. Some people add nginx to the mix, to allow for 80/443 access, but I can list a few reasons why I don't bother, just give users a link with :3000 in it and you are sorted :) RAM saved, work spared :) of course, if you need a domain and SSL certificate, you either put more work into it, or engage ELB, it all depends on your needs</li>
</ul>

#### Add IAM Role to the instance
I suggest adding the following AWS-managed policies which will future-proof your instance if you start thinking about properly managing it and take it out of a sandbox:
```
arn:aws:iam::aws:policy/AmazonSSMManagedInstanceCore
arn:aws:iam::aws:policy/AmazonSSMPatchAssociation
```

#### Add SSM access, because SSH is so 2003 :)
```
sudo yum install -y https://s3.amazonaws.com/ec2-downloads-windows/SSMAgent/latest/linux_amd64/amazon-ssm-agent.rpm
```

#### Storage:
I left it at the default 8GB ssd, after the deployment was done, `du` was at 63% and I don't see it growing very much over time (I introduced logrotate just in case, anticipating the logs being the biggest contributors to storage consumption).

#### Add swapfile.

```
sudo fallocate -l 1G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile && sudo swapon /swapfile
```

#### add to /etc/fstab to make swap mounted after reboot.

```
sudo echo '/swapfile swap swap defaults 0 0' >>/etc/fstab
```

### MySQL
That was a very frustrating stage as AL2022 is not an official distro yet, so default YUM/DNF configurations are useless. Once the AMI is released, I'm sure AWS will arm it with all sorts of repo definitions, but for now, here is my hacky way of deploying it.

```
yum install -y  https://dev.mysql.com/get/mysql80-community-release-fc35-3.noarch.rpm
```
Ok, let's do yum update - FAIL - as mentioned above - distro doesn't exist and it's not advertising it as FedoraCore distro, so let's hack  the repo file to let it think it's FedoraCore35 :)

edit the below with your editor of choice:
```
/etc/yum.repos.d/mysql-community.repo
```
add the following content (or manually change the existing one to match)

```
[mysql80-community]
name=MySQL 8.0 Community Server
baseurl=http://repo.mysql.com/yum/mysql-8.0-community/fc/35/x86_64
enabled=1
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-mysql-2022
       file:///etc/pki/rpm-gpg/RPM-GPG-KEY-mysql

[mysql-connectors-community]
name=MySQL Connectors Community
baseurl=http://repo.mysql.com/yum/mysql-connectors-community/fc/35/x86_64
enabled=1
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-mysql-2022
       file:///etc/pki/rpm-gpg/RPM-GPG-KEY-mysql

[mysql-tools-community]
name=MySQL Tools Community
baseurl=http://repo.mysql.com/yum/mysql-tools-community/fc/35/x86_64
enabled=1
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-mysql-2022
       file:///etc/pki/rpm-gpg/RPM-GPG-KEY-mysql

[mysql-tools-preview]
name=MySQL Tools Preview
baseurl=http://repo.mysql.com/yum/mysql-tools-preview/fc/35/x86_64
enabled=0
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-mysql-2022
       file:///etc/pki/rpm-gpg/RPM-GPG-KEY-mysql
```

Let's see if that worked

```
dnf update -y
sudo dnf install -y mysql-community-server
```

FAIL. Library I'm not familiar with is missing. A bit of digging and found a package providing it.

```
 yum install -y wget https://rpmfind.net/linux/fedora/linux/releases/35/Everything/x86_64/os/Packages/m/mecab-0.996-3.fc35.2.x86_64.rpm
```

OK, let's try again.

```
sudo dnf install -y mysql-community-server
```
It should install now, but I received an interesting notification about release upgrade. 

```
dnf update --releasever=2022.0.20220719
```
Once that's done, let's check if MySQL is running - probably not. But is the service defined?
```
systemctl| grep mysql
```
`mysqld.service` should be on the list already so let's enable it and start.

```sudo systemctl enable mysqld.service```
```43  sudo systemctl start mysqld.service```

Is it running?

```ps aux | grep mysql```


#### Run MySQL initial setup.

```
sudo grep 'temporary password' /var/log/mysqld.log 
```

Make a note of the temporary root password generated by the installer.

```
sudo mysql_secure_installation 
```
Use the password noted above, set a new one, remove anonymous users, decide if you require for root to be able to log in remotely (don't, use the tunnel instead, as mentioned earlier), reload permissions. Done.

Connect to the MySQL console, create a new db - let's call it `backbone`

```
CREATE DATABASE backbone;
USE backbone;
CREATE USER 'metabase'@'localhost' IDENTIFIED BY '<password>';
GRANT ALL ON backbone.* TO 'metabase'@'localhost';
```

### Metabase user setup
Your instance (given how tiny the specs are) should be solely dedicated to metabase purpose, so the below method is a total overkill, you could run everything from the default ec2-user, but I believe it's a good practice to separate out the services and data from a default system login, to avoid accidental damage.

```
sudo adduser metabase
sudo touch /etc/default/metabase
```

I made sure in /etc/passwd that my metabase user can't login to the system as a human :) 

```
metabase:x:995:993::/home/metabase:/bin/false 
```

### Java installation

```
sudo dnf install -y java-17-openjdk-devel
```

### Metabase download

Admittedly I was working as root at this time, so make sure permissions are set right after.

```
cd /home/metabase/
wget https://downloads.metabase.com/v0.40.1/metabase.jar
chown metabase:metabase metabase.jar
```

**Of course, in the future, make sure you download the latest version.**


### Set up the service

```
touch /etc/systemd/system/metabase.service
```

Use your favourite text editor to add the following. Make sure the users (system and MySQL),password are set to what you set up in your system.
You can also change the **StandardOutput, StandardError** variables to separate the messages to another file rather than dump everything into /var/log/messages
I'm doing it this way, to make my life simpler. a. this instance's sole purpose is to run metabase + mysql, so I don't have to monitor multiple streams of possible errors b. when I set up the CloudWatch agent, I have a template to push out system logs already, don't have to fiddle with it to add metabase specific logs. One less thing to remember :). But of course - you do what suits your needs. 


```
[Unit]
Description=Metabase server

[Service]
Type=simple
WorkingDirectory=/home/metabase
User=metabase
Group=metabase
ExecStart=/bin/java -jar /home/metabase/metabase.jar
Restart=on-failure
RestartSec=10
StandardOutput=syslog
StandardError=syslog
SyslogIdentifier=metabase
Environment=MB_PASSWORD_COMPLEXITY=normal
Environment=MB_PASSWORD_LENGTH=8
Environment=MB_DB_TYPE=mysql
Environment=MB_DB_DBNAME=backbone
Environment=MB_DB_PORT=3306
Environment=MB_DB_USER=<user>
Environment=MB_DB_PASS=<password>
Environment=MB_DB_HOST=localhost
Environmen=MB_EMOJI_IN_LOGS=true
```

Moment of truth :). Start the service.

```
systemctl enable  metabase
systemctl start metabase
```

Test.

```
ps aux | grep metabase
```
You should see the process running. If you don't, check `/var/log/messages` - metabase should dump errors there. In my case it was MySQL permissions issues.
If it's running, we are almost there, just one important test - check if metabase is using MySQL rather than h2. 

```
sudo ls -lha /home/metabase
```

The only elements you should see is metabase.jar and plugins directory. If you see any `.db.` files, it means that the connection to MySQL was not picked up from the config and metabase defaulted to its built-in functionality. DO NOT LEAVE IT AS IS. Investigate, fix, remove the db files, try again. Don't 'let it be, should be fine' - you **will** regret later.

### Testing

It's time to log in to metabase - using a browser, go to  http://<IP>:3000/
you should see the Metabase interface, go through the setup process, add your sources and have fun.


### Updates

Stop the service

```
systemctl stop metabase
```

Back up the current version

```
cd /home/metabase
mv metabase.jar metabase_$(date +%d-%m-%Y).jar
```

Download the latest version to the same place

```
wget https://downloads.metabase.com/{latest-version}/metabase.jar
```

Start the service

```
systemctl start metabase
```

### There are some useful tips that got me through my stumbles.

MySQL developers in their 'wisdom' decided to introduce certain settings that caused a lot of frustration.
One of them was inability to do 'group by' in some scenarios.  I know I could rework the metabase 'questions', but life is too short for that.

First, check the sql_mode by running this query:


```
SELECT @@sql_mode;
```

The result should be something like this:

```
ONLY_FULL_GROUP_BY,STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_ENGINE_SUBSTITUTION
```

Make a note of the values, you will be needing most of them.
Edit my.cnf under [mysqld] section:

sql_mode= (values from above, but remove ONLY_FULL_GROUP_BY)

e.g. 

```
sql_mode=STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_ENGINE_SUBSTITUTION
```

```
sudo systemctl restart mysqld.service
```



(c) Dawid Krysiak https://itisoktoask.me/ 