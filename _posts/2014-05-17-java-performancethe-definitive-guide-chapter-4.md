---
layout: post
title: "Java Performance:The Definitive Guide Chapter 4"
description: ""
category: "java performance:the definitive guide"
tags: performance
---
{% include JB/setup %}

#####Tuning the Code Cache
`-XX:ReservedCodeCacheSize=N`,`-XX:InitialCodeCacheSize=N`,调整code cache size的大小.查看方式:jmc--MBean Server--Memory--Active Memory Pools--Code Cache和jconsole--Memory--Chart--Memory pool "Code Cache"
 
jvm类型|默认code cache size
:---------------|:---------------
32-bit client,Java 8|32MB
32-bit server with tiered compilation, Java 8|240MB
64-bit server with tiered compilation, Java 8|240MB
32-bit client, Java 7|32MB
32-bit server, Java 7|32MB
64-bit server, Java 7|48MB
64-bit server with tiered compilation, Java 7|96MB
<!-- more -->
#####Compilation Thresholds
`OSR trigger=(CompileThreshold*((OnStackReplacePercentage-InterpreterProfilePercentage)/100))`

#####Inspecting the Compilation Process
`-XX:+PrintCompilation`,打印编译过程

各种属性说明

符号|说明
:---------------|:---------------
%|OSR编译
s|方法同步
!|异常处理的方法
b|编译发生在阻塞模式
n|编译放生在本地方法的包装器上

`jstat -compiler pid`

    Compiled Failed Invalid   Time   FailedType FailedMethod
         199      0       0     1.16          0 
`jstat -printcompilation pid Nms`

    Compiled  Size  Type Method
         208     37    1 java/awt/Rectangle <init>
         209    159    1 java/util/HashMap <init>
         210    836    1 sun/java2d/pipe/AlphaPaintPipe renderPathTile
         212    836    1 sun/java2d/pipe/AlphaPaintPipe renderPathTile
         213    319    1 java/security/AccessControlContext optimize
         215    361    1 java/awt/TexturePaintContext getRaster

#####Tiered Compilation Levels
level|说明
:---------------|:---------------
0|解释代码
1|简单C1编译代码
2|限制C1编译代码
3|完全C1编译代码
4|C2编译代码

#####总结
1. 不用担心小方法(尤其set/get),编译器会优化
2. 需要编译的代码会放到队列中.
3. 需要调节code cacehe,以免影响JIT
4. 越简单的方法,越容易优化.复杂的循环结果和大方法,将影响JIT


