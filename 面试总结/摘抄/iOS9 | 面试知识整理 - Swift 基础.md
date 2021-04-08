#### 1. 介绍一下 Swift?

`Swift`是苹果在2014年6月WWDC发布的全新编程语言，借鉴了`JS,Python,C#,Ruby`等语言特性,看上去偏脚本化,Swift 仍支持 `cocoa touch` 框架

他的优点:

1. Swift更加安全，它是类型安全的语言。
2. Swift容易阅读，语法和文件结构简易化。
3. Swift更易于维护，文件分离后结构更清晰。
4. Swift代码更少，简洁的语法，可以省去大量冗余代码
5. Swift速度更快，运算性能更高。

#### 2. Swift 和OC 如何相互调用?

- Swift 调用 OC代码
    需要创建一个 `Target-BriBridging-Header.h` 的桥文件,在乔文件导入需要调用的OC代码头文件即可
- OC 调用 Swift代码
    直接导入 `Target-Swift.h`文件即可, Swift如果需要被OC调用,需要使用@objc 对方法或者属性进行修饰

#### 3. 类(class) 和 结构体(struct) 有什么区别?

在 Swift 中,class 是引用类型(指针类型), struct 是值类型

**值类型**

- 值类型在传递和赋值时将进行复制; 赋值给var、let或者给函数传参，是直接将所有内容拷贝一份, 类似于对文件进行copy、paste操作，产生了全新的文件副本。属于深拷贝(deep copy)
- 值类型: 比如结构体,枚举,是在栈空间上存储和操作的

**引用类型**

- 引用类型只会使用引用对象的一个"指向"; 赋值给var、let或者给函数传参，是将内存地址拷贝一份,类似于制作一个文件的替身(快捷方式、链接)，指向的是同一个文件。属于浅拷贝(shallow copy)
- 引用类型: 比如 Class,是在堆空间上存储和操作的

#### 4. class 和 struct 比较,优缺点?

class 有以下功能,struct 是没有的:

1. class可以继承,子类可以使用父类的特性和方法
2. 类型转换可以在运行时检查和解释一个实例对象
3. class可以用 deinit来释放资源
4. 一个类可以被多次引用

struct 优势:

1. 结构较小,适用于复制操作,相比较一个class 实例被多次引用,struct 更安全
2. 无需担心内存泄露问题

#### 5. Swift 中,什么可选型(Optional)

- 在 Swift 中,可选型是为了表达一个变量为空的情况,当一个变量为空,他的值就是 nil
- 在类型名称后面加个问号? 来定义一个可选型
- 值类型或者引用类型都可以是可选型变量



```swift
var name: String? // 默认为 nil
var age: Int?     // 默认为nil
print(name, age) // 打印 nil, nil
```

#### 6.Swift,什么是泛型?

- 泛型主要是为增加代码的灵活性而生的,它可以是对应的代码满足任意类型的的变量或方法;
- 泛型可以将类型参数化，提高代码复用率，减少代码量



```swift
// 实现一个方法,可以交换实现任意类型
func swap<T>(a: inout T, b: inout T) {
    (a, b) = (b, a)
}
```

#### 7. 访问控制关键字 open, public, internal, fileprivate, private 的区别?

Swift 中有个5个级别的访问控制权限,从高到低依次是 `open, public, internal, fileprivate, private`

它们遵循的基本规则: 高级别的变量不允许被定义为低级别变量的成员变量,比如一个 private 的 class 内部允许包含 public的 String值,反之低级变量可以定义在高级别变量中;

- open: 具备最高访问权限,其修饰的类可以和方法,可以在任意 模块中被访问和重写.
- public: 权限仅次于 open，和 open 唯一的区别是: 不允许其他模块进行继承、重写
- internal: 默认权限, 只允许在当前的模块中访问，可以继承和重写,不允许在其他模块中访问
- fileprivate: 修饰的对象只允许在当前的文件中访问;
- private: 最低级别访问权限,只允许在定义的作用域内访问

#### 8.关键字:Strong,Weak,Unowned 区别?

