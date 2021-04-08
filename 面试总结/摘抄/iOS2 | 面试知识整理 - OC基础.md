# iOS | 面试知识整理 - OC基础 (二)

#### 1.C和 OC 如何混编

xcode可以识别一下几种扩展名文件:

- .m文件,可以编写 OC语言 和 C 语言代码
- .cpp: 只能识别C++ 或者C语言(C++兼容C)
- .mm: 主要用于混编 C++和OC代码,可以同时识别OC,C,C++代码

#### 2. Swift 和OC 如何调用?

- Swift 调用 OC代码
    需要创建一个 `Target-BriBridging-Header.h` 的桥文件,在乔文件导入需要调用的OC代码头文件即可
- OC 调用 Swift代码
    直接导入 `Target-Swift.h`文件即可, Swift如果需要被OC调用,需要使用@objc 对方法或者属性进行修饰

#### 3. Foundation 对象与 CoreFoundation 对象 有什么区别?

- `Foundation`对象是OC的,在MRC下需要手动管理内存,ARC下不需要手动管理
- `Core Foundation`对象是C对象, MRC和ARC都需要手动管理内存
- 数据类型之间的转换
    - ARC:__bridge_retained, __bridge_transfer(自动内存管理)
    - 非ARC: __bridge

#### 4.与OC比较.Swift有什么优点?

Swift 是一门新型语言,借鉴了`JS,Python,C#,Ruby`等语言特性,看上去偏脚本化,Swift 仍支持 `cocoa touch` 框架

优点:

1. Swift更加安全，它是类型安全的语言。
2. Swift容易阅读，语法和文件结构简易化。
3. Swift更易于维护，文件分离后结构更清晰。
4. Swift代码更少，简洁的语法，可以省去大量冗余代码
5. Swift速度更快，运算性能更高。

#### 5. delegate(代理,委托)

委托是协议的一种,顾名思义，就是委托他人帮自己去做什么事。

delegate:

1. 非常严格的语法。所有将听到的事件必须是在delegate协议中有清晰的定义,语法清晰,易读;
2. 如果delegate中的一个方法没有实现那么就会出现编译警告/错误
3. 在释放代理对象时，需要小心的将delegate改为nil。一旦设定失败，那么调用释放对象的方法将会出现内存crash

#### 6.Notification(通知)

在IOS应用开发中有一个`”Notification Center“`的概念。它是一个单例对象，允许当事件发生时通知一些对象。

notification:

1. 对于一个发出的通知，多个对象能够做出反应，即1对多的方式实现简单
2. controller能够传递context对象（dictionary），context对象携带了关于发送通知的自定义的信息
3. 在调试的时候应用的工作以及控制过程难跟踪；

#### 7.KVO

KVO是一个对象能够观察另外一个对象的属性的值，并且能够发现值的变化。

KVO：

1. 能够提供一种简单的方法实现两个对象间的同步。例如：model和view之间同步；
2. 能够对非我们创建的对象，即内部对象的状态改变作出响应，而且不需要改变内部对象（SKD对象）的实现；

#### 8. 如何选择delegate、notification、KVO？

三种模式都是一个对象传递事件给另外一个对象，并且不要他们有耦合。

1. delegate. 一对一
2. notification 一对多,多对多
3. KVO 一对一

三者各有自己的特点:

- delegate 语法简洁,方便阅读,易于调试
- notification 灵活多变,可以跨越多个类之间进行使用
- KVO 实现属性监听,实现model和view同步
- 可以根据实际开发遇到的场景来使用不同的方式

#### 9. 若你去设计一个通知中心，你会怎样设计？

个人理解: 参考现有的通知中心

1. 创建通知中心单例类,并在里面有个一个保存通知的全局NSDiction
2. 对于注册通知的类,将其注册通知名作为key, 执行的方法和类,以及一些参数做为一个数组为值
3. 发送通知可以调用通知中心,通过字典key(通知名),找到对应的 类和方法进行执行调用传值.

#### 10. Notification 和KVO区别

