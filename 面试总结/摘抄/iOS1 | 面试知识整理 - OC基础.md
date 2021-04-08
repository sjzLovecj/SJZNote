# iOS | 面试知识整理 - OC基础 (一)

### 前言:

最近公司项目不怎么忙, 闲暇时间把iOS 在面试中可能会遇到的问题整理了一番, 一部分题目是自己面试遇到的,一部分题目则是网上收录的, 方便自己巩固复习, 也分享给大家! 知识点比较多,比较杂,这里做了分类,下面是分类链接地址;

**面试知识点整理 - 目录:**
[iOS | 面试知识整理 - OC基础 (一)](https://www.jianshu.com/p/51c9eb362b71)
[iOS | 面试知识整理 - OC基础 (二)](https://www.jianshu.com/p/2057f9663d78)
[iOS | 面试知识整理 - OC基础 (三)](https://www.jianshu.com/p/f0504db3a456)
[iOS | 面试知识整理 - UI 相 关 (四)](https://www.jianshu.com/p/c6b7302b7411)
[iOS | 面试知识整理 - 内存管理 (五)](https://www.jianshu.com/p/713106afd7ef)
[iOS | 面试知识整理 - 多 线 程 (六)](https://www.jianshu.com/p/a937fd308669)
[iOS | 面试知识整理 - 网络相关 (七)](https://www.jianshu.com/p/2f82ea4627cb)
[iOS | 面试知识整理 - 数据持久化 (八)](https://www.jianshu.com/p/c8a39b53179e)
[iOS | 面试知识整理 - Swift 基础 (九)](https://www.jianshu.com/p/d18907e2654d)

# iOS | 面试知识整理 - OC基础 (一)

#### 1. #include、#import、@class的区别?

- 在C 语言中, 我们使用 `#include` 来引入头文件,如果需要防止重复导入需要使用`#ifndef...#define...#endif`
- 在OC语言中, 我们使用`#import`来引入头文件,可以防止重复引入头文件,可以避免出现头文件递归引入的现象。
- `@class`仅用来告诉编译器，有这样一个类，编译代码时，不报错,不会拷贝头文件.如果需要使用该类或者内部方法需要使用 `#import`导入

#### 2. id 和 instancetype的区别?

- `id`可以作为方法的返回以及参数类型 也可以用来定义变量
- `instancetype` 只能作为函数或者方法的返回值
- instancetype对比id的好处就是: 能精确的限制返回值的具体类型

#### 3. New 作用是什么?

1. 向计算机(堆区)申请内存空间;
2. 给实例变量初始化;
3. 返回所申请空间的首地址;

#### 4.OC实例变量的修饰符? 及作用范围?

@puplic

```
 1.可以在其他类中访问被@public修饰的成员变量
 2.也可以在本类中访问被@public修饰的成员变量
 3.可以在子类中访问父类中被@public修饰的成员变量
复制代码
```

@private

```
1.不可可以在其他类中访问被@private修饰的成员变量
2.也可以在本类中访问被@private修饰的成员变量
3.不可以在子类中访问父类中被@private修饰的成员变量
复制代码
```

@protected (默认情况下所有的实例变量都是protected)

```
1.不可可以在其他类中访问被@protected修饰的成员变量
2.也可以在本类中访问被@protected修饰的成员变量
3.可以在子类中访问父类中被@protected修饰的成员变量
复制代码
```

@package

```
介于public和private之间的,如果是在其他包中访问就是private,在当前代码中访问就是public.
复制代码
```

#### 5. @proprety的作用

```
@property = ivar + getter + setter;
复制代码
```

1. 在.h文件中帮我们自动生成`get`和`set`方法声明
2. 在.m文件中帮我们生成私有的实例变量(前提是没有在.h文件中没有手动生成)
3. 在.m文件中帮我们是实现`get和set`方法的实现

- 注意:
    在使用@property情况下,可以重写getter和setter方法.需要注意的是, 当把setter和getter方法都实现了之后,实例变量也需要手动添加.

#### 6. @proprety 参数说明?

- 原子性---`atomic/nonatomic` 如果不写默认情况为 `atomic`(系统会自动加上同步锁，影响性能),在 iOS 开发中尽量指定为 `nonatomic`，这样有助于提高程序的性能
- 读/写权限---`readwrite(读写)、readooly (只读)`
- 内存管理语义---`retain、assign、strong、 weak、unsafe_unretained、copy`
- 方法名---`getter=、setter=`

#### 7 NSObject和id的区别?

- NSObject和id都可以指向任何对象
- NSObject对象会在编译时进行检查,需要强制类型转换
- id类型不需要编译时检查,不需要强制类型转换

#### 8. id类型, nil , Nil ,NULL和NSNULL的区别?

- id类型: 是一个独特的数据类型，可以转换为任何数据类型，id类型的变量可以存放任何数据类型的对象，在内部处理上，这种类型被定义为指向对象的指针，实际上是一个指向这种对象的实例变量的指针; id 声明的对象具有运行时特性，既可以指向任意类型的对象
- nil 是一个实例对象值;如果我们要把一个对象设置为空的时候,就用nil
- Nil 是一个类对象的值,如果我们要把一个class的对象设置为空的时候,就用Nil
- NULL 指向基本数据类型的空指针(C语言的变量的指针为空)
- NSNull 是一个对象,它用在不能使用nil的场合

#### 9. atomic和nonatomic区别,以及作用?

`atomic`与`nonatom`的主要区别就是系统自动生成的`getter/setter`方法不一样

- atomic系统自动生成的getter/setter方法会进行加锁操作
- nonatomic系统自动生成的getter/setter方法不会进行加锁操作

**atomic不是线程安全的**

- 系统生成的getter/setter方法会进行加锁操作,注意:这个锁仅仅保证了getter和setter存取方法的线程安全.
- 因为getter/setter方法有加锁的缘故,故在别的线程来读写这个属性之前,会先执行完当前操作.
- atomic 可以保证多线程访问时候,对象是未被其他线程销毁的(比如:如果当一个线程正在get或set时,又有另一个线程同时在进行release操作,可能会直接crash)

#### 10. 什么情况使用 weak 关键字，相比 assign 有 什么不同?

- 在 ARC 中,在有可能出现循环引用的时候,往往要通过让其中一端使用 `weak` 来解决, 比如:`delegate` 代理属性， 自身已经对它进行一次强引用,没有必要再强引用一次,此时也会使用 `weak`,自定义 `IBOutlet` 控件属性一般也使用 `weak`;当然，也可以使用 `strong`，但是建议使用 `weak`

**weak 和 assign 的不同点**

- `weak` 策略在属性所指的对象遭到摧毁时，系统会将 `weak` 修饰的属性对象的指针指 向 `nil`，在 `OC` 给 `nil` 发消息是不会有什么问题的; 如果使用 `assign` 策略在属性所指 的对象遭到摧毁时，属性对象指针还指向原来的对象，由于对象已经被销毁，这时候就产生了野指针，如果这时候在给此对象发送消息，很容造成程序奔溃 `assigin` 可以用于修饰非 `OC`对象,而 `weak` 必须用于 `OC` 对象

#### 11. 代理使用 weak 还是 assign

- 建议使用 `weak`, 对于weak: 指明该对象并不负责保持delegate这个对象，delegate这个对象的销毁由外部控制。
- 可以使用 `assign`,也有weak的功效, 对于使用 assign 修饰delegate, 在对象释放前,需要将 delegate 指针设置为 nil,不然会产生野指针

#### 12. ARC 下，不显式指定任何属性关键字时，默认 的关键字都有哪些?

- 基本数据类型: `atomic,readwrite,assign`
- 普通的 OC 对象: `atomic,readwrite,strong`

#### 13. 怎么用 copy 关键字?

- `NSString、NSArray、NSDictionary 等等经常使用 copy` 关键字，是因为他们有对应 的可变类型:`NSMutableString、NSMutableArray、NSMutableDictionary`，为确保 对象中的属性值不会无意间变动，应该在设置新属性值时拷贝一份，保护其封装性
- `block` 也经常使用 `copy` 关键字,方法内部的 `block` 默认是 在栈区的,使用 `copy` 可以把它放到堆区.
- 对于 `block` 使用 `copy` 还是 `strong` 效果是一样的，但是 建议写上 `copy`，因为这样显示告知调用者“编译器会自动对 `block` 进行了 `copy` 操 作

#### 14. 如何让自定义类可以用 copy 修饰符?如何重写带 copy 关键字的 setter?

若想令自己所写的对象具有拷贝功能，则需实现 `NSCopying` 协议。如果自定义的对象分为可变版本与不可变版本，那么就要同时实现 `NSCopyiog 与 NSMutableCopying 协议`

```
// 实现不可变版本拷贝
- (id)copyWithZone:(NSZone *)zone; 
// 实现可变版本拷贝
- (id)mutableCopyWithZone:(NSZone *)zone;
// 重写带 copy 关键字的 setter
- (void)setName:(NSString *)name {
    _name = [name copy];
}
复制代码
```

#### 15. weak 属性需要在 dealloc 中置 nil 么

- 在 ARC 环境无论是强指针还是弱指针都无需在 dealloc 设置为 nil ， ARC 会自动帮我们处理
- 即便是编译器不帮我们做这些，weak 也不需要在 dealloc 中置 nil 在属性所指的对象遭到摧毁时，属性值也会清空

#### 16.说一下OC的反射机制;

- OC的反射机制主要是基于OC的动态语言特性;
- 系统Foundation框架为我们提供了一些方法反射的API;
- 我们可以通过这些API执行将字符串转为SEL等操作;
- 由于OC语言的动态性，这些操作都是发生在运行时的。

#### 17.手写单例

方式一: 不是线程安全的,如果多线程需要加锁

```
static ClassName *_instance;
+ (instancetype)sharedInstance{
   @synchronized (self) {
       if(!_instance)   {
           _instance = [self alloc]init];
       }
    }
    return _instance;
} 
复制代码
```

方式二: 注意多线程问题 GCDdispatch_once 默认是线程安全的

```
  static ClassName *_instance;
    + (instancetype)sharedInstance{
        static dispatch_one_t oneToken;
        dispatch_once(&onetoken,^{
           _instance = [self alloc]init];
        });
        return _instance;
    }
    
    + (instancetype)allocWithZone:(NSZone *) zone{
      static dispatch_t onetoken;
      dispatch_once(&oncetoken ^{
         _instance = [super allocwithzone:zone];
      })
      retun _instance
    }
复制代码
```

#### 18. 什么是僵尸对象?

- 已经被销毁的对象(不能再使用的对象),内存已经被回收的对象。

#### 19.野指针

- 指向僵尸对象(不可用内存/已经释放的内存地址)的指针

比如:

```
NSObject *obj = [NSObject new];
[obj release]; // obj 指向的内存地址已经释放了,
obj 如果再去访问的话就是野指针错误了.
野指针错误形式在Xcode中通常表现为：Thread 1：EXC_BAD_ACCESS，因为你访问了一块已经不属于你的内存。
复制代码
```

#### 20. 什么是内存泄露?

- 内存泄露 :一个对象不再使用,但是这个对象却没有被销毁,空间没有释放,则这个就叫做内存泄露.
- ARC导致的循环引用 block,delegate,NSTimer等.

#### 21.数组copy后里面的元素会复制一份新的吗

- 不会,数组里面存的是之前对象的地址,不会改变,可以自己测试一下

#### 22. 如下代码,会有什么问题吗?

```
@property (copy, nonatomic) NSMutableArray * array
复制代码
```

使用 copy 修饰,会生成不可变数组,在添加删除数组元素时候会崩溃

#### 23. OC中的NSInteger 和int 有什么区别

- 在32位操作系统时候, NSInteger 等价于 int,即32位
- 在64位操作系统时候, NSInteger 等价于 long,即64位

#### 24. @synthesize 和 @dynamic 分别有什么作用

- @property 有两个对应的词，一个是`@synthesize`，一个是`@dynamic`。
- 如果 `@synthesize 和@dynamic`都没写，那么默认的就是`@syntheszie var = _var`;
- `@synthesize` 的语义是如果你没有手动实现 `setter` 方法和 `getter` 方法，那么编译器 会自动为你加上这两个方法
- `@dynamic` 告诉编译器:属性的 `setter 与 getter`方法由用户自己实现，不自动生成(当然对于 readonly 的属性只需提供 getter 即可)

#### 25.NSMutableDictionary 中使用setValueForKey 和 setObjectForKey有什么区别?

- 根据官方文档说明: 一般情况下,如果给NSMutableDictionary 发送`setValue` 仍然是调用了 `setObject`方法, 如果参数 value 为 nil,则会调用`removeObject` 移除这个键值对;
- `setObjectForKey` 是 NSMutableDictionary特有的, value 不能为 nil,否则会崩溃
- `setValueForKey` 是KVC的,key 必须是字符串类型, setObject 的 key 可以是任意类型

#### 26.列举出延迟调用的几种方法?

- performSelector方法

```
[self performSelector:@selector(Delay) withObject:nil afterDelay:3.0f];
复制代码
```

- NSTimer定时器

```
[NSTimer scheduledTimerWithTimeInterval:3.0f target:self selector:@selector(Delay) userInfo:nil repeats:NO];
复制代码
```

- sleepForTimeInterval

```
[NSThread sleepForTimeInterval:3.0f];
复制代码
```

- GCD方式

```
dispatch_time_t popTime = dispatch_time(DISPATCH_TIME_NOW, (int64_t)(3 * NSEC_PER_SEC));
    dispatch_after(popTime, dispatch_get_main_queue(), ^(void){
        [self Delay];
    });
- (void)Delay {
 NSLog(@"执行"); 
}
复制代码
```

#### 27. NSCache 和NSDictionary 区别?

1. NSCache可以提供自动删减缓存功能，而且保证线程安全，与字典不同，不会拷贝键。
2. NSCache可以设置缓存上限，限制对象个数和总缓存开销。定义了删除缓存对象的时机。这个机制只对NSCache起到指导作用，不会一定执行。
3. NSPurgeableData搭配NSCache使用，可以自动清除数据。
4. 只有那种“重新计算很费劲”的数据才值得放入缓存。

#### 28.NSArray 和 NSSet区别

- NSSet和NSArray功能性质一样，用于存储对象，属于集合。
- NSSet属于 “无序集合”，在内存中存储方式是不连续
- NSArray是 “有序集合” 它内存中存储位置是连续的。
- NSSet，NSArray都是类，只能添加对象，如果需要加入基本数据类型（int，float，BOOL，double等），需要将数据封装成NSNumber类型。
- 由于NSSet是用hash实现的所以就造就了它查询速度比较快，但是我们不能把某某对象存在第几个元素后面之类的有关下标的操作。

#### 29.声明一个函数,传入值是一个输入输出参数都是 int的 block 函数

```
- (void)test_Function:(int(^)(int num)) block{}
复制代码
```

#### 30.面向对象和面向过程的区别?

- 面向过程:注重的是解决问题的步骤,比如C语言
- 面向对象:关注的是解决问题的去要那些对象,OC语言就是面向对象

#### 31.对象方法和类方法的区别?

- 对象方法:以减号开头,只可以被对象调用,可以访问成员变量
- 类方法:以加号开头只能用类名调用,对象不可以调用,类方法不能访问成员变量

#### 32. 什么是面向过程?(POP--Procedure Oriented Programming)

- “面向过程”`(Procedure Oriented)`是一种以过程为中心的编程思想。就是分析出解决问题所需要的步骤，然后用函数把这些步骤一步一步实现，使用的时候一个一个依次调用就可以了。注重的是实现过程!

#### 33. 什么是面向对象?(OOP--Object Oriented Programming)

- “面向对象”是一种以对象为中心的编程思想。
- 面向对象的三大特性：

1. 封装
    隐藏对象的属性和实现细节，仅对外提供公共访问方式，将变化隔离，便于使用，提高复用性和安全性。
2. 继承
    提高代码复用性；建立了类之间的关系；子类可以拥有父类的所有成员变量的方法；继承是多态的前提。
3. 多态
    父类或接口定义的引用变量可以指向子类或具体实现类的实例对象。提高了程序的拓展性。

- 正因为面向对象编程有着着三种特性，`继承、封装、多态`，从而使得面向对象编程更具有容易让人接受，更贴近与人们的生活，比面向对象编程更加方便与快捷，一定程度上降低了程序员的工作量，使程序的可读性也得到了提高，代码的效率也得到了提高。

#### 34. 什么是多态?

- 多态在面向对象语言中指同一个接口有多种不同的实现方式,在OC中,多态则是不同对象对同一消息的不同响应方式;子类通过重写父类的方法来改变同一方法的实现.体现多态性
- 通俗来讲: 多态就父类类型的指针指向子类的对象,在函数（方法）调用的时候可以调用到正确版本的函数（方法）。
- 多态就是某一类事物的多种形态.继承是多态的前提;

#### 35. 什么是分类?

- 分类: 在不修改原有类代码的情况下,可以给类添加方法
    Categroy 给类扩展方法,或者关联属性, Categroy底层结构也是一个结构体:内部存储这结构体的名字,那个类的分类,以及对象和类方法列表,协议,属性信息
- 通过Runtime加载某个类的所有Category数据
- 把所有Category的方法、属性、协议数据，合并到一个大数组中后面参与编译的Category数据，会在数组的前面
- 将合并后的分类数据（方法、属性、协议），插入到类原来数据的前面

#### 36.什么是协议?

- 协议：协议是一套标准，这个标准中声明了很多方法，但是不关心具体这些方法是怎么实现的，具体实现是由遵循这个协议的类去完成的。
- 在OC中，一个类可以实现多个协议，通过协议可以弥补单继承的缺陷但是协议跟继承不一样，协议只是一个方法列表，方法的实现得靠遵循这个协议的类去实现。

#### 37.正式协议&非正式协议?

- 非正式协议:凡是在NSObject或其子类 Foundation 框架中的类增加类别(分类),都是非正式协议
- 正式协议: @protocol

#### 38.如何实现多继承?

1. 类别
2. 协议
3. 消息转发 (后面会详细讲述)

#### 39.为什么说OC是一门动态语言？

- 动态语言:是指程序在运行时可以改变其结构，新的函数可以被引进,已有的函数可以被删除等在结构上的变化
- 动态类型语言: 就是类型的检查是在运行时做的。

OC的动态特性可从三方面:

- 动态类型（Dynamic typing）:最终判定该类的实例类型是在运行期间
- 动态绑定（Dynamic binding）：在运行时确定调用的方法
- 动态加载（Dynamic loading）：在运行期间加载需要的资源或可执行代码

#### 40.动态绑定?

- 动态绑定 将调用方法的确定也推迟到运行时。OC可以先跳过编译，到运行的时候才动态地添加函数调用，在运行时才决定要调用什么方法，需要传什么参数进去，这就是动态绑定。
- 在编译时，方法的 调用并不和代码绑定在一起，只有在消实发送出来之后，才确定被调用的代码。通过动态类型和动态绑定技术，

#### 41. cocoa 和 cocoa touch是什么?区别?

- Cocoa包含Foundation和AppKit框架，可用于开发Mac OS X系统的应用程序。
- Cocoa Touch包含Foundation和UIKit框架，可用于开发iPhone OS系统的应用程序。
- Cocoa是 Mac OS X 的开发环境，Cocoa Touch是 iPhone OS的开发环境。

#### 42. cocoa touch底层技术架构?

cocoa touch底层技术架构 主要分为4层:

- 可触摸层 Cocoa Touch : UI组件,触摸事件和事件驱动,系统接口
- 媒体层 Media: 音视频播放,动画,2D和3D图形
- Core Server: 核心服务层,底层特性,文件,网络,位置服务区等
- Core OS: 内存管理,底层网络,硬盘管理

#### 43. 什么是谓词?

谓词(`NSPredicate`)是OC针对数据集合的一种逻辑帅选条件,类似一个过滤器,简单实实用代码如下:

```
Person * p1 = [Person personWithName:@"alex" Age:20];
Person * p2 = [Person personWithName:@"alex1" Age:30];
Person * p3 = [Person personWithName:@"alex2" Age:10];
Person * p4 = [Person personWithName:@"alex3" Age:40];
Person * p5 = [Person personWithName:@"alex4" Age:80];
    
NSArray * persons = @[p1, p2, p3, p4, p5];
//定义谓词对象,谓词对象中包含了过滤条件
NSPredicate *predicate = [NSPredicate predicateWithFormat:@"age < 30"];
//使用谓词条件过滤数组中的元素,过滤之后返回查询的结果
NSArray *array = [persons filteredArrayUsingPredicate:predicate];
复制代码
```

#### 44. 什么是类工厂方法?

类工厂方法就是用来快速创建对象的类方法, 他可以直接返回一个初始化好的对象,具备以下特征:

1. 一定是类方法
2. 返回值需要是 id/instancetype 类型
3. 规范的方法名说说明类工厂方法返回的是一个什么对象,一般以类名首字母小写开始;

比如系统 UIButton 的buttonWithType 就是一个类工厂方法:

```
// 类工厂方法
+ (instancetype)buttonWithType:(UIButtonType)buttonType;
// 使用
+ UIButton * button = [UIButton buttonWithType:UIButtonTypeCustom];
复制代码
```

#### 45. 什么是糖衣语法?

糖衣语法又叫做`语法糖`或`语法盐`,是指在计算机语言中添加某种语法,这种语法对语言的功能没有影响,但更方便程序员使用,增加程序的可读性,减少代码出错机会

OC中的字面量,其实就是语法糖

```
NSNumber * number = @1;
NSArray * array = @[@1, @2, @3];
NSDictionary * dict = @{@"key":@"value"};
NSNumber * num1 = array[0];
NSString * value = dict[@"key"];
复制代码
```

#### 46.Svn 和 Git 区别

- svn 和 git 都是用来对项目进行版本控制以及代码管理的.可以监测代码及资源的更改变化.有利于实现高效的团队合作;
- svn 是集中式的,集中式是指只有一个远程版本库,git 是分布式的,分布式有本地和远程版本库,本地仓库都保留了整个项目的完整备份;
    如果存储远程版本库的服务器挂了，所有人的代码都无法提交，甚至丢失版本库, git则因为有本地版本库而不会有这个问题。
- 由于两者的架构不同,git 和 svn 的分支也是不同的, svn 的分支是一个完整的目录,包含所有的实际文件,和中心仓库是保持同步的,如果某个团队成员创建新的分支,那么会同步到所有的版本成员中,所有人都会收到影响. 而 git下创建的分支合并前是不会影响到任何人的.创建分支可以在本地脱机进行任何操作.测试无误后在合并到主分支,然后其他成员才可以看得到.

#### 47.OC中有二维数组吗? 如何实现?

OC中没有二维数组, 可以通过一维数组嵌套来实现二维数组;

```
// 字面量定义
NSArray * array = @[
                    @[@1,@2,@3,@4,@5],
                    @[@11,@12,@13,@14,@15],
                    @[@21,@22,@23,@24,@25],
                    @[@31,@32,@33,@34,@35],
                    @[@41,@42,@43,@44,@45],
                    ];

// 访问
NSLog(@"%@",array[1][1]);
复制代码
```

#### 48.CocoaPods理解

CocoaPods 是一个 objc 的依赖管理工具，而其本身是利用 ruby 的依赖管理 gem 进行构建的

- 想深入了解这个命令执行的详细内容，可以在这个命令后面加上 --verbose。现在运行这个命令 pod install --verbose
- CocoaPod三方库,会优先编译

#### 49. --verbose 和 --no-repo-update有什么用?

- verbose意思为 冗长的、啰嗦的，一般在程序中表示详细信息。此参数可以显示命令执行过程中都发生了什么。
- pod install或pod update可能会卡在Analyzing dependencies步骤，因为这两个命令会升级 CocoaPods 的 spec 仓库，追加该参数可以省略此步骤，命令执行速度会提升。

#### 50. KVC中的集合运算符

1. 简单集合运算符：@avg、@sum、@max、@min、@count (只能用在集合对象中，对象属性必须为数字类型)
2. 对象操作符：
    @unionOfObjects：返回指定属性的值的数组，不去重
    @distinctUnionOfObjects：返回指定属性去重后的值的数组
3. 数组 / 集体操作符：跟对象操作符很相似，只不过是在NSArray和NSSet所组成的集合中工作的。@unionOfArrays：返回一个数组，值由各个子数组的元素组成，不去重 @distinctUnionOfArrays：返回一个数组，值由各个子数组的元素组成，去重 @distinctUnionOfSets：和@distinctUnionOfArrays差不多, 只是它期望的是一个包含着NSSet对象的NSSet，并且会返回一个NSSet对象。因为集合不能有重复的值，所以只有distinct操作。

#### 51.简要说明const,宏,static,extern区分以及使用?

**const**

```
const常量修饰符,经常使用的字符串常量，一般是抽成宏，但是苹果不推荐我们抽成宏，推荐我们使用const常量。

- const 作用：限制类型
- 使用const修饰基本变量, 两种写法效果一致 , b都是只读变量
  const int b = 5; 
  int const b = 5;   
- 使用const修饰指针变量的变量 
  第一种: const int *p = &a 和 int const *q = &a; 效果一致,*p 的值不能改,p 的指向可以改; 
  第二种: int * const p = &a;  表示 p 的指向不能改,*p 的值可以改
  第三种: 
  const int * const p = &a; *p 值和 p 的指向都不能改
  
  const 在*左边, 指向可变, 值不可变
  const 在*的右边, 指向不可变, 值可变
  const 在*的两边, 都不可变
复制代码
```

**宏**

```
* 基本概念：宏是一种批量处理的称谓。一般说来，宏是一种规则或模式，或称语法替换 ，用于说明某一特定输入（通常是字符串）如何根据预定义的规则转换成对应的输出（通常也是字符串)。这种替换在预编译时进行，称作宏展开。编译器会在编译前扫描代码，如果遇到我们已经定义好的宏那么就会进行代码替换，宏只会在内存中copy一份，然后全局替换，宏一般分为对象宏和函数宏。 宏的弊端：如果代码中大量的使用宏会使预编译时间变长。

const与宏的区别？

* 编译检查 宏没有编译检查，const有编译检查；
* 宏的好处 定义函数，方法 const不可以；
* 宏的坏处 大量使用宏，会导致预编译时间过长
复制代码
```

**static**

```
* 修饰局部变量: 被static修饰局部变量，延长生命周期，跟整个应用程序有关，程序结束才会销毁,被 static 修饰局部变量，只会分配一次内存
* 修饰全局变量: 被static修饰全局变量，作用域会修改，也就是只能在当前文件下使用
复制代码
```

**extern**

```
声明外部全局变量(只能用于声明，不能用于定义)

常用用法（.h结合extern联合使用）
如果在.h文件中声明了extern全局变量，那么在同一个类中的.m文件对全局变量的赋值必须是：数据类型+变量名（与声明一致）=XXXX结构。并且在调用的时候，必须导入.h文件。代码如下：

.h
@interface ExternModel : NSObject
extern NSString *lhString;
@end 
.m     
@implementation ExternModel
NSString *lhString=@"hello";
@end

调用的时候：例如：在viewController.m中调用，则可以引入：ExternModel.h，否则无法识别全局变量。当然也可以通过不导入头文件的方式进行调用（通过extern调用）。
复制代码
```

#### 52.编译型和解释型的区别?

- 编译型语言: 首先是将源代码编译生成机器指令，再由机器运行机器码 (二进制)。
- 解释型语言: 源代码不是直接翻译成机器指令，而是先翻译成中间代码，再由解释器对中间代码进行解释运行。

#### 53.动态语言和静态语言?

- 动态类型语言: 是指数据类型的检查是在运行时做的。用动态类型语言编程时，不用给变量指定数据类型，该语言会在你第一次赋值给变量时，在内部记录数据类型。
- 静态类型语言: 是指数据类型的检查是在运行前（如编译阶段）做的。

#### 54.什么是指针常量和常量指针？

- 常量指针本质是指针，常量修饰它，表示这个指针乃是一个指向常量的指针（变量）。
    指针指向的对象是常量，那么这个对象不能被更改。
- 指针常量的本质是一个常量，而用指针修饰它，那么说明这个常量的值应该是一个指针。
    指针常量的值是指针，这个值因为是常量，所以不能被赋值

#### 55. 指针函数和函数指针

指针函数

- 指针函数： 顾名思义，它的本质是一个函数，不过它的返回值是一个指针。

```
// 指针函数
int *sum(int a, int b){
    int result = a + b;
    int *c = &result;
    return c;
}
int *p = sum(10, 20);
printf("sum:%d\n", *p);
复制代码
```

函数指针

- 与指针函数不同，函数指针 的本质是一个指针，该指针的地址指向了一个函数，所以它是指向函数的指针。

```
// 函数指针
int max(int a, int b){
    return (a > b)?a:b;
}
int (*p)(int, int) = max;
int result = p(10, 20);
printf("result:%d\n", result);
复制代码
```

#### 56.写一个标准的宏MAX,这个宏输入2个参数,返回最大一个

```
#define Max(a,b) a>b?a:b
复制代码
```

#### 57.自定义宏 #define MIN(A,B) A

```
float a = 1;
float b = MIN(a++,1.5);
问 a= ? b = ?
答案: a = 3; b = 2
a++ 会后执行, a++在表达式出现了2次,得3,  a++<1.5,返回a++,得2

// 扩展
float a = 1;
float b = [self getMix:a++ b:1.5];
- (CGFloat)getMix:(CGFloat ) a b:(CGFloat)b{
   return a>b?a:b;
}
运行 a = 2; b =1;
复制代码
```

