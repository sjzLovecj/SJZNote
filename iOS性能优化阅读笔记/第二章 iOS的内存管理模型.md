## 2.1 iOS的内存管理模型
### 2.1.1 关于内存消耗与引用计数
当引用程序启动后，除了代码和系统数据会消耗一部分内存外，开发者在程序中创建的任何对象都会消耗内存。iOS程序中，内存通常被分成如下5个区域：
- 栈区：存储局部变量，在作用域结束后内存会被回收
- 堆区：存储OC对象，需要开发者手动申请和释放
- BSS区：用来存储未初始化的全局变量和静态变量
- 数据区：用来存储已经初始化的全局变量、静态变量 和 常量
- 代码段：加载代码

除了堆区需要开发者手动进行内存管理外，其他区都由系统自动进行回收

引用计数是OC语言提供的内存管理技术，每一个OC对象都有一个retainCount属性。

### 2.1.2 MRC内存管理
关闭ARC：
将TARGETS -> Build Settings -> Objective-C Automatic Reference Counting设置为NO

MRC内存管理有如下两个原则：
- 谁持有，谁负责释放，不是自己持有的不能释放
- 当对象不再被需要时，需要主动释放

| 函数名 | 意义 |
| --- | --- |
| alloc | 进行内存分配 |
| new | 创建并初始化对象 |
| copy | 复制对象 |
| mutablecoy | 可变复制对象 |
| retain | 进行持有 |

alloc、new、copy新创建一个对象，该对象引用计数为1，当需要时，调用release使引用计数减1
retain使对象的引用计数加1，当不再使用对象时，需要调用release，使引用计数减1

### 2.1.3 关于ARC
所有权修饰符：
- __strong：修饰符通常用来对变量进行强引用
    1. 使用__strong修饰的变量如果是自己生成的，则会被添加到自动释放池中，并在作用域结束时，release一次
    2. 使用__strong修饰的变量如果不是自己生成的，则会被强引用，即会被持有使其引用计数增加1，在离开作用域后会被release一次
    3. 使用__strong修饰的变量如果重新复制或者置为nil，则会被release一次
  
- __weak：修饰符通常用来对变量进行弱引用，最大的用途是避免ARC环境下循环引用问题
    1. 被__weak修饰的变量仅提供弱引用，不会使引用计数增加。如果变量是自己生成，则会被添加到自动释放池，会在离开作用域时被release一次；如果不是自己生成，则在离开作用域之后，不会进行release操作
    2. 被__weak修饰的变量指针，变量如果失效，则指针会被自动置为nil，这是一种比较安全的设计方式，大量减少野指针造成的异常。

- __unsafe_unretained：这个修饰符是不安全的，这个修饰符也是对变量进行弱引用，不同的是，当变量对象失效时，其指针不会被自动置为nil，会产生野指针
    1. 被__unsafe_unretained修饰的变量仅提供弱引用，不会使其引用计数增加。变量如果是自己生成的，则会在离开作用域的时候被release一次，如果不是自己生成的，则在离开作用域后，不会进行release操作
    
- __autoreleasing：这个修饰符与自动释放池有关
  

ARC还需要牢记如下几条原则：
1. 不能使用retain、release、autorelease函数，不可访问retainCount属性
2. 不能调用dealloc函数，可以覆写dealloc函数，但在实现中不可调用父类的dealloc函数
3. 不能使用NSAutoreleasePool，可以使用@autoreleasepool替代
4. 对象型变量不能作为C语言的结构体

### 2.1.4 属性修饰符
在探讨类的定义时：类是属性与方法的集合
- 属性用来存储类中的数据
- 方法用来描述类中的行为


| 名称 | 作用 |
| --- | --- |
| assign | 直接赋值，与引用计数无关，用来声明简单数据类型的属性，如Int |
| retain | 对旧对象进行释放，并强引用新的对象，使其引用计数加1，用在MRC中 |
| strong | 对新对象进行强引用，使用旧对象，使其引用计数加1，作用与retain类似，用在ARC中 |
| copy | 在实现Setter方法时，采用copy函数，会生成新的对象被自己持有 |
| weak | 弱引用，不对所赋值的对象进行持有，但是是安全的，当对象不可用时，会被置为nil，用在ARC |
| unsafe_unretained | 弱引用，和weak不同的是，如果引用的对象不可用，则当指针不会被置为nil，会产生野指针 |

### 2.1.5 ARC 和 MRC进行混编
1. 将TARGETS -> Build Settings -> Objective-C Automatic Reference Counting设置为YES
2. 在工程的Build Phases选项中的Compile Sources中对应的文件添加-fno-objc-arc，（在MRC中添加ARC文件，则-fobjc-arc）

## 2.2 自动释放池
### 2.2.1 关于autorelease方法
1. 当引用计数为0时，内存进行释放
2. release会使对象的引用计数减1
3. autorelease，为自动释放，本质上是使release函数的调用被延迟了
4. 自动释放对象的内存管理是交给自动释放池处理的

### 系统维护的自动释放池
iOS系统在运行的时候，会自动创建一些线程，每一个线程都默认拥有自动释放池，在每次时间循环时，都会将其自动释放池清空

