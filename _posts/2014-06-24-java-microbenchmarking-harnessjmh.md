---
layout: post
title: "Java Microbenchmarking Harness(jmh)"
description: ""
category: "openjdk"
tags: jmh
---
{% include JB/setup %}

#####下载源代码,并放到本地库中
    hg clone http://hg.openjdk.java.net/code-tools/jmh/ jmh
    cd jmh/
    mvn clean install -DskipTests

#####创建项目
    mvn archetype:generate \
          -DinteractiveMode=false \
          -DarchetypeGroupId=org.openjdk.jmh \
          -DarchetypeArtifactId=jmh-java-benchmark-archetype \
          -DgroupId=org.sample \
          -DartifactId=test \
          -Dversion=1.0
<!-- more -->
#####或者把如下添加内容添加到pom.xml
    <dependency>
            <groupId>org.openjdk.jmh</groupId>
            <artifactId>jmh-core</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
        <dependency>
            <groupId>org.openjdk.jmh</groupId>
            <artifactId>jmh-generator-annprocess</artifactId>
            <version>1.0-SNAPSHOT</version>
            <scope>provided</scope>
        </dependency>
        <prerequisites>
        <maven>3.0</maven>
    </prerequisites>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.0</version>
                <configuration>
                    <compilerVersion>1.6</compilerVersion>
                    <source>1.6</source>
                    <target>1.6</target>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-surefire-plugin</artifactId>
                <version>2.14.1</version>
                <configuration>
                    <forkMode>always</forkMode>
                    <redirectTestOutputToFile>true</redirectTestOutputToFile>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-shade-plugin</artifactId>
                <version>2.0</version>
                <executions>
                    <execution>
                        <phase>package</phase>
                        <goals>
                            <goal>shade</goal>
                        </goals>
                        <configuration>
                            <finalName>benchmarks</finalName>
                            <transformers>
                                <transformer
                                        implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                                    <mainClass>org.openjdk.jmh.Main</mainClass>
                                </transformer>
                            </transformers>
                        </configuration>
                    </execution>
                </executions>
            </plugin>

        </plugins>
    </build>

#####实例代码
    @State(Scope.Benchmark)
    public class SplitWordTest {
    
        private IKAnalyzer analyzer;
    
        @State(Scope.Thread)
        public static class Ids {
            private List<String> ids;
    
            @Setup
            public void setup() {
                ids = readTextList("/Users/bboniao/Desktop/title");
    
            }
        }
    
        public static List<String> readTextList(String path) {
            String read = "";
            BufferedReader br = null;
            List<String> list = new ArrayList<String>();
            try {
                File file = new File(path);
                FileReader fileread = new FileReader(file);
                br = new BufferedReader(fileread);
                while ((read = br.readLine()) != null) {
                    list.add(read);
                }
            } catch (IOException e) {
                e.printStackTrace();
            } finally {
                try {
                    if (br != null) {
                        br.close();
                    }
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
            return list;
        }
    
        @Setup
        public void init() {
            analyzer = new IKAnalyzer();
            analyzer.setUseSmart(true);
            TokenStream tokenStream = analyzer.tokenStream("content", new StringReader("test"));
            tokenStream.addAttribute(CharTermAttribute.class);
            List<String> list = new ArrayList<String>();
            try {
                while (tokenStream.incrementToken()) {
                    CharTermAttribute charTermAttribute = tokenStream.getAttribute(CharTermAttribute.class);
                    list.add(charTermAttribute.toString());
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
            list.clear();
        }
    
        @Benchmark
        @BenchmarkMode({Mode.Throughput, Mode.AverageTime, Mode.SampleTime, Mode.SingleShotTime})
        @OutputTimeUnit(TimeUnit.MILLISECONDS)
        public List<List<String>> getAnalysisResult(Ids ids) throws IOException {
            List<List<String>> list = new ArrayList<List<String>>(ids.ids.size());
            for (String s : ids.ids) {
                list.add(get(s));
            }
            return list;
        }
    
        private List<String> get(String keyWord) throws IOException {
            TokenStream tokenStream = analyzer.tokenStream("content", new StringReader(keyWord));
            tokenStream.addAttribute(CharTermAttribute.class);
            List<String> list = new ArrayList<String>();
            while (tokenStream.incrementToken()) {
                CharTermAttribute charTermAttribute = tokenStream.getAttribute(CharTermAttribute.class);
                list.add(charTermAttribute.toString());
            }
            return list;
        }
    
        public static void main(String[] args) throws RunnerException {
            Options opt = new OptionsBuilder()
                    .include(".*" + SplitWordTest.class.getSimpleName() + ".*")
                    .warmupIterations(5)
                    .measurementIterations(5)
                    .forks(1)
                    .build();
    
            new Runner(opt).run();
        }
    }

#####执行如下命令
    mvn clean package
    java  -jar target/benchmarks.jar -wi 3 -i 3 '.*SplitWordTest.*'     

#####最后汇总结果
    Benchmark                                   Mode   Samples        Score  Score error    Units
    o.o.j.t.SplitWordTest.getAnalysisResult    thrpt        30        0.034        0.001   ops/ms
    o.o.j.t.SplitWordTest.getAnalysisResult     avgt        30       29.954        0.742    ms/op
    o.o.j.t.SplitWordTest.getAnalysisResult   sample      1109       30.195        0.306    ms/op
    o.o.j.t.SplitWordTest.getAnalysisResult       ss        30       89.442        8.043       ms
