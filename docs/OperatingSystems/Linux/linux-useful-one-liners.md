---
share: "true"
date: 2021-11-12T19:05:00Z
tags:
  - Linux
  - Bash
draft: false
---

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

### System load is high, but not caused by processes hammering CPU? Check I/O activity:

```
 iostat -x 1
```

### NTP time synchronisation
```
/usr/sbin/ntpdate -s pool.ntp.org
```
if you want to make it scheduled - cronjob solution:
```
0 * * * * /usr/sbin/ntpdate -s pool.ntp.org  >/dev/null 2>&1
```

### NTP time fails because firewall?

```
sudo date -s "$(wget -S  "http://www.google.com/" 2>&1 | grep -E '^[[:space:]]*[dD]ate:' | sed 's/^[[:space:]]*[dD]ate:[[:space:]]*//' | head -1l | awk '{print $1, $3, $2,  $5 ,"GMT", $4 }' | sed 's/,//')"
```

### Linux filesystem becomes read-only. The last trick before reboot:
```
 check real status of mounts
 /proc/mounts
 
 check if /etc/fstab entries have errors=remount-ro option on
  mount -o remount,rw /
```

###  Killing the processes by the name

```
ps -ef | grep procesname | grep -v grep | awk '{print $2}'
```

### Corrupted MySQL (MyISAM tables ONLY!!!)

```
myisamchk -r /var/lib/mysql/*/*.MYI
```

### Find Out The Top 5 Memory Consuming Process
```
ps auxf | sort -nr -k 4 | head -5
```
### Usage in percentage:
```
ps -A --sort -rss -o comm,pmem | head -n 11
```
### Find Out top 10 CPU Consuming Process
```
 ps auxf | sort -nr -k 3 | head -5
```

### /dev/null disappeared?
```
rm /dev/null; mknod -m 666 /dev/null c 1 3
```

### Find big logs
```
find . -name '*.log' -size +400M -exec ls -lha {} \;
```

### Inodes where did they went?

```
find . -xdev -printf '%h\n' | sort | uniq -c | sort -k 1 -n
```
Another aproach:
```
 for i in `find . -xdev -type d `; do echo `ls -a $i | wc -l` $i; done | sort -n
```
or another approach:
```
du --inodes -S | sort -rh | sed -n '1,50{/^.\{71\}/s/^\(.\{30\}\).*\(.\{37\}\)$/\1...\2/;p}'
```
### Tired of known hots error while using scp?
```
alias scp_nocheck='scp -o '\''StrictHostKeyChecking no'\'' -o '\''CheckHostIP no'\'' -o '\''HashKnownHosts no'\'' -o '\''UserKnownHostsFile /dev/null'\'''
```
### Tired of known hosts error?
```
alias ssh_nocheck='ssh -o '\''StrictHostKeyChecking no'\'' -o '\''CheckHostIP no'\'' -o '\''HashKnownHosts no'\'' -o '\''UserKnownHostsFile /dev/null'\'''
```

### Console version of the above if you have direct access to logs

```
cat logfile | grep -E -o "([0-9]{1,3}[.]){3}[0-9]{1,3}" | sort | uniq -c | sort -rn
```

### CPU load is low, but machine feels unresponsive?

```
iostat -d -x 5 3

        rrqm/s : The number of read requests merged per second that were queued to the hard disk
        wrqm/s : The number of write requests merged per second that were queued to the hard disk
        r/s : The number of read requests per second
        w/s : The number of write requests per second
        rsec/s : The number of sectors read from the hard disk per second
        wsec/s : The number of sectors written to the hard disk per second
        avgrq-sz : The average size (in sectors) of the requests that were issued to the device.
        avgqu-sz : The average queue length of the requests that were issued to the device
        await : The average time (in milliseconds) for I/O requests issued to the device to be served. This includes the time spent by the requests in queue and the time spent servicing them.
        svctm : The average service time (in milliseconds) for I/O requests that were issued to the device
        %util : Percentage of CPU time during which I/O requests were issued to the device (bandwidth utilization for the device). Device saturation occurs when this value is close to 100%.

```
### Which process opened a TCP/IP port?

```
netstat -tulpn
```

### Last 15 modified files

```
last 15 modified files
find . -type f -printf '%T@ %TY-%Tm-%Td %TH:%TM:%.2TS %P\n' | sort -nr | head -n 15 | cut -f2- -d" "
```
### Check file type of the files
```
find . -type f | xargs file
```
### Find duplicate files using MD5
```
find . -type f - print0 | xargs -0 md5sum | sort | uniq -w32 --all-repeated=separate
```
### Kill defunct processes
Not recommended (defunct processes should be killed automatically when the parent process finished) but sometimes 'you gotta do, what you gotta do'
```
ps -ef | grep -E "\[ process name \] <defunct>" | awk '{print "kill -9", $3}' | ksh > /dev/null 2>&1
```
### Command line json linting
```
json_xs -t none < file.json
```
### Linux swap file (I can't even count how many times I used it in the last few weeks, pasting it here).

```
sudo fallocate -l 1G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile && sudo swapon /swapfile
```

### Sed string replacement in multiple files at once.
Quick invention inspired by the need to tidy up someone else's code. This particular oneliner looks for files in the currect directory with .py etension, executes a replacemend of a comment tag with a lighter one, and applies to all files matching the pattern.

```
find . -type f -name "*.py" -exec sed -i "s/\"\"\"/'''/g" {} +
```



(c) Dawid Krysiak https://itisoktoask.me/ 