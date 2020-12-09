### App的启动过程
1. APP的启动分为两种
    - 冷启动：从零开始启动APP
    - 热启动：APP已经在内存中，在后台存活着，再次点击图标启动APP
    <!-- more -->

2. App的启动过程
    - 解析info.plist
        - 加载相关信息，例如闪屏
        - 沙盒建立，权限检查
    - Mach-O加载
        - 如果是胖二进制文件，寻找合适当前CPU类别的部分
        - 加载所有依赖的Mach-O文件（递归调用Mach-O加载的方法），进行rebase指针调整和bind符号绑定（dyld完成，运行时开始）
        - 定位内部、外部指针引用，例如字符串、函数等
        - 执行声明为`__attribute__((constructor))`的C函数
        - 加载类扩展（Category）中的方法
        - C++静态对象加载、调用ObjC的`+load`方法
        - 进行各种objc结构的初始化（注册Objc类、初始化对象等等）
        - 调用C++静态初始化器和__attribute__((constructor))装饰的函数
    - 程序执行
        - 调用`main()`
        - 调用`UIApplicationMain()`
        - 调用`application:didFinishLaunchingWithOptions:`

3. 如何测量启动过程耗时
    1.  main()函数之前
        - 通过添加环境变量可以打印出APP的启动时间分析（Edit scheme -> Run -> Arguments）
            - DYLD_PRINT_STATISTICS设置为1
            - 如果需要更详细的信息，那就将DYLD_PRINT_STATISTICS_DETAILS设置为1
        ```
        Total pre-main time: 3.0 seconds (100.0%)
                dylib loading time: 1.4 seconds (47.3%)
                rebase/binding time: 297.30 milliseconds (9.7%)
                    ObjC setup time: 245.42 milliseconds (8.0%)
                initializer time: 1.0 seconds (34.9%)
                slowest intializers :
                    libSystem.B.dylib :  11.39 milliseconds (0.3%)
                libglInterpose.dylib : 489.71 milliseconds (15.9%)
                        AFNetworking : 318.57 milliseconds (10.3%)
                        SJZCoreModule : 317.69 milliseconds (10.3%)
                        LYEmptyView : 131.32 milliseconds (4.2%)
        ```
        如何解读
        1). main()函数之前总共使用了3s
        2). 在3s中，加载动态库用了1.4s，指针重定位使用了297.30ms，ObjC类初始化使用了245.42ms，各种初始化使用了1s。
        3). 在初始化耗费的1s中，用时最多的五个初始化是libSystem.B.dylib、libglInterpose.dylib、AFNetworking、SJZCoreModule、LYEmptyView

    2. main()函数之后
        
    - 指的是从 main() 函数执行开始，到 appDelegate 的 didFinishLaunchingWithOptions 方法里首屏渲染相关方法执行完成。
    
