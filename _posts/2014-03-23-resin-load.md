---
layout: post
title: "记录一次resin load过高的解决过程"
description: ""
category: "java"
tags: jstack,ss,housemd
---
{% include JB/setup %}

#####1.使用[housemd](https://github.com/bboniao/bboniao.github.com/releases/download/0.2.6/housemd-0.2.6.zip)统计方法耗时,[中文说明](https://github.com/CSUG/HouseMD/wiki/UserGuideCN)
`./housewd $jvm_pid`

`trace -d  -l 50000 -t 50000 ClassName`

#####2.使用ss统计tcp信息,参照[文章](http://www.ttlsa.com/linux-command/ss-replace-netstat/)
`ss -s`

`ss -t -a | grep ESTAB`

<!-- more -->
#####3.统计java各个线程的个数
`jstack $jvm_pid | grep 'nid=' | awk -F '-' '{print $1}' | awk '{++S[$0]} END {for (a in S) print S[a],a}' | sort -nr`

#####4.统计各个状态线程的个数
`jstack $jvm_pid | grep java.lang.Thread.State | awk '{++S[$0]} END {for (a in S) print S[a],a}' | sort -nr`

#####5.查找哪些线程cpu使用过高
`jstack $jvm_pid > jstack01 && ps mp $jvm_pid -o THREAD,tid,time | sort -k2nr | awk '{printf("%x",$8)}{print " ",($2"%")," ",$9}' | head -30`


