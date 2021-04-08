#### 1.ViewController 生命周期

**单个viewController的生命周期**



```css
- initWithCoder:(NSCoder *)aDecoder：（如果使用storyboard或者xib）
- loadView：加载view
- viewDidLoad：view加载完毕
- viewWillAppear：控制器的view将要显示
- viewWillLayoutSubviews：控制器的view将要布局子控件
- viewDidLayoutSubviews：控制器的view布局子控件完成  
- viewDidAppear:控制器的view完全显示
- viewWillDisappear：控制器的view即将消失的时候
- viewDidDisappear：控制器的view完全消失的时候
- dealloc 控制器销毁
```

**两个控制器AB进行跳转调用顺序:**

A控制器先展示调用



```css
 - [ViewControllerA loadView]
 - [ViewControllerA viewWillAppear:]
 - [ViewControllerA viewWillLayoutSubviews]
 - [ViewControllerA viewDidLayoutSubviews]
 - [ViewControllerA viewDidAppear:]
```

B控制器跳转调用顺序



```css
 - [ViewControllerB loadView]
 - [ViewControllerB viewDidLoad]
 - [ViewControllerA viewWillDisappear:]
 - [ViewControllerB viewWillAppear:]
 - [ViewControllerB viewWillLayoutSubviews]
 - [ViewControllerB viewDidLayoutSubviews]
 - [ViewControllerA viewDidDisappear:]
 - [ViewControllerB viewDidAppear:]
```

B控制器返回A顺序



```css
 - [ViewControllerB viewWillDisappear:]
 - [ViewControllerA viewWillAppear:]
 - [ViewControllerB viewDidDisappear:]
 - [ViewControllerA viewDidAppear:]
```

#### 2. CALayer 和 UIView

- UIView 和 CALayer都是 UI 操作的对象
- UIView是 CALayer用于交互的对象,UIView是CALayer的delegate ,UIView是UIResponder的子类,其中提供了很多CALayer所没有的交互接口,主要负责处理用户触发的各种操作;
- CALayer主要负责绘制,在图像和动画上渲染性能更好;

#### 3.UIView 的frame，bounds，center

- frame: 描述当前界面元素在其父界面元素中的位置和大小。
- bounds: 描述当前界面元素在其自身坐标系统中的位置和大小。
- center: 描述当前界面元素的中心点在其父界面元素中的位置.

#### 4.CALayer的frame，bounds，anchorPoint,position

- frame：与view中的frame概念相同，（x,y）subLayer左上角相对于supLayer坐标系的位置关系；width, height表示subLayer的宽度和高度。
- bounds：与view中的bounds概念相同，（x,y）subLayer左上角相对于自身坐标系的关系；width, height表示subLayer的宽度和高度。
- anchorPoint(锚点)：锚点在自身坐标系中的相对位置，默认值为（0.5，0.5），左上角为（0，0），右下角为（1，1），其他位置以此类推；锚点都是对于自身来讲的. 确定自身的锚点,通常用于做相对的tranform变换.当然也可以用来确定位置；
- position：锚点在supLayer坐标系中的位置；

#### 5.iOS 为什么必须在主线程中操作UI

- UIKit不是线程安全的(多个线程访问修改,可能一个线程已经释放了,另一个线程会访问,以及资源抢夺问题等)
- 主线程上默认是开始 runloop 的,子线程没有 runloop 也无法监听一些事件,手势刷新UI等操作
- 在子线程更新UI可能会无效,也可能会崩溃

#### 6. 如何处理UITableVier 中Cell 动态计算高度的问题，都有哪些方案

- 你的Cell要使用AutoLayout来布局约束这是必须的；设置tableview的estimatedRowHeight为一个非零值，这个属性是设置一个预估的高度值，不用太精确。 设置tableview的rowHeight属性为UITableViewAutomaticDimension
- 第三方 UITableView+FDTemplateLayoutCell(计算布局高度缓存的)
- 手动计算每个控件的 高度并相加,最后缓存高度

#### 7. AutoLayout 中的优先级是什么?

AutoLayout中添加的约束也有优先级,优先级的数值是1~1000

- 一种情况是我们经常添加的各种约束,默认的优先级是1000，也就是最高级别，条件允许的话系统会满足我们所有的约束需求。
- 另外一种情况就是固有约束(intinsic content size)
- Content Hugging Priority 抗拉伸优先级值越小，越容易被拉伸
- Content Compression Resistance 抗压缩优先级 优先级越小，越先被压缩

