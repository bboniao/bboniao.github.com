---
layout: post
title: "A Better Log4j SMTPAppender"
description: ""
category: "java"
tags: log4j
---
{% include JB/setup %}

###log4j配置mail发送log,会有多少发多少,如下改动可以在指定时间内发送指定次数的log

###java代码

    package com.sohu.rc.util;
    import org.apache.log4j.net.SMTPAppender;
    public class LimitedSMTPAppender extends SMTPAppender {
        private int limit = 10; // max at 10 mails ...
        private int cycleSeconds = 3600; // ... per hour
        public void setLimit(int limit) {
            this.limit = limit;
        }
        public void setCycleSeconds(int cycleSeconds) {
            this.cycleSeconds = cycleSeconds;
        }
        private int lastVisited;
        private long lastCycle;
        @Override
        protected boolean checkEntryConditions() {
            final long now = System.currentTimeMillis();
            final long thisCycle = now - (now % (1000L * cycleSeconds));
            if (lastCycle != thisCycle) {
                lastCycle = thisCycle;
                lastVisited = 0;
            }
            lastVisited++;
            return super.checkEntryConditions() && lastVisited <= limit;
        }
    }

###log4j.properties的配置
    log4j.rootLogger=info,mail
    log4j.threshhold=ALL
    
    log4j.appender.mail=com.sohu.rc.util.LimitedSMTPAppender
    log4j.appender.mail.limit=3
    log4j.appender.mail.cycleSeconds=60
    log4j.appender.mail.Threshold=ERROR
    log4j.appender.mail.BufferSize=32
    log4j.appender.mail.From = bboniao@gmail.com
    log4j.appender.mail.SMTPHost=10.11.132.229
    log4j.appender.mail.Subject=Rc_Strategy_Log4J_Message
    log4j.appender.mail.To= bboniao@163.com
    log4j.appender.mail.layout=org.apache.log4j.PatternLayout
    log4j.appender.mail.layout.ConversionPattern=[%-5p] %d(%r) --> [%t] %l: %m %x %n

###hadoop/hbase配置
#####hadoop:打开bin/hadoop-daemon.sh,配置`HADOOP_ROOT_LOGGER`
#####hbase:打开bin/hbase-daemon.sh,配置`HBASE_ROOT_LOGGER`
#####否则log4j.properties中的`log4j.rootLogger`不会生效

###[参考链接](http://blog.cherouvim.com/a-better-smtpappender/)