4. 影响启动性能的因素
    每个库本身都有依赖关系，苹果公司建议使用更少的动态库，并建议在使用动态库的数量较多时，尽量将多个动态库进行合并。数量上，苹果公司最多可以支持6个非系统动态库合并为一个
    +load()方法里的内容可以放到首屏渲染完成后再执行，或使用+initialize()方法替换掉。因为，在一个 +load() 方法里，进行运行时方法替换操作会带来 4 毫秒的消耗。不要小看这 4 毫秒，积少成多，执行 +load() 方法对启动速度的影响会越来越大。

    - mian()函数之前耗时的影响因素
        - 动态库加载越多，启动越慢
        - ObjC类越多，启动越慢
        - C的constructor函数越多，启动越慢
        - ObjC的+load越多，启动越慢


    > 优化
    > 1. 减少动态库、合并一些动态库（定期清理不必要的动态库）
    > 2. 减少Objc类、分类的数量、减少Selector数量（定期清理不必要的类、分类）
    > 3. 合并功能类似的类和扩展（Category）
    > 4. 减少C++虚函数数量
    > 5. Swift尽量使用struct
    > 6. 用+initialize方法和dispatch_once取代所有的__attribute__((constructor))、C++静态构造器、ObjC的+load方法
    > 7. 压缩资源图片


    - main() 函数之后耗时的影响因素
        - 执行mian() 函数耗时
        - 执行application:didFinishLaunchingWithOptions:耗时
        - rootViewController及其childViewController的加载、view及其subviews的加载
    
    ```
    - (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
        SJZTabBarController * tabBar = [[SJZTabBarController alloc] init];
    
        ViewController * vc1 = [[ViewController alloc] init];
        UINavigationController * nav1 = [[UINavigationController alloc] initWithRootViewController:vc1];
        nav1.tabBarItem = [[UITabBarItem alloc] initWithTitle:@"首页" image:nil selectedImage:nil];
        [tabBar addChildViewController:nav1];
        
        ViewController * vc2 = [[ViewController alloc] init];
        UINavigationController * nav2 = [[UINavigationController alloc] initWithRootViewController:vc2];
        nav2.tabBarItem = [[UITabBarItem alloc] initWithTitle:@"首页" image:nil selectedImage:nil];
        [tabBar addChildViewController:nav2];


​        
        UIWindow * window = [[UIWindow alloc] initWithFrame:UIScreen.mainScreen.bounds];
        window.rootViewController = tabBar;
        [window makeKeyAndVisible];
        self.window = window;
        
        return YES;
     }
    ```
    那么`[SJZTabBarController viewDidLoad]`  `[ViewController viewDidLoad]`  `[AppDelegate application: didFinishLaunchingWithOptions:]` 调用顺序是怎么样的？
    > 执行顺序为：
    > 1. didFinishLaunchingWithOptions 开始执行 
    > 2. 开始加载 SJZTabBarController 的 viewDidLoad
    >  3. didFinishLaunchingWithOptions 跑完了
    >  4. 开始加载 ViewController 的 viewDidLoad, 然后执行一堆初始化的操作
    
    但如果在didFinishLaunchingWithOptions执行期间，操作了ViewController的view 以及 subView（初始化好的）的话
    > 执行顺序将变为:
    > 1. didFinishLaunchingWithOptions 开始执行 
    > 2. 开始加载 SJZTabBarController 的 viewDidLoad
    > 3. 开始加载 ViewController 的 viewDidLoad, 然后执行一堆初始化的操作
    > 4. didFinishLaunchingWithOptions 跑完了
    
    如果将界面的初始化、网络请求、数据解析、视图渲染等操作都放到了viewDidLoad，每次启动APP的时候，在用户看到第一个页面之前，APP将执行完以上所有的操作，才会进入视图渲染阶段。
    
    所以，main() 函数开始执行后到首屏渲染完成前只处理首屏相关的业务，其他非首屏业务的初始化、监听注册、配置文件读取等都放到首屏渲染完成后去做。
    - 必要的配置信息，如埋点、日志等放到 didFinishLaunchingWithOptions中
    - 初始化首页相关的代码，放到首页的viewDidLoad
    - 其他与首页无关的SDK配置，网络请求，监听注册、配置文件读取等，放到首页viewDidAppear中
    
    > 优化
    > 1. 必须第一时间启动，仍然把它留在 didFinishLaunchingWithOptions 里启动。
    > 2. 某些在用户进入APP主体之前必须要加载的功能，放到首页viewDidLoad 或 广告页的viewDidLoad
    > 3. 非必须的，我们放到第一个界面的viewDidAppear

### 优化的目标
> 苹果给出的标准：
> 应该在400ms内完成main()函数之前的加载
> 整体过程耗时不能超过20秒，否则系统会kill掉进程，App启动失败

这里要根据自己App的需求和规模，我们的目标应该有所取舍

### 总结
- 重新梳理架构，减少动态库（合并动态库）、ObjC类的数目，减少Category的数目
- 定期扫描不再使用的动态库、类、函数
- 用dispatchonce()代替所有的__attribute__((constructor))函数、C++静态对象初始化、ObjC的+load
- 控制 C++ 全局变量的数量。
- 

### 参考(其实大部分照抄😎)
[iOS App 启动性能优化](https://mp.weixin.qq.com/s?__biz=MzA3NTYzODYzMg==&mid=2653579242&idx=1&sn=8f2313711f96a62e7a80d63851653639&chksm=84b3b5edb3c43cfb08e30f2affb1895e8bac8c5c433a50e74e18cda794dcc5bd53287ba69025&mpshare=1&scene=1&srcid=081075Vt9sYPaGgAWwb7xd1x&key=4b95006583a3cb388791057645bf19a825b73affa9d3c1303dbc0040c75548ef548be21acce6a577731a08112119a29dfa75505399bba67497ad729187c6a98469674924c7b447788c7370f6c2003fb4&ascene=0&uin=NDA2NTgwNjc1&devicetype=iMac16%2C2+OSX+OSX+10.12.6+build(16G29)&version=12020110&nettype=WIFI&fontScale=100&pass_ticket=IDZVtt6EyfPD9ZLcACRVJZYH8WaaMPtT%2BF3nfv7yZUQBCMKM4H1rDCbevGd7bXoG)
[优化 App 的启动时间实践 iOS](https://www.jianshu.com/p/0858878e331f)












