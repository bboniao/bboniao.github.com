---
layout: post
title: "JVM致命错误日志（hs_err_pid.log）解读"
description: ""
category: "jvm"
tags: hs err pid
---
{% include JB/setup %}

###淘宝的一个[ppt](https://github.com/bboniao/bboniao.github.com/releases/download/javacrash/javacrash-120504004849-phpapp02.pptx)

###[转自《四火的唠叨》](http://www.raychase.net/1459)

#####致命错误出现的时候，JVM生成了hs_err_pid<pid>.log这样的文件，其中往往包含了虚拟机崩溃原因的重要信息。因为经常遇到，在这篇文章里，我挑选了一个，并且逐段分析它包含的内容（文件可以在文章最后下载）。默认情况下文件是创建在工作目录下的（如果没权限创建的话JVM会尝试把文件写到/tmp这样的临时目录下面去），当然，文件格式和路径也可以通过参数指定，比如：
    java -XX:ErrorFile=/var/log/java/java_error%p.log

<!-- more -->

#####这个文件将包括：
    触发致命错误的操作异常或者信号；
    版本和配置信息；
    触发致命异常的线程详细信息和线程栈；
    当前运行的线程列表和它们的状态；
    堆的总括信息；
    加载的本地库；
    命令行参数；
    环境变量；
    操作系统CPU的详细信息。
#####首先，看到的是对问题的概要介绍：
    # SIGSEGV (0xb) at pc=0x03568cf4, pid=16819, tid=3073346448
#####一个非预期的错误被JRE检测到，其中：
    SIGSEGV是信号名称
    0xb是信号码
    pc=0x03568cf4指的是程序计数器的值
    pid=16819是进程号
    tid=3073346448是线程号
#####如果你对JVM有了解，应该不会对这些东西陌生。
#####接下来是JRE和JVM的版本信息：
    # JRE version: 6.0_32-b05
    # Java VM: Java HotSpot(TM) Server VM (20.7-b02 mixed mode linux-x86 )
#####运行在mixed模式下。
#####然后是问题帧的信息：
    # Problematic frame:
 
    # C  [libgtk-x11-2.0.so.0+0x19fcf4]  __float128+0x19fcf4
#####C：帧类型为本地帧，帧的类型包括
    C：本地C帧
    j：解释的Java帧
    V：虚拟机帧
    v：虚拟机生成的存根栈帧
    J：其他帧类型，包括编译后的Java帧
#####libgtk-x11-2.0.so.0+0x19fcf4：和程序计数器（pc）表达的含义一样，但是用的是本地so库+偏移量的方式。
#####接下去第一部分是线程信息：
    Current thread (0x09f30c00):  JavaThread "main" [_thread_in_native, id=16822, stack(0xb72a8000,0xb72f9000)]
#####当前线程的：
    0x09f30c00：指针
    JavaThread：线程类型，可能的类型包括：
      JavaThread
      VMThread
      CompilerThread
      GCTaskThread
      WatcherThread
      ConcurrentMarkSweepThread
    main：名字
      _thread_in_native：线程当前状态，状态枚举包括：
      _thread_uninitialized：线程还没有创建，它只在内存原因崩溃的时候才出现
      _thread_new：线程已经被创建，但是还没有启动
      _thread_in_native：线程正在执行本地代码，一般这种情况很可能是本地代码有问题
      _thread_in_vm：线程正在执行虚拟机代码
      _thread_in_Java：线程正在执行解释或者编译后的Java代码
      _thread_blocked：线程处于阻塞状态
      …_trans：以_trans结尾，线程正处于要切换到其它状态的中间状态
    id=16822：线程ID
    0xb72a8000,0xb72f9000：栈区间 

    siginfo:si_signo=SIGSEGV: si_errno=0, si_code=1 (SEGV_MAPERR), si_addr=0x00000010
#####这部分是导致虚拟机终止的非预期的信号信息，含义前面已经大致提到过了。其中si_errno和si_code是Linux下用来鉴别异常的，Windows下是一个ExceptionCode。
    EAX=0x00000000, EBX=0x0375dd84, ECX=0x00000000, EDX=0x00000000
    ESP=0xb72f0fa0, EBP=0xb72f0fb8, ESI=0x00000000, EDI=0x0a6c1800
    EIP=0x03568cf4, EFLAGS=0x00010246, CR2=0x00000010

