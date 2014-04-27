---
layout: post
title: "System Performance Enterprise and the Cloud Chapter 1-2"
description: ""
category: "System Performance Enterprise and the Cloud"
tags: performance
---
{% include JB/setup %}

#####Generic system software stack

![image]({{ IMAGE_PATH }}/generic_system_software_stack.png)

<!-- more -->

#####Activities
1. Setting performance objectives and performance modeling
2. Performance characterization of prototype software or hardware
3. Performance analysis of development code, pre-integration
4. Performing non-regression testing of software builds, pre- or post-release
5. Benchmarking/ benchmarketing for software releases
6. Proof-of-concept testing in the target environment
7. Configuration optimization for production deployment
8. Monitoring of running production software
9. Performance analysis of issues

#####Perspectives

![image]({{ IMAGE_PATH }}/analysis_perspectives.png)

#####Terminology
1. IOPS
2. Throughput
3. Response time
4. Latency
5. Utilization
6. Saturation
7. Bottleneck
8. Workload
9. Cache

#####Models
1. System under Test
2. Queueing System

#####Concepts
1. Latency

    ![Network connection latency]({{ IMAGE_PATH }}/network_connection_latency.png)
2. Time Scales
![Time Scale of System Latencies]({{ IMAGE_PATH }}/Time_Scale_of_System_Latencies.png)
3. Trade-offs
![Trade offs]({{ IMAGE_PATH }}/Trade-offs.png)
4. Tuning Efforts
![Tuning Efforts]({{ IMAGE_PATH }}/Tuning_Efforts.png)
5. Level of Appropriateness
6. Point-in-Time Recommendations
7. Load versus Architecture
![Load versus Architecture]({{ IMAGE_PATH }}/Load_versus_Architecture.png)
8. Scalability
![Throughput versus load]({{ IMAGE_PATH }}/Throughput_versus_load.png)
![Performance degradation]({{ IMAGE_PATH }}/Performance_degradation.png)
9. Known-Unknowns
 - Known-knowns
 - Known-unknowns
 - Unknown-unknowns
10. Metrics
 - IOPS
 - Throughput
 - Utilization
 - Latency
11. Utilization
 - Time-Based
 - Capacity-Based
 - Non-Idle Time
12. Saturation
![Utilization versus saturation]({{ IMAGE_PATH }}/Utilization_versus_saturation.png)
13. Profiling
14. Caching
 - Most recently used(MRU)
 - Last recently used(LRU)
 - Most frequently used(MFU)
 - Last frequently used(LFU)
 - Not frequently used(NFU)

#####Perspectives
1. Resource Analysis
2. Workload Analysis

#####Methodology
![Generic System Performance Methodologies]({{ IMAGE_PATH }}/analysis_perspectives.png)

1. Streetlight Anti-Method
2. Random Change Anti-Method
 - Pick a random item to change(e.g.,a tunable parameter) 
 - Change it in one direction
 - Measure performance
 - Chang it in the other direction
 - Measure performance
 - Were the results in step 3 or step 5 better than the baseline?If so,keep the change and go back to step 1
3. Blame-Someone-Else Anti-Method
 - Find a system or environment component for which you are not responsible
 - Hypothesize that the issue is with that component
 - Redirect the issue to the team responsible for that component
 - When proven wrong,go back to step 1
4. Ad Hoc Checklist Method
5. Problem Statement
 - What makes you think there is a performance problem?
 - Has this system ever performed well?
 - What changed recently? Software? Hardware? Load?
 - Can the problem be expressed in terms of latency or runtime?
 - Does the problem affect other perple or applications(or is it just you)?
 - What is the environment? What software and hardware are used? Versions? Configuration?
6. Scientific Method
 - Question
 - Hypothesis
 - Prediction
 - Test
 - Analysis
7. Diagnosis Cycle
 - hypothesis--> instrumentation--> data--> hypothesis 
8. Tools Method
 - List available performance tools(optionlly, install or purchase more)
 - For each tool, list useful metrics it provides
 - For each metric, list possible rules for interpretation
9. The USE(utilization saturation errors) Method

 - Procedure
![The USE method flow]({{ IMAGE_PATH }}/Procedure.png)

 - Metrics
![Example USE Method Metrics]({{ IMAGE_PATH }}/Example_USE_Method_Metrics.png)
![Example USE Method Advanced Metrics]({{ IMAGE_PATH }}/Example_USE_Method_Advanced_Metrics.png)

 - Software Resources
  - Mutex locks
  - Thread pools
  - Process/thread capacity
  - File descriptor capacity 

10. Workload Characterization
 - Who is causing the load? Process ID, user ID, remote IP address?
 - Why is the load being called? Code path, stack trace?
 - What are the load characteristics? IOPS, throoughput, direction(read/write)
 - How is the load changing over time? Is there a dialy pattern
