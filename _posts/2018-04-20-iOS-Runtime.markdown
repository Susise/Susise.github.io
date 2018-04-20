---
layout: post
title: iOS - Runtime 的理解
date: 2018-04-20 17:41:00.000000000 +09:00
---

Runtime


1、什么是Runtime？

Runtime 又叫运行时，是一套底层的C语言API，是iOS内部的核心之一，OC代码的底层都是基于它来实现的，最终都是转化成了Runtime的C语言代码。其中最主要的就是消息机制

OC中一切都被设计成对象，我们都知道一个类被初始化成一个实例，这个实例是一个对象。实际上一个类本质上也是一个对象。


为什么说oc是面向对象的动态语言？

动态加载 例如2x 3x 图片

动态类型，比如说id类型，在运行时才能确定到底是什么类型

动态绑定，比如说，在编译时候并不能决定真正调用哪个函数，只有在真正运行的时候才能根据函数名称找到对应的函数来调用

2、Runtime的运行机制

OC代码在程序运行的过程中，最终都转换为runtime的C语言代码。

3、Runtime的常见方法？

发送消息

objc_msgSend 只有对象才能发送消息，调用方法的本质就是发送消息，objc_msgSend([Person class], @selector(eat)); 发送消息的本质是，对象根据方法编号SEL去映射表查找对应的方法实现

获取列表 - 属性列表 、方法列表、成员变量列表、协议列表  

unsigned int count;

class_copyPropertyList([self class], &count)   属性列表 

class_copyMethodList([self class], &count)  方法列表

class_copyIvarList([self class], &count)  成员变量列表

class_copyProtocolList([self class], &count)  协议列表

获取类方法、获取实例方法

class_getClassMethod([self class], @selector(testClassMethod))  获取类方法

class_getInstanceMethod([self class], @selector(testObjectMethod))  获取实例方法


替换方法实现

交换两个方法 — 当某个系统的方法不能满足需求的时候，在系统方法的需求上添加。例如，统计进入vc时间

  {% highlight ruby %}
  
Method me1 = class_getInstanceMethod([self class], @selector(meth1));
Method me2 = class_getInstanceMethod([self class], @selector(meth2));
method_exchangeImplementations(me1, me2);

 {% endhighlight %}
 

拦截调用、添加方法：再找不到调用方法，程序崩溃之前，有机会重写NSObject的四个方法来处理（其实拦截调用的恶原理就是动态添加方法）

  {% highlight ruby %}
  
class_addMethod([self class], NSSelectorFromString(@"aaa"), (IMP)aaa, "V@:")

 {% endhighlight %}

+ (BOOL )resolveInstanceMethod:(SEL)sel  //当你调用一个不存在的类方法的时候，会调用这个方法，默认返回NO，你可以加上自己的处理然后返回YES

+ (BOOL)resolveClassMethod:(SEL)sel

//后两个方法需要转发到其他的类处理
- (id)forwardingTargetForSelector:(SEL)aSelector //将你调用的不存在的方法重定向到一个其他声明了这个方法的类，只需要你返回一个有这个方法的target

- (void)forwardInvocation:(NSInvocation *)anInvocation //将你调用的不存在的方法打包成NSInvocation传给你。做完你自己的处理后，调用invokeWithTarget:方法让某个target触发这个方法

关联对象、动态添加属性 -  应用：给Category添加属性

  {% highlight ruby %}

static const char *key = "name";

- (void)setName:(NSString *)name{
objc_setAssociatedObject(self, key, name, OBJC_ASSOCIATION_COPY);
}
- (NSString *)name{
return objc_getAssociatedObject(self, key);
}

 {% endhighlight %}
 
实现NSCoding的自动归档和解档


如果有100个属性，就要写一百遍

  {% highlight ruby %}

#import "Movie.h" 

#import <objc/runtime.h>

@implementation Movie 

- (void)encodeWithCoder:(NSCoder *)encoder { 

unsigned int count = 0; 
Ivar *ivars = class_copyIvarList([Movie class], &count);
for (int i = 0; i<count; i++) { 
Ivar ivar = ivars[i]; 
const char *name = ivar_getName(ivar);
NSString *key = [NSString stringWithUTF8String:name]; 
id value = [self valueForKey:key];
[encoder encodeObject:value forKey:key]; 
} 
} 
- (id)initWithCoder:(NSCoder *)decoder {
if (self = [super init]) {
unsigned int count = 0; 
Ivar *ivars = class_copyIvarList([Movie class], &count); 
for (int i = 0; i<count; i++) { 
Ivar ivar = ivars[i];
const char *name = ivar_getName(ivar); 
NSString *key = [NSString stringWithUTF8String:name]; 
id value = [decoder decodeObjectForKey:key]; 
[self setValue:value forKey:key];    
}
} 
return self;
}
@end

 {% endhighlight %}

