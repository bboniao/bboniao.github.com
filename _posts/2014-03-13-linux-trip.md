---
layout: post
title: "Linux小技巧"
description: ""
category: "linux"
tags: linux
---
{% include JB/setup %}

###查找wio过高进程

1. /etc/init.d/syslog stop
2. echo 1 > /proc/sys/vm/block_dump
3. dmesg | egrep "READ|WRITE|dirtied" | egrep -o '([a-zA-Z]*)' | sort | uniq -c | sort -rn | head
4. echo 0 > /proc/sys/vm/block_dump
5. /etc/init.d/syslog start

<!-- more -->

###Linux分析jvm的cpu性能瓶颈

1. 使用ps aux找到jvm的pid
2. 执行top -H -p \<pid\>，可显示出该进程下的所有线程。找到占用cpu最多的子线程pid，并将其转换为16进制
3. jstack \<pid\> 查找"nid=16进制子pid"
