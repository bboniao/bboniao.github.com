---
layout: post
title: "Java Mission Control"
description: ""
category: "jvm"
tags: jmc
---
{% include JB/setup %}

###Mac上使用

下载`jdk7u40`以后的版本,执行`sudo jmc`,使用管理员启动jmc.

###启动Java Flight Recorder (JFR)

1. 启动参数:

    -Dcom.sun.management.jmxremote.port=7091 -Dcom.sun.management.jmxremote.rmi.port=7091 -Dcom.sun.management.jmxremote.authenticate=false -Dcom.sun.management.jmxremote.ssl=false  -XX:+UnlockCommercialFeatures -XX:+FlightRecorder

<!-- more -->
2. 额外启动端口:

    jcmd 5596 ManagementAgent.start jmxremote.ssl=false jmxremote.port=7091 jmxremote.rmi.port=7091 jmxremote.authenticate=false jmxremote.autodiscovery=true 