#### 8. 怎么高效的实现控件的圆角效果?

//直接对图片进行重绘 (使用Core Graphics),实际开发加异步处理,也可以给 SDWebImage 也做扩展;



```objectivec
 - (UIImage *)imageWithCornerRadius:(CGFloat)radius {
        CGRect rect = (CGRect){0.f, 0.f, self.size};
        UIGraphicsBeginImageContextWithOptions(self.size, NO, UIScreen.mainScreen.scale);
        CGContextAddPath(UIGraphicsGetCurrentContext(), [UIBezierPath bezierPathWithRoundedRect:rect cornerRadius:radius].CGPath);
        CGContextClip(UIGraphicsGetCurrentContext());
        [self drawInRect:rect];
        UIImage *image = UIGraphicsGetImageFromCurrentImageContext();
        UIGraphicsEndImageContext();
        return image;
    }
```

// 利用CAShapeLayer圆角,替换原本的layer,达到圆角效果



```objectivec
 UIBezierPath *maskPath = [UIBezierPath bezierPathWithRoundedRect:self.bounds byRoundingCorners:UIRectCornerAllCorners cornerRadii:self.bounds.size];
    CAShapeLayer *maskLayer = [[CAShapeLayer alloc]init];
    maskLayer.frame = self.bounds;
    maskLayer.path = maskPath.CGPath;
    self.layer.mask = maskLayer;
```

#### 9.CALayer如何添加点击事件

1. 通过 touchesBegan: withEvent 方法,监听屏幕点击事件,在这个方法中通过 convertPoint 找到点击位置,进行判断,如果点击了 layer 视图内坐标,就触发点击事件
2. 通过 hitTest方法找到包含坐标系的 layer 视图



```objectivec
- (void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event{
//  方法一,通过 convertPoint 转为为 layer 坐标系进行判断
    CGPoint point = [[touches anyObject]   locationInView:self.view];
    CGPoint redPoint = [self.redLayer  convertPoint:point fromLayer:self.view.layer];
    if ([self.redLayer containsPoint:redPoint]) {
    NSLog(@"点击了calayer");
    }
//  方法二 通过 hitTest 返回包含坐标系的 layer 视图
    CGPoint point1 = [[touches anyObject] locationInView:self.view];
    CALayer *layer = [self.view.layer hitTest:point1];
    if (layer == self.redLayer) {
       NSLog(@"点击了calayer");
    }
}
```

#### 10.介绍下layoutSubview和drawRect

- layoutSubviews调用情况:
    - init初始化UIView不会触发调用
    - addSubview会触发调用
    - 改变view的width和height的时候回触发调用
    - 一个UIScrollView滚动会触发调用
    - 旋转screen会触发调用
    - 改变一个UIView大小的时候会触发superView的layoutSubviews事件
    - 直接调用setLayoutSubviews会触发调用
    - -(void)viewWillAppear:(BOOL)animated会触发一次调用
    - -(void)viewDidAppear:(BOOL)animated 看情况，可能有调用
- drawRect调用情况
    - 如果UIView没有设置frame大小，直接导致drawRect不能被自动调用。
    - drawRect在loadView和viewDidLoad这两个方法之后调用
    - 调用sizeToFit后自动调用drawRect
    - 通过设置contentMode值为UIViewContentModeRedraw。那么每次设置或者更改frame自动调用drawRect。
    - 直接调用setNeedsDisplay或者setNeedsDisplayInRect会触发调用

#### 11. layoutIfNeeded , layoutSubViews和 setNeedsLayout区别?

- layoutIfNeeded 方法一点被调用,主线程会立即强制重新布局,它会从当前视图开始,一直到完成所有子视图的布局
- layoutSubViews 用来自定义视图尺寸,他是系统自动调用的,开发者不能手动调用,可以重写改方法,让系统在调整布局时候按照我们希望的方式进行布局.这个方法在旋转屏幕,滑动或者触摸屏幕,修改子视图时候被触发.
- setNeedsLayout 和 layoutIfNeeded相似,唯一不同的是他不会立即强制视图重新布局,而是在下一个布局周期才会触发更新.他主要用于多个视图布局先后更新的场景;

#### 12.假如Controller太臃肿，如何优化?

