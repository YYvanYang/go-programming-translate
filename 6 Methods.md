# 第6章 方法

Since the early 1990s, object-oriented programming (OOP) has been the dominant programming paradigm in industry and education, and nearly all widely used languages developed since then have included support for it. Go is no exception.

自1990年代初以来，面向对象编程（OOP）一直是业界和教育界的主导编程范式，几乎所有此后开发的广泛使用的语言都包含了对它的支持。Go也不例外。

Although there is no universally accepted definition of object-oriented programming, for our purposes, an object is simply a value or variable that has methods, and a method is a function associated with a particular type. An object-oriented program is one that uses methods to express the properties and operations of each data structure so that clients need not access the object's representation directly.

虽然面向对象编程没有一个普遍接受的定义，但就我们的目的而言，**对象**（object）就是一个拥有方法的值或变量，而**方法**（method）就是与特定类型关联的函数。面向对象的程序是使用方法来表达每个数据结构的属性和操作的程序，这样客户端就不需要直接访问对象的表示。

In earlier chapters, we have made regular use of methods from the standard library, like the Seconds method of type time.Duration:

在前面的章节中，我们经常使用标准库中的方法，比如`time.Duration`类型的`Seconds`方法：

```go
const day = 24 * time.Hour
fmt.Println(day.Seconds()) // "86400"
```

and we defined a method of our own in Section 2.5, a String method for the Celsius type:

我们在2.5节中定义了自己的方法，为`Celsius`类型定义了一个`String`方法：

```go
func (c Celsius) String() string { return fmt.Sprintf("%g°C", c) }
```

In this chapter, the first of two on object-oriented programming, we'll show how to define and use methods effectively. We'll also cover two key principles of object-oriented programming, encapsulation and composition.

在这一章中，作为面向对象编程两章中的第一章，我们将展示如何有效地定义和使用方法。我们还将涵盖面向对象编程的两个关键原则：**封装**（encapsulation）和**组合**（composition）。

## 6.1. 方法声明

A method is declared with a variant of the ordinary function declaration in which an extra parameter appears before the function name. The parameter attaches the function to the type of that parameter.

方法是用普通函数声明的一个变体来声明的，其中在函数名前面出现一个额外的参数。这个参数将函数附加到该参数的类型上。

Let's write our first method in a simple package for plane geometry:

让我们在一个简单的平面几何包中编写我们的第一个方法：

```go
gopl.io/ch6/geometry
package geometry

import "math"

type Point struct{ X, Y float64 }

// traditional function
func Distance(p, q Point) float64 {
    return math.Hypot(q.X-p.X, q.Y-p.Y)
}

// same thing, but as a method of the Point type
func (p Point) Distance(q Point) float64 {
    return math.Hypot(q.X-p.X, q.Y-p.Y)
}
```

The extra parameter p is called the method's receiver, a legacy from early object-oriented languages that described calling a method as "sending a message to an object."

额外的参数`p`被称为方法的**接收者**（receiver），这是早期面向对象语言的遗留术语，它们将调用方法描述为"向对象发送消息"。

In Go, we don't use a special name like this or self for the receiver; we choose receiver names just as we would for any other parameter. Since the receiver name will be frequently used, it's a good idea to choose something short and to be consistent across methods. A common choice is the first letter of the type name, like p for Point.

在Go中，我们不对接收者使用像`this`或`self`这样的特殊名称；我们选择接收者名称就像选择任何其他参数一样。由于接收者名称会被频繁使用，选择简短的名称并在各个方法中保持一致是个好主意。一个常见的选择是类型名称的第一个字母，比如用`p`表示`Point`。

In a method call, the receiver argument appears before the method name. This parallels the declaration, in which the receiver parameter appears before the method name.

在方法调用中，接收者参数出现在方法名之前。这与声明平行，在声明中接收者参数出现在方法名之前。

```go
p := Point{1, 2}
q := Point{4, 6}
fmt.Println(Distance(p, q)) // "5", function call
fmt.Println(p.Distance(q))  // "5", method call
```

There's no conflict between the two declarations of functions called Distance above. The first declares a package-level function called geometry.Distance. The second declares a method of the type Point, so its name is Point.Distance.

上面两个名为`Distance`的函数声明之间没有冲突。第一个声明了一个名为`geometry.Distance`的包级函数。第二个声明了`Point`类型的一个方法，所以它的名称是`Point.Distance`。

The expression p.Distance is called a selector, because it selects the appropriate Distance method for the receiver p of type Point. Selectors are also used to select fields of struct types, as in p.X. Since methods and fields inhabit the same name space, declaring a method X on the struct type Point would be ambiguous and the compiler will reject it.

表达式`p.Distance`被称为**选择器**（selector），因为它为`Point`类型的接收者`p`选择了适当的`Distance`方法。选择器也用于选择结构体类型的字段，如`p.X`。由于方法和字段占用相同的命名空间，在结构体类型`Point`上声明方法`X`会产生歧义，编译器会拒绝它。

Since each type has its own name space for methods, we can use the name Distance for other methods so long as they belong to different types. Let's define a type Path that represents a sequence of line segments and give it a Distance method too.

由于每种类型都有自己的方法命名空间，我们可以为其他方法使用名称`Distance`，只要它们属于不同的类型。让我们定义一个表示线段序列的类型`Path`，并给它也定义一个`Distance`方法。

```go
// A Path is a journey connecting the points with straight lines.
type Path []Point
```

```go
// Distance returns the distance traveled along the path.
func (path Path) Distance() float64 {
    sum := 0.0
    for i := range path {
        if i > 0 {
            sum += path[i-1].Distance(path[i])
        }
    }
    return sum
}
```