字典转模型

字典转模型的方式

（1） kvc

  {% highlight ruby %}
  
+ (instancetype)statusWithDict:(NSDictionary *)dict { 
Status *status = [[self alloc] init]; 
[status setValuesForKeysWithDictionary:dict];
return status;
}

 {% endhighlight %}
 
Kvc 转模型必须保证，模型中的属性和字典中的key 一一对应。如果不一致，就报 setvalue forundefinedKey 找不到key

解决办法，是重写对象的setValue：forUndefinedKey 把系统的方法覆盖，就可以了

（2）Runtime

利用运行时，遍历模型中的所有属性，根据属性的属性名，去查找字典中的key，取出对应的值，给模型的属性赋值。





4、Runtime的相关应用/具体实现/你在开发中用到的runtime

导入头文件 #import <objc/runtime.h>

MJExtention的底层实现 — 获取成员变量 - 赋值

给分类添加属性

在Category里面实现方法交换在系统方法的基础上加自定义功能- 方法交换 - 黑魔法

字典转模型

归档/解档

如有需要遍历成员变量（变量属性、方法、协议）

实现未定义的方法（这个不是很理解，没用过，没定义，会调用吗？）

5、Runtime的数据结构

相关定义

typedef struct objc_method *Method; 方法

typedef struct objc_ivar *Ivar; 成员变量

typedef struct objc_category *Category; 类别

typedef struct objc_property *objc_property_t ; 属性

类 

struct objc_class {
Class _Nonnull isa;   //实例对象的isa指向类，类的isa 指向元类

#if !__OBJC2__ // 如果不是 OC 2.0
Class _Nullable super_class    //指向父类
const char * _Nonnull name          //类名  
long version                         
long info                                            
long instance_size                                    
struct objc_ivar_list * _Nullable ivars          //成员变量列表         
struct objc_method_list **methodLists;       //方法列表
struct objc_cache * _Nonnull cache             //缓存
struct objc_protocol_list * _Nullable protocols      //协议列表  
#endif

} OBJC2_UNAVAILABLE;

修改methodlists 添加成员方法，是category实现的原理

6、黑魔法 Method Swizzling，什么情况下调用

简单的说就是方法交换，达到给系统的方法挂钩。

Method Swizzling指的是改变一个已存在的选择器对应的实现的过程，oc中方法的调用能够在运行时通过改变 - 通过改变类的调度表中选择器到最终函数间的映射关系

在OC中调用一个方法，其实是向一个对象发送消息，查找消息的唯一依据，
* 再没有一个类的实现源码的情况下，想改变其中一个方法的实现，可以灵活的使用 Method Swizzling


关于Runtime的面试题

1、Runtime 的方法调用是怎么实现的？

如果用实例对象调用实例方法，会到实例的isa指针指向的对象（类对象）操作，如果调用的是类方法，就会到类对象的isa指针指向的对象 （元类对象）中操作。

首先，在相应的操作的对象的缓存方法列表中找调用的方法，如果找到，转向相应的实现并执行

如果没有找到，在相应操作的对象中的方法列表中找调用的方法，如果找到，转向相应的实现并执行

如果没有找到，去父类指针所指向的对象中执行1、 2

以此类推，如果一直找到根类还没有找到，转向拦截调用

如果没有重写拦截调用的方法，程序报错。

感悟：重写父类的方法，并没有覆盖掉父类的方法，只是不再去父类找了； 如果想调用已经重写过的父类的方法的实现，只需要使用super 这个编译器标志符，就会在运行时跳过当前类对象中寻找方法的过程。

2 、Runtime 是什么？

Runtime 又叫运行时，是一套底层的C语言API，是iOS内部的核心之一。OC代码的底层都是基于它来实现的。

3、为什么需要Runtime