- KVO提供一种机制,当指定的被观察的对像的属性被修改后,KVO会自动通知响应的观察者,KVC(键值编码)是KVO的基础
- 通知:是一种广播机制,在实践发生的时候,通过通知中心对象,一个对象能够为所有关心这个时间发生的对象发送消息,两者都是观察者模式,不同在于KVO是被观察者直接发送消息给观察者,是对象间的直接交互,通知则是两者都和通知中心对象交互,对象之间不知道彼此
- 本质区别,底层原理不一样.kvo 基于 runtime, 通知则是有个通知中心来进行通知

#### 11.结构体与数组有什么区别?

1. 结构体可以存不同类型的元素,而数组只能存同一类型
2. 结构体类型需要我们自已定义.数组是用别的类型加[元素个数]
3. 结构体内存分配方式很特别,使用对齐原则,不一定是所有元素的字节数和,而数组一定是所有元素的字节数和.
4. 结构体指针可以指针名->结构体元素名(取元素);数组不行.

#### 12. NSDictionary的实现原理是什么？

- 哈希表(NSDictionary 是通过hash表来实现key和value之间的映射和存储的)

#### 13.说一下静态库和动态库之间的区别

- 静态库：以.a 和 .framework为文件后缀名。
- 动态库：以.tbd(之前叫.dylib) 和 .framework 为文件后缀名。
- 静态库：链接时会被完整的复制到可执行文件中，被多次使用就有多份拷贝。
- 动态库：链接时不复制，程序运行时由系统动态加载到内存，系统只加载一次，多个程序共用（如系统的UIKit.framework等），节省内存。
- // 静态库.a 和 framework区别.a 主要是二进制文件,不包含资源,需要自己添加头文件
    .framework 可以包含头文件+资源信息

#### 14.对于 oc 来讲,他的最大优缺点是什么? 如何缺点如何绕过这些不足

优点:

1. OC是C语言的超集, 在C语言基础上增加了面向对象特性, 开发使用起来会方便高效.
2. 分类可以快速扩展类的方法.扩展模块之间相互不影响
3. 运行时特性,动态特性(动态类型,动态绑定,动态加载),提高了编程的灵活性
4. OC可以与C / C++进行混编

缺点:

1. 不支持多继承,多继承可以使用分类,协议,消息转发来弥补
2. 不支持运算符重载
3. 使用动态运行时类型，所有的方法都是函数调用，所以很多编译时优化方法都用不到，如内联函数等，性能低劣。
4. 执行效率比C低,语法怪异

#### 15. OC与 JS交互方式有哪些?

1. 通过拦截URL
2. 使用MessageHandler(WKWebView)
3. JavaScriptCore (UIWebView)
4. 使用三方库WebViewJavascriptBridge,可提供 js 调OC,以及OC掉JS

#### 16. 通过JS调用OC代码(url拦截)一

- 通过拦截url（适用于UIWebView和WKWebView）



```objectivec
- (BOOL)webView:(UIWebView *)webView shouldStartLoadWithRequest:(NSURLRequest *)request navigationType:(UIWebViewNavigationType)navigationType {
    NSString *url = request.URL.absoluteString;
    if ([url rangeOfString:@"需要跳转源生界面的URL判断"].location != NSNotFound) {
        //跳转原生界面
        return NO;
    }
    return YES;
}
```

#### 17. JS调用OC代码(messageHander)二

- 当JS端想传一些数据给iOS.那它们会调用下方方法来发送.
- window.webkit.messageHandlers.<方法名>.postMessage(<数据>)上方代码在JS端写会报错,导致网页后面业务不执行.可使用try-catch执行.
- 那么在OC中的处理方法如下.它是WKScriptMessageHandler的代理方法.name和上方JS中的方法名相对应.



```objectivec
- (void)addScriptMessageHandler:(id <WKScriptMessageHandler>)scriptMessageHandler name:(NSString *)name;
```

#### 18 JS调用OC代码(WebViewJavascriptBridge)三



