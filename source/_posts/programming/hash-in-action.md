---
title: Hash In Action
date: 2020-10-06 22:48:57
updated: 2020-10-06 22:48:57
categories:
- Programming
tags:
- System Design
---
# Intro
In the past three months, I was occupied by the intensive work and have no extra energy to divert my attention. But thanks to National Day! Finally, I have the time to review and to share. Vive la Chine! Let's keep on track, in this post, I share a simple use case of using hash technique in action.

<!-- more -->
# What is Hash?
I have summarized Hash technique a year ago when I was preparing for on-campus recruitment season. To summarized it more simply, I draw an illustration below. Be aware that we don't talk about how to design but how to use it in this post. In short, hash function is the core of a hash algorithm, a well-designed hash function should output different results when received high similarity inputs.

<div style="width: 300px; margin: auto">

![What is Hash?](https://raw.githubusercontent.com/zhongqin0820/zhongqin0820.github.io/source-articles/source/images/programming/what-is-hash.png)
</div>

With this in mind, here goes the [user story](#User Story).

# User Story
> As a customer manager, I want to monitor all my clients' machines which distributed in different regions and I should receive an alert whenever, wherever and whichever server crashed, thus the OPS team who use MS-DOS will be notified promptly and the problems will be fixed quickly and I can serve my clients better.

> As a service leader, I don't want your implementation to become too heavy. Just implement it simply with no user perception and without any extra expenses on it. To achieve the requirement, you may use our provided OBS which compatible in each server.

# System Design
Since we have different servers in different regions and each server is capable of uploading files to an OBS. The case is easier than those using telegraf or kafka things. Just as the illustration shows below, we separate the system into two parts, monitoring and notifying.

<div style="width: 300px; margin: auto">

![System Design](https://raw.githubusercontent.com/zhongqin0820/zhongqin0820.github.io/source-articles/source/images/programming/hash-simple-use-case-system-design.png)
</div>

## Monitoring
**Monitor** all servers by installing a cron job with a pre-defined threshold in each server to check the required metrics. If the server reaches the threshold, it will only upload the log file once to the OBS and won't upload any more until the problem was fixed.

> Note that cron service needs to be restart if you edit a cron job by using `crontab -e`.

## Notifying
**Notify** the OPS team every time the OBS file list changed by popping a notification window, which will give enough information to fix the crashed server. This is the highlighted part I intend to share in this post, where hash technique was used by me in action. Since the OBS will receive a log file whenever a server crashed, what we need to do is to list the bucket and compare the file list to the previous one with the help of a hash algorithm. The logic behind is, **if the hash value is different from each other, it means that the file list is different and there must be some new log files uploaded. Then, we need to diff and notify those new log files.**

### Cron job in MS-DOS
Since we need to retrieve the file list regularly, we need to set up a cron job to do so. Unlike *nix system, we cannot use cron services in MS-DOS. Although MS-DOS has `Task Scheduler`, it's still not user-friendly if your program was implemented in shell scripts. Thus, I highly recommend you to set a cron job by using `infinite loop` and run the script by using `nohup bash xxx.sh 2>&1 &`, which will save your time a lot.

### How to Notify in MS-DOS
Now comes to notification, it's easier to pop up a notification window in MS-DOS than you thought by using vb-scripts. Here is the snippet, which reads content from a file and display it on a pop up window, you may customize it to your use case.

```vb
set fs = creatobject("scripting.filesystemobject")
set ts = fs.opentextfile("path_to/filename.type", 1, true)
value = ts.readall
Msgbox(value, 64, FormatDateTime(Now))
```

# Summary
Although the whole system isn't very complicated and is a kind of a toy-system, at least it worked without extra costs. And above all, I demonstrate a simple use case of using hash technique in action. Hope this post helped and happy coding :)

# Changelog
- 2020/10/06：First edition
