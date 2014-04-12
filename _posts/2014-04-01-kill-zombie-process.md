---
layout: post
title: "Kill Zombie Process"
description: ""
category: "linux"
tags: kill
---
{% include JB/setup %}

###查找僵尸进程
`ps aux |awk '{print $8 " " $2}' |grep -w Z`

###查找此进程信息和父进程pid
`ps -ef |grep $pid`

<!-- more -->