```objectivec
1. 设置 webViewBridge
_bridge = [WKWebViewJavascriptBridge bridgeForWebView:self.webView];
    [_bridge setWebViewDelegate:self];
2. 注册handler方法,需要和 前段协商好 方法名字,是供 JS调用Native 使用的。
    [_bridge registerHandler:@"scanClick" handler:^(id data, WVJBResponseCallback responseCallback) {
        // OC调用
        NSString *scanResult = @"http://www.baidu.com";
        // js 回调传参
        responseCallback(scanResult);
    }];
3. OC掉用JS
  [_bridge callHandler:@"testJSFunction" data:@"一个字符串" responseCallback:^(id responseData) {
        NSLog(@"调用完JS后的回调：%@",responseData);
    }];
```

#### 19.OC调用JS代码



```objectivec
// 直接运行 使用 
NSString *jsStr = @"执行的JS代码";
[webView stringByEvaluatingJavaScriptFromString:jsStr];

// 使用JavaScriptCore框架
#import <JavaScriptCore/JavaScriptCore.h>  
- (void)webViewDidFinishLoad:(UIWebView *)webView {
    //获取webview中的JS内容
    JSContext *context = [webView valueForKeyPath:@"documentView.webView.mainFrame.javaScriptContext"];
    NSString *runJS = @"执行的JS代码";
    //准备执行的JS代码
    [context evaluateScript:runJS];
}
```

#### 20. 遇到过BAD_ACCESS的错误吗？你是怎样调试的？

BAD_ACCESS 报错属于内存访问错误，会导致程序崩溃，错误的原因是访问了野指针(悬挂指针)。

1. 设置全局断点快速定位问题代码所在行。
2. 开启僵尸对象诊断
3. Analyze分析
4. 重写object的respondsToSelector方法，现实出现EXEC_BAD_ACCESS前访问的最后一个object。
5. Xcode 7 已经集成了BAD_ACCESS捕获功能：Address Sanitizer。 用法如下：在配置中勾选✅Enable Address Sanitizer。

#### 21.什么是函数式编程？链式

- 函数式编程是一种编程模型，他将计算机运算看做是数学中函数的计算，并且避免了状态以及变量的概念。函数式编程就像流水线一样，一顺顺的把问题解决完，从一个起点开始，一个个的调用函数，因为上一个函数有返回值是工具类本身，所以一个函数执行完之后，可以用上一个函数继续调用，有点链式思维在里面。
- Masonry 就是我们最常见的函数式编程,通过对象.方法1().方法2.....



```swift
[view mas_makeConstraints:^(MASConstraintMaker *make){     
make.top.bottom.left.right.equalTo(self.view);
}];
```

链式编程?

- top.bottom.left.right.equalTo(self.view)通过"."语法，将需要执行的代码连续的书写，就叫做链式编程，它使得代码简单易懂。

#### 22. 响应式编程?

- 响应式编程是一种面向数据流和变化传播的编程范式。
- 例如，在命令式编程环境中，a:=b+c表示将表达式的结果赋给a，而之后改变b或c的值不会影响a。但在响应式编程中，a的值会随着b或c的更新而更新。
- Reactive Cocoa就是一个响应式编程的经典作品！

#### 23.Block和Protocol的区别，Block是为了解决什么问题而使用的。

- “代理和block的共同特性是回调机制，不同的是，代理的方法比较多，比较分散,公共接口，方法较多也选择用delegate进行解耦
- 使用block的代码比较集中统一,异步和简单的回调用block更好”

Block是为了解决什么问题而使用的? 个人认为:

- block为了多线程之间调度产生的;
- block 也是一个OC对象,可以当参数传递,使用方便简单,灵活,很少的代码就可以实现代码回调.比协议省很多代码

#### 24.说一下ios代码签名

- 确保从app store下载的app是没被恶意篡改，如果修改则无法安装, 以及验证app开发者身份;

#### 25.什么是app thinning(app 瘦身)

`App Thinning`可以译成“应用瘦身”。指的是App store 和操作系统在安装iOS或者watchOS的 app 的时候通过一些列的优化，尽可能减少安装包的大小，使得 app 以最小的合适的大小被安装到你的设备上。而这个过程包括了三个过程：slicing, bitcode, and on-demand resources。

1. slicing 可以打包对应的 app 资源文件
2. Bitcode 苹果会对包含Bitcode的二进制app进行二次优化，而不需要提交一个新的app版本到app store中。
3. On-Demand Resources 按需加载

