---
layout: post
title: "StochasticLoadBalancer"
description: ""
category: "hbase"
tags: balance
---
{% include JB/setup %}

#####均衡影响的因素
权重|name(xml配置)|默认值
:---------------|:---------------|:---------------
region数量|hbase.master.balancer.stochastic.regionCountCost|500
region迁移的数量|hbase.master.balancer.stochastic.moveCost|100
每个regiion server, region数量的倾斜度|hbase.master.balancer.stochastic.tableSkewCost|35
根据StoreFile的本地程度|hbase.master.balancer.stochastic.localityCost|25
StoreFile的大小|hbase.master.balancer.stochastic.storefileSizeCost|5
memstore的大小|hbase.master.balancer.stochastic.memstoreSizeCost|5
写请求|hbase.master.balancer.stochastic.writeRequestCost|5
读请求|hbase.master.balancer.stochastic.readRequestCost|5
<!-- more -->
#####说明
1. 基于表的均衡

#####[参考](http://www.cnblogs.com/cenyuhai/p/3650943.html)