## 2.3 杜绝内存泄漏
1. 循环引用
2. Block循环引用，使用__weak解决
3. 代理与循环引用，使用weak修饰符解决
4. 定时器引起的内存泄漏，定时器对象会持有当前视图控制器，只有当定时器失效后，其才会释放所持有的当前视图控制器。调用invalidate方法

## 2.4 关于僵尸对象
### 2.4.1 捕获僵尸对象
在Objective-C中，内存的使用包含下面几个阶段：
- 请求创建对象，向系统申请一块内存空间，在申请完成后，这块内存空间不能再做他用 
- 对象被释放，此时这块内存空间变为闲置，可以被申请使用
- 在此块内存重新被申请使用前，这块内存中的数据依然存在
- 此时如果依然有指针指向这块内存，则此指针为野指针
- 当野指针对这块内存进行访问时，如果这块内存已经被重新分配，则会出现系统问题，如果没有被分配，则不会出现系统问题。

Xcode提供的可以捕获僵尸对象的工具：
Product->Edit SCheme->Run->勾选Zombie Objects

在ARC中，使用__weak修饰和__strong修饰的变量指针变量释放后会被自动置为nil，这就大大减少了野指针的问题，向nil指针发送任何消息都是无效的。


## 2.5 CoreFoundation框架中的内存管理
CoreFoundation也提供了字符串、数组、集合、颜色、时间和URL等对象。

### 2.5.1 CoreFoundation中的引用计数
在CoreFoundation框架中，有几条内存管理法则：
1. 自己创建的对象要自己负责释放
2. 如果使用别人创建的对象，要保证其可用，则需要对对象进行持有
3. 如果对对象进行了持有，则当不再需要此对象时，要进行释放

- 在CoreFoundation框架中，使用带有Create、Copy这样的字段的函数获取的对象会被认为是我们自己创建的对象，我们自己要负责这些对象的释放
- 当使用带有Get这样的字段的函数获取对象时，默认并不对此对象进行持有，可以手动调用CGRetain()函数进行持有

### 2.5.2 CoreFoundation框架与Foundation框架混用
可以通过桥接的方式在CoreFoundation和Foundation框架间灵活地转换对象

- 使用__bridge关键字可以进行CoreFoundation对象和Foundation对象的互相转化，使用__bridge进行转换的对象的对象的引用计数并不会做任何额外操作

```
CFUUIDRef uuid = CFUUIDRef(NULL);
__weak NSUUID * uid = (__bridge NSUUID *)(uuid);
CFRelease(uuid);
```

- 还有一种专门用来将Foundation对象转换成CoreFoundation对象的桥接关键字__bridge_retained，不同于__bridge关键字转换后不会主动对对象引用计数操作，__bridge_retained关键字转换后会对对象进行强引用

```
NSUUID * uuid = [[NSUUID alloc] init];
CFUUIDRef uid = (__bridge_retained CFUUIDRef)uuid;
```
与__bridge_retained对应，__bridge_transfer专门用来将CoreFoundation对象转换为Foundation对象。

```
CFUUIDRef uuid = CFUUIDCreate(NULL);
NSUUID * uid = (__bridge NSUUID *)(uuid);
CFRelease(uuid);
```
ARC环境下默认生了了uid的修饰符__strong，因此实际上uid对uuid对象进行了一次强引用，所以当向uuid发送release消息后，内存并没有被释放，依然可以被正常访问。

```
CFUUIDRef uuid = CGUUIDCreate(NULL);
NSUUID * uid = (__bridge_transfer NSUUID*)(uuid);
CGRelease(uuid);
```
运行工程，会发现程序会直接发生Crash，__bridge_transfer的作用是在转换后主动进行一次释放操作，可以简单解释，__bridge_transfer将原对象的所有权交给了转换后的指针，虽然上面uid默认使用__strong修饰，会使引用计数加1，但是其主动的release又会使引用计数减1，后面我们又进行了release操作，因此对象被释放，之后的访问产生Crash。

## 2.6 扩展：关于id 与 void*
### 2.6.1 关于id类型
id是Objective-C中定义的一种泛型实现，它可以表示任何对象类型。
- 作为参数或返回值
- id类型的参数不会进行类型检查
    - 声明为id类型的对象就相当于告诉了编译器不进行类型检查，使用id类型的对象可以调用任何方法，都不会进行类型检查
- id<protocol>是一种优雅的编程方式
    - id类型不会进行编译检查，因此约束类型方法实现的最好方式就是通过协议。

### 2.6.2 关于void与void*
Objective-C语言的函数必须有一个确定的返回值类型，如果没有返回值，则需要使用void来标记返回值类型

void* 描述的实际上是任意类型的指针，这里和id很像，虽然id描述的是Objective-C对象，但是本质上也是指针，根据推测，id类型的数据和void* 类型的数据是可以进行类型转换的

事实上，MRC环境下确实如此，ARC下需对OC对象进行引用计数管理，对C语言指针并不会，ARC下不允许直接将id与void* 进行转换，需要用桥接的方式

类型检查都是编译时的特性，真正传递的数据依然在运行时决定