- Swift 的内存管理机制同OC一致,都是ARC管理机制; Strong,和 Weak用法同OC一样
- Unowned(无主引用), 不会产生强引用，实例销毁后仍然存储着实例的内存地址(类似于OC中的unsafe_unretained), 试图在实例销毁后访问无主引用，会产生运行时错误(野指针)

#### 9. 如何理解copy-on-write?

值类型(比如:struct),在复制时,复制对象与原对象实际上在内存中指向同一个对象,当且仅当修改复制的对象时,才会在内存中创建一个新的对象,

- 为了提升性能，Struct, String、Array、Dictionary、Set采取了Copy On Write的技术
- 比如仅当有“写”操作时，才会真正执行拷贝操作
- 对于标准库值类型的赋值操作，Swift 能确保最佳性能，所有没必要为了保证最佳性能来避免赋值

#### 10.什么是属性观察?

属性观察是指在当前类型内对特性属性进行监测,并作出响应,属性观察是 swift 中的特性,具有2种, `willset` 和 `didset`



```swift
var title: String {
    willSet {
        print("willSet", newValue)

    }
    didSet {
        print("didSet", oldValue, title)
    }
}
```

- willSet会传递新值，默认叫newValue
- didSet会传递旧值，默认叫oldValue
- 在初始化器中设置属性值不会触发willSet和didSet

#### 11. swift 为什么将 String,Array,Dictionary设计为值类型?

- 值类型和引用类型相比,最大优势可以高效的使用内存,值类型在栈上操作,引用类型在堆上操作,栈上操作仅仅是单个指针的移动,而堆上操作牵涉到合并,位移,重链接,Swift 这样设计减少了堆上内存分配和回收次数,使用 copy-on-write将值传递与复制开销降到最低

#### 12.如何将Swift 中的协议(protocol)中的部分方法设计为可选(optional)?

- 在协议和方法前面添加 `@objc`,然后在方法前面添加 `optional`关键字,改方式实际上是将协议转为了OC的方式



```swift
@objc protocol someProtocol {
  @objc  optional func test()
}
```

- 使用扩展(extension),来规定可选方法,在 swift 中,协议扩展可以定义部分方法的默认实现



```swift
protocol someProtocol {
    func test()
}

extension someProtocol{
    func test() {
        print("test")
    }
}
```

#### 13.比较Swift 和OC中的初始化方法 (init) 有什么不同?

swift 的初始化方法,更加严格和准确, swift初始化方法需要保证所有的`非optional`的成员变量都完成初始化, 同时 swfit 新增了`convenience和 required`两个修饰初始化器的关键字

- convenience只提供一种方便的初始化器,必须通过一个指定初始化器来完成初始化
- required是强制子类重写父类中所修饰的初始化方法

#### 14.比较 Swift和OC中的 protocol 有什么不同?

- Swift 和OC中的 protocol相同点在于: 两者都可以被用作代理;
- 不同点: Swift中的 protocol还可以对接口进行抽象,可以实现面向协议,从而大大提高编程效率,Swift中的protocol可以用于值类型,结构体,枚举;

#### 15.swift 和OC 中的自省 有什么区别?

自省在OC中就是判断某一对象是否属于某一个类的操作,有以下2中方式



```json
[obj iskinOfClass:[SomeClass class]]
[obj isMemberOfClass:[SomeClass class]]
```

在 Swift 中由于很多 class 并非继承自 NSObject, 故而 Swift 使用 `is` 来判断是否属于某一类型, `is` 不仅可以作用于`class`, 还是作用于`enum`和`struct`

#### 16.什么是函数重载? swift 支不支持函数重载?

- 函数重载是指: 函数名称相同,函数的参数个数不同, 或者参数类型不同,或参数标签不同, 返回值类型与函数重载无关
- swift 支持函数重载

#### 17.swift 中的枚举,关联值 和 原始值的区分?

- 关联值--有时会将枚举的成员值跟其他类型的变量关联存储在一起，会非常有用

    

    ```swift
    // 关联值
    enum Date {
    case digit(year: Int, month: Int, day: Int)
    case string(String)
    }
    ```

- 原始值--枚举成员可以使用**相同类型**的默认值预先关联，这个默认值叫做:原始值

    

    ```swift
    // 原始值
    enum Grade: String {
      case perfect = "A"
      case great = "B"
      case good = "C"
      case bad = "D"
    }
    ```