Path is a named slice type, not a struct type like Point, yet we can still define methods for it. In allowing methods to be associated with any type, Go is unlike many other object-oriented languages. It is often convenient to define additional behaviors for simple types such as numbers, strings, slices, maps, and sometimes even functions. Methods may be declared on any named type defined in the same package, so long as its underlying type is neither a pointer nor an interface.

`Path`是一个命名的切片类型，不像`Point`那样是结构体类型，但我们仍然可以为它定义方法。Go允许方法与任何类型关联，这与许多其他面向对象语言不同。为数字、字符串、切片、映射，有时甚至是函数等简单类型定义额外的行为通常很方便。方法可以在同一包中定义的任何命名类型上声明，只要其底层类型既不是指针也不是接口。

The two Distance methods have different types. They're not related to each other at all, though Path.Distance uses Point.Distance internally to compute the length of each segment that connects adjacent points.

这两个`Distance`方法有不同的类型。它们彼此完全无关，尽管`Path.Distance`内部使用`Point.Distance`来计算连接相邻点的每个线段的长度。

Let's call the new method to compute the perimeter of a right triangle:

让我们调用新方法来计算直角三角形的周长：

```go
perim := Path{
    {1, 1},
    {5, 1},
    {5, 4},
    {1, 1},
}
fmt.Println(perim.Distance()) // "12"
```

In the two calls above to methods named Distance, the compiler determines which function to call based on both the method name and the type of the receiver. In the first, path[i-1] has type Point so Point.Distance is called; in the second, perim has type Path, so Path.Distance is called.

在上面对名为`Distance`的方法的两次调用中，编译器根据方法名和接收者的类型来确定调用哪个函数。在第一次调用中，`path[i-1]`的类型是`Point`，所以调用`Point.Distance`；在第二次调用中，`perim`的类型是`Path`，所以调用`Path.Distance`。

All methods of a given type must have unique names, but different types can use the same name for a method, like the Distance methods for Point and Path; there's no need to qualify function names (for example, PathDistance) to disambiguate. Here we see the first benefit to using methods over ordinary functions: method names can be shorter. The benefit is magnified for calls originating outside the package, since they can use the shorter name and omit the package name:

给定类型的所有方法必须有唯一的名称，但不同的类型可以为方法使用相同的名称，比如`Point`和`Path`的`Distance`方法；不需要限定函数名称（例如，`PathDistance`）来消除歧义。这里我们看到使用方法而不是普通函数的第一个好处：方法名称可以更短。对于来自包外部的调用，这个好处会被放大，因为它们可以使用更短的名称并省略包名：

```go
import "gopl.io/ch6/geometry"

perim := geometry.Path{{1, 1}, {5, 1}, {5, 4}, {1, 1}}
fmt.Println(geometry.PathDistance(perim)) // "12", standalone function
fmt.Println(perim.Distance())             // "12", method of geometry.Path
```

## 6.2. 指针接收者的方法

Because calling a function makes a copy of each argument value, if a function needs to update a variable, or if an argument is so large that we wish to avoid copying it, we must pass the address of the variable using a pointer. The same goes for methods that need to update the receiver variable: we attach them to the pointer type, such as *Point.

因为调用函数会复制每个参数值，如果函数需要更新变量，或者如果参数太大我们希望避免复制它，我们必须使用指针传递变量的地址。对于需要更新接收者变量的方法也是如此：我们将它们附加到指针类型，比如`*Point`。

```go
func (p *Point) ScaleBy(factor float64) {
    p.X *= factor
    p.Y *= factor
}
```

The name of this method is (*Point).ScaleBy. The parentheses are necessary; without them, the expression would be parsed as *(Point.ScaleBy).

此方法的名称是`(*Point).ScaleBy`。括号是必需的；没有它们，表达式会被解析为`*(Point.ScaleBy)`。

In a realistic program, convention dictates that if any method of Point has a pointer receiver, then all methods of Point should have a pointer receiver, even ones that don't strictly need it. We've broken this rule for Point so that we can show both kinds of method.

在现实的程序中，约定规定如果`Point`的任何方法有指针接收者，那么`Point`的所有方法都应该有指针接收者，即使是那些严格来说不需要的方法。我们为`Point`打破了这个规则，这样我们可以展示两种方法。

Named types (Point) and pointers to them (*Point) are the only types that may appear in a receiver declaration. Furthermore, to avoid ambiguities, method declarations are not permitted on named types that are themselves pointer types:

命名类型（`Point`）和指向它们的指针（`*Point`）是可以出现在接收者声明中的唯一类型。此外，为了避免歧义，不允许在本身是指针类型的命名类型上声明方法：

```go
type P *int
func (P) f() { /* ... */ } // compile error: invalid receiver type
```

The (*Point).ScaleBy method can be called by providing a *Point receiver, like this:

`(*Point).ScaleBy`方法可以通过提供`*Point`接收者来调用，像这样：

```go
r := &Point{1, 2}
r.ScaleBy(2)
fmt.Println(*r) // "{2, 4}"
```

or this:

或者这样：

```go
p := Point{1, 2}
pptr := &p
pptr.ScaleBy(2)
fmt.Println(p) // "{2, 4}"
```

or this:

或者这样：

```go
p := Point{1, 2}
(&p).ScaleBy(2)
fmt.Println(p) // "{2, 4}"
```

But the last two cases are ungainly. Fortunately, the language helps us here. If the receiver p is a variable of type Point but the method requires a *Point receiver, we can use this shorthand:

但最后两种情况很笨拙。幸运的是，语言在这里帮助了我们。如果接收者`p`是`Point`类型的变量，但方法需要`*Point`接收者，我们可以使用这个简写：

```go
p.ScaleBy(2)
```

and the compiler will perform an implicit &p on the variable. This works only for variables, including struct fields like p.X and array or slice elements like perim[0]. We cannot call a *Point method on a non-addressable Point receiver, because there's no way to obtain the address of a temporary value.

编译器会对变量执行隐式的`&p`操作。这只对变量有效，包括像`p.X`这样的结构体字段和像`perim[0]`这样的数组或切片元素。我们不能在不可寻址的`Point`接收者上调用`*Point`方法，因为没有办法获得临时值的地址。

```go
Point{1, 2}.ScaleBy(2) // compile error: can't take address of Point literal
```

But we can call a Point method like Point.Distance with a *Point receiver, because there is a way to obtain the value from the address: just load the value pointed to by the receiver. The compiler inserts an implicit * operation for us. These two function calls are equivalent:

但我们可以用`*Point`接收者调用像`Point.Distance`这样的`Point`方法，因为有办法从地址获得值：只需加载接收者指向的值。编译器为我们插入了隐式的`*`操作。这两个函数调用是等价的：

```go
pptr.Distance(q)
(*pptr).Distance(q)
```

Let's summarize these three cases again, since they are a frequent point of confusion. In every valid method call expression, exactly one of these three statements is true.

让我们再次总结这三种情况，因为它们是经常混淆的地方。在每个有效的方法调用表达式中，以下三种情况中恰好有一种是真的：

Either the receiver argument has the same type as the receiver parameter, for example both have type T or both have type *T:

要么接收者参数与接收者参数具有相同的类型，例如都具有类型`T`或都具有类型`*T`：

```go
Point{1, 2}.Distance(q) // Point
pptr.ScaleBy(2)         // *Point
```

Or the receiver argument is a variable of type T and the receiver parameter has type *T. The compiler implicitly takes the address of the variable:

或者接收者参数是类型`T`的变量，接收者参数具有类型`*T`。编译器隐式地获取变量的地址：

```go
p.ScaleBy(2) // implicit (&p)
```

Or the receiver argument has type *T and the receiver parameter has type T. The compiler implicitly dereferences the receiver, in other words, loads the value:

或者接收者参数具有类型`*T`，接收者参数具有类型`T`。编译器隐式地解引用接收者，换句话说，加载值：

```go
pptr.Distance(q) // implicit (*pptr)
```

If all the methods of a named type T have a receiver type of T itself (not *T), it is safe to copy instances of that type; calling any of its methods necessarily makes a copy. For example, time.Duration values are liberally copied, including as arguments to functions. But if any method has a pointer receiver, you should avoid copying instances of T because doing so may violate internal invariants. For example, copying an instance of bytes.Buffer would cause the original and the copy to alias (§2.3.2) the same underlying array of bytes. Subsequent method calls would have unpredictable effects.

如果命名类型`T`的所有方法都有`T`本身的接收者类型（不是`*T`），那么复制该类型的实例是安全的；调用其任何方法都必须进行复制。例如，`time.Duration`值被自由地复制，包括作为函数的参数。但如果任何方法有指针接收者，你应该避免复制`T`的实例，因为这样做可能违反内部不变量。例如，复制`bytes.Buffer`的实例会导致原始值和副本别名（§2.3.2）相同的底层字节数组。后续的方法调用会产生不可预测的效果。

## 6.3. Composing Types by Struct Embedding

## 6.3. 通过结构体嵌入组合类型

Consider the type ColoredPoint:

考虑 ColoredPoint 类型：

```go
gopl.io/ch6/coloredpoint

import "image/color"

type Point struct{ X, Y float64 }

type ColoredPoint struct {
    Point
    Color color.RGBA
}
```

We could have defined ColoredPoint as a struct of three fields, but instead we embedded a Point to provide the X and Y fields. As we saw in Section 4.4.3, embedding lets us take a syntactic shortcut to defining a ColoredPoint that contains all the fields of Point, plus some more. If we want, we can select the fields of ColoredPoint that were contributed by the embedded Point without mentioning Point:

我们本可以将 ColoredPoint 定义为一个包含三个字段的结构体，但我们选择嵌入一个 Point 来提供 X 和 Y 字段。正如我们在 4.4.3 节中看到的，嵌入让我们能够使用语法快捷方式来定义一个包含 Point 所有字段及其他字段的 ColoredPoint。如果需要，我们可以选择由嵌入的 Point 贡献的 ColoredPoint 字段，而无需提及 Point：

```go
var cp ColoredPoint
cp.X = 1
fmt.Println(cp.Point.X) // "1"
cp.Point.Y = 2
fmt.Println(cp.Y) // "2"
```

A similar mechanism applies to the methods of Point. We can call methods of the embedded Point field using a receiver of type ColoredPoint, even though ColoredPoint has no declared methods:

类似的机制也适用于 Point 的方法。我们可以使用 ColoredPoint 类型的接收者调用嵌入的 Point 字段的方法，尽管 ColoredPoint 没有声明任何方法：

```go
red := color.RGBA{255, 0, 0, 255}
blue := color.RGBA{0, 0, 255, 255}
var p = ColoredPoint{Point{1, 1}, red}
var q = ColoredPoint{Point{5, 4}, blue}
fmt.Println(p.Distance(q.Point)) // "5"
p.ScaleBy(2)
q.ScaleBy(2)
fmt.Println(p.Distance(q.Point)) // "10"
```

