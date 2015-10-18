# 与 Objective-C API 交互

本页包含内容：

-   [初始化（Initialization）](#initialization)

-   [访问属性（Accessing Properties）](#accessing_properties)

-   [方法（Working with Methods）](#working_with_methods)

-   [id 兼容性](#id_compatibility)

-   [为空性和可选（Nullability and Optionals）](#Nullability_and_Optionals)

-   [扩展（Extensions）](#extensions)

-   [闭包（Closures）](#closures)

-   [对象比较（Object Comparison）](#object_comparison)

-   [Swift 类型兼容性（Swift Type Compatibility）](#swift_type_compatibility)

-   [轻量级泛型（Lightweight Generics）](#Lightweight_Generics)

-   [Objective-C 选择器（Objective-C Selectors）](#objective_c_selectors)

**互用性**是让 Swift 和 Objective-C 相接合的一种特性，使你能够在一种语言编写的文件中使用另一种语言。当你准备开始把 Swift 融入到你的开发流程中时，你应该学会如何利用互用性来重新定义，改善并提高你编写 Cocoa 应用的方式。

互用性很重要的一点就是允许你在写 Swift 代码时使用 Objective-C 的 API。当你导入一个 Objective-C 框架后，你可以使用原生的 Swift 语法实例化这些类并与之交互。

<a name="initialization"></a>
## 初始化（Initialization）

为了使用 Swift 实例化一个 Objective-C 的类，你应该使用 Swift 语法调用它的一个初始化器。当 Objective-C 的`init`方法转换到 Swift，它们将用原生 Swift 初始化语法呈现。“init”前缀被截断，当作一个关键字，用来表明该方法是初始化方法。那些以“initWith”开头的`init`方法，“With”也会被去除。原初始化方法的方法名的第一部分去除“init”或者“initWith”后，首字母变成小写，并且被当做是新初始化方法第一个参数的参数名。原初始化方法的方法名其余的每一部分方法名依次变为参数名。这些方法名片段都跑到了圆括号中，方法调用时都是必须写的。

举个例子，你在使用 Objective-C 时会这样做：

```objective-c
UITableView *myTableView = [[UITableView alloc] initWithFrame:CGRectZero 
                                                        style:UITableViewStyleGrouped];
```
在 Swift 中，你应该这样做：

```swift
let myTableView: UITableView = UITableView(frame: CGRectZero, style: .Grouped)
```

你不需要调用`alloc`，Swift 能够正确的为你处理。注意，当使用 Swift 风格的初始化方法的时候，“init”不会出现。

你可以在初始化时显式地声明对象的类型，也可以忽略它，Swift 能够正确推断对象的类型。

```swift
let myTextField = UITextField(frame: CGRect(x: 0.0, y: 0.0, width: 200.0, height: 40.0))
```

这里的`UITableView`和`UITextField`对象和你在 Objective-C 中使用它们时具有相同的功能。你可以用一样的方式使用它们，访问属性或者调用各自类中的方法。

为了统一和简洁，Objective-C 的工厂方法也在 Swift 中映射为**便捷初始化方法**（**convenience initializers**）。这种映射能够让它们使用同样简洁明了的初始化方法。例如，在 Objective-C 中你可能会像下面这样调用一个工厂方法：

```objective-c
UIColor *color = [UIColor colorWithRed:0.5 green:0.0 blue:0.5 alpha:1.0];
```

在 Swift 中，你应该这样做：

```swift
let color = UIColor(red: 0.5, green: 0.0, blue: 0.5, alpha: 1.0)
```

### 可失败初始化（Failable Initialization）

在 Objective-C 中，构造器会直接返回他们初始化的对象。为了告知调用者初始化失败，Objective-C 中的构造器会返回`nil`。在 Swift 中，这种模式被内置到语言特性中，被称为**可失败初始化**（**failable initialization**）。在 iOS 和 OS X 系统框架中许多 Objective-C 构造器会被检查是否会初始化失败。你可以在你的 Objective-C 代码中，使用**为空性标注**（**nullability annotations**）来指明初始化是否会失败，正如 [为空性和可选（Nullability and Optionals）](#Nullability_and_Optionals)中所描述。如果初始化不会失败，这些 Objective-C 构造器便会以`init(...)`被导入，如果初始化可能会失败，则会以`init?(...)`被导入。否则，构造器会以`init!(...)`被导入。

例如，当图片文件在指定路径中不存在时，`UIImage(contentsOfFile:)`构造器初始化`UIImage`对象便会失败。如果初始化成功，我们可以用可选绑定来对可失败初始化的结果进行解包。

```swift
if let image = UIImage(contentsOfFile: "MyImage.png") {
    // loaded the image successfully
} else {
    // could not load the image
}
```

<a name="accessing_properties"></a>
## 访问属性（Accessing Properties）

Objective-C 使用`@property`关键字声明属性，导入 Swift 中时，遵循以下规则：
- 拥有`nonnull`，`nullable`，`null_resettable`修饰符的属性，作为可选或者非可选属性导入。参见 [为空性和可选（Nullability and Optionals）](#Nullability_and_Optionals)中的描述。
- 拥有`readonly`修饰符的属性，作为只读计算属性（`{ get }`）导入。
- 拥有`weak`修饰符的属性，作为标记`weak`关键字的属性（`weak var`）导入。
- 拥有`assign`，`copy`，`strong`，`unsafe_unretained`修饰符的属性，导入后会拥有相应的内存管理策略。
- `atomic`，`nonatomic`修饰符会被忽略。Swift 中所有属性都是`nonatomic`的。
- `getter=`，`setter=`修饰符会被忽略。

在 Swift 中访问和设置 Objective-C 对象的属性时，使用点语法，直接使用属性名即可，不需要附加圆括号：

```swift
myTextField.textColor = UIColor.darkGrayColor()
myTextField.text = "Hello world"
```
>注意

>`darkGrayColor()`跟着一对圆括号，因为它是一个类方法，而不是一个属性。

在 Objective-C 中，一个有返回值的无参数方法可以像属性那样使用点语法调用。但在 Swift 中不再能够这样做了，只有在 Objective-C 中 使用`@property`关键字声明的属性才会被作为属性引入。方法的导入和调用见 [方法（Working with Methods）](#working_with_methods)中的描述。

<a name="working_with_methods"></a>
## 方法（Working with Methods）

在 Swift 中调用 Objective-C 方法时，使用点语法。

当 Objective-C 方法转换到 Swift 时，Objective-C 的`selector`的第一部分将会成为方法名并出现在圆括号的前面，而第一个参数将直接在括号中出现，并且没有参数名，而剩下的参数名与参数则一一对应的填入圆括号中。方法名的所有部分在调用时都是必须写的。

举个例子，你在使用 Objective-C 时会这样做：

```objective-c
[myTableView insertSubview:mySubview atIndex:2];
```

在 Swift 中，你应该这样做：

```swift
myTableView.insertSubview(mySubview, atIndex: 2)
```

如果你调用一个无参方法，仍必须在方法名后面加上一对圆括号：

```swift
myTableView.layoutIfNeeded()
```

<a name="id_compatibility"></a>
## id 兼容性

Swift 包含一个叫做`AnyObject`的协议类型，表示任意类型的对象，就像 Objective-C 中的`id`一样。`AnyObject`协议允许你编写类型安全的 Swift 代码同时维持无类型对象的灵活性。因为`AnyObject`协议保证了这种安全，Swift 将`id`对象导入为`AnyObject`。

举个例子，跟`id`一样，你可以为`AnyObject`类型的常量或变量分配任何类型的对象，如果是变量，你还可以再为其重新分配任意类型的对象。

```swift
var myObject: AnyObject = UITableViewCell()
myObject = NSDate()
```

你也可以在调用 Objective-C 方法或者访问属性时不将它转换为具体类的类型。这包括了 Objective-C 中标记为`@objc`的 Objective-C 兼容方法。

```swift
let futureDate = myObject.dateByAddingTimeInterval(10)
let timeSinceNow = myObject.timeIntervalSinceNow
```

然而，由于直到运行时才知道`AnyObject`的对象类型，所以有可能在不经意间写出不安全代码。和 Objective-C 中一样，如果用`AnyObject`对象调用不存在的方法或者访问不存在的属性，将会引发运行时错误。比如下面的代码在运行时将会引发一个`unrecognized selector error`错误：

```swift
myObject.characterAtIndex(5)
// crash, myObject does't respond to that method
```

但是，你可以通过 Swift 中可选的特性来消除这个 Objective-C 中常见的错误，当你用`AnyObject`对象调用一个 Objective-C 方法时，方法调用行为实际上类似于隐式解包可选。你可以像调用协议中的可选方法那样通过可选链接语法来用`AnyObject`对象调用方法。

> 注意  

> 通过`AnyObject`访问属性总是返回一个可选值。

举个例子，在下面的代码中，第一和第二行代码将不会被执行，因为`count`属性和`characterAtIndex:`方法不存在于`NSDate`对象中。`myCount`常量会被推断成可选的`Int`类型并且被赋值为`nil`。同样你可以使用`if-let`语句来有条件地展开这种不一定存在的方法的返回值。就像第三行做的一样。

```swift
let myCount = myObject.count
let myChar = myObject.characterAtIndex?(5)
if let fifthCharacter = myObject.characterAtIndex?(5) {
    print("Found \(fifthCharacter) at index 5")
}
```

对于 Swift 中的强制类型转换，从`AnyObject`类型的对象转换成明确的类型可能会失败，所以它会返回一个可选值。你需要检查该可选值来确认转换是否成功。

```swift
let userDefaults = NSUserDefaults.standardUserDefaults()
let lastRefreshDate: AnyObject? = userDefaults.objectForKey("LastRefreshDate")
if let date = lastRefreshDate as? NSDate {
    println("\(date.timeIntervalSinceReferenceDate)")
}
```

当然，如果你能确定这个对象的类型（并且确定不是`nil`），你可以用`as`操作符强制调用。

```swift
let myDate = lastRefreshDate as! NSDate
let timeInterval = myDate.timeIntervalSinceReferenceDate
```

<a name="Nullability_and_Optionals"></a>
## 为空性和可选（Nullability and Optionals）

在 Objective-C 中，对象的引用可以是值为`NULL`的原始指针（在 Objective-C 中称为`nil`）。而在 Swift 中，所有的值，包括结构体与对象的引用，都保证为非空。作为替代，你将可以为空的值包装为**可选类型**（**optional type**）。当你需要指明值为空时，你需要使用`nil`赋值。想要了解更多关于可选的信息，可以看看[《The Swift Programming Language 中文版》](http://wiki.jikexueyuan.com/project/swift/)中的 [可选（Optionals）](http://wiki.jikexueyuan.com/project/swift/chapter2/01_The_Basics.html#optionals)部分。

Objective-C 可以使用**为空性标注**（**nullability annotations**）来指明一个参数类型，属性类型或者返回值类型是否可以为`NULL`或者`nil`值。单个类型声明可以使用`_Nullable`和`_Nonnull`标注，单个属性声明可以使用`nullable`，`nonnull`，`null_resettable`属性修饰符，范围性标注可以使用`NS_ASSUME_NONNULL_BEGIN`和`NS_ASSUME_NONNULL_END`宏。如果一个类型没有任何为空性标注，Swift 就不能分辨出其到底是可选还是非可选，并且将作为隐式解包可选导入。

-  以`_Nonnull`或者范围宏标注的类型，作为**非可选**（**non-optional**）导入到 Swift。
-  以`_Nullable`标注的类型，作为**可选**（**optional**）导入到 Swift。
-  没有为空性标注的类型，作为**隐式解包可选**（**implicitly unwrapped optional**）导入到 Swift。

例如，考虑如下的 Objective-C 声明：

```objective-c
@property (nullable) id nullableProperty;
@property (nonnull) id nonNullProperty;
@property id unannotatedProperty;
 
NS_ASSUME_NONNULL_BEGIN
- (id)returnsNonNullValue;
- (void)takesNonNullParameter:(id)value;
NS_ASSUME_NONNULL_END
 
- (nullable id)returnsNullableValue;
- (void)takesNullableParameter:(nullable id)value;
 
- (id)returnsUnannotatedValue;
- (void)takesUnannotatedParameter:(id)value;
```

下面是它们如何被导入 Swift 中的：

```Swift
var nullableProperty: AnyObject?
var nonNullProperty: AnyObject
var unannotatedProperty: AnyObject!
 
func returnsNonNullValue() -> AnyObject
func takesNonNullParameter(value: AnyObject)
 
func returnsNullableValue() -> AnyObject?
func takesNullableParameter(value: AnyObject?)
 
func returnsUnannotatedValue() -> AnyObject!
func takesUnannotatedParameter(value: AnyObject!)
```
大多数 Objective-C 的系统框架，包括Foundation，都已经提供了为空性标注，符合你的使用习惯，且让你能以类型安全的方式与各种值打交道。

<a name="extensions"></a>
## 扩展（Extensions）

Swift 的扩展和 Objective-C 的分类(Category)相似。扩展可以为原有的类，结构和枚举扩充功能，即使它们是在 Objective-C 中定义的。你可以为系统框架中的类型或者你的自定义类型定义扩展，只需要导入合适的模块。

举个例子，你可以扩展`UIBezierPath`类来创建一个简单的三角形路径，这个方法只需提供三角形的边长与起点。

```swift
extension UIBezierPath {
    convenience init(triangleSideLength: CGFloat, origin: CGPoint) {
        self.init()
        let squareRoot = CGFloat(sqrt(3.0))
        let altitude = (squareRoot * triangleSideLength) / 2
        moveToPoint(origin)
        addLineToPoint(CGPoint(x: origin.x + triangleSideLength, y: origin.y))
        addLineToPoint(CGPoint(x: origin.x + triangleSideLength / 2, y: origin.y + altitude))
        closePath()
    }
}
```

你也可以使用扩展来增加属性（包括类的属性与静态属性）。然而，这些属性必须是计算属性，扩展不能为类，结构体，枚举添加存储属性。

下面这个例子为`CGRect`类增加了一个叫`area`的属性。

```swift
extension CGRect {
    var area: CGFloat {
        return width * height
    }
}
let rect = CGRect(x: 0.0, y: 0.0, width: 10.0, height: 50.0)
let area = rect.area
```

你同样可以使用扩展让类，结构或枚举符合协议而无需继承，即使它们是在 Objective-C 中定义的。

你不能使用扩展来覆盖 Objective-C 类型中存在的方法与属性。

<a name="closures"></a>
## 闭包（Closures）

Objective-C 中的`block`会被自动作为 Swift 中的闭包导入。（Objective-C blocks are automatically imported as Swift closures with Objective-C block calling convention, denoted by the @convention(block) attribute.）例如，下面是一个 Objective-C 中的 block 变量：

```objective-c
void (^completionBlock)(NSData *, NSError *) = ^(NSData *data, NSError *error) {
    // ...
}
```

而它在 Swift 中的形式为

```swift
let completionBlock: (NSData, NSError) -> Void = { (data, error) in 
    // ...
}
```

Swift 的闭包与 Objective-C 的 block 能够兼容，所以你可以把一个 Swift 闭包传递给一个接收 block 作为参数的 Objective-C 方法。Swift 闭包与函数具有相同的类型，所以你甚至可以传递 Swift 函数的函数名。

闭包与 block 有相似的捕获语义，但有个关键的不同：捕获的变量是可以直接修改的，而不是像 block 那样会拷贝变量。换句话说，Swift 中变量的默认行为与 Objective-C 中`__block`变量一致。

<a name="object_comparison"></a>
## 对象比较（Object Comparison）

当比较两个 Swift 中的对象时，可以使用两种方式。第一种，使用 ==，判断两个对象内容是否相同。第二种，使用 === ，判断常量或者变量是否引用同一个对象的实例。

Swift 与 Objective-C 一般使用 == 与 === 操作符来做比较。Swift 的 == 操作符为源自`NSObject` 的对象提供了默认的实现。在 == 操作符的实现中，Swift 调用`NSObject`定义的`isEqual:`方法。

`NSObject`类仅仅做了同一性比较（地址相同），所以你需要在你自己的类中重新实现`isEqual:`方法。因为你可以直接传递 Swift 对象（包括那些不继承自`NSObject`的）给 Objective-C 的 API，所以如果你希望比较两个对象的内容是否相同而不是仅仅比较其同一性的话，你应该为这些对象实现`isEqual:` 方法。

作为实现比较的一部分，确保根据 [Object comparison](https://developer.apple.com/library/prerelease/ios/documentation/General/Conceptual/DevPedia-CocoaCore/ObjectComparison.html#//apple_ref/doc/uid/TP40008195-CH37) 实现对象的`hash`属性。更进一步的说，如果你希望你的类能够作为字典中的键，还需要遵从`Hashable`协议以及实现`hashValues`属性。

<a name="swift_type_compatibility"></a>
## Swift 类型兼容性

当你在 Swift 中创建了一个继承自 Objective-C 类的子类时，该类以及该类的成员：属性，方法，下标和构造器，便会在 Objective-C 中自动可用。在某些情况下，你需要更细粒度地控制如何将 Swift API 暴露给 Objective-C。如果你的 Swift 类没有继承自 Objective-C 的类，又或者你想更改暴露给 Objective-C 代码的接口中的符号名称，你便可以使用`@objc`属性。如果你正在使用诸如键值观察这种需要动态替换方法实现的 API，还可以通过使用`dynamic`修饰符来要求访问成员变量时通过 Objective-C 运行时的动态派发。

### 在 Objective-C 中暴露 Swift 接口（Exposing Swift Interfaces in Objective-C）

当你定义一个继承自`NSObject`类或者其他 Objective-C 类的 Swift 子类时，该类便会自动兼容 Objective-C。Swift 编译器已经为你做好了这部分所需要的工作。所有的步骤都由 Swift 编译器自动完成，如果你不会在 Objective-C 代码中导入 Swift 类，你也不需要担心类型适配问题。否则，如果你的 Swift 类并不继承于 Objective-C 类而你希望能在 Objective-C 的代码中使用它，你可以使用下面描述的`@objc`属性。

`@objc`可以让你的 Swift API 在 Objective-C 以及 Objective-C runtime 中使用。换句话说，你可以通过在任何 Swift 方法，属性，下标，构造器，类，枚举前添加`@objc`，从而让它们可以在 Objective-C 代码中使用。

> 注意  

> 嵌套类型不能标记`objc`属性。  
> 另外只有原始值为整形的 Swift 枚举，例如`Int`，才能使用`objc`属性。

如果你的类继承自 Objective-C，编译器会自动帮助你完成这一步。而如果一个类前面加上了`@objc`关键字，编译器就会在类中所有成员前加`@objc`。当你使用`@IBOutlet`，`@IBAction`，或者是`@NSManaged`属性时，`@objc`属性也会自动添加。当一个 Objetive-C 类使用选择器来实现 target-action 设计模式时，也要用到此属性，例如，`NSTimer`或者`UIButton`。

> 注意

> 如果标记了`private`访问级别修饰符，编译器将不会自动插入`@objc`属性。

当你在 Objective-C 中使用 Swift API 时，编译器通常会对语句做直接翻译。

例如，Swift API `func playSong(name: String)`在 Objective-C 中会被解释为`- (void)playSong:(NSString *)name`。然而，有一个例外：当在 Objective-C 中使用 Swift 的初始化函数时，编译器会在方法前添加“initWith”并且将原初始化函数的第一个参数首字母大写。

例如，这个 Swift 初始化函数`init(songName: String, artist: String)`在 Objective-C 中将被翻译为`- (instancetype)initWithSongName:(NSString *)songName artist:(NSString *)artist`。

Swift 同时也提供了一个`@objc`属性的变体，通过它你可以指定转换到 Objective-C 后的符号名。例如，如果你的 Swift 类的名字包含 Objective-C 中不支持的字符，你就可以为 Objective-C 提供一个可供替代的名字。如果你要为 Swift 函数提供一个 Objective-C 选择器风格的名字，记得在参数名后添加冒号：

```swift
@objc(Squirrel)
class Белка: NSObject {

    @objc(initWithName:)
    init (имя: String) { 
        // ...
    }
    
    @objc(hideNuts:inTree:)
    func прячьОрехи(Int, вДереве: Дерево) {
       // ...
    }
}
```

当你在 Swift 类中使用`@objc(name)`关键字时，该类可在 Objective-C 没有任何命名空间。因此，这个关键字在你迁徙被归档的 Objecive-C 类到 Swift 时非常有用。由于归档过的对象存储了类的名字，你应该使用`@objc(name)`来声明与旧的归档过的类相同的名字，这样旧的类才能被新的 Swift 类解档。

### 要求动态派发（Requiring Dynamic Dispatch）

`@objc`属性将你的 Swift API 暴露给了 Objective-C runtime，但是它并不能保证属性，方法，下标，或构造器的动态派发。Swift 编译器可能仍然通过绕过 Objective-C runtime 来消虚拟化（devirtualize）或内联成员访问来优化代码的性能。当一个成员声明用`dynamic`修饰符标记时，对该成员的访问将始终是动态派发的。由于标记`dynamic`修饰符后需要用 Objective-C runtime 来派发，因此会被隐式地标记`@objc`属性。

一般很少需要动态派发。但是，当你要在运行时替换一个 API 的实现时你必须使用`dynamic`修饰符。例如，你可以使用 Objective-C runtime 的`method_exchangeImplementations`函数在应用程序运行中替换某个方法的实现。如果 Swift 编译器内联了方法的实现或者消虚拟化对它的访问，新的实现将不会被使用。

<a name="Lightweight_Generics"></a>
## 轻量级泛型（Lightweight Generics）

使用轻量级泛型参数化声明的 Objective-C 类型，如`NSArray`，`NSSet`以及`NSDictionary`，在被导入到 Swift 时会附带上它们保存的内容的类型。

例如，考虑下面的 Objective-C 属性声明：

```objective-c
@property NSArray<NSDate *>* dates;
@property NSSet<NSString *>* words;
@property NSDictionary<NSURL *, NSData *>* cachedData;
```

下面是 Swift 导入后：

```swift
var dates: [NSDate]
var words: Set<String>
var cachedData: [NSURL: NSData]
```

> 注意

> 除了 Foundation 中的集合类，其他 Objective-C 类型使用轻量级泛型会被 Swift 忽略。

<a name="objective_c_selectors"></a>
## Objective-C 选择器

Objective-C 选择器是一种用于引用 Objective-C 方法名的类型。在 Swift 中，Objective-C 选择器用`Selector`结构体表示。你可以通过字符串字面量创建一个选择器，比如`let mySelector: Selector = "tappedButton:"`。因为字符串字面量能够自动被转换为选择器，所以你可以把字符串字面量直接传递给任何能够接受选择器的方法。

```swift
import UIKit
class MyViewController: UIViewController {

    let myButton = UIButton(frame: CGRect(x: 0, y: 0, width: 100, height: 50))
    
    override init?(nibName nibNameOrNil: String?, bundle nibBundleOrNil: NSBundle?) {
        super.init(nibName: nibNameOrNil, bundle: nibBundleOrNil)
        myButton.addTarget(self, action: "tappedButton:", forControlEvents: .TouchUpInside)
    }
    
    func tappedButton(sender: UIButton!) {
        print("tapped button")
    }
    
    required init?(coder: NSCoder) {
        super.init(coder: coder)
    }
}
```

如果你的 Swift 类继承自 Objective-C 类，那么所有方法和属性都可以用于 Objective-C 选择器。反之，如果你的 Swift 类不是继承自 Objective-C 类，你需要在要用于选择器的方法前面加上`@objc`属性前缀，详情请看 [Swift 类型兼容性](#swift_type_compatibility)。

### 使用 performSelector 发送消息

你可以使用`performSelector(_:)`方法以及它的变体向 Objective-C 兼容的对象发送消息。

`performSelector`系列 API 可以向指定线程发送消息，或者延迟发送没有返回值的消息。该系列 API 同步执行，并返回隐式解包可选的非托管实例（`Unmanaged<AnyObject>!`），因为返回值的类型和所有权无法在编译期决定。可以查看[非托管对象](https://github.com/949478479/Using-Swift-with-Cocoa-and-Objective-C/blob/master/02Interoperability/03Working%20with%20Cocoa%20Data%20Types.md#%E9%9D%9E%E6%89%98%E7%AE%A1%E5%AF%B9%E8%B1%A1)获取更多信息。

```swift
let string: NSString = "Hello, Cocoa!"
let selector: Selector = "lowercaseString"
if let result = string.performSelector(selector) {
    print(result.takeUnretainedValue())
}
// prints "hello, cocoa!"
```

向一个对象发送无法识别的选择器将造成接收者调用`doesNotRecognizeSelector(_:)`方法，其默认实现是引发`NSInvalidArgumentException`异常。

```swift
let array: NSArray = ["delta", "alpha", "zulu"]
let invalidSelector: Selector = "invalid"
array.performSelector(invalidSelector) // raises an exception
```

在 Objective-C 运行时中直接向对象发送消息是内在不安全的，因为编译器无法保证消息发送的结果，或是消息是否能在第一时间被处理。同样的，不鼓励使用`performSelector`系列 API，除非你的代码确实依赖于 Objective-C 运行时提供的动态方法决议。否则，正如 [id 兼容性](https://github.com/949478479/Using-Swift-with-Cocoa-and-Objective-C/blob/master/02Interoperability/01Interacting%20with%20Objective-C%20APIs.md#id-%E5%85%BC%E5%AE%B9%E6%80%A7)中所描述的那样，使用可选链向`AnyObject`类型发送消息更为安全方便。