---
layout: post
title: "Hbase Batch Get"
description: ""
category: "hbase"
tags: hbase,get
---
{% include JB/setup %}

实现|描述|分组|运行时间(ms)|丢失率|avg|min|max|stddev|qps
:---------------|:---------------|:---------------|:---------------|:---------------|:---------------|:---------------|:---------------|:---------------|:---------------
AsyncHbaseBatchGetServiceImpl|采用asynchbase和CountDownLatch|1|7070|0.01%|1|0|332|10.66|610
AsyncJdkBatchGetServiceImpl|采用Future, 父子线程处理|5|12055|0%|1|0|277|13.64|594
DeferredBatchGetServiceImpl|Deferred和CountDownLatch|5|50096|0%|5|0|505|49.84|410
HystrixBatchGetServiceImpl|采用Hystrix的Request Collapsing|1|154824|0%|15|0|2086|154.34|218
NativeBatchGetServiceImpl|原生batch get, 内部实现采用AsyncProcess|1|386787|0%|41|0|13605|446.93|102
AsyncJdkBatchGetServiceImplV1|采用CountDownLatch|5|49698|0%|5|0|502|49.84|406
AsyncJdkBatchGetServiceImplV2|采用Future, 父子线程处理, 没有分组|1|10075|0%|1|0|330|10.87|605
NativeGetServiceImpl|原生get|1|112326|0%|11|0|2047|111.07|265
<!-- more -->

#####说明

1. "运行时间"为单线程10000个请求, 额外有100个请求用于预热; "qps","avg","min,"max","stddev"都是jmeter测试的数据
2. 每次批量get, 抓取100条
3. 分组的意思是, 每个线程有多少个请求采用原生batch get. 避免线程池爆满
4. 异步操作的timeout时间为500ms

#####总结, 性能从高到低

实现|时间|描述
:---------------|:---------------|:---------------
AsyncHbaseBatchGetServiceImpl|7070|性能最高; 单个异步请求, 线程池不会跑满; 内部有个多个请求合并的流程, 直接节省线程使用; 不依赖hbase和hadoop代码, 兼容性俱佳. 但会有一些结果丢失|
AsyncJdkBatchGetServiceImplV2|10075|相比AsyncJdkBatchGetServiceImpl, 性能提高, 是因为没有分组, 采用原生get. 很容易跑满线程池|
AsyncJdkBatchGetServiceImpl|12055|推荐使用. |
AsyncJdkBatchGetServiceImplV1|49698|采用CountDownLatch,然后每5个批量操作|
DeferredBatchGetServiceImpl|50096|主要是为了了解Deferred的性能如何, 与AsyncJdkBatchGetServiceImplV1基本一致|
NativeGetServiceImpl|112326|原生get, 没有做批量处理
HystrixBatchGetServiceImpl|154824|单次请求, Hystrix底层进行合并|
NativeBatchGetServiceImpl|386787|原生batch get|

#####[jmeter配置文件](https://github.com/bboniao/bboniao.github.com/releases/download/hbase-client.jmx/hbase-client.jmx)

#####[源代码](https://github.com/bboniao/hbase)