OC 是一门动态语言，会将一些工作放在代码运行时才处理，不是编译的时候，也就是说，很多类和成员变量在编译的时候 是不知道的，在运行时候，编写的代码才会转换成完整的确定的代码执行。所以，编译器是不够的，我们需要一个运行时系统（Runtime system），来处理编译后的代码。（Runtime是基于C和汇编写的，由此可见苹果为了动态系统的搞笑而做出的努力）

4、Runtime 有什么作用？（也可以从实际应用出发回答）

Oc在三种层面上与Runtime系统进行交互

* 通过Objective-C源代码，只需要编写oc代码，Runtime系统自动在幕后搞定一切，调用方法，编译器会将oc代码转换成运行时代码，在运行时确定数据结构和函数
* 通过Foundation框架的NSObject类定义的方法
* 通过对Runtime库函数的直接调用

5、Runtime 有哪些相关的术语？

SEL  是selector 在objc中的表示，selector 是方法选择器，作用和名字一样，日常生活中，我们通过人名辨别谁是谁，注意在相同类中不会有命名相同的同哥方法，selector 对方法名进行包装，以便找到对应的方法实现。

id 参数类型，指向某个类的实例的指针。 typedef struct objc_object *id;
struct objc_object { Class isa; };


Class  typedef struct objc_class *Class;

Class 其实是指向objc_class 结构体的指针，objc_class的数据结构如下

struct objc_class {

Class isa  OBJC_ISA_AVAILABILITY;

#if !__OBJC2__

Class super_class

OBJC2_UNAVAILABLE;

const char *name

OBJC2_UNAVAILABLE;

long version

OBJC2_UNAVAILABLE;

long info

OBJC2_UNAVAILABLE;

long instance_size

OBJC2_UNAVAILABLE;

struct objc_ivar_list *ivars

OBJC2_UNAVAILABLE;

struct objc_method_list **methodLists

OBJC2_UNAVAILABLE;

struct objc_cache *cache

OBJC2_UNAVAILABLE;

struct objc_protocol_list *protocols

OBJC2_UNAVAILABLE; 

#endif
 } OBJC2_UNAVAILABLE; 

从objc_class 可以看到，一个运行时类中 关联了他的父类指针，类名、成员变量、方法、缓存以及附属的协议。


IMP  IMP在objc.h 中的定义是：

typedef id (*IMP)(id, SEL, ...); 

他就是一个函数指针，由编译器生成的，当你发起一个objc消息之后，最终他会执行的那段代码，就是由这个函数指针制定的，而IMP 这个函数指针就是指向了这个方法的实现。

如果得到了执行某个实例、某个方法的入口，我们可以绕开消息传递阶段，直接执行方法。

IMP指向的方法与objc_msgSend 函数类型相同，参数都包括 id 和 SEL类型， 每个方法名都对应一个SEL 类型的方法选择器，而每个实例对象中的SEL对应的方法实现肯定是唯一的，通过一组id 和SEL 参数就能确定唯一的方法实现地址。

Cache 定义如下

typedef struct objc_cache *Cache

struct objc_cache {

unsigned int mask /* total = mask + 1 */

OBJC2_UNAVAILABLE;

unsigned int occupied

OBJC2_UNAVAILABLE;

Method buckets[1]

OBJC2_UNAVAILABLE;

};

Cache 为方法调用的性能进行优化，每当实例对象接收到一个消息时，他不会直接在isa 指针指向的类的方法列表中遍历找到能够响应的方法，因为每次都要查找，效率太低了，而是优先在Cache中查找

Runtime 系统会把被调用的方法存到Cache中，如果一个方法被调用，那么他有可能还会被调用，下次查找的时候就会效率更高，就像计算机原理中CPU 绕过主寸先访问Cache 一样

Property typedef struct objc_property *Property;

typedef struct objc_property *objc_property_t;//  这个更常用，可以通过

class_copyPropertyList  和 protocol_copyPropertyList  方法获取类 和协议中的属性。

objc_property_t *class_copyPropertyList(Class cls, unsigned int*outCount)

objc_property_t *protocol_copyPropertyList(Protocol *proto,unsigned int *outCount)

注意：返回的是属性列表，列表中每个元素都是一个 objc_property_t  指针， name--------T@"NSString",&,N,V_name

property_getName  用来查找属性名称，返回C字符串

property_getAttributes  函数挖掘属性 的真实名称 和 @encode 类型，返回C字符串 


6、Runtime 和消息？

