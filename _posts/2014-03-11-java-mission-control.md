---
layout: post
title: "Java Mission Control"
description: ""
category: "jvm"
tags: jmc
---
{% include JB/setup %}

###Mac上使用

1. 下载`jdk7u40`以后的版本,执行`sudo jmc`,使用管理员启动jmc.
2. 使用eclipse:
  [eclipse3.8](http://archive.eclipse.org/eclipse/downloads/drops/R-3.8-201206081200/download.php?dropFile=eclipse-platform-3.8-macosx-cocoa-x86_64.tar.gz)
  [插件路径](https://download.oracle.com/technology/products/missioncontrol/updatesites/experimental/5.2.0/eclipse/)

###启动Java Flight Recorder (JFR)

1. 启动参数:

    -Dcom.sun.management.jmxremote.port=7091 -Dcom.sun.management.jmxremote.rmi.port=7091 -Dcom.sun.management.jmxremote.authenticate=false -Dcom.sun.management.jmxremote.ssl=false  -XX:+UnlockCommercialFeatures -XX:+FlightRecorder

<!-- more -->
2. 启动代理(5596是jvm的pid,jvm启动时必须添加-XX:+UnlockCommercialFeatures -XX:+FlightRecorder):

    jcmd 5596 ManagementAgent.start jmxremote.ssl=false jmxremote.port=7091 jmxremote.rmi.port=7091 jmxremote.authenticate=false jmxremote.autodiscovery=true 