#####这是寄存器上下文。
    Top of Stack: (sp=0xb72f0fa0)
    0xb72f0fa0:   00000000 00402250 0040217f 0375dd84
    0xb72f0fb0:   00000000 0a6c1800 b72f0fe8 0356c2c0
    0xb72f0fc0:   00000000 0a6c1800 b72f0fe8 003b3e77
    0xb72f0fd0:   003e6c8b 0a1a70d0 0a193358 0375dd84
    0xb72f0fe0:   0a276418 0a276418 b72f1048 03536c56
    0xb72f0ff0:   0acad000 0b3ca978 0000000c 00dd0674
    0xb72f1000:   00000003 0a2c7d50 b72f1038 0000330c
    0xb72f1010:   ffffffff ffffffff 00000001 00000001
     
    Instructions: (pc=0x03568cf4)
    0x03568cd4:   89 14 24 89 75 f8 89 d6 89 7d fc 89 c7 e8 7e 1b
    0x03568ce4:   ea ff 89 34 24 89 87 d4 02 00 00 e8 30 00 ea ff
    0x03568cf4:   8b 40 10 89 3c 24 c7 44 24 08 00 00 00 00 89 87
    0x03568d04:   d0 02 00 00 8b 83 88 24 00 00 89 44 24 04 e8 dd

#####栈顶程序计数器旁的操作码，它们可以被反汇编成系统崩溃前执行的指令。

    Register to memory mapping:
 
    EAX=0x00000000 is an unknown value
    EBX=0x0375dd84: <offset 0x394d84> in /usr/lib/libgtk-x11-2.0.so.0 at 0x033c9000
    ECX=0x00000000 is an unknown value
    EDX=0x00000000 is an unknown value
    ESP=0xb72f0fa0 is pointing into the stack for thread: 0x09f30c00
    EBP=0xb72f0fb8 is pointing into the stack for thread: 0x09f30c00
    ESI=0x00000000 is an unknown value
    EDI=0x0a6c1800 is an unknown value

#####寄存器和内存映射信息。
    Stack: [0xb72a8000,0xb72f9000],  sp=0xb72f0fa0,  free space=291k
    Native frames: (J=compiled Java code, j=interpreted, Vv=VM code, C=native code)
    C  [libgtk-x11-2.0.so.0+0x19fcf4]  __float128+0x19fcf4
    C  [libgtk-x11-2.0.so.0+0x1a32c0]  __float128+0xc0
    ... ...
    C  [libswt-pi-gtk-3738.so+0x33f6a]  Java_org_eclipse_swt_internal_gtk_OS__1Call+0xf
    J  org.eclipse.swt.internal.gtk.OS._Call(III)I
    J  org.eclipse.swt.internal.gtk.OS.Call(III)I
     
    Java frames: (J=compiled Java code, j=interpreted, Vv=VM code)
    J  org.eclipse.swt.internal.gtk.OS._Call(III)I
    J  org.eclipse.swt.internal.gtk.OS.Call(III)I
    j  org.eclipse.swt.widgets.Widget.fixedSizeAllocateProc(II)I+5
    j  org.eclipse.swt.widgets.Display.fixedSizeAllocateProc(II)I+17
    v  ~StubRoutines::call_stub
    ... ...

#####线程栈。包含了地址、栈顶、栈计数器和线程尚未使用的栈信息，由于栈可能非常长，打印的长度有限制，但是至少本地栈和Java栈都打印出来了（很多时候本地栈打印不出来，但是Java栈一般都能打印出来）。从中可以看到，Eclipse的虚拟机崩溃了。
    Java Threads: ( => current thread )
      0x0b4c1000 JavaThread "Worker-247" [_thread_blocked, id=25417, stack(0x741bc000,0x7420d000)]
      0x0a300c00 JavaThread "Worker-246" [_thread_blocked, id=25235, stack(0x7d30c000,0x7d35d000)]
    ... ...

#####线程信息。一目了然，不解释了。
    VM state:not at safepoint (normal execution)

#####虚拟机状态。包括：
    not at a safepoint：正常运行状态；
    at safepoint：所有线程都因为虚拟机等待状态而阻塞，等待一个虚拟机操作完成；
    synchronizing：一个特殊的虚拟机操作，要求虚拟机内的其它线程保持等待状态。
#####
    VM Mutex/Monitor currently owned by a thread: None
