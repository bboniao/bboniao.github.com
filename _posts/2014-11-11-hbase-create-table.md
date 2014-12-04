---
layout: post
title: "hbase create table"
description: ""
category: "hbase"
tags: table
---
{% include JB/setup %}

#####HTableDescriptor
方法|作用
:---------------|:---------------
setCompactionEnabled|Compaction的开关, 默认开启
setDurability|设置wal的存储模型, 默认同SYNC_WAL
setMaxFileSize|设置split触发的最大文件大小, 默认为-1, 采用全局配置
setMemStoreFlushSize|设置memstore的flush大小, 默认为-1, 采用全局配置
setReadOnly|设置该表只读, 默认为false
<!-- more -->
#####HColumnDescriptor
方法|作用
:---------------|:---------------
setBlockCacheEnabled|开启blogcache, 默认为为true
setBlocksize|谁知blogsize的大小, 默认为65536
setBloomFilterType|bloomfilter的类型, 默认为row
setCacheBloomsOnWrite|在写的时候, 缓存bloomfilter block, 默认为false
setCacheDataOnWrite|是否在写的时候cache data block, 默认false
setCacheIndexesOnWrite|是否在写的时候缓存index blocks, 默认false
setCompactionCompressionType|compaction时压缩方式, 默认没有
setCompressionType|数据压缩方式, 默认没有
setCompressTags|是否使用DataBlockEncoding, 如果DataBlockEncoding没有设置将不会起作用
setDataBlockEncoding|在cache中block数据的编码算法, 默认没有. 查询没有优化, 只是降低内存使用, 提高cpu使用
setEncryptionKey|加密的key
setEncryptionType|加密的方式, 可以为"AES"
setEvictBlocksOnClose|是否在关闭的时候, 剔除block cache, 默认为false
setInMemory|整个表的数据是否保存到内存, 重启后丢失, 默认为false
setKeepDeletedCells|是否保留删除的数据, 默认false
setMaxVersions|存储的版本数, 默认为1
setMinVersions|ttl到期后, 保留数据的版本数, 默认为0
setPrefetchBlocksOnOpen|在开启的时候, 是否预先缓存block, 默认为false
setScope|设置是否可以复制; 默认值为0, 不能, 1可以
setTimeToLive|数据存活时间, 默认Integer最大值, 单位秒