#### 26. 如果没有instruments，该如何检测memory leak, zombie object 之类的问题。

- 查看MLeaksFinder源码分析,国内三方

#### 27.字典的工作原理 ？怎100w个中是怎么快速去取value？

1. NSDictionary（字典）是使用 hash表来实现key和value之间的映射和存储的，hash函数设计的好坏影响着数据的查找访问效率。
    - (void)setObject:(id)anObject forKey:(id <NSCopying>)aKey;
2. Objective-C 中的字典 NSDictionary 底层其实是一个哈希表，实际上绝大多数语言中字典都通过哈希表实现，



```cpp
哈希表的本质是一个数组，数组中每一个元素称为一个箱子(bin)，箱子中存放的是键值对。数组长度即箱子数。
哈希表还有一个重要的属性: 负载因子(load factor)，它用来衡量哈希表的 空/满 程度，一定程度上也可以体现查询的效率，计算公式为:
负载因子 = 总键值对数 / 箱子个数
负载因子越大，意味着哈希表越满，越容易导致冲突，性能也就越低。因此，一般来说，当负载因子大于某个常数(可能是 1，或者 0.75 等)时，哈希表将自动扩容。
参考: https://www.jianshu.com/p/88dfc8f405ab
```

#### 28. isEquel和hash的关系

1. isEquel 用于比较2个对象是否相等, 与==(地址比较)不一样, 可以重写isEquel方法,来进行2个对象的比较
2. hash 是一个类方法，任何类都可以调用这个方法，返回的结果是一个NSInteger值(如果两个对象相等，那么他们的hash值一定相等，但是，如果两个对象的哈希值相等是不能一定推出来这两个对象是相等的)

#### 29.isEquel 和 isEquelToString

- isEquel 比较的是2个NSObject的方法,
- isEquelToString是比较2个字符串值是否相等
- isEquel 首先比较2个对象地址,如果相同就返回YES,如果不同,就比较对象类型,以及属性的值,一般重写 isEquel 来比较2个对象

#### 30. iOS 9 以后通知不再需要手动移除

- 通知 NSNotification 在注册者被回收时需要手动移除，是一直以来的使用准则。 原因是在 MRC 时代，通知中心持有的是注册者的 unsafe_unretained 指针，在注册者被回收时若不对通知进行手动移除，则指针指向被回收的内存区域，变为野指针。此时发送通知会造成 crash 。 而在 iOS 9 以后，通知中心持有的是注册者的 weak 指针，这时即使不对通知进行手动移除，指针也会在注册者被回收后自动置空。因为向空指针发送消息是不会有问题的。

#### 31. 如何对 NSMutableArray 进行 KVO

- 一般情况下只有通过调用 set 方法对值进行改变才会触发 KVO。但是在调用NSMutableArray的 addObject或removeObject 系列方法时，并不会触发它的 set 方法。所以为了实现NSMutableArray的 KVO，官方为我们提供了如下方法:



```objectivec
@property (nonatomic, strong) NSMutableArray *arr;

//添加元素操作
[[self mutableArrayValueForKey:@"arr"] addObject:item];
//移除元素操作
[[self mutableArrayValueForKey:@"arr"] removeObjectAtIndex:0];
```

#### 32. 编译过程做了哪些事情；



```undefined
* Objective,Swift都是编译语言。编译语言在执行的时候，必须先通过编译器生成机器码，机器码可以直接在CPU上执行，所以执行效率较高。Objective,Swift二者的编译都是依赖于Clang + LLVM. OC和Swift因为原理上大同小异，知道一个即可！
* iOS编译 不管是OC还是Swift，都是采用Clang作为编译器前端,LLVM(Low level vritual machine)作为编译器后端。
* 编译器前端 :编译器前端的任务是进行：语法分析，语义分析，生成中间代码(intermediate representation )。在这个过程中，会进行类型检查，如果发现错误或者警告会标注出来在哪一行
* 编译器后端 :编译器后端会进行机器无关的代码优化，生成机器语言，并且进行机器相关的代码优化。LVVM优化器会进行BitCode的生成，链接期优化等等,LLVM机器码生成器会针对不同的架构，比如arm64等生成不同的机器码。
```

