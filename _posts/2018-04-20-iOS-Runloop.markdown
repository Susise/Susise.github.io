---
layout: post
title: iOS - Runloop
date: 2018-04-20 18:04:00.000000000 +09:00
---



RunLoop

RunLoop和线程紧密相连。`runloop` 是为了线程而生，没有线程，就没有存在的必要。每个线程，都有与之对应的 `runloop` 对象。主线程的 `runloop` 默认是开启的。对于其他线程来说，`runloop` 默认是不启动的。如果需要更多的线程交互则可以手动配置和启动。如果线程知识去执行一个长时间的已确定的任务则不需要。

{% highlight ruby %}

int main(int argc,char *argv[]){

@autoreleasepool{

return UIApplicationMain(argc, argv,nil,NSStringFromClass([appDelegateclass]));

}

}

 {% endhighlight %}


`UIApplicationMain()` 函数，这个方法会为 `main thread` 设置一个 `NSRunLoop` 对象。应用无人操作的时候休息，让他干活的时候能立马响应。


iOS中有两个API来访问和使用 `RunLoop`

Foundation框架中的, `NSRunLoop`

CoreFoundtion 中的 `CFRunLoopRef`

一、Runloop 与线程

1、每条线程都有唯一的一个与之对应的 `Runloop` 对象

2、主线程的 `Runloop` 已经自动创建好了，子线程的 `RunLoop` 需要主动创建

3、`RunLoop` 在第一次获取时创建，在线程结束时销毁

4、获取 `RunLoop` 对象

{% highlight ruby %}

Foundation  [NSRunLoop currentRunLoop]; 获取当前线程的runloop

[NSRunLoop mainRunLoop]；获得主线程的Runloop

CoreFoundation  CFRunLoopGetCurrent(); // 获得当前线程的RunLoop对象
 
CFRunLoopGetMain(); // 获得主线程的RunLoop对象  

如果给子线程添加runloop，不能直接alloc init，只要调用currentrunloop ，系统就会自动创建一个 Runloop 添加到当前线程中。

[NSRunLoop currentRunLoop]; // 这个方法是懒加载

 {% endhighlight %}

二、RunLoop 相关类

CoreFoundation 中关于 `RunLoop` 的5个类

{% highlight ruby %}

CFRunLoopRef

CFRunLoopModeRef

CFRunLoopSourceRef

CFRunLoopTimerRef

CFRunLoopObserverRef

 {% endhighlight %}

1、CFRunLoopModeRef 

CFRunLoopModeRef 表示RunLoop 的运行模式，一个 RunLoop 包含若干个 Mode，每个 Mode 又包含若干个 Source/Timer/Observer。每次 RunLoop 启动时，只能指定其中一个 Mode，这个 Mode 被称作 CurrentMode。

如果需要切换 Mode，只能退出 Loop，在重新指定 Mode 进入，这样做主要为了分隔开不同组的 Source/Timer/Observer 让其互不影响

系统默认注册了 5个Mode。

{% highlight ruby %}

KCFRunLoopDefaultMode App的默认Mode，通常主线程就是在这个Mode下运行的

UITrackingRunLoopMode 界面跟踪 Mode，用于ScrollView追踪触摸滑动，保证界面滑动时不受其他Mode影响

UIInitializationRunLoopMode 在刚启动App 时 进入的第一个 Mode，启动完成后就不再使用

GSEventReceiveRunLoopMode 接受系统事件的内部Mode，通常用不到，

KCFRunLoopCommonModes 这是一个占位用的 Mode，不是一种真正的Mode

 {% endhighlight %}

详细见代码：四种倒计时方式  [RunLoop] 


2、CFRunLoopSourceRef  

CFRunLoopSourceRef 是事件源（输入源），以前的分法 Port-BasedSources、Custom InputSources、Cocoa PerformSelector Sources。现在分法：Source0：非基于Port的，用于用户主动触发的事件、Source1：基于Port的，通过内核和其它线程相互发送消息

3、CFRunLoopTimerRef

CFRunLoopTimerRef 是基于时间的触发器，基本上说的就是NSTimer，它会受runloop的mode影响。GCD 定时器不受Runloop 的mode的影响。

4、CFRunLoopObserverRef

CFRunLoopObserverRef 是观察者，能狗监听RunLoop 的状态改变。可以监听的时间点有以下几个 

![图片](/assets/images/iOS/runloop1.png)

可以监测Timer

三、runloop的使用 详细见代码 [RunLoop]

1、NSTimer 

2、ImageView显示

3、PerformSelector

4、常驻线程

5、自动释放池

四 RunLoop面试题

1、什么是RunLoop？

1)从字面意思看：运行循环、跑圈

2)其实它内部就是do-while循环，在这个循环内部不断地处理各种任务（比如Source、Timer、Observer

3)一个线程对应一个RunLoop，主线程的RunLoop默认已经启动，子线程的RunLoop得手动启动（调用run方法)

4)RunLoop只能选择一个Mode启动，如果当前Mode中没有任何Source(Sources0、Sources1)、Timer，那么就直接退出RunLoop

2、自动释放池什么时候释放

通过Observer监听RunLoop的状态

3、在开发中如何使用RunLoop，什么应用场景

开启一个常驻线程（让一个子线程不进入消亡状态，等待其他线程发来消息，处理其他事件）。在子线程中开启一个定时器，在子线程中进行一些长期监控

可以控制定时器在特定模式下执行。可以让某些事件（行为、任务）在特定模式下执行。可以添加Observer监听RunLoop的状态，比如监听点击事件的处理（在所有点击事件之前做一些事情）

Core Foundation框架 和Foundation框架 

Core Foundation框架 是一组C语言接口，为iOS应用程序提供基本数据管理和服务功能。下面是该框架支持进行管理的数据以及可提供的服务

群体数据类型 (数组、集合等)

程序包

字符串管理

日期和时间管理

原始数据块管理

偏好管理

URL及数据流操作

线程和RunLoop

端口和soket通讯

Foundation 框架 提供Objective-C接口，如果要把Foundation 对象和 Core Foundation 类型参杂使用，则可利用两个框架之间的 “toll-free bridging”，所谓的Toll-free bridging 是说您可以在某个框架的方法或函数同时使用Core Foundation和Foundation 框架中的某些类型，很多数据支持这一特性。其中包括群体和字符串数据类型。每个框架的类和类型描述都会对某个对象是否为toll-free bridged 应和什么对象桥接进行说明。

    

[RunLoop]:https://github.com/Susise/RunLoop
    


