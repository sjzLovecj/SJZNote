# iOS | 面试知识整理 - OC底层 (三)

#### 1. 一个OC对象占用多少内存

- 系统分配了16个字节给NSObject对象（通过`malloc_size`函数获得）
- 但NSObject对象内部只使用了8个字节的空间（64bit环境下，可以通过`class_getInstanceSize`函数获得）

#### 2. 对象的isa指针指向哪里？

- instance对象的isa指向class对象
- class对象的isa指向meta-class对象
- meta-class对象的isa指向基类的meta-class对象

#### 3.OC的类信息存放在哪里？

- 对象方法、属性、成员变量、协议信息，存放在class对象中
- 类方法，存放在meta-class对象中
- 成员变量的具体值，存放在instance对象

#### 4.iOS用什么方式实现对一个对象的KVO？(KVO的本质是什么？)

- 利用RuntimeAPI动态生成一个子类，并且让instance对象的isa指向这个全新的子类
- 当修改instance对象的属性时，会调用Foundation的_NSSetXXXValueAndNotify函数
    willChangeValueForKey:
    父类原来的setter
    didChangeValueForKey:
- 内部会触发监听器（Oberser）的监听方法(`observeValueForKeyPath:ofObject:change:context:`）

#### 5.如何手动触发KVO？

手动调用`willChangeValueForKey:和didChangeValueForKey:`



```objectivec
- (void)viewDidLoad {
[super viewDidLoad];
    
    Person *person = [[Person alloc]init];;
    [p addObserver:self forKeyPath:@"name" options:NSKeyValueObservingOptionNew | NSKeyValueObservingOptionOld context:nil];
    [p willChangeValueForKey:@"name"];
    [p didChangeValueForKey:@"name"];
}
-(void)observeValueForKeyPath:(NSString *)keyPath ofObject:(id)object change:(NSDictionary<NSKeyValueChangeKey,id> *)change context:(void *)context{
    NSLog(@"被观测对象：%@, 被观测的属性：%@, 值的改变: %@\n, 携带信息:%@", object, keyPath, change, context);
}
```

#### 6.直接修改成员变量会触发KVO么？

- 不会触发KVO

#### 7.通过KVC修改属性会触发KVO么？

- 会触发KVO
- KVC在赋值时候,内部会触发监听器（Oberser）的监听方法(observeValueForKeyPath:ofObject:change:context:） 发送通知

#### 8.KVC的赋值和取值过程是怎样的？原理是什么？

- KVC的全称是Key-Value Coding，俗称“键值编码”，可以通过一个key来访问某个属性
- 调用 setValue:forKey:
    setKey,_setKey ->找到了则进行赋值,未找到调用 `accessInstanceVarlableDirctly` 是否允许修改value值,返回YES, 调用_key, _isKey, key, isKey 进行赋值

#### 9.Category的使用场合是什么？

- 在不修改原有类代码的情况下,为类添对象方法或者类方法
- 或者为类关联新的属性
- 分解庞大的类文件

使用场合:

- 添加实例方法
- 添加类方法
- 添加协议
- 添加属性
- 关联成员变量

#### 10.Category的实现原理

- Category编译之后的底层结构是`struct category_t`，里面存储着分类的对象方法、类方法、属性、协议信息
- 在程序运行的时候，runtime会将Category的数据，合并到类信息中（类对象、元类对象中）

#### 11.Category和Class Extension的区别是什么？

- Class Extension在编译的时候，它的数据就已经包含在类信息中
- Category是在运行时，才会将数据合并到类信息中

#### 12.Category中有load方法吗？load方法是什么时候调用的？load 方法能继承吗？

- 有load方法
- load方法在runtime加载类、分类的时候调用
- load方法可以继承，但是一般情况下不会主动去调用load方法，都是让系统自动调用

#### 13. initialize方法如何调用,以及调用时机

- 当类第一次收到消息的时候会调用类的initialize方法
- 是通过 runtime 的消息机制 objc_msgSend(obj,@selector()) 进行调用的
- 优先调用分类的 initialize, 如果没有分类会调用 子类的,如果子类未实现则调用 父类的

#### 13. load、initialize方法的区别什么？它们在category中的调用的顺序？以及出现继承时他们之间的调用过程？

- load 是类加载到内存时候调用, 优先父类->子类->分类
- initialize 是类第一次收到消息时候调用,优先分类->子类->父类
- 同级别和编译顺序有关系
- load 方法是在 main 函数之前调用的

#### 14. Category能否添加成员变量？如果可以，如何给Category添加成员变量？

- 不能直接给Category添加成员变量，但是可以间接实现Category有成员变量的效果
- Category是发生在运行时,编译完毕,类的内存布局已经确定,无法添加成员变量(Category的底层数据结构也没有成员变量的结构)
- 可以通过 runtime 动态的关联属性

#### 15. block的原理是怎样的？本质是什么？

- block 本质其实是OC对象
- block 内部封装了函数调用以及调用环境

#### 16. __block的作用是什么？有什么使用注意点？

- 如果需要在 block 内部修改外部的 局部变量的值,就需要使用__block 修饰(全局变量和静态变量不需要加__block 可以修改)
- __block 修饰以后,局部变量的数据结构就会发生改变,底层会变成一个结构体的对象,结构内部会声明 一个 __block修饰变量的成员, 并且将 __block修饰变量的地址保存到堆内存中. 后面如果修改 这个变量的值,可以通过 isa 指针找到这个结构体,进来修改 这个变量的值;
- 可以在 block 内部修改 变量的值

#### 17. block的属性修饰词为什么是copy？使用block有哪些使用注意？

- block 一旦没有进行copy操作，就不会在堆上
- 使用注意：循环引用问题 (外部使用__weak 解决)

#### 17. block在修改NSMutableArray，需不需要添加__block？

- 如果是操作 NSMutableArray 对象不需要,因为 block 内部拷贝了 NSMutableArray对象的内存地址,实际是通过内存地址操作的
- 如果 NSMutableArray 对象要重新赋值,就需要加__block

#### 18. Block 内部为什么不能修改局部变量,需要加__block

- 通过查看Block 源码,可以发现, block 内部如果单纯使用 外部变量, 会在 block 内部创建同样的一个变量,并且将 外部变量的值引用过来..(只是将外部变量值拷贝到 block 内部), 内部这个变量和外部 实际已经没关系了
- 从另一方面分析,block 本质也是一个 函数指针, 外部的变量也是一个局部变量,很有可能 block 在使用这个变量时候,外部变量已经释放了,会造成错误
- 加了__block 以后, 会将外部变量的内存拷贝到堆中, 内存由 block 去管理.

#### 19.讲一下 OC 的消息机制

- OC中的方法调用其实都是转成了objc_msgSend函数的调用，给receiver（方法调用者）发送了一条消息（selector方法名）
- objc_msgSend底层有3大阶段
    - 消息发送（当前类、父类中查找）、
    - 动态方法解析、
    - 消息转发

#### 20. 消息发送流程

- 当我们的一个 receiver(实例对象)收到消息的时候, 会通过 isa 指针找到 他的类对象, 然后在类对象方法列表中查找 对应的方法实现,如果 未找到,则会通过 superClass 指针找到其父类的类对象, 找到则返回,未找打则会一级一级往上查到,最终到NSObject 对象, 如果还是未找到就会进行动态方法解析
- 类方法调用同上,只不过 isa 指针找到元类对象;

#### 21. 动态方法解析机制

当我们发送消息未找到方法实现,就会进入第二步,动态方法解析: 代码实现如下



```objectivec
//  动态方法绑定- 实例法法调用
+ (BOOL)resolveInstanceMethod:(SEL)sel{
    if (sel == @selector(run)) {
        Method method = class_getInstanceMethod(self, @selector(test));
        class_addMethod(self, sel, method_getImplementation(method), method_getTypeEncoding(method));
        return YES;
    }
    return [super resolveInstanceMethod:sel];
}
// 类方法调用
+(BOOL) resolveClassMethod:(SEL)sel....
```

#### 22.消息转发机制流程

未找到动态方法绑定,就会进行消息转发阶段



```kotlin
// 快速消息转发- 指定消息处理对象
- (id)forwardingTargetForSelector:(SEL)aSelector{
    if (aSelector == @selector(run)) {
        return [Student new];
    }
    return  [super forwardingTargetForSelector:aSelector];
} 

// 标准消息转发-消息签名
- (NSMethodSignature *)methodSignatureForSelector:(SEL)aSelector{
    if(aSelector == @selector(run))
    {
        return [NSMethodSignature signatureWithObjCTypes:"v@:"];
    }
    return [super methodSignatureForSelector:aSelector];
}
- (void)forwardInvocation:(NSInvocation *)anInvocation{
   //内部逻辑自己处理 
}
```

#### 23. 什么是Runtime？平时项目中有用过么？

- Objective-C runtime是一个`运行时`库，它为Objective-C语言的动态特性提供支持，我们所写的OC代码在运行时都转成了runtime相关的代码，类转换成C语言对应的结构体，方法转化为C语言对应的函数，发消息转成了C语言对应的函数调用。通过了解runtime以及源码,可以更加深入的了解OC其特性和原理
- OC是一门动态性比较强的编程语言，允许很多操作推迟到程序运行时再进行
- OC的动态性就是由Runtime来支撑和实现的，Runtime是一套C语言的API，封装了很多动态性相关的函数
- 平时编写的OC代码，底层都是转换成了Runtime API进行调用

#### 22.runtime具体应用

- 利用关联对象（AssociatedObject）给分类添加属性
- 遍历类的所有成员变量（修改textfield的占位文字颜色、字典转模型、自动归档解档）
- 交换方法实现（交换系统的方法）
- 利用消息转发机制解决方法找不到的异常问题

#### 23.unrecognized selector sent to instance 错误

该错误是基于OC的消息机制:

1. 在方法列表中未找到方法实现
2. 尝试动态方法解析,也未绑定犯法
3. 进行消息转发,也未处理
4. 最后进行报错

#### 24.如果向一个nil对象发消息不会crash的话,那么message sent to deallocated instance的错误是怎么回事？

- 这是因为这个对象已经被释放了（引用计数为0了），那么这个时候再去调用方法肯定是会Crash的，因为这个时候这个对象就是一个野指针（指向僵尸对象（对象的引用计数为0，指针指向的内存已经不可用）的指针）了，安全的做法是释放后将对象重新置为nil，使它成为一个空指针

#### 25. 向一个nill对象发送消息会发生什么？

- OC中向nil发消息，什么都不会方式,程序是不会崩溃的。
- 因为OC的函数都是通过objc_msgSend进行消息发送来实现的，相对于C和C++来说，对于空指针的操作会引起crash问题，而objc_msgSend会通过判断self来决定是否发送消息，如果self为nil，那么selector也会为空，直接返回，不会出现问题。视方法返回值，向nil发消息可能会返回nil（返回值为对象），0（返回值为一些基础数据）或0X0（返回值为id）等。但对于[NSNull null]对象发送消息时，是会crash的，因为NSNull类只有一个null方法

#### 26.代码打印结果:



```objectivec
@interface Person : NSObject
@end
@implementation Person
@end

@interface Student : Person
@end
@implementation Student
- (instancetype)init{
    if (self= [super init]) {
        NSLog(@"%@", [self class]);
        NSLog(@"%@", [super class]);
        NSLog(@"%@", [self superclass]);
        NSLog(@"%@", [super superclass]);
    }
}
[self class] 和 [super class] 都是给当前类返送消息,spuer 表示在父类中查找
[self superClass]  和 [super superclass] 也是也当前类发消息,返回父类
第一个打印:
Student / Student/ Person / Person
```

#### 27.代码运行结果?



```objectivec
BOOL res1 = [[NSObject class] isKindOfClass:[NSObject class]];
BOOL res2 = [[NSObject class] isMemberOfClass:[NSObject class]];
BOOL res3 = [[Person class] isKindOfClass:[Person class]];
BOOL res4 = [[Person class] isMemberOfClass:[Person class]];
NSLog(@"%d-%d-%d-%d",res1, res2, res3, res4);
```

- isKindOfClass 表示对象是否为当前类或者子类的 类型
- isMemberOfClass 表示是否为当前类的的类型
- isMemberOfClass 分为- 对象方法 和+ 类方法2中
    `- (bool)isMemberOfClass; 比较的是类对象`
    `+ (bool)isMemberOfClass; 比较的是元类`

打印结果: 1 ,0, 0, 0

#### 28.讲讲 RunLoop，项目中有用到吗？

- runloop运行循环,保证程序一直运行,主线程默认开启
- 用于处理线程上的各种事件,定时器等
- 可以提高程序性能,节约CPU资源,有事情做就做,没事情做就让线程休眠
- 应用范畴:
    定时器,事件响应,手势识别,界面刷新,以及autoreleasePool 等等

#### 29.runloop内部实现逻辑？

- 实际上 RunLoop 就是这样一个函数，其内部是一个 do-while 循环。当你调用 CFRunLoopRun() 时，线程就会一直停留在这个循环里；直到超时或被手动停止，该函数才会返回。

#### 30.runloop和线程的关系？

- 每条线程都有唯一的一个与之对应的RunLoop对象
- RunLoop保存在一个全局的Dictionary里，线程作为key，RunLoop作为value
- 线程刚创建时并没有RunLoop对象，RunLoop会在第一次获取它时创建
- RunLoop会在线程结束时销毁
- 主线程的RunLoop已经自动获取（创建），子线程默认没有开启RunLoop

#### 31.timer 与 runloop 的关系？

- timer 定时器,是基于 runloop 来实现的, runloop 在运行循环当中,监听到了定制器 就会执行;所以 timer 需要添加到 runloop 中去, 注意子线程的 runloop 默认是不开启的,如果在子线程执行 timer 需要手动开启 runloop

#### 32.程序中添加每3秒响应一次的NSTimer，当拖动tableview时timer可能无法响应要怎么解决？

- 将 timer 对象添加到 runloop 中,并修改 runloop 的运行 mode



```objectivec
 NSTimer *timer = [NSTimer timerWithTimeInterval:1 repeats:YES block:nil];
 [[NSRunLoop currentRunLoop] addTimer:timer forMode:NSRunLoopCommonModes];
```

#### 33. runloop的mode作用是什么？

runloop 只能在一种 mode 下运行, 做不同的事情,runloop 会切换到对应的 model 下来执行,默认是 kCFRunLoopDefaultMode 如果视图滑动再回切换到 UITrackingRunLoopMode,如果需要在多种 mode 下运行则需要手动设置 kCFRunLoopCommonModes;

1. kCFRunLoopDefaultMode：App的默认Mode，通常主线程是在这个Mode下运行
2. UITrackingRunLoopMode：界面跟踪 Mode，用于 ScrollView 追踪触摸滑动，保证界面滑动时不受其他 Mode 影响
3. UIInitializationRunLoopMode: 在刚启动 App 时第进入的第一个 Mode，启动完成后就不再使用，会切换到kCFRunLoopDefaultMode
4. GSEventReceiveRunLoopMode: 接受系统事件的内部 Mode，通常用不到
5. kCFRunLoopCommonModes: 这是一个占位用的Mode，作为标记kCFRunLoopDefaultMode和UITrackingRunLoopMode用，并不是一种真正的Mode

#### 34.使用method swizzling要注意什么?

1. 方式无限循环
2. 进行版本迭代的时候需要进行一些检验，防止系统库的函数发生了变化

#### 35. 一个系统方法被 多次交换,会有什么影响吗?以及调用顺序?原理



```rust
都会执行,后交换的会先调用.
                         
第一次交换   viewwillAppAppear 和 test1 的指向的方法实现地址发生变化
第二次交换   viewwillAppAppear 和 test2 实际上等于是 test2 和 test1 进行了交换,因为 viewwillAppAppear 已经变为了 test1了.

调用 --> viewwillAppAppear
实际调用顺序 -->test2--->test1-->viewwillAppAppear
形成一个闭环:viewwillAppAppear 也只会调用一次
```

#### 36.runloop 主线程监听卡顿

- 用户层面感知的卡顿都是来自处理所有UI的主线程上，包括在主线程上进行的大计算，大量的IO操作，或者比较重的绘制工作。
- 如何监控主线程呢，首先需要知道的是主线程和其它线程一样都是靠NSRunLoop来驱动的。可以先看看CFRunLoopRun的大概的逻辑 ,不难发现NSRunLoop调用方法主要就是在kCFRunLoopBeforeSources和kCFRunLoopBeforeWaiting之间,还有kCFRunLoopAfterWaiting之后,也就是如果我们发现这两个时间内耗时太长,那么就可以判定出此时主线程卡顿.只需要另外再开启一个线程,实时计算这两个状态区域之间的耗时是否到达某个阀值,便能揪出这些性能杀手.
- 用GCD里的dispatch_semaphore_t开启一个新线程，设置一个极限值和出现次数的值，然后获取主线程上在kCFRunLoopBeforeSources到kCFRunLoopBeforeWaiting再到kCFRunLoopAfterWaiting两个状态之间的超过了极限值和出现次数的场景，将堆栈dump下来，最后发到服务器做收集，通过堆栈能够找到对应出问题的那个方法。



```objectivec
- (void)start
{
    if (observer)
        return;
    
    // // 创建信号
    semaphore = dispatch_semaphore_create(0);
    
    // 注册RunLoop状态观察
    CFRunLoopObserverContext context = {0,(__bridge void*)self,NULL,NULL};
    observer = CFRunLoopObserverCreate(kCFAllocatorDefault,
                                       kCFRunLoopAllActivities,
                                       YES,
                                       0,
                                       &runLoopObserverCallBack,
                                       &context);
    CFRunLoopAddObserver(CFRunLoopGetMain(), observer, kCFRunLoopCommonModes);
    
    // 在子线程监控时长
    dispatch_async(dispatch_get_global_queue(0, 0), ^{
        while (YES)
        {
            // 假定连续5次超时50ms认为卡顿(当然也包含了单次超时250ms)
            long st = dispatch_semaphore_wait(semaphore, dispatch_time(DISPATCH_TIME_NOW, 50*NSEC_PER_MSEC));
            // Returns zero on success, or non-zero if the timeout occurred.
            if (st != 0)
            {
                if (!observer)
                {
                    timeoutCount = 0;
                    semaphore = 0;
                    activity = 0;
                    return;
                }
                
                // kCFRunLoopBeforeSources 即将处理source kCFRunLoopAfterWaiting 刚从睡眠中唤醒
                // RunLoop会一直循环检测，从线程start到线程end，检测检测到事件源（CFRunLoopSourceRef）执行处理函数，首先会产生通知，corefunction向线程添加runloopObservers来监听事件，并控制NSRunLoop里面线程的执行和休眠，在有事情做的时候使当前NSRunLoop控制的线程工作，没有事情做让当前NSRunLoop的控制的线程休眠。
                
                if (activity == kCFRunLoopBeforeSources || activity == kCFRunLoopAfterWaiting)
                {
                    
                    if (++timeoutCount < 3)
                        continue;
                    
                     NSLog(@"有点儿卡");
                }
            }
            timeoutCount = 0;
        }
    });
}
```

#### 37. _objc_msgForward 函数是做什么的?直接 调用它将会发生什么?

- _objc_msgForward 是 IMP 类型，用于消息转发的:当向一个对象发送一条消息，但 它并没有实现的时候，_objc_msgForward 会尝试做消息转发
- 直接调用_objc_msgForward 是非常危险的事，这是把双刃刀，如果用不好会直接 导致程序 Crash，但是如果用得好，能做很多非常酷的事
- JSPatch 就是直接调用_objc_msgForward 来实现其核心功能的

#### 38. 如何打印一个类中的所有实例变量

OC的类实际上是一个objc_class类型的结构体,包含了实例变量列表: (objc_ivar_list),可以通过 runtime 函数来获取这个列表:
`OBJC_EXPORT Ivar _Nonnull * _Nullable class_copyIvarList(Class _Nullable cls, unsigned int * _Nullable outCount)`

例子:



```objectivec
Student *stu = [[Student alloc]init];
stu.stu_name = @"alex";
stu.stu_age = 10;

unsigned int count = 0;
Ivar *list = class_copyIvarList([stu class], &count);
NSMutableDictionary * dict = [NSMutableDictionary dictionary];
for (int i = 0; i< count; i++){
    id iVarName = [NSString stringWithUTF8String:ivar_getName(list[i])];
    dict[iVarName] = [stu valueForKey:iVarName];
}
    
NSLog(@"%@",dict);
```

#### 39. 如何使用 rumtime 动态添加一个类

runtime 很强大.可以动态的创建一个全新的类或对象



```objectivec
// 添加一个继承NSObject的类 类名是MyClass
Class MyClass = objc_allocateClassPair([NSObject class], "MyClass", 0);
// 增加实例变量
class_addIvar(MyClass, "_age", sizeof(NSString *), 0, "@");
//注册这个类到runtime系统中就可以使用他了
objc_registerClassPair(MyClass);
//生成了一个实例化对象
id myobj = [[MyClass alloc] init];
//给刚刚添加的变量赋值
[myobj setValue:@30 forKey:@"age"];
// 打印
NSLog(@"age= %@",[myobj valueForKey:@"age"]);
```

#### 下一篇入口:



作者：Leon_520
链接：https://www.jianshu.com/p/f0504db3a456
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。