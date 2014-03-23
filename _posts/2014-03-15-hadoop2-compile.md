---
layout: post
title: "hadoop2 compile"
description: ""
category: "hadoop"
tags: hadoop2
---
{% include JB/setup %}


###使用Maven 3.0

###安装依赖的包:

    yum install gcc-c++ cmake zlib-devel

###安装protobuf

###编辑hadoop-common-project/hadoop-auth/pom.xml.添加

    <dependency>
      <groupId>org.mortbay.jetty</groupId>
      <artifactId>jetty-util</artifactId>
      <scope>test</scope>
    </dependency>

###执行命令:

    mvn package -Pdist,native -DskipTests -Dtar

###编译好的tar在hadoop-common-project/hadoop-common/target

<!-- more -->
