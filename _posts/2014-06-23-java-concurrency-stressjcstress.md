---
layout: post
title: "Java Concurrency Stress(jcstress)"
description: ""
category: "openjdk"
tags: jcstress
---
{% include JB/setup %}

#####下载源代码:hg clone http://hg.openjdk.java.net/code-tools/jcstress/jcstress

#####在tests-custom/pom.xml添加即将测试代码的依赖

#####在tests-custom下编写测试代码

#####在tests-custom下resources/org/openjdk/jcstress/desc下编写xml描述文件

#####在源代码根目录,执行`mvn clean install -pl tests-custom -am`

#####再执行 `java -jar tests-custom/target/jcstress.jar -t '.*TreeMapUpdateTest.*'`

<!-- more -->
#####java实例

    @JCStressTest
    @State
    public class TreeMapUpdateTest {
    
        private static final Random R = new Random();
    
        private static final Map<String, String> CONCURRENT_MAP = new ConcurrentSkipListMap<String, String>();
    
        @Actor
        public void foreach(IntResult1 result) {
            for (Map.Entry<String, String> e : CONCURRENT_MAP.entrySet()) {
                result.r1 = 1;
            }
        }
    
        @Actor
        public void add2() {
            CONCURRENT_MAP.put(String.valueOf(R.nextInt()), String.valueOf(R.nextInt()));
        }
    }

#####xml实例
    <testsuite>
    <test name="com.sohu.tv.jcstress.test.TreeMapUpdateTest">
        <contributed-by>Michael Nitschinger</contributed-by>
        <description>
            Tests the thread-safeness of the StringUtil class.
        </description>
        <case>
            <match>[1]</match>
            <expect>ACCEPTABLE</expect>
            <description>
                Acceptable to see true.
            </description>
        </case>
        <unmatched>
            <expect>FORBIDDEN</expect>
            <description>
                Other cases are not expected. -1 would mean an exception raised.
            </description>
        </unmatched>
    </test>
    </testsuite>