1. 将网络请求抽象到单独的类中,方便在基类中处理公共逻辑；方便在基类中处理缓存逻辑，以及其它一些公共逻辑；方便做对象的持久化。
2. 将界面的封装抽象到专门的类中, 构造专门的 UIView 的子类，来负责这些控件的拼装。这是最彻底和优雅的方式，不过稍微麻烦一些的是，你需要把这些控件的事件回调先接管，再都一一暴露回 Controller。
3. 构造 ViewModel, 借鉴MVVM。具体做法就是将 ViewController 给 View 传递数据这个过程，抽象成构造 ViewModel 的过程。
4. 专门构造存储类,专门来处理本地数据的存取。
5. 整合常量

#### 13.程序启动过程

**main 函数执行前:**

1. 首先当程序启动时，系统会读取程序的可执行文件（mach-o）, 从里面获取动态加载器(dylb)的路径;
2. 加载dylb, dylb会初始化运行环境，配合ImageLoader将二进制文件加载到内存中去;
3. 动态链接依赖库, 初始化依赖库，初始化 runtime;
4. runtime 会对项目中的所有类进行类结构初始化，调用所有的 load 方法;
5. 最后 dylb 会返回 main 函数地址，main 函数被调用，进入程序入口

**main 函数执行后:**

1. 内部会调用 UIApplicationMain 函数，创建一个UIApplication对象和它的代理，就是我们项目中的 Appdelegate 类
2. 开启一个事件循环(main runloop), 监听系统事件
3. 程序启动完毕时，通知代理Appdelegate, 调用 didFinishLaunching 代理方法，在这里会创建 UIWindow,设置它的rootViewController,
4. 最后调用 self.window makeKeyAndVisable显示窗口

#### 14.渲染以及图像显示原理过程?

- 每一个UIView都有一个layer，每一个layer都有个content，这个content指向的是一块缓存，叫做backing store。
- UIView的绘制和渲染是两个过程，当UIView被绘制时，CPU执行drawRect，通过context将数据写入backing store。
- 当backing store写完后，通过render server交给GPU去渲染，将backing store中的bitmap数据显示在屏幕上。
- 说到底CPU就是做绘制的操作把内容放到缓存里，GPU负责从缓存里读取数据然后渲染到屏幕上。

#### 15.离屏渲染是什么

- 离屏渲染，指的是 GPU （图形处理器）在当前屏幕缓冲区以外新开辟一个缓冲区进行渲染操作。为什么离屏这么耗时？原因主要有创建缓冲区和上下文切换。创建新的缓冲区代价都不算大，付出最大代价的是上下文切换。
- GPU屏幕渲染有两种方式:
    - On-Screen Rendering (当前屏幕渲染) 指的是GPU的渲染操作是在当前用于显示的屏幕缓冲区进行。
    - Off-Screen Rendering (离屏渲染) 指的是在GPU在当前屏幕缓冲区以外开辟一个缓冲区进行渲染操作。

#### 16. 那些情况会造成离屏渲染?

1. 为图层设置遮罩（layer.mask）
2. 将图层的layer.masksToBounds / view.clipsToBounds属性设置为true
3. 将图层layer.allowsGroupOpacity属性设置为YES和layer.opacity小于1.0
4. 为图层设置阴影（layer.shadow *）。
5. 为图层设置layer.shouldRasterize=true
6. 具有layer.cornerRadius，layer.edgeAntialiasingMask，layer.allowsEdgeAntialiasing的图层
7. 文本（任何种类，包括UILabel，CATextLayer，Core Text等）。

#### 17.手机适配方案

1. 使用宏,针对不同的设备抽取导航,状态栏,以及 tabbar 高度信息
2. 宽高等比适配(X 的特殊处理)
3. 图片美工需要提供@2x,@3x进行适配
4. 字体根据屏幕大小适配
5. 权限针对不同系统进行适配
6. api 适配

#### 18. 什么是懒加载?

懒加载 也叫做 `延迟加载`,他的核心思想就是把对象的实例化尽量延迟,在需要使用的时候才会初始化,这样做的好处可以减轻大量对象实例化对资源的消耗.
另外懒加载把对象的实例化代码抽取出来独立出来,提高代码的可读性,以便代码更好的被组织

#### 19.什么是响应者链?

响应者链是用于确定`事件响应`的一种机制, 事件主要是指触摸事件(touch Event),该机制与UIKIT中的UIResponder类密切相关,响应触摸事件的必须是继承自UIResponder的类,比如UIView 和UIViewController