The methods of Point have been promoted to ColoredPoint. In this way, embedding allows complex types with many methods to be built up by the composition of several fields, each providing a few methods.

Point 的方法已被提升到 ColoredPoint。通过这种方式，嵌入允许我们通过组合多个字段来构建具有许多方法的复杂类型，每个字段提供一些方法。

Readers familiar with class-based object-oriented languages may be tempted to view Point as a base class and ColoredPoint as a subclass or derived class, or to interpret the relationship between these types as if a ColoredPoint "is a" Point. But that would be a mistake. Notice the calls to Distance above. Distance has a parameter of type Point, and q is not a Point, so although q does have an embedded field of that type, we must explicitly select it. Attempting to pass q would be an error:

熟悉基于类的面向对象语言的读者可能会倾向于将 Point 视为基类，将 ColoredPoint 视为子类或派生类，或者将这些类型之间的关系解释为 ColoredPoint "是一个" Point。但这是错误的。注意上面对 Distance 的调用。Distance 有一个 Point 类型的参数，而 q 不是 Point，所以尽管 q 确实有一个该类型的嵌入字段，我们必须显式地选择它。尝试传递 q 将会出错：

```go
p.Distance(q) // compile error: cannot use q (ColoredPoint) as Point
```

A ColoredPoint is not a Point, but it "has a" Point, and it has two additional methods Distance and ScaleBy promoted from Point. If you prefer to think in terms of implementation, the embedded field instructs the compiler to generate additional wrapper methods that delegate to the declared methods, equivalent to these:

ColoredPoint 不是 Point，但它"有一个" Point，并且它有两个从 Point 提升的额外方法 Distance 和 ScaleBy。如果你更喜欢从实现的角度来思考，嵌入字段指示编译器生成额外的包装方法，这些方法委托给声明的方法，等价于：

```go
func (p ColoredPoint) Distance(q Point) float64 {
    return p.Point.Distance(q)
}

func (p *ColoredPoint) ScaleBy(factor float64) {
    p.Point.ScaleBy(factor)
}
```

When Point.Distance is called by the first of these wrapper methods, its receiver value is p.Point, not p, and there is no way for the method to access the ColoredPoint in which the Point is embedded.

当第一个包装方法调用 Point.Distance 时，它的接收者值是 p.Point，而不是 p，并且该方法无法访问嵌入了 Point 的 ColoredPoint。

The type of an anonymous field may be a pointer to a named type, in which case fields and methods are promoted indirectly from the pointed-to object. Adding another level of indirection lets us share common structures and vary the relationships between objects dynamically. The declaration of ColoredPoint below embeds a *Point:

匿名字段的类型可以是指向命名类型的指针，在这种情况下，字段和方法从指向的对象间接提升。添加另一层间接引用让我们能够共享公共结构并动态改变对象之间的关系。下面的 ColoredPoint 声明嵌入了一个 *Point：

```go
type ColoredPoint struct {
    *Point
    Color color.RGBA
}

p := ColoredPoint{&Point{1, 1}, red}
q := ColoredPoint{&Point{5, 4}, blue}
fmt.Println(p.Distance(*q.Point)) // "5"
q.Point = p.Point // p and q now share the same Point
p.ScaleBy(2)
fmt.Println(*p.Point, *q.Point) // "{2 2} {2 2}"
```

A struct type may have more than one anonymous field. Had we declared ColoredPoint as

一个结构体类型可以有多个匿名字段。如果我们将 ColoredPoint 声明为：

```go
type ColoredPoint struct {
    Point
    color.RGBA
}
```

then a value of this type would have all the methods of Point, all the methods of RGBA, and any additional methods declared on ColoredPoint directly. When the compiler resolves a selector such as p.ScaleBy to a method, it first looks for a directly declared method named ScaleBy, then for methods promoted once from ColoredPoint's embedded fields, then for methods promoted twice from embedded fields within Point and RGBA, and so on. The compiler reports an error if the selector was ambiguous because two methods were promoted from the same rank.

那么该类型的值将拥有 Point 的所有方法、RGBA 的所有方法，以及直接在 ColoredPoint 上声明的任何额外方法。当编译器将诸如 p.ScaleBy 这样的选择器解析为方法时，它首先查找直接声明的名为 ScaleBy 的方法，然后查找从 ColoredPoint 的嵌入字段提升一次的方法，然后查找从 Point 和 RGBA 内的嵌入字段提升两次的方法，以此类推。如果选择器因为两个方法从相同层级提升而产生歧义，编译器将报告错误。

Methods can be declared only on named types (like Point) and pointers to them (*Point), but thanks to embedding, it's possible and sometimes useful for unnamed struct types to have methods too.

方法只能在命名类型（如 Point）及其指针（*Point）上声明，但得益于嵌入，无名结构体类型也可以拥有方法，这有时很有用。

Here's a nice trick to illustrate. This example shows part of a simple cache implemented using two package-level variables, a mutex (§9.2) and the map that it guards:

这里有一个很好的技巧来说明。这个例子展示了使用两个包级变量实现的简单缓存的一部分，一个互斥锁（§9.2）和它保护的映射：

```go
var (
    mu sync.Mutex // guards mapping
    mapping = make(map[string]string)
)

func Lookup(key string) string {
    mu.Lock()
    v := mapping[key]
    mu.Unlock()
    return v
}
```

The version below is functionally equivalent but groups together the two related variables in a single package-level variable, cache:

下面的版本在功能上是等价的，但将两个相关变量组合在一个包级变量 cache 中：

