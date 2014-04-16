---
layout: post
title: "为什么要控制hbase cloumnFamily的数量"
description: ""
category: "hbase"
tags: hbase cloumnFamily
---
{% include JB/setup %}

设计Hbase schema,要尽量使用一个column family.原因主要是flush和compact机制造成的.

flush和compact都是基于region级别的:当一个column family数据导入过多,发生flush,也会触发其他所有column family的(此时很小的memstore也会被触发的).而flush会生成很多storefile,紧接着就会触发compact.过多没必要的flush/compact会大大增加io的开销,影响hbase整体性能.

<!-- more -->