#### 18. swift 中的闭包结构是什么样子的?



```swift
{
    (参数列表) -> 返回值类型 in 函数体代码
}
```

#### 19. 什么是尾随闭包?

- 将一个很长的闭包表达式作为函数的最后一个实参
- 使用尾随闭包可以增强函数的可读性
- 尾随闭包是一个被书写在函数调用括号外面(后面)的闭包表达式



```swift
// fn 就是一个尾随闭包参数
func exec(v1: Int, v2: Int, fn: (Int, Int) -> Int) {
    print(fn(v1, v2))
}

// 调用
exec(v1: 10, v2: 20) {
    $0 + $1
}
```

#### 20. 什么是逃逸闭包?

当闭包作为一个实际参数传递给一个函数或者变量的时候，我们就说这个闭包逃逸了，可以在形式参数前写 `@escaping` 来明确闭包是允许逃逸的。

- 非逃逸闭包、逃逸闭包，一般都是当做参数传递给函数
- 非逃逸闭包:闭包调用发生在函数结束前，闭包调用在函数作用域内
- 逃逸闭包:闭包有可能在函数结束后调用，闭包调用逃离了函数的作用域，需要通过@escaping声明



```swift
// 定义一个数组用于存储闭包类型
var completionHandlers: [() -> Void] = []

//  在方法中将闭包当做实际参数,存储到外部变量中
func someFunctionWithEscapingClosure(completionHandler: @escaping () -> Void) {
    completionHandlers.append(completionHandler)
}
```

如果你不标记函数的形式参数为 @escaping ，你就会遇到编译时错误。

#### 21. 什么是自动闭包?

自动闭包是一种自动创建的用来把作为实际参数传递给函数的表达式打包的闭包。它不接受任何实际参数，并且当它被调用时，它会返回内部打包的表达式的值。这个语法的好处在于通过写普通表达式代替显式闭包而使你省略包围函数形式参数的括号。



```swift
func getFirstPositive(_ v1: Int, _ v2: @autoclosure () -> Int) -> Int? {
    return v1 > 0 ? v1 : v2()
}
getFirstPositive(10, 20)
```

- 为了避免与期望冲突，使用了@autoclosure的地方最好明确注释清楚:这个值会被推迟执行
- @autoclosure 会自动将 20 封装成闭包 { 20 }
- @autoclosure 只支持 () -> T 格式的参数
- @autoclosure 并非只支持最后1个参数
- 有@autoclosure、无@autoclosure，构成了函数重载

如果你想要自动闭包允许逃逸，就同时使用 @autoclosure 和 @escaping 标志。

#### 22. swift中, 存储属性和计算属性的区别?

Swift中跟实例对象相关的属性可以分为2大类

**存储属性(Stored Property)**

- 类似于成员变量这个概念
- 存储在实例对象的内存中
- 结构体、类可以定义存储属性
- 枚举不可以定义存储属性

**计算属性(Computed Property)**

- 本质就是方法(函数)
- 不占用实例对象的内存
- 枚举、结构体、类都可以定义计算属性



```swift
struct Circle {
    // 存储属性
    var radius: Double
    // 计算属性
    var diameter: Double {
        set {
            radius = newValue / 2
        }
        get {
            return radius * 2
        }
    }
}
```

#### 23. 什么是延迟存储属性(Lazy Stored Property)?

使用lazy可以定义一个延迟存储属性，在第一次用到属性的时候才会进行初始化(类似OC中的懒加载)

- lazy属性必须是var，不能是let
    - let必须在实例对象的初始化方法完成之前就拥有值
- 如果多条线程同时第一次访问lazy属性
    - 无法保证属性只被初始化1次



```swift
class PhotoView {
    // 延迟存储属性
    lazy var image: Image = {
        let url = "https://...x.png"        
        let data = Data(url: url)
        return Image(data: data)
    }() 
} 
```

#### 24. 什么是属性观察器?

可以为非lazy的var存储属性设置属性观察器,通过关键字`willSet`和`didSet`来监听属性变化