一个事件响应者的完成主要分为2个过程: hitTest方法命中视图和响应者链确定响应者; hitTest的调用顺序是从UIWindow开始，对视图的每个子视图依次调用，也可以说是从显示最上面到最下面,直到找命中者; 然后命中者视图沿着响应者链往上传递寻找真正的响应者.

**事件传递过程**

- 当我们触控手机屏幕时系统便会将这一操作封装成一个UIEvent放到事件队列里面，然后Application从事件队列取出这个事件，接着需要找到命中者, 所以开始的第一步应该是找到命中者, 那么又是如何找到的呢？那就不得不引出UIView的2个方法：



```objectivec
// 返回能够相应该事件的视图
-(UIView *)hitTest:(CGPoint)point withEvent:(UIEvent *)event  
// 查看点击的位置是否在这个视图上
-(BOOL)pointInside:(CGPoint)point withEvent:(UIEvent *)event 
```

- 寻找事件的命中视图是通过对视图调用hitTest和pointInside完成的 hitTest的调用顺序是从UIWindow开始，对视图的每个子视图依次调用，也可以说是从显示最上面到最下面 遍历直到找到命中视图;

**响应链传递**

- 找到命中者,任务并未完成,因为命中者不一定是事件的响应者,所谓响应就是开发中为事件绑定一个触发函数,事件发生后,执行响应函数里的代码
- 找到命中视图后事件会从此视图开始沿着响应链nextResponder传递，直到找到处理事件的响应视图,如果没有处理的事件会被丢弃。
- 如果视图有父视图则nextResponder指向父视图，如果是根视图则指向控制器，最终指向AppDelegate, 他们都是通过重写nextResponder来实现。
- 自下往上查找

**无法响应的事件情况**

- Alpha=0、
- 子视图超出父视图的情况、
- userInteractionEnabled=NO、
- hidden=YES

#### 20.动画相关有哪几种方式?

- UIView animation --- 可以实现基于 UIView 的简单动画,他是CALayer Animation封装,可以实现移动,旋转,变色,缩放等基本操作,他实现的动画无法回撤,暂停,与手势交互,常用方法如下



```objectivec
    [UIView animateWithDuration: 10 animations:^{
        // 动画操作
    }];
```

- UIViewPropertyAnimator --- 是 iOS10 中引入的处理交互式动画接口,他是基于 UIView 实现的, 用法同 UIView animation比较类似,增加了一些新的属性以及方法;

- CALayer Animation --- 是在更底层CALayer 上的动画接口, 他可以实现各种复杂的动画效果, 实现的动画可以回撤,暂停与手势交互等,常用的类有以下几个:



```objectivec
1. CABasicAnimation——基本动画
2. CAKeyframeAnimation——关键帧动画
     与CABasicAnimation的区别是：CABasicAnimation：只能从一个数值（fromValue）变到另一个数值（toValue）CAKeyframeAnimation：会使用一个NSArray保存这些数值
3. CAAnimationGroup——动画组
    动画组，是CAAnimation的子类，可以保存一组动画对象，将CAAnimationGroup对象加入层后，组中所有动画对象可以同时并发运行
4. 转场动画——CATransition
    CATransition是CAAnimation的子类，用于做转场动画，能够为层提供移出屏幕和移入屏幕的动画效果。iOS比Mac OS X的转场动画效果少一点。
```

#### 21. UIScrollView 原理

- UIScrollView继承自UIView，内部有一个 UIPanGestureRecongnizer手势。 frame 是相对父视图坐标系来决定自己的位置和大小，而bounds是相对于自身坐标系的位置和尺寸的。改视图 bounds 的 origin 视图本身没有发生变化，但是它的子视图的位置却发生了变化，因为 bounds 的 origin 值是基于自身的坐标系，当自身坐标系的位置被改变了，里面的子视图肯定得变化， bounds 和 panGestureRecognize 是实现 UIScrollView 滑动效果的关键技术点。

#### 23. loadView 的作用?

- loadView 用来自定义 view，只要实现了这个方法，其他通过 xib 或 storyboard 创 建的 view 都不会被加载

#### 24. IBOutlet 连出来的视图属性为什么可以被设 置成 weak?

- 因为父控件的 subViews 数组已经对它有一个强引用

