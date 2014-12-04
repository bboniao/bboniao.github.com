---
layout: post
title: "HBase Performance Tuning"
description: ""
category: "hbase"
tags: performance,tuning
---
{% include JB/setup %}

1. 关闭swap
`nohup swapoff -a > /dev/null &`
`echo 0 > /proc/sys/vm/swappiness`
修改/etc/sysctl.conf，新增vm.swappiness = 0
2. 减少full gc, 采用BucketCache, [性能测试](http://www.n10k.com/blog/blockcache-showdown/)
name|value
:---------------|:---------------
hfile.block.cache.size|0.01
hbase.bucketcache.ioengine|file:/tmp/bucketcache_tmpfs/cache.data
hbase.bucketcache.percentage.in.combinedcache|0.99
hbase.bucketcache.size|20480
hbase.regionserver.global.memstore.upperLimit|0.60
hbase.regionserver.global.memstore.lowerLimit|0.55
编辑/etc/cron.daily/tmpwatch, 不包含目录/tmp/bucketcache_tmpfs
3. 重要配置
name|value|description
:---------------|:---------------|:---------------
hbase.master.wait.on.regionservers.mintostart|5|regionserver一半
dfs.datanode.failed.volumes.tolerated|2|挂载6个, 2个坏了可以容忍

4. 启动backup master
5. MTTR配置
name|value|description
:---------------|:---------------|:---------------
hbase.lease.recovery.dfs.timeout|23000|recover lease的超时时间, 大于dfs的
如下hdfs配置||
dfs.client.socket-timeout|10000|60s变为10s
dfs.datanode.socket.write.timeout|10000|8 * 60变为10s
ipc.client.connect.timeout|3000|60s变为3s
ipc.client.connect.max.retries.on.timeouts|2|
dfs.namenode.avoid.read.stale.datanode|true|
dfs.namenode.stale.datanode.interval|20000|默认30s
dfs.namenode.avoid.write.stale.datanode|true|
6. 队列配置
name|value|description
:---------------|:---------------|:---------------
hbase.ipc.server.max.callqueue.length|handlerCount*10|调用队列的大小, 默认handlerCount*10
hbase.ipc.server.callqueue.read.share|0.6|默认为0, 读写不分
hbase.ipc.server.callqueue.handler.factor|1|默认为0, 共享一个队列
7. 放弃版本
8. family不超过三个
9. Leveraging local data
10. Bloom Filters
