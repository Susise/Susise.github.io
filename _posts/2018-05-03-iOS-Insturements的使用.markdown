---
layout: post
title: iOS-Insturements 的使用
date: 2018-05-03 14:32:00.000000000 +09:00
---

Analyze 静态分析


它让你可以跟踪一个或多个进程

应用程序性能分析。在开始进行应用程序性能分析的时候,一定要使用真机,模拟器运行在Mac上，然而Mac上的CPU往往比iOS设备要快

另外在开始性能分析前另外一件重要的事情是，应用程序运行一定要发布配置 而不是调试配置. Release。

[链接1]、[链接2]、[链接3] Leaks

Undefined symbols for architecture i386:  "_OBJC_CLASS_$_JPUSHService", referenced from:  objc-class-ref in AppDelegate.o 

意思是无法找到 名为  JPUSHService 的类，进而还会导致一个无法连接的一个报错，没有这个类，添加这个类

选择项目文件 ，选择 Build Phases，选择 Compile Sources，添加 XXX.m 类（切记，添加的是.m文件）

第二种，还有一些 第三方库不在支持 “从3.0.0版本开始不再支持i386模拟器”

2.Time Profiler

选择Time Profiler启动.

分析代码的执行时间，可以找出程序变慢的原因

time profile时间分析工具用来检测应用CPU的使用情况.可以看到应用程序中各个方法正在消耗CPU时间.使用大量CPU不一定是个问题.类似我们客户端中不同场景的天气动画[类似大雨]的路径就对CPU依赖就非常高，动画本身也是非常苛刻且耗费资源较多的任务.

3.Leaks

监控内存泄漏

一般的措施内存使用情况，检查泄漏的内存，并提供了所有活动的分配和泄漏模块的类对象分配统计信息以及内存地址历史记录。用大白话来说，就是调查程序内存泄露的具体内容，比如泄露的内存地址、哪段代码内存泄露，泄露的内存大小......

![图片](/assets/images/iOS/Insturements1.png)
  
![图片](/assets/images/iOS/Insturements2.png)

![图片](/assets/images/iOS/Insturements3.png)

![图片](/assets/images/iOS/Insturements4.png)

在下面call Tree 里面选择。反向调用树、按线程做分析、隐藏系统库文件，过滤各种系统库调用
4.Allocations

监测内存分布情况


在 Xcode 中, 共提供了两种工具帮助查找泄漏点


1 > Analyze command + shift + B

- 学 名:  静态分析工具- 查 找:  可以通过 Product ->Analyze 菜单项启动- 快捷键:  CMD+shift +b.- Analyze主要分析以下四种问题:
1) 逻辑错误：访问空指针或未初始化的变量等；
2) 内存管理错误：如内存泄漏等；
3) 声明错误：从未使用过的变量；
4) Api调用错误：未包含使用的库和框架

2 >Instruments
- 学 名: 动态分析工具- 查 找:   Product ->Profile 菜单项启动- 快捷键:  CMD + i.- 简 介:它有很多跟踪模块可以动态分析和跟踪内存, CPU 和文件系统.

[链接4]

内存溢出

指程序在申请内存时候，没有足够的内存空间供其使用，出现 out of memory，比如申请了 一个integer 但是他存了 long 才能存下的数，这就是内存溢出

在xcode 里面也可以查看内存使用情况

![图片](/assets/images/iOS/Insturements5.png)

内存空间的划分：

一个进程占用的内存空间 包括不同的5种数据区：

1、BSS段，通常是存放未初始化的全局变量

2、数据段 通常是存放已初始化的全局变量

3、代码段 通常是存放程序执行代码

4、堆  通常是用于存放进程运行中 被动态分配的内存段 OC对象 （所有继承自NSObject的对象 ）就存放在堆里

5、栈 由编译器自动分配释放，存放函数的参数值

4.Core Animation FPS

在性能优化中一个最具参考价值的属性是FPS。Frames Per second，其实就是屏幕刷新率，苹果的iPhone推荐的刷新率是 60 Hz。也就是说GPU每秒钟刷新屏幕 60次，这每刷新一次就是一帧的frame，FPS 也就是每秒钟刷新多少帧画面。静止不变的页面FPS 值是 0，这个值没有参考意义，当地狱45的时候卡顿会比较明显 

注意点，

测试使用真机调试

最好使用release包测试，release 是发布版本，苹果会在release 包中做很多优化工作，因此用release包测试出来的性能才是最真实的。

![图片](/assets/images/iOS/Insturements6.png)

关于Core  Animation FPS 一个不错的 [博客]

5、Activity Monitor 
个人觉的很像Windows的任务管理器，可以查看所有的进程，以及进程的内存、cpu使用百分比等数据等，就不多介绍了，打开一看大概就知道怎么回事


UIScreen.main.scale。设备的自然分辨率

scale = 1; 的时候是代表当前设备是320*480的分辨率（就是iphone4之前的设备）
scale = 2; 的时候是代表分辨率为640*960的分辨率

[判断屏幕的分辨率]

![图片](/assets/images/iOS/Insturements7.png)

其实，[[UIScreen mainScreen] scale]是计算屏幕分辨率的喔。
[[UIScreen main] scale] == 1; //代表320 x 480 的分辨率
[[UIScreen main] scale] == 2; //代表640 x 960 的分辨率
[[UIScreen main] scale] == 3; //代表1242 x 2208 的分辨率

6、 真机调试

Color Blended Layer  （混合过度绘制）

Color Copied Images (拷贝的图片)

Color Misaligned Images (图片对齐方式)

Color Offscreen - Rendered

![图片](/assets/images/iOS/Insturements8.png)


[链接1]:https://www.jianshu.com/p/f87455e47da1

[链接2]:http://www.cocoachina.com/ios/20150225/11163.html

[链接3]:http://www.cocoachina.com/special/20161013/17759.html

[链接4]:http://www.cocoachina.com/special/20161013/17759.html 

[博客]:https://www.jianshu.com/p/a93375df8547

[判断屏幕的分辨率]:https://blog.csdn.net/u010545480/article/details/48707411?locationNum=4&fps=1