#### 25.请简述 UITableViewCell的复用机制

- 每次创建 cell 的时候通过 dequeueReusableCellWithIdentifier:方法创建 cell，它先到 缓存池中找指定标识的 cell，如果没有就直接返回 nil
- 如果没有找到指定标识的 cell，那么会通过 initWithStyle:reuseIdentifier:创建一个 cell
- 当 cell 离开界面就会被放到缓存池中，以供下次复用

#### 26.使用 drawRect 有什么影响?

- drawRect 方法依赖 Core Graphics 框架来进行自定义的绘制
- 缺点:它处理 touch 事件时每次按钮被点击后，都会用 setNeddsDisplay 进行强制
- 重绘;而且不止一次，每次单点事件触发两次执行。这样的话从性能的角度来 说，对 CPU 和内存来说都是欠佳的。特别是如果在我们的界面上有多个这样的 UIButton 实例，那就会很糟糕了
- 这个方法的调用机制也是非常特别. 当你调用 setNeedsDisplay 方法时, UIKit 将会 把当前图层标记为 dirty,但还是会显示原来的内容,直到下一次的视图渲染周期,才会 将标记为 dirty 的图层重新建立 Core Graphics 上下文,然后将内存中的数据恢复出 来, 再使用 CGContextRef 进行绘制

#### 27.能否在一个控制器 嵌入2个 TableViewController 控制器

- 可以, 控制器可以添加子控制器

#### 28.一个 TableView 是否可以关联2个不同的dataSource?

- 可以关联多个数据源,重点只要处理好数据源和 tableView 的对接工作即可

#### 29.masksToBounds 和clipsToBounds

- masksToBounds 是指子 layer 在超出父 layer时是否被裁剪,YES表示参见,NO 表示不裁剪, 默认是NO
    clipsToBounds 是指子 View 在超出父 View时是否被裁剪

#### 30.tintColor 是什么?

- tintColor 是 ios7以后 UIView类添加的一个新属性,用于改变应用的主色调,默认是 nil

#### 31.imageNamed 和 imageWithContentsOfFile区别

- imageNamed 会自动缓存新加载的图片,并切重复利用缓存图片,一般用于App 内经常使用的尺寸不大的图片
- imageWithContentsOfFile 根据路径加载没有 取缓存和缓存的过程,用于一些大图,使用完毕会释放内存

#### 32.View 和 View 传值方式有哪些

1. 通过视图类对外提供的属性直接传值
2. 通过代理传值
3. 通过通知传值
4. 通过 Block 传值
5. 通过NSUserDefault, 不建议

#### 33. 为什么iOS提供 UIView 和CAlayer 两个个平行的层级结构

- UIView 和CAlayer2个平行的层级结构主要是用于职责分离,实现视图的绘制,显示,布局解耦,避免重复代码
- 在iOS 和 Mac OS两个平台上,事件和用户交互有很多不同的地方,创建2个层级结构,可以在2个平台上共享代码,从而使得开发快捷.

#### 34.UIWindow是什么,有什么特点?

- UIWindow 继承自 UIView, 作为根视图来装置 View元素, UIWindow提供一个区域用于显示UIView,并且将事件分发给 UIView,一般一个应用只有一个 UIWindow;

#### 35. 什么是隐式动画和显示动画

- 隐式动画是系统框架自动完成的。Core Animation在每个runloop周期中自动开始一次新的事务，即使你不显式的用[CATransaction begin]开始一次事务，任何在一次runloop循环中属性的改变都会被集中起来，然后做一次0.25秒的动画。在iOS4中，苹果对UIView添加了一种基于block的动画方法：+animateWithDuration:animations:。这样写对做一堆的属性动画在语法上会更加简单，但实质上它们都是在做同样的事情。CATransaction的+begin和+commit方法在+animateWithDuration:animations:内部自动调用，这样block中所有属性的改变都会被事务所包含,多用于简单动画效果



```objectivec
    [UIView animateWithDuration:1 animations:^{
        view.center = self.view.center;
    }];
```

- 显式动画，Core Animation提供的显式动画类型，既可以直接对layer层属性做动画，也可以覆盖默认的图层行为。我们经常使用的CABasicAnimation，CAKeyframeAnimation，CATransitionAnimation，CAAnimationGroup等都是显式动画类型，这些CAAnimation类型可以直接提交到CALayer上。显式动画可用于实现更为复杂的动画效果.



