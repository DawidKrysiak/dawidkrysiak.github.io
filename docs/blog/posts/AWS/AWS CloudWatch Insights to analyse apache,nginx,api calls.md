---
share: "true"
tags:
  - aws
  - CloudWatch
  - CloudWatchInsights
---

count the unique IP addresses in the apache/nginx/flow logs

```
fields @timestamp, @message
  | parse @message /(?<@ip>([0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3})[.]*)/
  | stats count() as requests by @ip
  | sort requests desc

```

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


(c) Dawid Krysiak https://itisoktoask.me/