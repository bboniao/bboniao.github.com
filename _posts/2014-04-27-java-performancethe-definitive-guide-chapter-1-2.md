---
layout: post
title: "Java Performance:The Definitive Guide Chapter 1 2"
description: ""
category: "Java Performance:The Definitive Guide"
tags: performance
---
{% include JB/setup %}

#####The Complete Performance Story
1. Write Better Algorithms
2. Write Less Code
3. Prematurely Optimize
4. Look Elsewhere: The Database Is Always the Bottleneck
5. Optimize for the Common Case
 - Optimize code by profiling it and focusing on the operations in the profile taking the most time
 - Apply Occam's Razor to diagnosing performance problems
 - Write simple algorithms for the most common operations in an application

<!-- more -->

##### Test a Real Application
1. Microbenchmarks
 - Microbenchmarks must use their results
 - Microbenchmarks must not include extraneous operations
 - Microbenchmarks must measure the correct input
2. Macrobenchmarks
3. Mesobenchmarks

##### Understand Throughput, Batching, and Response Time
1. Elapsed Time(Batch)Measurements
2. Throughput Measurements
3. Response Time Tests
[Faban](http://faban.org/)

##### Understand Variability

##### Test Early, Test Often
