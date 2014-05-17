---
layout: post
title: "Java Performance:The Definitive Guide Chapter 3"
description: ""
category: "java performance:the definitive guide"
tags: performance
---
{% include JB/setup %}

#####Network Usage
[nicstat](http://jaist.dl.sourceforge.net/project/nicstat/nicstat-src-1.95.tar.gz)

`nicstat 1`

        Time      Int   rKB/s   wKB/s   rPk/s   wPk/s    rAvs    wAvs %Util    Sat
    21:45:53       lo    1.45    1.45    8.54    8.54   174.0   174.0  0.00   0.00
    21:45:53     eth0   35.45   12.14   72.33   86.34   501.9   144.0  0.04   0.00
    21:45:53     eth1    0.43    0.36    6.01    1.05   73.54   354.8  0.00   0.00
        Time      Int   rKB/s   wKB/s   rPk/s   wPk/s    rAvs    wAvs %Util    Sat
    21:45:54       lo    2.80    2.80   12.99   12.99   220.3   220.3  0.00   0.00
    21:45:54     eth0    1.35    1.78   20.99   21.99   66.00   82.86  0.00   0.00
    21:45:54     eth1    0.41    0.00    7.00    0.00   60.00    0.00  0.00   0.00
        Time      Int   rKB/s   wKB/s   rPk/s   wPk/s    rAvs    wAvs %Util    Sat
    21:45:55       lo    0.30    0.30    6.00    6.00   50.67   50.67  0.00   0.00
    21:45:55     eth0    2.70    3.93   36.99   50.99   74.65   78.88  0.01   0.00
    21:45:55     eth1    0.23    0.00    4.00    0.00   60.00    0.00  0.00   0.00
<!-- more  -->

#####jcmd
命令|说明
:---------------|:---------------
jcmd pid VM.uptime|jvm启动时长
jcmd pid VM.system_properties|列出jvm系统参数
jcmd pid VM.version|jvm版本
jcmd pid VM.command_line|jvm启动参数
jcmd pid VM.flags [-all]|jvm性能参数
jcmd pid Thread.print|等价于jstack pid
jcmd pid PerfCounter.print|jvm所有性能指标

#####Working with tuning flags
`jcmd pid VM.flags -all | grep manageable`列出的jvm参数,可以使用`jinfo -flag [+-]parameter pid`动态修改

#####Enable JFR via the command line
`-XX:+UnlockCommercialFeatures -XX:+FlightRecorder`开启JFR之后,可以通过`-XX:+FlightRecorderOptions=string`和jcmd pid JFR.start [option_list]激活记录
`string`和`[option_list]`,是逗号隔开的键值对,如下:

参数|说明
:---------------|:---------------
name=name|记录名
defaultrecording=<true|false>|是否初始开始记录,默认值为false
settings=path|包含JFR配置的文件名
delay=time|延迟时间(例如:30s,1h)
duration=time|记录持续时间
filename=path|记录输出的目录
compress=<true|false>|是否使用gzip压缩,默认值为false
maxage=time|记录文件保留时间
maxsize=size|记录文件最大size,(例如:1024K,1M)

`jcmd pid JFR.dump [option_list]`,随时dump记录文件,option如下:

参数|说明
:---------------|:---------------
name=name|记录名
recording=n|记录数
filename=path|记录文件的路径

`jcmd pid JFR.check [verbose]`,检查记录是否在运行

`jcmd pid JFR.stop [option_list]`,终止记录进程

参数|说明
:---------------|:---------------
name=name|记录名
recording=n|停止记录数,可以参考JFR.check
discard=boolean|如果为true,抛弃数据,不会写入文件
filename=path|记录文件的路径

