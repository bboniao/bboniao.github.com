---
layout: post
title: "Java Mission Control"
description: ""
category: "jvm"
tags: jmc
---
{% include JB/setup %}

###Mac上使用

1. 下载`jdk7u40`以后的版本,执行jmc.mac os上暂时用不了,但可以用方法2
2. 使用eclipse:
  [eclipse3.8](http://archive.eclipse.org/eclipse/downloads/drops/R-3.8-201206081200/download.php?dropFile=eclipse-platform-3.8-macosx-cocoa-x86_64.tar.gz)
  使用其他平台jdk的jmc更新完插件,拷贝JAVA_HOME/lib/missioncontrol/plugins下的jar包到eclipse/plugins即可

###启动Java Flight Recorder (JFR)

1. 启动参数:

    -Dcom.sun.management.jmxremote.port=7091 -Dcom.sun.management.jmxremote.rmi.port=7091 -Dcom.sun.management.jmxremote.authenticate=false -Dcom.sun.management.jmxremote.ssl=false  -XX:+UnlockCommercialFeatures -XX:+FlightRecorder

<!-- more -->
2. 启动代理(5596是jvm的pid,jvm启动时必须添加-XX:+UnlockCommercialFeatures -XX:+FlightRecorder):

    jcmd 5596 ManagementAgent.start jmxremote.ssl=false jmxremote.port=7091 jmxremote.rmi.port=7091 jmxremote.authenticate=false jmxremote.autodiscovery=true 