messages  aren’t bound to method implementations until Runtime  消息知道运行时才会与方法实现进行绑定。 
这里要清楚一点，objc_msgSend 方法看起来好像是返回了数据，其实objc_msgSend 从不返回数据，而是你的方法在运行时被调用后才会返回数据，下面详细叙述消息发送的步骤

* 首先检测这个selector 是不是要忽略，比如Mac OS X 开发，有了垃圾回收就不会理会retain、release 这些函数

* 检测这个selector的tatget是不是nil，Objc 允许我们对一个nil对象执行任何方法不会Crash，因为运行时会被忽略掉

* 如果上面两步都通过了，那么就开始查找这个类的实现IMP，先从cache里查找，如果找到了就运行对应的函数去执行相应的代码

* 如果cache找不到，就找类的方法列表中查找是否有对应的方法

* 如果类的方法列表中找不到就到父类的方法列表中查找，一直找到NSObject 类为止

* 如果还找不到，就要开始进入动态方法解析了


在消息的传递中，编译器会根据情况在objc_msgSend \ objc_msgSend_stret \ objc_msgSendSuper \ objc_msgSendSuper_stret 这四个方法中选择一个调用，如果消息是传递给父类，那么会调用名字带有super的函数，如果消息返回值是数据结构而不是简单值时，会调用名字带有stret的函数


7、消息转发

- 重定向

消息转发机制执行前，Runtime 系统允许我们替换消息的接收者为其他对象。通过- (id)forwardingTargetForSelector:(SEL)aSelector  方法。 

  {% highlight ruby %}

- (id)forwardingTargetForSelector:(SEL)aSelector
{
if(aSelector == @selector(mysteriousMethod:)){
return alternateObject;
}
return [super forwardingTargetForSelector:aSelector];
}

 {% endhighlight %}

如果此方法返回nil 或者self。则会计入消息转发机制 (forwardInvocation:)  ，否则将向返回的对象重新发送消息

- 转发

当动态方法解析不做处理返回NO 时，则会触发消息转发机制。这时，forwardInvocation: 方法会被执行，我们可以重写这个方法来自定义我们的转发逻辑

  {% highlight ruby %}

- (void)forwardInvocation:(NSInvocation *)anInvocation
{
if ([someOtherObject respondsToSelector: [anInvocation selector]])
[anInvocation invokeWithTarget:someOtherObject];
else
[super forwardInvocation:anInvocation];
}

 {% endhighlight %}

唯一参数  是个NSInvocation 类型的对象，该对象封装了原始的消息和消息的参数，我们可以实现forwardInvocation: 方法来对不能处理的消息做一些处理。也可以将消息转发给其他对象处理，而不排除错误。

转发和继承相似，可用于为objc 编程添加一些多继承的效果。一个对象把消息转发出去，就好像它把另一个对象中的方法接过来或者“继承”过来一样。

8、转发与继承

虽然转发可以实现继承的功能，但是NSObject 还是必须表面上很严谨，像respondsToSelector ：和 isKindOfClass: 这类方法只会考虑继承体系，不会考虑转发链。

Warrior 对象被问到是否能响应 negotiate消息：

if ( [aWarrior respondsToSelector:@selector(negotiate)] ) 

回答当然是NO，尽管他能接受 negotiate消息而不报错，因为它靠转发消息给Diploat 类响应消息。

如果你就是想要别人以为Warrior 继承到了 Diplomat的 negotiate 方法，你得重新实现 respondsToSelector：和 isKindOfClass：来加入你的转发算法
 
  {% highlight ruby %}
  
- (BOOL)respondsToSelector:(SEL)aSelector
if ( [super respondsToSelector:aSelector] )
return YES;
else {
/* Here, test whether the aSelector message can     *
* be forwarded to another object and whether that  *
* object can respond to it. Return YES if it can.  */
}
return NO; } 

 {% endhighlight %}

除了 respondsToSelector:  和  isKindOfClass:  之外，instancesRespondToSelector:  中也应该写一份转发算法，如果使用了协议 conformsToProtocol:  同样也要加入到这一行列中。如果一个对象想要转发他接受的任何远程消息，他得给出一个方法标签来返回准确的方法描述，methodSignatureForSelector：这个方法会最终响应被转发的消息，从而生成一个确定的NSInvocation 对象 描述消息和消息参数，这个方法最中响应被转发的消息。他要像下面这样实现

  {% highlight ruby %}
  