11. Drill-Down Analysis
 - three stages
  - Monitoring
  - Identification
  - Analysis
 - Five Whys(database)
  - A database has begun to perform poorly for many queries. Why?
  - It is delayed by disk I/O due to memory paging. Why?
  - Database memory usage has grown too large. Why?
  - The allocator is consuming more memory than it should. Why?
  - The allocator has a memory fragmentation issue.
12. Latency Analysis(Mysql query latency)
 - Is there a query latency issue?(yes)
 - Is the query time largely spent on-CPU or waiting off-CPU?(off-CPU)
 - What is the off-CPU time spent waiting for?(file system I/O)
 - Is the file system I/O time due to disk I/O or lock contention?(disk I/O)
 - Is the disk I/O time likely due to random seeks or data transfer time?(transfer time)

 ![Latency analysis procedure]({{ IMAGE_PATH }}/Latency_analysis_procedure.png)
13. Method R
14. Event Tracing
 - Input: all attributes of an event request: type, direction, size, and so on
 - Times: start time, end time, latency(difference)
 - Result: error status, result of event(size)
15. Baseline Statistics
16. Static Performance Tuning
 - Does the component make sense?
 - Does the configuration make sense for the intended workload?
 - Was the component autoconfigured in the best state for the intended workload?
 - Has the component experienced an error and is it in a degraded state?
 Examples
 - Network interface negotiation: selecting 100 Mbits/s instead of 1 Gbit/s
 - Failed disk in a RAID pool
 - Older version of the operating system, application, or firmware used
 - Mismatched file system record size compared to workload I/O size
 - Server accidentally configured as a router
 - Server configured to use resources, such as authentication, from a remote data center instead of locally
17. Cache Tuning
 - Aim to cache as high in the stack as possible, close to where the work is performed, reducing the operational overhead of cache hits.
 - Check that the cache is enabled and working.
 - Check the cache hit/miss ratios and miss rate.
 - If the cache is dynamic, check its current size.
 - Tune the cache for the workload. This task depends on available cache tunable parameters
 - Tune the workload for the cache. Doing this includes reducing unnecessary comsumers of the cache, which frees up more space for the target workload
18. Micro-Benchmarking
 - Syscall time: for fork(), exec(), open(), read(), close()
 - File system reads: from a cache file, varying the read size from 1 byte to 1 Mbyte
 - Network throughput: transferring data between TCP endpoints, for varying socket buffer sizes

#####Modeling
1. Enterprise versus Cloud
2. Visual Identification
 - Linear scalability
 - Contention
 - Coherence
 - Knee point
 - Scalability ceiling

 ![scalability profile]({{ IMAGE_PATH }}/scalability_profile.png)
3. Amdahl's Law of Scalability   C(N) = N/1 + a(N - 1)
 - Collect data of range of N, either by observation of an existing system or experimentally using micro-benchmarking or load generators
 - Perform regression analysis to determine the Amdahl parameter(a); this may be done using statistical software, such as gnuplot or R
 - Present the result for analysis.The collected data points can be plotted along with the model function to predict scaling and reveal differences between the data and the model. This may also be done using gnuplot or R
4. Universal Scalability Law  C(N)=N/1 + a(N - 1) + BN(N - 1)
5. Queueing Theory
 - Arrival process
 - Service time distribution
 - Number of server centers

#####Capacity Planning
1. Resource Limits
 - Measure the rate of server requests, and monitor this rate over time
 - Measure hardware and software resource usage. Monitor this rate over time
 - Express server requests in terms of resources used
 - Extrapolate server requests to known(or experimentally determined)limits for earch resource
2. Factor Analysis
 - Test performance with all factors configured to maximum
 - Change factors one by one, testing performance(it should drop for each)
 - Attribute a percentage performance drop to each factor, based on measurements, along with the cost savings
 - Starting with maximum performance(and cost),choose factors to save cost, while maintaining the required requests per second based on their combined performance drop
3. Scaling Solutions

#####Statistics
1. Quantifying Performance
 - Observation-Based
  - Choose a reliable metric
  - Estimate the performance gain from resolving the issue
 - Experimentation-Based
  - Apply the fix
  - Quantify before versus after using a reliable metric
2. Averages
 - Geometric Mean
 - Harmonic Mean
 - Averages over Time
 - Decayed Average
3. Standard Deviations,Percentiles,Median

 ![staticsis values]({{ IMAGE_PATH }}/staticsis_values.png)
4. Coefficient of Variation
5.Multimodal Distributions
6.Outliers

#####Monitoring
1. Time-Based Patterns
2. Monitoring Products
3. Summary-since-Boot

#####Visualizations
1. Line Chart

 ![Line Chart]({{ IMAGE_PATH }}/Line_Chart.png)
2. Scatter Plots
![Scatter Plots]({{ IMAGE_PATH }}/Scatter_Plots.png)
3. Heat Maps
![heat map]({{ IMAGE_PATH }}/heat_map.png)
4. Surface Plot
![Surface Plot]({{ IMAGE_PATH }}/Surface_Plot.png)
5. Visualization Tools
