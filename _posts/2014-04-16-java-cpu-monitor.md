---
layout: post
title: "java cpu monitor"
description: ""
category: "jvm"
tags: jstack jvm
---
{% include JB/setup %}

###转自[java-cpu-monitor shell脚本](http://www.javaarch.net/jiagoushi/55.htm)
    #!/bin/bash                                                                                                                           
 
    threshold=${1-95}
    now=$(date '+%Y-%m-%d %H:%M:%S')
    cache=()
     
    jps -q | xargs ps hH k -pcpu o pid,lwp,pcpu,args \
      | awk "\$3 >= $threshold" | while read line; do
     
      array=($(echo "$line"))
      pid=${array[0]}
      lwp=$(printf '%#x' ${array[1]})
     
      if [ -z "${cache[$pid]}" ]; then
        cache[$pid]=$(jstack $pid)
      fi
     
      echo "[$now] ${line}"
      echo
      echo "${cache[$pid]}" | sed -n "/nid=$lwp/,/^$/p"
      echo
     
    done

threshold设置一下cpu的使用率

<!-- more -->