#### 33. 容错处理你们一般是注意哪些？

在团队协作开发当中，由于每个团队成员的水平不一，很难控制代码的质量，保证代码的健壮性，经常会发生由于后台返回异常数据造成app崩溃闪退的情况，为了避免这样的情况项目中做一些容错处理，显得格外重要，极大程度上降低了因为数据容错不到位产生崩溃闪退的概率。



```objectivec
例如：
1.字典
2.数组；
3.野指针；
4.NSNull
等~
// AvoidCrash github 三方不错
```

#### 34. 项目开始容错处理没做？如何防止拦截潜在的崩溃？

例：
1、category给类添加方法用来替换掉原本存在潜在崩溃的方法。
2、利用runtime方法交换技术，将系统方法替换成类添加的新方法。
3、利用异常的捕获来防止程序的崩溃，并且进行相应的处理。
4、使用 @try__Catch__方法进行拦截

总结：
1、不要过分相信服务器返回的数据会永远的正确。
2、在对数据处理上，要进行容错处理，进行相应判断之后再处理数据，这是一个良好的编程习惯。

#### 35.@try @catch异常机制

Objective-C 异常机制 :
-- 作用 : 开发者将引发异常的代码放在 @try 代码块中, 程序出现异常 使用 @catch 代码块进行捕捉;
-- 每个代码块作用 : @try 代码块存放可能出现异常的代码, @catch 代码块 异常处理逻辑, @finally 代码块回收资源;
-- 语法示例 :



```python
try{
   //..执行的代码，其中可能有异常。一旦发现异常，则立即跳到catch执行。否则不会执行catch里面的内容
}catch(){
  //...除非try里面执行代码发生了异常，否则这里的代码不会执行
}finally{
  //..不管什么情况都会执行，包括try catch 里面用了return ,可以理解为只要执行了try或者catch，就一定会执行 finally
}
可以用于查找 bug,或者调试,防止崩溃使用
```

#### 36.单元测试是什么?

单元测试（unit testing），是指对软件中的最小可测试单元进行检查和验证。



```objectivec
1. 单元测试可以进行 逻辑测试/异步测试/性能测试
2. 单元测试是以代码测试代码。不是靠 NSLog 来测试,而是使用断言来测试的，提前预判条件必须满足。XCTAssert(条件, 不满足条件的描述)
3. 可以在单元测试类中编写单独的测试用例方法。这些方法与普通的方法类似，但是方法名称必须以 test 开头，且不能有参数，不然不会识别为测试方法。
4. 不是所有的方法都需要测试。例如私有方法不需要测试，只有暴露在 .h 中的方法需要测试。
一般而言，代码的覆盖度大概在 50% ~ 70%。从 github 上得知：YYModel 测试覆盖度为 83%，AFNetworking 测试覆盖度为 77%，两者都是比较高的。
 
总结: 单元测试可以根据项目需要,针对一些关键业务,编写一些测试用例,可以方便的排查业务逻辑可能出现的问题.在后续改动时候也可以方便的测试等等.
```

#### 37. 一个上线的项目，知道这个方法可能会出问题，在不破坏改方法前提下，怎么搞？

1. 做一些容错处理,防止崩溃
2. 加一些日志收集,收集问题再具体分析
3. try_catch

#### 38.Xcode编译器发展简史



```css
Xcode3 以前：  GCC；
Xcode3：             增加LLVM，GCC(前端) + LLVM(后端)；
Xcode4.2：           出现Clang - LLVM 3.0成为默认编译器；
Xcode4.6：         LLVM 升级到4.2版本；
Xcode5：         GCC被废弃，新的编译器是LLVM 5.0，从GCC过渡到Clang-LLVM的时代正式完成
```

#### 39.代码从 Git 上拉下来到生成 .ipa 都有哪些过程，期间都生成了什么文件。

1. git clone 远程地址到本地
2. pod 三方集成
3. 配置证书信息,签名
4. 打包 ipa