- (NSMethodSignature*)methodSignatureForSelector:(SEL)selector
{
NSMethodSignature* signature = [super
methodSignatureForSelector:selector];
if (!signature) {
signature = [surrogate
methodSignatureForSelector:selector];
}
return signature;
}

 {% endhighlight %}


9、健壮的实例变量

在Runtime的现行版本中，最大的特点就是健壮的实例变量了。

我们让自己的类继承自 NSObject 不仅仅是因为基类 有很多复杂的内存分类问题，更是因为这使得我们可以享受到Runtime系统带来的便利。虽然平时我们平时很少会考虑一句简单的调用方法，发送消息底层所做的复杂的操作，但深入理解Runtime 系统的细节使得我们可以利用消息机制写出功能更强大的代码。

10、runtime实现的机制是什么，怎么用，一般用于干什么，你还能记得你所使用的相关的头文件或者某些方法名称吗？

* 需要导入     <objc/message.h><objc/runtime.h> 

* Runtime 运行时机制，是一套C语言库

* 实际上我们编写的所有OC代码，最中都是转成了runtime 库的东西，比如类转成了 runtime 库里面的结构体等数据类型，方法转成了runtime 库里面的c语言函数，平时调用方法都是转成了 objc_msgSend函数，（所以说OC有个消息发送机制）

* 因此，runtime是OC的底层实现，是OC的幕后执行者

* 有了runtime库，能做什么事情呢？runtime库里面包含了类、成员变量、方法相关的API，比如获取类里面的所有成员变量，为类动态添加成员变量，动态改变类的方法实现，为类动态添加新方法等。

11、OC如何对已有的方法，添加自己的功能代码以实现类似的记录日志这样的功能？

Category - 方法交换 

这里主要考察方法交换，现在分类中添加一个方法，注意不能重写系统方法，会覆盖

12、如何让Category 支持属性？

关联属性

  {% highlight ruby %}
  
@interface NSObject (test)
@property (nonatomic, copy) NSString *name;
@end 
.m 
@implementation NSObject (test)
// key 
static const char *key = "name";
-(NSString *)name
{ // key  根据关联的key，获取关联的值
return objc_getAssociatedObject(self, key);
}
-(void)setName:(NSString *)name
{
// 第一个参数 ：给哪个对象添加关联
// 第二个参数 ：关联的key，通过这个key 获取
// 第三个参数 ：关联的value
// 第四个参数 ：关联的策略
objc_setAssociatedObject(self, key, name,
OBJC_ASSOCIATION_RETAIN_NONATOMIC);
}

 {% endhighlight %}
 
13、Toll-Free Bridging 是什么，什么情况下会使用？

Toll-Free Bridging 用于在 Foundation  对象与 Core Foundation 对象之间交换数据，俗称桥接。在ARC环境下，Foundation对象转成Core Foundation对象，使用__bridge 桥接以后ARC 会自动管理2个对象。使用__bridge __retained桥接需要手动释放 Core Foundation对象。在ARC 环境下，Core Foundation对象转成了Foundation对象。使用__bridge 桥接，如果Core Fouundation 对象被释放，Foundation对象同时也不能使用了，需要手动管理Core Foundation对象。使用__bridge_transfer桥接，系统会自动管理2个对象。


14、performSelector ：withObject ：faterDelay 内容大概是怎么实现的，有什么注意事项？

创建一个定时器，时间结束后，系统会使用runtime通过方法名称（Selector 本质就是方法名称）去方法列表中找到对应的方法并调用方法

注意事项：调用 performSelector ：withObject ：faterDelay  方法时，先判断希望调用的方法是否存在respondsToSelector 
这个方法是异步方法，必须在主线程调用，在子线程调用永远不会调用到想调用的方法

15 、能否向编译后得到的类中增加实例变量？能否向运行时创建的类中添加实例变量？为什么？

不能向编译后得到的类中添加实例变量

能像运行时创建的类中添加实例变量

因为编译后的类已经注册在runtime中，类结构体 objc_ivar_list  实例变量的链表 和 instance_size  实例变量的内存大小已经确定，同时runtime 会调用 class_setIvarLayout 或 class_setWeakIvarLayout 来处理 strong weak 饮用，所以不能向存在的类中添加实例变量。

运行时创建的类可以添加实例变量，调用class_addIvar 函数，但是得到调用 objc_allocateClassPair  之后，objc_registerClassPair 之前，原因同上。