```swift
struct Circle {
    var radius: Double {
        willSet {
            print("willSet", newValue)
        } 
        didSet {
            print("didSet", oldValue, radius)
        }
    } 
    init() {
        self.radius = 1.0
        print("Circle init!")
    }
}
```

#### 25. swift中什么类型属性(Type Property)?

严格来说，属性可以分为

实例属性(Instance Property): 只能通过实例对象去访问

- 存储实例属性(Stored Instance Property):存储在实例对象的内存中，每个实例对象都有1份
- 计算实例属性(Computed Instance Property)

类型属性(Type Property):只能通过类型去访问

- 存储类型属性(Stored Type Property):整个程序运行过程中，就只有1份内存(类似于全局变量)
- 计算类型属性(Computed Type Property)

可以通过static定义类型属性 p如果是类，也可以用关键字class



```swift
struct Car {
    static var count: Int = 0
    init() {
        Car.count += 1
    }
}
```

不同于存储实例属性，你必须给存储类型属性设定初始值

- 因为类型没有像实例对象那样的init初始化器来初始化存储属性

存储类型属性默认就是lazy，会在第一次使用的时候才初始化

- 就算被多个线程同时访问，保证只会初始化一次
- 存储类型属性可以是let

枚举类型也可以定义类型属性(存储类型属性、计算类型属性)

#### 26. swift 中如何使用单例模式?

可以通过`类型属性+let+private` 来写单例; 代码如下如下:



```swift
 public class FileManager {
    public static let shared = {
        // ....
        // ....
        return FileManager()
}()
    private init() { }
}
```

#### 27.swift 中的下标是什么?

- 使用subscript可以给任意类型(枚举、结构体、类)增加下标功能，有些地方也翻译为:下标脚本
- subscript的语法类似于实例方法、计算属性，本质就是方法(函数)

使用如下:



```swift
class Point {
    var x = 0.0, y = 0.0
    subscript(index: Int) -> Double {
        set {
            if index == 0 {
                x = newValue
            } else if index == 1 {
                y = newValue }
        }
        get {
            if index == 0 {
                return x
            } else if index == 1 {
                return y
            }
            return 0
        }
    }
}

var p = Point()
// 下标赋值
p[0] = 11.1
p[1] = 22.2
// 下标访问
print(p.x) // 11.1
print(p.y) // 22.2
```

#### 27.简要说明Swift中的初始化器?

- 类、结构体、枚举都可以定义初始化器
- 类有2种初始化器: `指定初始化器(designated initializer)`、`便捷初始化器(convenience initializer)`



```swift
 // 指定初始化器 
init(parameters) {
    statements 
}
// 便捷初始化器
convenience init(parameters) {
    statements 
} 
```

规则:

- 每个类至少有一个指定初始化器，指定初始化器是类的主要初始化器
- 默认初始化器总是类的指定初始化器
- 类偏向于少量指定初始化器，一个类通常只有一个指定初始化器

初始化器的相互调用规则

- 指定初始化器必须从它的直系父类调用指定初始化器
- 便捷初始化器必须从相同的类里调用另一个初始化器
- 便捷初始化器最终必须调用一个指定初始化器

#### 28.什么可选链?

可选链是一个调用和查询可选属性、方法和下标的过程，它可能为 nil 。如果可选项包含值，属性、方法或者下标的调用成功；如果可选项是 nil ，属性、方法或者下标的调用会返回 nil 。多个查询可以链接在一起，如果链中任何一个节点是 nil ，那么整个链就会得体地失败。

- 多个?可以链接在一起
- 如果链中任何一个节点是nil，那么整个链就会调用失败

#### 29. 什么是运算符重载(Operator Overload)?

类、结构体、枚举可以为现有的运算符提供自定义的实现，这个操作叫做:运算符重载



```swift
struct Point {
    var x: Int
    var y: Int
    
    // 重载运算符
    static func + (p1: Point, p2: Point) -> Point   {
        return Point(x: p1.x + p2.x, y: p1.y + p2.y)
    }
}

var p1 = Point(x: 10, y: 10)
var p2 = Point(x: 20, y: 20)
var p3 = p1 + p2
```