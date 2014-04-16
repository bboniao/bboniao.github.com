---
layout: post
title: "HBase在单Column和多Column情况下批量Put的性能对比分析"
description: ""
category: "hbase"
tags: hbase column put
---
{% include JB/setup %}

###转自[量子恒道官方博客](http://blog.linezing.com/?p=2106)

针对HBase在单column family单column qualifier和单column family多column qualifier两种场景下，分别批量Put写入时的性能对比情况，下面是结合HBase的源码来简单分析解释这一现象。

###1. 测试结果
在客户端批量写入时，单列族单列模式和单列族多列模式的TPS和RPC次数相差很大，以客户端10个线程，开启WAL的两种模式下的测试数据为例，
#####1)单列族单列模式下，TPS能够达到12403.87，实际RPC次数为53次；
#####2)单列族多列模式下，TPS只有1730.68，实际RPC次数为478次。
二者TPS相差约7倍，RPC次数相差约9倍。详细的测试环境这里不再罗列，我们这里关心的只是在两种条件下的性能差别情况。

<!-- more -->

###2. 粗略分析
下面我们先从HBase存储原理层面“粗略”分析下为什么出现这个现象：
#####HBase 的KeyValue类中自带的字段占用大小约为50~60 bytes左右（参考HBase源码org/apache/hadoop/hbase/KeyValue.java），那么客户端Put一行数据时（53 个字段，row key为64 bytes，value为751 bytes）：
#####1)开WAL，单column family单column qualifier，批量Put：(50~60) + 64 + 751 = 865~875 bytes；
#####2)开WAL，单column family多column qualifier，批量Put：((50~60) + 64) * 53 + 751 = 6793~7323 bytes。
因 此，总体来看，后者实际传输的数据量是前者的：(6793~7323 bytes) / (865~875 bytes) = 7.85~8.36倍，与测试结果478 / 53 = 9.0倍基本相符（由于客户端write buffer大小一样，实际请求数的比例关系即代表了实际传输的数据量的比例关系）。

###3. 源码分析
接下来我们通过对HBase的源码分析来进一步验证以上理论估算值：
#####HBase客户端执行put操作后，会调用put.heapSize()累加当前客户端buffer中的数据，满足以下条件则调用flushCommits()将客户端数据提交到服务端：
#####1)每次put方法调用时可能传入的是一个List<Put>，此时每隔DOPUT_WB_CHECK条（默认为10条），检查当前缓存数据是否超过writeBufferSize（测试中被设置为5MB），超过则强制执行刷新；
#####2)autoFlush被设置为true，此次put方法调用后执行一次刷新；
#####3)autoFlush被设置为false，但当前缓存数据已超过设定的writeBufferSize，则执行刷新。
    private void doPut(final List<Put> puts) throws IOException {
        int n = 0;
        for (Put put : puts) {
            validatePut(put);
            writeBuffer.add(put);
            currentWriteBufferSize += put.heapSize();
            // we need to periodically see if the writebuffer is full instead 
            // of waiting until the end of the List
            n++;
            if (n % DOPUT_WB_CHECK == 0
                    && currentWriteBufferSize > writeBufferSize) {
                flushCommits();
            }
        }
        if (autoFlush || currentWriteBufferSize > writeBufferSize) {
            flushCommits();
        }
    }
由上述代码可见，通过put.heapSize()累加客户端的缓存数据，作为判断的依据；那么，我们可以按照测试数据的实际情况，编写代码生成Put对象后就能得到测试过程中的一行数据（由53个字段组成，共计731 bytes）实际占用的客户端缓存大小：
    
    import org.apache.hadoop.hbase.client.Put;
    import org.apache.hadoop.hbase.util.Bytes;
    
    public class PutHeapSize {
        /**
         * @param args
         */
        public static void main(String[] args) {
            // single column Put size
            byte[] rowKey = new byte[64];
            byte[] value = new byte[751];
            Put singleColumnPut = new Put(rowKey);
            singleColumnPut.add(Bytes.toBytes("t"), Bytes.toBytes("col"), value);
            System.out.println("single column Put size: " + singleColumnPut.heapSize());
    
            // multiple columns Put size
            value = null;
            Put multipleColumnsPut = new Put(rowKey);
            for (int i = 0; i < 53; i++) {
                multipleColumnsPut.add(Bytes.toBytes("t"), Bytes.toBytes("col" + i), value);
            }
            System.out.println("multiple columns Put size: " + (multipleColumnsPut.heapSize() + 751));
        }
    }
程序输出结果如下：
#####single column Put size: 1208
#####multiple columns Put size: 10575
由运行结果可得到，9719/1192 = 8.75，与上述理论分析值（7.85~8.36倍）、实际测试结果值（9.0倍）十分接近，基本可以验证测试结果的准确性

如 果你还对put.heapSize()方法感兴趣，可以继续阅读其源码实现，你会发现对于一个put对象来说，其中KeyValue对象的大小最主要决定 了整个put对象的heapSize大小，为了进一步通过实例验证，下面的这段代码分别计算单column和多columns两种情况下一行数据的 KeyValue对象的heapSize大小：

    import org.apache.hadoop.hbase.KeyValue;
    public class KeyValueHeapSize {
        /**
         * @param args
         */
        public static void main(String[] args) {
    
            // single column KeyValue size
            byte[] row = new byte[64]; // test row length
            byte[] family = new byte[1]; // test family length
            byte[] qualifier = new byte[4]; // test qualifier length
            long timestamp = 123456L; // ts
            byte[] value = new byte[751]; // test value length
            KeyValue singleColumnKv = new KeyValue(row, family, qualifier, timestamp, value);
            System.out.println("single column KeyValue size: " + singleColumnKv.heapSize());
    
            // multiple columns KeyValue size
            value = null;
            KeyValue multipleColumnsWithoutValueKv = new KeyValue(row, family, qualifier, timestamp, value);
            System.out.println("multiple columns KeyValue size: " + (multipleColumnsWithoutValueKv.heapSize() * 53 + 751));
        }
    
    }
程序输出结果如下：

#####single column KeyValue size: 920
#####multiple columns KeyValue size: 10079
与前面PutHeapSize程序的输出结果对比发现，KeyValue确实占据了整个Put对象的大部分heapSize空间，同时发现从KeyValue对象级别对比两种情况下的传出数据量情况：10079/920 = 10.9倍，也与实际测试值比较接近。

###4. 相关结论
经过以上分析可以得出以下结论：
#####1)在实际应用场景中，对于单column qualifier和多column qualifier两种情况，如果value长度越长，row key长度越短，字段数（column qualifier数）越少，前者和后者在实际传输数据量上会相差小些；反之则相差较大。
#####2)如果采用多column qualifier的方式存储，且客户端采取批量写入的方式，则可以根据实际情况，适当增大客户端的write buffer大小，以便能够提高客户端的写入吞吐量。
