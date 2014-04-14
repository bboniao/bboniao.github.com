---
layout: post
title: "memcache使用一致性hash,找出指向的节点"
description: ""
category: "memcache"
tags: memcache
---
{% include JB/setup %}

###代码如下
    //地址列表
    String[] array = addrs.split(";");
    List<InetSocketAddress> isa = new ArrayList<InetSocketAddress>();
    for(int i = 0;i < array.length;i ++){
        String[] addr = array[i].split(":");
        isa.add(new InetSocketAddress(addr[0], Integer.parseInt(addr[1])));
    }
    MemcachedClientBuilder builder = new XMemcachedClientBuilder(isa);
    //一致性hash
    builder.setSessionLocator(new KetamaMemcachedSessionLocator());
    builder.setOpTimeout(200);

    builder.build();
    String memcacheKey = "test";
    InetSocketAddress i = builder.getSessionLocator().getSessionByKey(memcacheKey).getRemoteSocketAddress();
    System.out.println(i.getHostString());
    System.out.println(i.getPort());
<!-- more -->