16、为什么其他语言里叫函数调用，OC 里则是给对象发消息（或者谈下runtime的理解）

17、runtime 如何实现weak属性？

通过关联属性来实现

声明一个weak 属性，这里假设delegate 其实weak 关键字可以不使用，因为我们重写了getter/setter 方法，

  {% highlight ruby %}
  
@property (nonatomic, weak) id delegate; 

- (id)delegate {

return objc_getAssociatedObject(self, @"__delegate__key"); 
}

 {% endhighlight %}
 
指定使用 OBJC_ASSOCIATION_ASSIGN 官方注释 Specifies a weak reference to the associated object 

也就是说 对于对象类型，就是weak 了

  {% highlight ruby %}
  
- (void)setDelegate:(id)delegate {
objc_setAssociatedObject(self, @"__delegate__key", delegate, OBJC_ASSOCIATION_ASSIGN);
}

 {% endhighlight %}

通过 objc_storeWeak 函数来实现，不过这种方式几乎没有遇到过有人这么实用，所以不细说。 

18、runtime 如何通过selector 找到对应的IMP地址？

每个selector 都与对应的IMP是一一对应的关系，通过selector 就可以直接找到对应的IMP；

19、objc_msgForward 函数是做什么的，直接调用他将会发生什么？

objc_msgForward 是IMP 类型，用于消息转发的，当向一个对象发送一条消息，但是它并没有实现的时候objc_msgForward 会尝试做消息转发， IMP msgForward = _objc_msgForward; 

如果手动调用  objc_msgForward 将跳过查找 IMP 的过程，而是直接触发 “消息转发”，进入如下流程

第一步 ：+ (BOOL)resolveInstanceMethod:(SEL)sel  实现方法，指定是否动态添加方法，如果返回 NO 则进行下一步，若返回 YES ，则通过class_addMethod 函数动态的添加方法，消息得到处理，此流程完毕

第二步：在第一步返回的是 NO 时候，就会进入(id)forwardingTargetForSelector:(SEL)aSelector  方法，这是运行时给我们的第二次机会，用于制定哪个对象响应这个slector ，不能指定为self，若返回nil，表示没有响应者，则进入第三步，若返回某个对象，则会调用该对象的方法。

第三步，若第二步返回的是nil，则我们首先是通过 - (NSMethodSignature *)methodSignatureForSelector:(SEL)aSelector  指定方法签名，若返回nil 则表示不处理，若返回方法签名，则会进入下一步

第四步：当第三步返回方法方法签名后，就会调用 (void)forwardInvocation:(NSInvocation *)anInvocation 方法，我们可以通过 anInvocation  对象做很多处理，比如修改实现方法，修改响应对象等。

第五步，如果没有实现 - (void)forwardInvocation:(NSInvocation *)anInvocation  方法，那么会进入 -(void)doesNotRecognizeSelector:(SEL)aSelector  方法，如果我们没有实现这个方法，那么就会crash 然后提示找不到相应的方法，到此，动态解析的流程就结束了。


20、runtime 如何实现weak变量的自动置nil？

Runtime 对注册的类 会进行布局，对于weak 对象 会放入一个hash 表中，用weak 指向的对象内存地址作为key，档次对象的引用计数为0的时候就会dealloc。加入weak指向的对象 内存地址是a，那么就会以a为键，在这个weak 表中搜索，找到所有以a为键的weak 对象，从而设置为 nil。 weak 修饰的指针默认值是nil ，在objective - c中向nil 发送消息是安全的。

21、动态绑定？

在运行时确定要调用的方法，动态绑定将调用方法的确定也推迟到了运行时。在编译时，方法的调用并不和代码绑定在一起，只有在消息发送出来以后，才确定被调用的代码。通过 动态类型 和动态绑定技术，代码每次执行都可以得到不同的结果。运行时因子负责确定消息的接收者 和被调用的方法，运行时的消息分发机制为 动态绑定提供支持。当向一个动态类型确定了对象发送消息时候，运行环境系统会通过接收着的isa 指针定位对象的类，并以此为起点确定被调用的方法，方法和消息是动态绑定的。而且，不必在objective - c代码中做任何工作，就可以自动获取动态绑定的好处，在每次发送消息的时候，特别是当消息的接收者 是动态类型已经确定的对象时，动态绑定就会例行而透明的发生。