#####虚拟机的Mutex和Monitor目前没有被线程持有。Mutex是虚拟机内部的锁，而Monitor则关联到了Java对象。

    Heap
     PSYoungGen      total 149056K, used 125317K [0xa9700000, 0xb41a0000, 0xb41a0000)
      eden space 123520K, 95% used [0xa9700000,0xb0ac0de0,0xb0fa0000)
      from space 25536K, 26% used [0xb28b0000,0xb2f50748,0xb41a0000)
      to   space 25600K, 0% used [0xb0fa0000,0xb0fa0000,0xb28a0000)
     PSOldGen        total 261248K, used 239964K [0x941a0000, 0xa40c0000, 0xa9700000)
      object space 261248K, 91% used [0x941a0000,0xa2bf7018,0xa40c0000)
     PSPermGen       total 163328K, used 130819K [0x841a0000, 0x8e120000, 0x941a0000)
      object space 163328K, 80% used [0x841a0000,0x8c160c40,0x8e120000)
#####堆信息。新生代、老生代、永久代。对JVM有了解的人应该都清楚，不解释了。

    Code Cache  [0xb4262000, 0xb5ac2000, 0xb7262000)
     total_blobs=5795 nmethods=5534 adapters=209 free_code_cache=25103616 largest_free_block=38336
#####代码缓存（Code Cache）。这是一块用于编译和保存本地代码的内存，注意是本地代码，它和PermGen（永久代）是不一样的，永久带是用来存放Java类定义的。

    Dynamic libraries:
    00101000-00122000 r-xp 00000000 08:01 3483560    /usr/lib/libjpeg.so.62.0.0
    00122000-00123000 rwxp 00020000 08:01 3483560    /usr/lib/libjpeg.so.62.0.0
    00125000-00130000 r-xp 00000000 08:01 9093202    /lib/libgcc_s-4.1.2-20080825.so.1
    00130000-00131000 rwxp 0000a000 08:01 9093202    /lib/libgcc_s-4.1.2-20080825.so.1
    ... ...
#####内存映射。这些信息是虚拟机崩溃时的虚拟内存列表区域。在定位崩溃原因的时候，它可以告诉你哪些类库正在被使用，位置在哪里，还有堆栈和守护页信息。就以列表中第一条为例说明：

    00101000-00122000：内存区域
    r-xp：权限，r/w/x/p/s分别表示读/写/执行/私有/共享
    00000000：文件内的偏移量
    08:01：文件位置的majorID和minorID
    3483560：索引节点号
    /usr/lib/libjpeg.so.62.0.0：文件位置
#####每一个lib都有两块虚拟内存区域——代码和数据，它们的权限不同，代码区域是r-xp；数据区域是rwxp。守护页（guard page）由权限为--xp和rwxp的一对组成。

    VM Arguments:
    jvm_args: -Dosgi.requiredJavaVersion=1.5 -XX:MaxPermSize=256m -Xms40m -Xmx512m -Dorg.eclipse.swt.browser.XULRunnerPath=''
    java_command: /.../eclipse/plugins/org.eclipse.equinox.launcher_1.2.0.v20110502.jar -os linux -ws gtk -arch x86 -showsplash -launcher /.../eclipse/eclipse -name Eclipse ...
    Launcher Type: SUN_STANDARD
     
    Environment Variables:
    PATH=...
    DISPLAY=:0.0
#####虚拟机参数和环境变量。

    Signal Handlers:
    SIGSEGV: [libjvm.so+0x726440], sa_mask[0]=0x7ffbfeff, sa_flags=0x10000004
    SIGBUS: [libjvm.so+0x726440], sa_mask[0]=0x7ffbfeff, sa_flags=0x10000004
    ... ...
#####信号句柄。对于Linux下的信号机制，参阅wiki百科，[链接](http://en.wikipedia.org/wiki/C_signal_handling)。

    OS:Red Hat Enterprise Linux Client release 5.4 (Tikanga)
     
    uname:Linux 2.6.18-164.el5 #1 SMP Tue Aug 18 15:51:54 EDT 2009 i686
    libc:glibc 2.5 NPTL 2.5 
    rlimit: STACK 10240k, CORE 0k, NPROC 65536, NOFILE 1024, AS infinity
    load average:1.78 1.58 1.54
     
    /proc/meminfo:
    ...
     
    CPU:total 4 (4 cores per cpu, 1 threads per core) family 6 model 42 stepping 7, cmov, cx8, fxsr, mmx, sse, sse2, sse3, ssse3
     
    /proc/cpuinfo:
    ...
     
    Memory: 4k page, physical 3631860k(155144k free), swap 5124724k(5056452k free)
#####系统信息。

###[文中使用的hs_err_pid文件在此下载](http://www.raychase.net/wp-content/uploads/2013/06/hs_err_pid16819.rar)