```go
var cache = struct {
    sync.Mutex
    mapping map[string]string
}{
    mapping: make(map[string]string),
}

func Lookup(key string) string {
    cache.Lock()
    v := cache.mapping[key]
    cache.Unlock()
    return v
}
```

The new variable gives more expressive names to the variables related to the cache, and because the sync.Mutex field is embedded within it, its Lock and Unlock methods are promoted to the unnamed struct type, allowing us to lock the cache with a self-explanatory syntax.

新变量为与缓存相关的变量提供了更有表现力的名称，由于 sync.Mutex 字段嵌入其中，它的 Lock 和 Unlock 方法被提升到无名结构体类型，允许我们使用自解释的语法来锁定缓存。

## 6.4. Method Values and Expressions

## 6.4. 方法值和方法表达式

Usually we select and call a method in the same expression, as in p.Distance(), but it's possible to separate these two operations. The selector p.Distance yields a method value, a function that binds a method (Point.Distance) to a specific receiver value p. This function can then be invoked without a receiver value; it needs only the non-receiver arguments.

通常我们在同一个表达式中选择并调用方法，如 p.Distance()，但也可以将这两个操作分开。选择器 p.Distance 产生一个方法值（method value），这是一个将方法（Point.Distance）绑定到特定接收者值 p 的函数。然后可以在没有接收者值的情况下调用该函数；它只需要非接收者参数。

```go
p := Point{1, 2}
q := Point{4, 6}

distanceFromP := p.Distance        // method value
fmt.Println(distanceFromP(q))      // "5"
var origin Point                   // {0, 0}
fmt.Println(distanceFromP(origin)) // "2.23606797749979", √5

scaleP := p.ScaleBy // method value
scaleP(2)           // p becomes (2, 4)
scaleP(3)           // then (6, 12)
scaleP(10)          // then (60, 120)
```

Method values are useful when a package's API calls for a function value, and the client's desired behavior for that function is to call a method on a specific receiver. For example, the function time.AfterFunc calls a function value after a specified delay. This program uses it to launch the rocket r after 10 seconds:

当包的 API 需要一个函数值，而客户端期望的行为是在特定接收者上调用方法时，方法值很有用。例如，函数 time.AfterFunc 在指定延迟后调用一个函数值。这个程序使用它在 10 秒后发射火箭 r：

```go
type Rocket struct { /* ... */ }
func (r *Rocket) Launch() { /* ... */ }
r := new(Rocket)
time.AfterFunc(10 * time.Second, func() { r.Launch() })
```

The method value syntax is shorter:

方法值语法更简洁：

```go
time.AfterFunc(10 * time.Second, r.Launch)
```

Related to the method value is the method expression. When calling a method, as opposed to an ordinary function, we must supply the receiver in a special way using the selector syntax. A method expression, written T.f or (*T).f where T is a type, yields a function value with a regular first parameter taking the place of the receiver, so it can be called in the usual way.

与方法值相关的是方法表达式（method expression）。当调用方法时，与普通函数不同，我们必须使用选择器语法以特殊方式提供接收者。方法表达式，写作 T.f 或 (*T).f（其中 T 是类型），产生一个函数值，其常规的第一个参数代替了接收者，因此可以按通常方式调用。

```go
p := Point{1, 2}
q := Point{4, 6}

distance := Point.Distance   // method expression
fmt.Println(distance(p, q))  // "5"
fmt.Printf("%T\n", distance) // "func(Point, Point) float64"

scale := (*Point).ScaleBy
scale(&p, 2)
fmt.Println(p)            // "{2 4}"
fmt.Printf("%T\n", scale) // "func(*Point, float64)"
```

Method expressions can be helpful when you need a value to represent a choice among several methods belonging to the same type so that you can call the chosen method with many different receivers. In the following example, the variable op represents either the addition or the subtraction method of type Point, and Path.TranslateBy calls it for each point in the Path:

当你需要一个值来表示属于同一类型的多个方法中的选择，以便可以用许多不同的接收者调用所选方法时，方法表达式会很有帮助。在下面的例子中，变量 op 表示 Point 类型的加法或减法方法，Path.TranslateBy 为 Path 中的每个点调用它：

```go
type Point struct{ X, Y float64 }

func (p Point) Add(q Point) Point { return Point{p.X + q.X, p.Y + q.Y} }
func (p Point) Sub(q Point) Point { return Point{p.X - q.X, p.Y - q.Y} }

type Path []Point

func (path Path) TranslateBy(offset Point, add bool) {
    var op func(p, q Point) Point
    if add {
        op = Point.Add
    } else {
        op = Point.Sub
    }
    for i := range path {
        // Call either path[i].Add(offset) or path[i].Sub(offset).
        path[i] = op(path[i], offset)
    }
}
```

## 6.5. Example: Bit Vector Type

## 6.5. 示例：位向量类型

Sets in Go are usually implemented as a map[T]bool, where T is the element type. A set represented by a map is very flexible but, for certain problems, a specialized representation may outperform it. For example, in domains such as dataflow analysis where set elements are small non-negative integers, sets have many elements, and set operations like union and intersection are common, a bit vector is ideal.

Go 中的集合通常实现为 map[T]bool，其中 T 是元素类型。用映射表示的集合非常灵活，但对于某些问题，专门的表示方法可能性能更好。例如，在数据流分析这样的领域中，集合元素是小的非负整数，集合有许多元素，并且并集和交集等集合操作很常见，位向量是理想的选择。

A bit vector uses a slice of unsigned integer values or "words," each bit of which represents a possible element of the set. The set contains i if the i-th bit is set. The following program demonstrates a simple bit vector type with three methods:

位向量使用无符号整数值或"字"的切片，其中每个位表示集合的一个可能元素。如果第 i 位被设置，则集合包含 i。以下程序演示了一个具有三个方法的简单位向量类型：

```go
gopl.io/ch6/intset

// An IntSet is a set of small non-negative integers.
// Its zero value represents the empty set.
type IntSet struct {
    words []uint64
}

// Has reports whether the set contains the non-negative value x.
func (s *IntSet) Has(x int) bool {
    word, bit := x/64, uint(x%64)
    return word < len(s.words) && s.words[word]&(1<<bit) != 0
}

// Add adds the non-negative value x to the set.
func (s *IntSet) Add(x int) {
    word, bit := x/64, uint(x%64)
    for word >= len(s.words) {
        s.words = append(s.words, 0)
    }
    s.words[word] |= 1 << bit
}

// UnionWith sets s to the union of s and t.
func (s *IntSet) UnionWith(t *IntSet) {
    for i, tword := range t.words {
        if i < len(s.words) {
            s.words[i] |= tword
        } else {
            s.words = append(s.words, tword)
        }
    }
}
```

Since each word has 64 bits, to locate the bit for x, we use the quotient x/64 as the word index and the remainder x%64 as the bit index within that word. The UnionWith operation uses the bitwise OR operator | to compute the union 64 elements at a time. (We'll revisit the choice of 64-bit words in Exercise 6.5.)

由于每个字有 64 位，为了定位 x 的位，我们使用商 x/64 作为字索引，余数 x%64 作为该字内的位索引。UnionWith 操作使用按位 OR 运算符 | 一次计算 64 个元素的并集。（我们将在练习 6.5 中重新讨论选择 64 位字的问题。）

This implementation lacks many desirable features, some of which are posed as exercises below, but one is hard to live without: a way to print an IntSet as a string. Let's give it a String method as we did with Celsius in Section 2.5:

这个实现缺少许多理想的特性，其中一些作为下面的练习，但有一个是必不可少的：将 IntSet 打印为字符串的方法。让我们像在 2.5 节中为 Celsius 所做的那样，给它一个 String 方法：

```go
// String returns the set as a string of the form "{1 2 3}".
func (s *IntSet) String() string {
    var buf bytes.Buffer
    buf.WriteByte('{')
    for i, word := range s.words {
        if word == 0 {
            continue
        }
        for j := 0; j < 64; j++ {
            if word&(1<<uint(j)) != 0 {
                if buf.Len() > len("{") {
                    buf.WriteByte(' ')
                }
                fmt.Fprintf(&buf, "%d", 64*i+j)
            }
        }
    }
    buf.WriteByte('}')
    return buf.String()
}
```

Notice the similarity of the String method above with intsToString in Section 3.5.4; bytes.Buffer is often used this way in String methods. The fmt package treats types with a String method specially so that values of complicated types can display themselves in a user-friendly manner. Instead of printing the raw representation of the value (a struct in this case), fmt calls the String method. The mechanism relies on interfaces and type assertions, which we'll explain in Chapter 7.

注意上面的 String 方法与 3.5.4 节中的 intsToString 的相似性；bytes.Buffer 经常在 String 方法中以这种方式使用。fmt 包特殊对待具有 String 方法的类型，以便复杂类型的值可以以用户友好的方式显示自己。fmt 不是打印值的原始表示（在这种情况下是结构体），而是调用 String 方法。该机制依赖于接口和类型断言，我们将在第 7 章中解释。

We can now demonstrate IntSet in action:

我们现在可以演示 IntSet 的实际应用：

```go
var x, y IntSet
x.Add(1)
x.Add(144)
x.Add(9)
fmt.Println(x.String()) // "{1 9 144}"

y.Add(9)
y.Add(42)
fmt.Println(y.String()) // "{9 42}"

x.UnionWith(&y)
fmt.Println(x.String()) // "{1 9 42 144}"
fmt.Println(x.Has(9), x.Has(123)) // "true false"
```

A word of caution: we declared String and Has as methods of the pointer type *IntSet not out of necessity, but for consistency with the other two methods, which need a pointer receiver because they assign to s.words. Consequently, an IntSet value does not have a String method, occasionally leading to surprises like this:

需要注意的是：我们将 String 和 Has 声明为指针类型 *IntSet 的方法，不是出于必要，而是为了与其他两个方法保持一致，后者需要指针接收者，因为它们要对 s.words 赋值。因此，IntSet 值没有 String 方法，偶尔会导致这样的意外：

```go
fmt.Println(&x)         // "{1 9 42 144}"
fmt.Println(x.String()) // "{1 9 42 144}"
fmt.Println(x)          // "{[4398046511618 0 65536]}"
```

In the first case, we print an *IntSet pointer, which does have a String method. In the second case, we call String() on an IntSet variable; the compiler inserts the implicit & operation, giving us a pointer, which has the String method. But in the third case, because the IntSet value does not have a String method, fmt.Println prints the representation of the struct instead. It's important not to forget the & operator. Making String a method of IntSet, not *IntSet, might be a good idea, but this is a case-by-case judgment.

在第一种情况下，我们打印一个 *IntSet 指针，它确实有 String 方法。在第二种情况下，我们在 IntSet 变量上调用 String()；编译器插入隐式的 & 操作，给我们一个指针，它有 String 方法。但在第三种情况下，因为 IntSet 值没有 String 方法，fmt.Println 打印结构体的表示。重要的是不要忘记 & 运算符。让 String 成为 IntSet 而不是 *IntSet 的方法可能是个好主意，但这需要具体情况具体分析。

Exercise 6.1: Implement these additional methods:

练习 6.1：实现这些额外的方法：

```go
func (*IntSet) Len() int      // return the number of elements
func (*IntSet) Remove(x int)  // remove x from the set
func (*IntSet) Clear()        // remove all elements from the set
func (*IntSet) Copy() *IntSet // return a copy of the set
```

Exercise 6.2: Define a variadic (*IntSet).AddAll(...int) method that allows a list of values to be added, such as s.AddAll(1, 2, 3).

练习 6.2：定义一个可变参数的 (*IntSet).AddAll(...int) 方法，允许添加一个值列表，例如 s.AddAll(1, 2, 3)。

Exercise 6.3: (*IntSet).UnionWith computes the union of two sets using |, the word-parallel bitwise OR operator. Implement methods for IntersectWith, DifferenceWith, and SymmetricDifference for the corresponding set operations. (The symmetric difference of two sets contains the elements present in one set or the other but not both.)

练习 6.3：(*IntSet).UnionWith 使用 |（字并行按位 OR 运算符）计算两个集合的并集。为相应的集合操作实现 IntersectWith、DifferenceWith 和 SymmetricDifference 方法。（两个集合的对称差包含存在于一个集合或另一个集合中但不同时存在于两者中的元素。）

Exercise 6.4: Add a method Elems that returns a slice containing the elements of the set, suitable for iterating over with a range loop.

练习 6.4：添加一个 Elems 方法，返回包含集合元素的切片，适合用 range 循环进行迭代。

Exercise 6.5: The type of each word used by IntSet is uint64, but 64-bit arithmetic may be inefficient on a 32-bit platform. Modify the program to use the uint type, which is the most efficient unsigned integer type for the platform. Instead of dividing by 64, define a constant holding the effective size of uint in bits, 32 or 64. You can use the perhaps too-clever expression 32 << (^uint(0) >> 63) for this purpose.

练习 6.5：IntSet 使用的每个字的类型是 uint64，但 64 位算术在 32 位平台上可能效率低下。修改程序以使用 uint 类型，这是平台上最高效的无符号整数类型。不要除以 64，而是定义一个常量来保存 uint 的有效位数大小，32 或 64。为此，你可以使用可能过于巧妙的表达式 32 << (^uint(0) >> 63)。

## 6.6. Encapsulation

## 6.6. 封装

A variable or method of an object is said to be encapsulated if it is inaccessible to clients of the object. Encapsulation, sometimes called information hiding, is a key aspect of object-oriented programming.

如果对象的变量或方法对对象的客户端不可访问，则称其为封装的。封装（有时称为信息隐藏）是面向对象编程的一个关键方面。

Go has only one mechanism to control the visibility of names: capitalized identifiers are exported from the package in which they are defined, and uncapitalized names are not. The same mechanism that limits access to members of a package also limits access to the fields of a struct or the methods of a type. As a consequence, to encapsulate an object, we must make it a struct.

Go 只有一种机制来控制名称的可见性：大写的标识符从定义它们的包中导出，而小写的名称则不会。限制对包成员访问的相同机制也限制了对结构体字段或类型方法的访问。因此，要封装一个对象，我们必须使其成为结构体。

That's the reason the IntSet type from the previous section was declared as a struct type even though it has only a single field:

这就是前一节中的 IntSet 类型被声明为结构体类型的原因，尽管它只有一个字段：

```go
type IntSet struct {
    words []uint64
}
```

We could instead define IntSet as a slice type as follows, though of course we'd have to replace each occurrence of s.words by *s in its methods:

我们可以将 IntSet 定义为切片类型，如下所示，当然我们必须在其方法中将每次出现的 s.words 替换为 *s：

```go
type IntSet []uint64
```

Although this version of IntSet would be essentially equivalent, it would allow clients from other packages to read and modify the slice directly. Put another way, whereas the expression *s could be used in any package, s.words may appear only in the package that defines IntSet.

尽管这个版本的 IntSet 本质上是等价的，但它将允许其他包的客户端直接读取和修改切片。换句话说，虽然表达式 *s 可以在任何包中使用，但 s.words 只能出现在定义 IntSet 的包中。

Another consequence of this name-based mechanism is that the unit of encapsulation is the package, not the type as in many other languages. The fields of a struct type are visible to all code within the same package. Whether the code appears in a function or a method makes no difference.

这种基于名称的机制的另一个结果是，封装的单位是包，而不是像许多其他语言那样是类型。结构体类型的字段对同一包内的所有代码都是可见的。代码出现在函数中还是方法中没有区别。

Encapsulation provides three benefits. First, because clients cannot directly modify the object's variables, one need inspect fewer statements to understand the possible values of those variables.

封装提供了三个好处。首先，因为客户端不能直接修改对象的变量，所以只需要检查较少的语句就能理解这些变量的可能值。

Second, hiding implementation details prevents clients from depending on things that might change, which gives the designer greater freedom to evolve the implementation without breaking API compatibility.

其次，隐藏实现细节可以防止客户端依赖可能会改变的内容，这给设计者更大的自由来演进实现而不破坏 API 兼容性。

As an example, consider the bytes.Buffer type. It is frequently used to accumulate very short strings, so it is a profitable optimization to reserve a little extra space in the object to avoid memory allocation in this common case. Since Buffer is a struct type, this space takes the form of an extra field of type [64]byte with an uncapitalized name. When this field was added, because it was not exported, clients of Buffer outside the bytes package were unaware of any change except improved performance. Buffer and its Grow method are shown below, simplified for clarity:

作为例子，考虑 bytes.Buffer 类型。它经常用于累积非常短的字符串，因此在对象中预留一点额外空间以避免在这种常见情况下的内存分配是一个有益的优化。由于 Buffer 是结构体类型，这个空间采用类型为 [64]byte 的额外字段的形式，具有小写名称。当添加此字段时，由于它未导出，bytes 包外部的 Buffer 客户端除了性能改进外，对任何更改都不知情。Buffer 及其 Grow 方法如下所示，为清晰起见进行了简化：

```go
type Buffer struct {
    buf     []byte
    initial [64]byte
    /* ... */
}

// Grow expands the buffer's capacity, if necessary,
// to guarantee space for another n bytes. [...]
func (b *Buffer) Grow(n int) {
    if b.buf == nil {
        b.buf = b.initial[:0] // use preallocated space initially
    }
    if len(b.buf)+n > cap(b.buf) {
        buf := make([]byte, b.Len(), 2*cap(b.buf) + n)
        copy(buf, b.buf)
        b.buf = buf
    }
}
```

The third benefit of encapsulation, and in many cases the most important, is that it prevents clients from setting an object's variables arbitrarily. Because the object's variables can be set only by functions in the same package, the author of that package can ensure that all those functions maintain the object's internal invariants. For example, the Counter type below permits clients to increment the counter or to reset it to zero, but not to set it to some arbitrary value:

封装的第三个好处，在许多情况下也是最重要的，是它防止客户端任意设置对象的变量。因为对象的变量只能由同一包中的函数设置，该包的作者可以确保所有这些函数都维护对象的内部不变量。例如，下面的 Counter 类型允许客户端递增计数器或将其重置为零，但不能将其设置为任意值：

```go
type Counter struct { n int }
func (c *Counter) N() int     { return c.n }
func (c *Counter) Increment() { c.n++ }
func (c *Counter) Reset()     { c.n = 0 }
```

Functions that merely access or modify internal values of a type, such as the methods of the Logger type from log package, below, are called getters and setters. However, when naming a getter method, we usually omit the Get prefix. This preference for brevity extends to all methods, not just field accessors, and to other redundant prefixes as well, such as Fetch, Find, and Lookup.

仅访问或修改类型内部值的函数，如下面来自 log 包的 Logger 类型的方法，称为 getter 和 setter。然而，在命名 getter 方法时，我们通常省略 Get 前缀。这种简洁性偏好扩展到所有方法，而不仅仅是字段访问器，也扩展到其他冗余前缀，如 Fetch、Find 和 Lookup。

```go
package log
type Logger struct {
    flags  int
    prefix string
    // ...
}
func (l *Logger) Flags() int
func (l *Logger) SetFlags(flag int)
func (l *Logger) Prefix() string
func (l *Logger) SetPrefix(prefix string)
```

Go style does not forbid exported fields. Of course, once exported, a field cannot be unexported without an incompatible change to the API, so the initial choice should be deliberate and should consider the complexity of the invariants that must be maintained, the likelihood of future changes, and the quantity of client code that would be affected by a change.

Go 风格并不禁止导出字段。当然，一旦导出，字段就不能在不进行不兼容的 API 更改的情况下取消导出，因此初始选择应该是深思熟虑的，应该考虑必须维护的不变量的复杂性、未来更改的可能性以及受更改影响的客户端代码的数量。

Encapsulation is not always desirable. By revealing its representation as an int64 number of nanoseconds, time.Duration lets us use all the usual arithmetic and comparison operations with durations, and even to define constants of this type:

封装并不总是可取的。通过将其表示为 int64 纳秒数，time.Duration 让我们可以对持续时间使用所有常用的算术和比较操作，甚至可以定义此类型的常量：

```go
const day = 24 * time.Hour
fmt.Println(day.Seconds()) // "86400"
```

As another example, contrast IntSet with the geometry.Path type from the beginning of this chapter. Path was defined as a slice type, allowing its clients to construct instances using the slice literal syntax, to iterate over its points using a range loop, and so on, whereas these operations are denied to clients of IntSet.

作为另一个例子，将 IntSet 与本章开头的 geometry.Path 类型进行对比。Path 被定义为切片类型，允许其客户端使用切片字面量语法构造实例，使用 range 循环迭代其点等等，而这些操作对 IntSet 的客户端是被拒绝的。

Here's the crucial difference: geometry.Path is intrinsically a sequence of points, no more and no less, and we don't foresee adding new fields to it, so it makes sense for the geometry package to reveal that Path is a slice. In contrast, an IntSet merely happens to be represented as a []uint64 slice. It could have been represented using []uint, or something completely different for sets that are sparse or very small, and it might perhaps benefit from additional features like an extra field to record the number of elements in the set. For these reasons, it makes sense for IntSet to be opaque.

这是关键的区别：geometry.Path 本质上是一个点序列，不多也不少，我们不预见会向其添加新字段，因此 geometry 包揭示 Path 是一个切片是有意义的。相比之下，IntSet 只是碰巧表示为 []uint64 切片。它可以使用 []uint 表示，或者对于稀疏或非常小的集合使用完全不同的表示，并且可能会从额外的功能中受益，例如记录集合中元素数量的额外字段。出于这些原因，IntSet 是不透明的是有意义的。

In this chapter, we learned how to associate methods with named types, and how to call those methods. Although methods are crucial to object-oriented programming, they're only half the picture. To complete it, we need interfaces, the subject of the next chapter.

在本章中，我们学习了如何将方法与命名类型关联，以及如何调用这些方法。尽管方法对面向对象编程至关重要，但它们只是整体的一半。要完成整体，我们需要接口，这是下一章的主题。