```objectivec
CABasicAnimation *positionAnima = [CABasicAnimation animationWithKeyPath:@"position.y"];
positionAnima.duration = 0.8;
positionAnima.fromValue = @(self.imageView.center.y);
positionAnima.toValue = @(self.imageView.center.y-30);
positionAnima.timingFunction = [CAMediaTimingFunction functionWithName:kCAMediaTimingFunctionEaseIn];
positionAnima.repeatCount = HUGE_VALF;
positionAnima.repeatDuration = 2;
positionAnima.removedOnCompletion = NO;
positionAnima.fillMode = kCAFillModeForwards;
[self.imageView.layer addAnimation:positionAnima forKey:@"AnimationMoveY"];
```

#### 36. UIButton 和UITableView的层级结构

- 继承结构
    - UIButton -> UIControl -> UIView -> UIResponder -> NSObject
    - UITableView -> UIScrollView -> UIView -> UIResponder -> NSObject
- 内部子控件结构
    - UIButton内部子控件结构: 默认有两个, 一个UIImageView, 一个UILable, 分别可以设置图片和文字, button设置属性基本都是set方法
    - UITableView内部子控件结构: UITableView中每一行数据都是UITableViewCell, UITableViewCell内部有一个UIView控件 (contentView, 作为其它元素的父控件) , 两个UILable 控件 (textLable, detailTextLable) , 一个UIImageView控件 (imageView) , 分别用于容器, 显示内容, 详情和图片

#### 37. Storyboard/xib 和 纯代码UI相比,有哪些优缺点

优点:

- 简单直接快速, 通过拖拽和点选即可配置UI,界面所见即所得
- 在 Storybord可以清楚的区分ViewController 界面之间的跳转关系

缺点:

- 协作冲突,多人编辑时,容易发生冲突,很难解决冲突
- 很难做到界面继承和重用
- 不便于进行模块化管理
- 影响性能

#### 38.AutoLayout 和 Frame在UI布局和渲染上有什么区别?

- AutoLayout是针对多尺寸屏幕的设计,其本质是通过线性不等式设置UI控件的相对位置,从而适配多种屏幕设备
- Frame 是基于XY坐标轴系统布局机制,它从数学上限定了UI 控件的具体位置,是 iOS'开发中最低层,最基本的界面布局方式
- AutoLayout性能比 Frame 差很多,AutoLayout布局过程是首先求解线性不等式,然后在转化为Frame进行布局,其中求解计算量非常大,很损耗性能;

#### 39.SafeArea, SafeAreaLayoutGuide, SafeAreaInsets 关键词的比较说明?

由于 iphoneX 采用了`刘海`设计,iOS11引入了安全区域`(SafeArea)`概念

- SafeArea是指App 显示内容的区域,不包括StatusBar,Navigationbar,tabbar,和 toolbar, 在 iPhoneX 中一般是指扣除了statusBar(44像素),和底部home indicator(高度为34像素)的区域.这样操作不会被刘海或者底部手势影响了.
- SafeAreaLayoutGuide 是指 Safe Area 的区域范围和限制, 在布局设置中,可以取得他的上下左右4个边界位置进行相应的布局
- SafeAreaInsets限定了Safe Area区域与整个屏幕之间的布局关系,一般用上下左右4个值来获取 SafeArea 与屏幕边缘之间的距离;

#### 40.UIScrollView 的 contentView, contentInset, contentSize, contentOffset 关键字比较?

- contentView 是指 UIScrollView上显示内容的区域,用户 addSubView 都是在 contentView上进行的;
- contentInset 是指 contentView与 UIScrollView的边界;
- contentSize 是指 contentView 的大小,表示可以滑动范围;
- contentOffset 是当前 contentView 浏览位置左上角点的坐标;

#### 41. 图片png与jpg的区别是什么?

- png:
    优点：无损格式，不论保存多少次，理论上图片质量都不会受任何影响；支持透明
    缺点：尺寸过大；打开速度与保存速度和jpg没法比

- jpg:
    优点：尺寸较小，节省空间；打开速度快
    缺点：有损格式，在修图时不断保存会导致图片质量不断降低；不支持透明

    在开发中，尺寸比较大的图片（例如背景图片）一般适用jpg格式，减小对内存的占用！

#### 下一篇入口:



作者：Leon_520
链接：https://www.jianshu.com/p/c6b7302b7411
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。