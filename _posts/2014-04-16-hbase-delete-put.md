---
layout: post
title: "hbase delete put的陷阱"
description: ""
category: "hbase"
tags: hbase delete put
---
{% include JB/setup %}

###问题
删除数据之后,再插入,查询的时候怎么也找不到.删除和插入都是同一个rowkey,同一个列,同一个版本.
###原因
如果rowkey/family/qulifier/timestamp都一样,多次删除和插入之后,获取哪个值取决于KeyValue.Type.Put之前是Delete就不会被插入了.

<!-- more -->

###分析
    KeyValue(byte[] row, int roffset, int rlength,  
       byte[] family, int foffset, int flength, byte[] qualifier, int qoffset,  
       int qlength, long timestamp, Type type, byte[] value, int voffset,  
       int vlength)
![image]({{ IMAGE_PATH }}/keyvalue.jpg)
由代码和图片看出,除了rowkey/family/qulifier/timestamp之外还需要KeyValue.Type才能真正定位到value.想要delete之后还能put,就需要compact,最好major_compact,之后再put就没问题了
