---
layout: post
title: "G1(Garbage First)的使用"
description: ""
category: "jvm"
tags: g1 gc
---
{% include JB/setup %}

###为什么采用G1

Hbase开启SLAB之后还是会产生很多碎片,导致Full GC.原因是BlockCache产生很多碎片,CMS对碎片无能为力.采用G1吞吐量与CMS相当

###我的参数配置
    -Xmx24g -Xms24g -XX:PermSize=96m -XX:MaxPermSize=96m -XX:+UseG1GC -XX:SurvivorRatio=6 -XX:MaxGCPauseMillis=400 -XX:G1ReservePercent=15  -XX:InitiatingHeapOccupancyPercent=40 -XX:ConcGCThreads=8

1. 新生代和region大小没有指定,G1可以更好的自动分配资源.
2. 提高MaxGCPauseMillis,实时性要求没有太严格,可以提高吞吐量
3. 提高G1ReservePercent,降低InitiatingHeapOccupancyPercent,提高ConcGCThreads因为遇到内存不足而产生了Full GC

<!-- more -->
###G1常用参数

参数|描述
:---------------|:---------------
-XX:+UseG1GC|开启G1
-XX:MaxGCPauseMillis=n|设置GC暂停的最大时间,这只是目标,尽量达到,默认值是 200 毫秒,`过小影响吞吐量`
-XX:InitiatingHeapOccupancyPercent=n|整个堆(而不是某个年代)使用量达到此值,便会触发并发GC周期.值为0则是连续触发,默认值为45
-XX:NewRatio=n|老年代与新生代的比值,默认值为2
-XX:SurvivorRatio=n|伊甸园代与生存代的比率,默认值为8
-XX:MaxTenuringThreshold=n|生存代存活的最大门限,默认值为15
-XX:ParallelGCThreads=n|设置垃圾回收器并行阶段的线程数,默认值与JVM运行的平台有关,`将 n 的值设置为逻辑处理器的数量`
-XX:ConcGCThreads=n|设置并发垃圾回收器使用的线程数,默认值与JVM运行的平台有关
-XX:G1ReservePercent=n|设置剩余的内存量,减少跃迁失败的可能,默认值为10
-XX:G1HeapRegionSize=n|设置G1平分java堆而产生区域的大小,默认值可以提供最大的工效性.最小值为1M,最大为32M,最多划分1024个,`建议使用默认值`

###标记周期的各个阶段

1. __初始标记阶段：__在此阶段，G1 GC 对根进行标记。该阶段与常规的 (`STW`) 年轻代垃圾回收密切相关。
2. __根区域扫描阶段：__G1 GC 在初始标记的存活区扫描对老年代的引用，并标记被引用的对象。该阶段与应用程序（`非STW`）同时运行，并且只有完成该阶段后，才能开始下一次 `STW` 年轻代垃圾回收。
3. __并发标记阶段：__G1 GC 在整个堆中查找可访问的（存活的）对象。该阶段与应用程序同时运行，可以被 `STW` 年轻代垃圾回收中断。
4. __重新标记阶段：__该阶段是 `STW` 回收，帮助完成标记周期。G1 GC 清空 SATB 缓冲区，跟踪未被访问的存活对象，并执行引用处理。
5. __清理阶段：__在这个最后阶段，G1 GC 执行统计和 RSet 净化的 `STW` 操作。在统计期间，G1 GC 会识别完全空闲的区域和可供进行混合垃圾回收的区域。清理阶段在将空白区域重置并返回到空闲列表时为部分并发。

###内存用尽造成Full GC

查找`to-space exhausted`和`to-space overflow`,表示因内存不够产生Full GC

1. -XX:G1ReservePercent 增加预存内存量
2. -XX:InitiatingHeapOccupancyPercent 减少此值,提前启动标记周期
2.  -XX:ConcGCThreads 增加并行标记线程的数目
