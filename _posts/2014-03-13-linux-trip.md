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

###memcache的启动参数

    /opt/memcache/bin/memcached -d -f 1.1 -M -m 4096 -o slab_reassign slab_automove -u root -l 10.11.6.31 -p 12336 -c 1024 -P /opt/memcache/memcached.pid

参数|描述
:---------------|:---------------
-d|守护进程方式启动
-f|增长因子
-M|内存用光时报错
-m|使用内存大小,单位m
-o|子参数:slab_reassign,slab_automove,cache就会以每10秒一次的频率进行重分配
-u|启动的用户
-l|绑定地址
-p|绑定端口
-c|连接数
-P|pid保存文件