#### 40.Pods的原理

- 简单理解：快速的搜索多第三方框架，然后自动集成多工程里面。并编译成一个libPod.a的静态库给我们的项目用。

#### 41. 函数指针和 Block区别

相同点:

- 二者都可以看成是一个代码片段。
- 函数指针类型和 Block 类型都可以作为变量和函数参数的类型

不同点:

- 函数指针只能指向预先定义好的函数代码块，函数地址是在编译链接时就已经确定好的。从内存的角度看，函数指针只不过是指向代码区的一段可执行代码
- block 本质是 OC对象，是 NSObject的子类，是程序运行过程中在栈内存动态创建的对象，可以向其发送copy消息将block对象拷贝到堆内存，以延长其生命周期。

#### 42. 符号表

iOS 构建时产生的符号表，是内存地址、函数名、文件名和行号的映射表。格式大概是：



```xml
<起始地址> <结束地址> <函数> [<文件名:行号>]
```

Crash 时的堆栈信息，全是二进制的地址信息。如果利用这些二进制的地址信息来定位问题是不可能的，因此我们需要将这些二进制的地址信息还原成源代码种的函数以及行号，这时候符号表就起作用了。利用符号表将原始的 Crash 的二进制堆栈信息还原成包含行号的源代码文件信息，可以快速定位问题。iOS 中的符号表文件(DSYM) 是在编译源代码后，处理完 Asset Catalog 资源和 info.plist 文件后开始生成，生成符号表文件(DSYM)之后，再进行后续的链接、打包、签名、校验等步骤。

#### 43. 应用瘦身(Thinning)

App Thinning“应用瘦身”,iOS9之后发布的新特性。它能对App store 和操作系统在安装iOS app 的时候通过一些列的优化，尽可能减少安装包的大小，使得 app 以最小的合适的大小被安装到你的设备上。而这个过程包括了三个过程：
`slicing, bitcode, on-demand resources，`



```undefined
* slicing
appStore 会根据用户的设备型号创建相应的应用变体,这些变体只包含可执行的结构和资源必要部分,不需要用户下载完整的安装包

* bitcode
bitcode系统会对编译后的二进制文件进行二次优化, 使用最新的编译器自动编译app并且针对特定架构进行优化。不会下载应用针对不同架构的优化，而仅下载与特定设备相关的优化，使得下载量更小，

* On Demand Resources
按需加载资源是在 app 第一次安装后可下载的文件。举例说明，当玩家解锁游戏的特定关卡后可以下载新关卡（和这个关卡相关的特定内容）。此外，玩家已经通过的关卡可以被移除以便节约设备上的存储空间。
```

#### 44.埋点处理

> 埋点是什么? 用户行为统计，俗称埋点

埋点分为两种：

- 页面统计，即在进入页面和离开页面的时候埋点，统计停留页面时长
- 交互事件统计

无痕埋点(自动埋点)解决方案:
技术原理：Method-Swizzling

对于一个给定的事件，UIControl会调用sendAction:to:forEvent:来将行为消息转发到UIApplication对象，再由UIApplication对象调用其sendAction:to:fromSender:forEvent:方法来将消息分发到指定的target上，那么，我们写一个UIControl的类别，通过替换它的sendAction:to:forEvent:方法，结合本地配置的埋点json或者plist文件(若埋点需要额外的参数，需要给UIControl的类别通过Runtime添加属性)，便可以实现自动埋点的功能。

参考链接:
https://www.jianshu.com/p/b8a67c4acfb3
https://www.jianshu.com/p/ae8d45e10ac5

#### 45.说一下iOS 中的APNS,远程推送原理?

`Apple push Notification Service`,简称 `APNS`,是苹果的远程消息推送,原理如下:

1. iOS 系统向APNS服务器请求手机端的`deviceToken`
2. App 接收到手机端的 deviceToken,然后传给 App 对应的服务器.
3. App 服务端需要发送推送消息时, 需要先通过 APNS 服务器
4. 然后根据对应的 deviceToken 发送给对应的手机

#### 下一篇入口:



作者：Leon_520
链接：https://www.jianshu.com/p/2057f9663d78
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。