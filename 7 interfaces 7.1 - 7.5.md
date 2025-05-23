好的，这是更新后的译文，已根据之前的校对意见进行了修正，并移除了“校对意见”和“我的解读”部分。

---

**第7章 接口 (Interfaces)**

Interface types express generalizations or abstractions about the behaviors of other types. [cite: 2] By generalizing, interfaces let us write functions that are more flexible and adaptable because they are not tied to the details of one particular implementation. [cite: 2] Many object-oriented languages have some notion of interfaces, but what makes Go's interfaces so distinctive is that they are satisfied implicitly. [cite: 3] In other words, there's no need to declare all the interfaces that a given concrete type satisfies; [cite: 4] simply possessing the necessary methods is enough. [cite: 5] This design lets you create new interfaces that are satisfied by existing concrete types without changing the existing types, which is particularly useful for types defined in packages that you don't control. [cite: 5] In this chapter, we'll start by looking at the basic mechanics of interface types and their values. [cite: 6] Along the way, we'll study several important interfaces from the standard library. [cite: 7] Many Go programs make as much use of standard interfaces as they do of their own ones. [cite: 8] Finally, we'll look at type assertions (§7.10) and type switches (§7.13) and see how they enable a different kind of generality. [cite: 9]

接口类型（interface types）表达了关于其他类型行为的泛化（generalizations）或抽象（abstractions） 。[cite: 2] 通过泛化，接口使我们能够编写更为灵活和适应性更强的函数，因为它们不依赖于某一特定实现的细节 。[cite: 2] 许多面向对象的语言都有某种接口的概念，但Go语言接口的显著特点在于它们是被**隐式满足**的（satisfied implicitly） 。[cite: 3] 换句话说，无需声明一个给定的具体类型（concrete type）满足了哪些接口；[cite: 4] 仅仅拥有必要的方法就足够了 。[cite: 5] 这种设计允许你创建新的接口，这些接口可以被现有的具体类型满足而无需改变现有类型，这对于那些定义在你无法控制的包中的类型尤其有用 。[cite: 5]

在本章中，我们将首先考察接口类型及其值的基本机制 。[cite: 6] 在此过程中，我们将学习标准库中的几个重要接口 。[cite: 7] 许多Go程序对其标准接口的运用，与对其自定义接口的运用同样频繁 。[cite: 8] 最后，我们将研究类型断言（type assertions）（§7.10）和类型分支（type switches）（§7.13），并了解它们如何促成一种不同类型的通用性（generality） 。[cite: 9]

---

**7.1. 接口作为合约 (Interfaces as Contracts)**

All the types we've looked at so far have been concrete types. [cite: 10] A concrete type specifies the exact representation of its values and exposes the intrinsic operations of that representation, such as arithmetic for numbers, or indexing, append, and range for slices. [cite: 11] A concrete type may also provide additional behaviors through its methods. [cite: 12] When you have a value of a concrete type, you know exactly what it is and what you can do with it. [cite: 13] There is another kind of type in Go called an interface type. [cite: 14] An interface is an abstract type. [cite: 14] It doesn't expose the representation or internal structure of its values, or the set of basic operations they support; it reveals only some of their methods. [cite: 15] When you have a value of an interface type, you know nothing about what it is; [cite: 17] you know only what it can do, or more precisely, what behaviors are provided by its methods. [cite: 18]

到目前为止，我们所考察的所有类型都是具体类型（concrete types） 。[cite: 10] 具体类型指定了其值的确切表示（exact representation），并公开了该表示的固有操作（intrinsic operations），例如数字的算术运算，或切片的索引、追加（append）和遍历（range） 。[cite: 11] 具体类型也可以通过其方法提供额外的行为 。[cite: 12] 当你拥有一个具体类型的值时，你就确切地知道它是什么以及你能用它做什么 。[cite: 13]

Go语言中还有另一种类型，称为接口类型（interface type）。[cite: 14] 接口是一种抽象类型（abstract type） 。[cite: 14] 它不暴露其值的表示或内部结构，也不暴露它们支持的基本操作集；它仅仅揭示了它们的某些方法 。[cite: 15] 当你拥有一个接口类型的值时，你对它是什么（what it is）一无所知；[cite: 17] 你只知道它能做什么（what it can do），或者更准确地说，它的方法提供了哪些行为 。[cite: 18]

Throughout the book, we've been using two similar functions for string formatting: fmt.Printf, which writes the result to the standard output (a file), and fmt.Sprintf, which returns the result as a string. [cite: 19] It would be unfortunate if the hard part, formatting the result, had to be duplicated because of these superficial differences in how the result is used. [cite: 20] Thanks to interfaces, it does not. [cite: 21] Both of these functions are, in effect, wrappers around a third function, fmt.Fprintf, that is agnostic about what happens to the result it computes: [cite: 21]

在本书中，我们一直在使用两个相似的函数进行字符串格式化：`fmt.Printf`，它将结果写入标准输出（一个文件）；以及`fmt.Sprintf`，它将结果作为字符串返回 。[cite: 19] 如果因为这些在使用结果上的表面差异（superficial differences），而不得不重复格式化结果这个困难的部分，那将是十分遗憾的 。[cite: 20] 得益于接口，这并没有发生 。[cite: 21] 这两个函数实际上都是第三个函数`fmt.Fprintf`的封装（wrappers），`fmt.Fprintf`对其计算结果的处理方式是无关乎具体细节的（agnostic） 。[cite: 21]

```go
package fmt

func Fprintf(w io.Writer, format string, args...interface{}) (int, error)

func Printf(format string, args...interface{}) (int, error) {
    return Fprintf(os.Stdout, format, args...)
}

func Sprintf(format string, args...interface{}) string {
    var buf bytes.Buffer
    Fprintf(&buf, format, args...)
    return buf.String()
}
```
```go
package fmt

func Fprintf(w io.Writer, format string, args ...interface{}) (int, error)

func Printf(format string, args ...interface{}) (int, error) {
    return Fprintf(os.Stdout, format, args...)
}

func Sprintf(format string, args ...interface{}) string {
    var buf bytes.Buffer
    Fprintf(&buf, format, args...)
    return buf.String()
}
```

The F prefix of Fprintf stands for file and indicates that the formatted output should be written to the file provided as the first argument. In the Printf case, the argument, os.Stdout, is an *os.File. In the Sprintf case, however, the argument is not a file, though it superficially resembles one: &buf is a pointer to a memory buffer to which bytes can be written. The first parameter of Fprintf is not a file either. It's an io.Writer, which is an interface type with the following declaration: 

`Fprintf`的`F`前缀代表文件（file），并指明格式化的输出应被写入作为第一个参数提供的文件 。[cite: 22] 在`Printf`的例子中，参数`os.Stdout`是一个`*os.File` 。[cite: 23] 然而，在`Sprintf`的例子中，参数并非文件，尽管它表面上看起来相似：`&buf`是一个指向内存缓冲区的指针，字节可以被写入该缓冲区 。[cite: 24]
`Fprintf`的第一个参数也不是文件。[cite: 25] 它是一个`io.Writer`，这是一个接口类型，其声明如下 ：[cite: 25]

```go
package io

// Writer is the interface that wraps the basic Write method.
type Writer interface {
    // Write writes len(p) bytes from p to the underlying data stream.
    // It returns the number of bytes written from p (0 <= n <= len(p))
    // and any error encountered that caused the write to stop early.
    // Write must return a non-nil error if it returns n < len(p).
    // Write must not modify the slice data, even temporarily.
    //
    // Implementations must not retain p.
    Write(p[]byte) (n int, err error)
}
```
```go
package io

// Writer is the interface that wraps the basic Write method.
type Writer interface {
    // Write writes len(p) bytes from p to the underlying data stream.
    // It returns the number of bytes written from p (0 <= n <= len(p))
    // and any error encountered that caused the write to stop early.
    // Write must return a non-nil error if it returns n < len(p).
    // Write must not modify the slice data, even temporarily.
    //
    // Implementations must not retain p.
    Write(p []byte) (n int, err error)
}
```

The io.Writer interface defines the contract between Fprintf and its callers. On the one hand, the contract requires that the caller provide a value of a concrete type like *os.File or *bytes.Buffer that has a method called Write with the appropriate signature and behavior. On the other hand, the contract guarantees that Fprintf will do its job given any value that satisfies the io.Writer interface. Fprintf may not assume that it is writing to a file or to memory, only that it can call Write. 

`io.Writer`接口定义了`Fprintf`与其调用者之间的合约（contract） 。[cite: 32] 一方面，该合约要求调用者提供一个具体类型（如`*os.File`或`*bytes.Buffer`）的值，该类型必须拥有一个名为`Write`且具有适当签名和行为的方法 。[cite: 33] 另一方面，该合约保证`Fprintf`在给定任何满足`io.Writer`接口的值时都能完成其工作 。[cite: 34] `Fprintf`可能不会假定它正在写入文件或内存，只假定它可以调用`Write`方法 。[cite: 35]

Because fmt.Fprintf assumes nothing about the representation of the value and relies only on the behaviors guaranteed by the io.Writer contract, we can safely pass a value of any concrete type that satisfies io.Writer as the first argument to fmt.Fprintf. This freedom to substitute one type for another that satisfies the same interface is called substitutability, and is a hallmark of object-oriented programming. 

由于`fmt.Fprintf`对值的表示不作任何假设，并且仅依赖于`io.Writer`合约所保证的行为，因此我们可以安全地将任何满足`io.Writer`的具体类型的值作为第一个参数传递给`fmt.Fprintf` 。[cite: 37, 38] 这种为一个类型替换另一个满足相同接口的类型的自由称为**可替换性**（substitutability），这是面向对象编程的一个标志（hallmark） 。[cite: 38]

Let's test this out using a new type. The Write method of the *ByteCounter type below merely counts the bytes written to it before discarding them. (The conversion is required to make the types of len(p) and *c match in the += assignment statement.) 

让我们用一个新的类型来验证这一点。[cite: 39] 下面`*ByteCounter`类型的`Write`方法仅仅计算写入它的字节数，然后将其丢弃 。[cite: 40] （需要类型转换以确保`len(p)`和`*c`在`+=`赋值语句中类型匹配。）[cite: 41]

_gopl.io/ch7/bytecounter_
```go
type ByteCounter int

func (c *ByteCounter) Write(p []byte) (int, error) {
    *c += ByteCounter(len(p)) // convert int to ByteCounter
    return len(p), nil
}
```
_gopl.io/ch7/bytecounter_
```go
type ByteCounter int

func (c *ByteCounter) Write(p []byte) (int, error) {
    *c += ByteCounter(len(p)) // convert int to ByteCounter
    return len(p), nil
}
```

Since *ByteCounter satisfies the io.Writer contract, we can pass it to Fprintf, which does its string formatting oblivious to this change; the ByteCounter correctly accumulates the length of the result. 

由于`*ByteCounter`满足`io.Writer`合约，我们可以将其传递给`Fprintf`，`Fprintf`会进行字符串格式化而对这种变化一无所知（oblivious）；[cite: 42] `ByteCounter`会正确地累积结果的长度 。[cite: 43]

```go
var c ByteCounter
c.Write([]byte("hello"))
fmt.Println(c) // "5", = len("hello")

c = 0 // reset the counter
var name = "Dolly"
fmt.Fprintf(&c, "hello, %s", name)
fmt.Println(c) // "12", = len("hello, Dolly")
```
```go
var c ByteCounter
c.Write([]byte("hello"))
fmt.Println(c) // "5", = len("hello")

c = 0 // reset the counter
var name = "Dolly"
fmt.Fprintf(&c, "hello, %s", name)
fmt.Println(c) // "12", = len("hello, Dolly")
```

Besides io.Writer, there is another interface of great importance to the fmt package. Fprintf and Fprintln provide a way for types to control how their values are printed. In Section 2.5, we defined a String method for the Celsius type so that temperatures would print as "100°C", and in Section 6.5 we equipped *IntSet with a String method so that sets would be rendered using traditional set notation like "{1 2 3}". Declaring a String method makes a type satisfy one of the most widely used interfaces of all, fmt.Stringer: 

除了`io.Writer`，`fmt`包中还有另一个至关重要的接口 。[cite: 44] `Fprintf`和`Fprintln`为类型提供了一种控制其值如何被打印的方式 。[cite: 45] 在2.5节中，我们为`Celsius`类型定义了一个`String`方法，使得温度可以打印为 "100°C"；在6.5节中，我们为`*IntSet`配备了一个`String`方法，以便集合可以使用传统的集合表示法（如 "{1 2 3}"）进行渲染 。[cite: 46] 声明一个`String`方法会使一个类型满足所有接口中应用最广泛的接口之一：`fmt.Stringer` ：[cite: 47]

```go
package fmt

// The String method is used to print values passed
// as an operand to any format that accepts a string
// or to an unformatted printer such as Print.
type Stringer interface {
    String() string
}
```
```go
package fmt

// The String method is used to print values passed
// as an operand to any format that accepts a string
// or to an unformatted printer such as Print.
type Stringer interface {
    String() string
}
```

We'll explain how the fmt package discovers which values satisfy this interface in Section 7.10. 
Exercise 7.1: Using the ideas from ByteCounter, implement counters for words and for lines. You will find bufio.Scanwords useful. 
Exercise 7.2: Write a function CountingWriter with the signature below that, given an io.Writer, returns a new Writer that wraps the original, and a pointer to an int64 variable that at any moment contains the number of bytes written to the new Writer. 
`func CountingWriter(w io.Writer) (io.Writer, *int64)`
Exercise 7.3: Write a String method for the *tree type in gopl.io/ch4/treesort (§4.4) that reveals the sequence of values in the tree. 

我们将在§7.10解释`fmt`包如何发现哪些值满足此接口 。[cite: 49]

练习 7.1：借鉴ByteCounter的思想，实现单词计数器和行计数器。[cite: 49] 你会发现bufio.ScanWords很有用 。[cite: 49]
练习 7.2：编写一个具有以下签名的函数CountingWriter，它接收一个io.Writer，返回一个新的Writer，该Writer封装了原始的Writer，并返回一个指向int64类型变量的指针，该变量在任何时刻都记录了写入新Writer的字节数 。[cite: 50, 51]
`func CountingWriter(w io.Writer) (io.Writer, *int64)`
练习 7.3：为gopl.io/ch4/treesort (§4.4)中的*tree类型编写一个String方法，用于显示树中值的序列 。[cite: 51]

---

**7.2. 接口类型 (Interface Types)**

An interface type specifies a set of methods that a concrete type must possess to be considered an instance of that interface. The io.Writer type is one of the most widely used interfaces because it provides an abstraction of all the types to which bytes can be written, which includes files, memory buffers, network connections, HTTP clients, archivers, hashers, and so on. The io package defines many other useful interfaces. A Reader represents any type from which you can read bytes, and a Closer is any value that you can close, such as a file or a network connection. (By now you've probably noticed the naming convention for many of Go's single-method interfaces.) 

一个接口类型指定了一组方法，具体类型必须拥有这些方法才能被认为是该接口的一个实例 。[cite: 52] `io.Writer`类型是使用最广泛的接口之一，因为它提供了对所有可以写入字节的类型的抽象，这些类型包括文件、内存缓冲区、网络连接、HTTP客户端、归档器（archivers）、哈希器（hashers）等等 。[cite: 53] `io`包定义了许多其他有用的接口。[cite: 54] `Reader`代表任何可以从中读取字节的类型，而`Closer`是任何可以关闭的值，例如文件或网络连接 。[cite: 54] （到目前为止，你可能已经注意到Go语言中许多单方法接口的命名约定。）[cite: 55]

```go
package io

type Reader interface {
    Read(p[]byte) (n int, err error)
}

type Closer interface {
    Close() error
}
```
```go
package io

type Reader interface {
    Read(p []byte) (n int, err error)
}

type Closer interface {
    Close() error
}
```

Looking farther, we find declarations of new interface types as combinations of existing ones. Here are two examples: 

更进一步，我们会发现将现有接口组合起来声明新的接口类型 。[cite: 56] 这里有两个例子：[cite: 56]

```go
type ReadWriter interface {
    Reader
    Writer
}

type ReadWriteCloser interface {
    Reader
    Writer
    Closer
}
```
```go
type ReadWriter interface {
    Reader
    Writer
}

type ReadWriteCloser interface {
    Reader
    Writer
    Closer
}
```

The syntax used above, which resembles struct embedding, lets us name another interface as a shorthand for writing out all of its methods. This is called embedding an interface. We could have written io.ReadWriter without embedding, albeit less succinctly, like this: 

上面使用的语法，类似于结构体嵌入（struct embedding），允许我们通过指定另一个接口的名称来作为写出其所有方法的简写。[cite: 57] 这被称为**接口嵌入**（embedding an interface） 。[cite: 57] 我们本可以不使用嵌入来编写`io.ReadWriter`，尽管会不那么简洁，像这样：[cite: 57]

```go
type ReadWriter interface {
    Read(p[]byte) (n int, err error)
    Write(p[]byte) (n int, err error)
}
```
```go
type ReadWriter interface {
    Read(p []byte) (n int, err error)
    Write(p []byte) (n int, err error)
}
```

or even using a mixture of the two styles:

甚至可以混合使用两种风格：

```go
type ReadWriter interface {
    Read(p[]byte) (n int, err error)
    Writer
}
```
```go
type ReadWriter interface {
    Read(p []byte) (n int, err error)
    Writer
}
```

All three declarations have the same effect. The order in which the methods appear is immaterial. All that matters is the set of methods. 
Exercise 7.4: The strings.NewReader function returns a value that satisfies the io.Reader interface (and others) by reading from its argument, a string. Implement a simple version of NewReader yourself, and use it to make the HTML parser (§5.2) take input from a string. 
Exercise 7.5: The LimitReader function in the io package accepts an io.Reader r and a number of bytes n, and returns another Reader that reads from r but reports an end-of-file condition after n bytes. Implement it. 
`func LimitReader (r io.Reader, n int64) io.Reader`

所有这三个声明具有相同的效果 。[cite: 59] 方法出现的顺序无关紧要（immaterial）；[cite: 59] 重要的是方法的集合 。[cite: 59]

练习 7.4：strings.NewReader函数返回一个通过从其参数（一个字符串）读取数据来满足io.Reader接口（以及其他接口）的值 。[cite: 60, 61] 请自行实现一个NewReader的简单版本，并用它来使HTML解析器（§5.2）从字符串接收输入 。[cite: 62]
练习 7.5：io包中的LimitReader函数接受一个io.Reader r和一个字节数n，并返回另一个Reader，该Reader从r读取数据，但在读取n个字节后报告文件结束（end-of-file）条件 。[cite: 63, 64] 请实现它 。[cite: 65]
`func LimitReader(r io.Reader, n int64) io.Reader`

---

**7.3. 接口的满足 (Interface Satisfaction)**

A type satisfies an interface if it possesses all the methods the interface requires. For example, an *os.File satisfies io.Reader, Writer, Closer, and ReadWriter. A *bytes.Buffer satisfies Reader, Writer, and ReadWriter, but does not satisfy Closer because it does not have a Close method. As a shorthand, Go programmers often say that a concrete type "is a" particular interface type, meaning that it satisfies the interface. For example, a *bytes.Buffer is an io.Writer; an *os.File is an io.ReadWriter. 

如果一个类型拥有接口所要求的所有方法，那么它就满足该接口 。[cite: 66] 例如，一个`*os.File`满足`io.Reader`、`Writer`、`Closer`和`ReadWriter` 。[cite: 67] 一个`*bytes.Buffer`满足`Reader`、`Writer`和`ReadWriter`，但不满足`Closer`，因为它没有`Close`方法 。[cite: 68] 作为一种简写，Go程序员常说一个具体类型“是”某个特定的接口类型，意指它满足该接口 。[cite: 69] 例如，一个`*bytes.Buffer`是一个`io.Writer`；一个`*os.File`是一个`io.ReadWriter` 。[cite: 70]

The assignability rule (§2.4.2) for interfaces is very simple: an expression may be assigned to an interface only if its type satisfies the interface. So: 
```go
var w io.Writer
w = os.Stdout           // OK: *os.File has Write method
w = new(bytes.Buffer)   // OK: *bytes.Buffer has Write method
// w = time.Second         // compile error: time.Duration lacks Write method

var rwc io.ReadWriteCloser
rwc = os.Stdout           // OK: *os.File has Read, Write, Close methods
// rwc = new(bytes.Buffer) // compile error: *bytes.Buffer lacks Close method
```
接口的赋值规则（§2.4.2）非常简单：一个表达式只有当其类型满足接口时才能赋值给该接口 。[cite: 71] 所以：[cite: 72]
```go
var w io.Writer
w = os.Stdout           // OK: *os.File 拥有 Write 方法
w = new(bytes.Buffer)   // OK: *bytes.Buffer 拥有 Write 方法
// w = time.Second         // 编译错误: time.Duration 缺少 Write 方法

var rwc io.ReadWriteCloser
rwc = os.Stdout           // OK: *os.File 拥有 Read, Write, Close 方法
// rwc = new(bytes.Buffer) // 编译错误: *bytes.Buffer 缺少 Close 方法
```

This rule applies even when the right-hand side is itself an interface: 
```go
w = rwc // OK: io.ReadWriteCloser has Write method
// rwc = w // compile error: io.Writer lacks Close method
```
此规则即使在右侧本身是一个接口时也适用：[cite: 73]
```go
w = rwc // OK: io.ReadWriteCloser 拥有 Write 方法
// rwc = w // 编译错误: io.Writer 缺少 Close 方法
```

Because ReadWriter and ReadWriteCloser include all the methods of Writer, any type that satisfies ReadWriter or ReadWriteCloser necessarily satisfies Writer. 

由于`ReadWriter`和`ReadWriteCloser`包含了`Writer`的所有方法，任何满足`ReadWriter`或`ReadWriteCloser`的类型必然满足`Writer` 。[cite: 75]

Before we go further, we should explain one subtlety in what it means for a type to have a method. Recall from Section 6.2 that for each named concrete type T, some of its methods have a receiver of type T itself whereas others require a *T pointer. Recall also that it is legal to call a *T method on an argument of type T so long as the argument is a variable; the compiler implicitly takes its address. But this is mere syntactic sugar: a value of type T does not possess all the methods that a *T pointer does, and as a result it might satisfy fewer interfaces. 

在我们进一步讨论之前，我们应该阐释一个类型拥有一个方法的含义中的一个微妙之处（subtlety） 。[cite: 76] 回想一下第6.2节，对于每个已命名的具体类型`T`，它的一些方法接收者是类型`T`本身，而另一些则需要一个`*T`指针 。[cite: 77] 还要回想一下，只要参数是一个变量（variable），就可以在类型为`T`的参数上调用`*T`的方法；[cite: 78] 编译器会隐式地获取其地址 。[cite: 79] 但这仅仅是语法糖：类型`T`的值并不拥有`*T`指针所拥有的所有方法，因此它可能满足更少的接口 。[cite: 80]

An example will make this clear. The String method of the IntSet type from Section 6.5 requires a pointer receiver, so we cannot call that method on a non-addressable IntSet value: 
```go
type IntSet struct { /*...*/ }
func (*IntSet) String() string
// var _ = IntSet{}.String() // compile error: String requires *IntSet receiver
```
一个例子将清楚地阐明这一点。[cite: 80] 第6.5节中`IntSet`类型的`String`方法需要一个指针接收者，所以我们不能在一个不可寻址的（non-addressable）`IntSet`值上调用该方法：[cite: 80]
```go
type IntSet struct { /*...*/ }
func (*IntSet) String() string
// var _ = IntSet{}.String() // compile error: String requires *IntSet receiver
```

but we can call it on an IntSet variable: 
```go
var s IntSet
// var _ = s.String() // OK: s is a variable and &s has a String method
```
但是我们可以在一个`IntSet`变量上调用它：[cite: 80]
```go
var s IntSet
// var _ = s.String() // OK：s 是一个变量，&s 拥有 String 方法
```

However, since only *IntSet has a String method, only *IntSet satisfies the fmt.Stringer interface: 
```go
var _ fmt.Stringer = &s // OK
// var _ fmt.Stringer = s  // compile error: IntSet lacks String method
```
然而，由于只有`*IntSet`拥有`String`方法，因此只有`*IntSet`满足`fmt.Stringer`接口：[cite: 81]
```go
var _ fmt.Stringer = &s // OK
// var _ fmt.Stringer = s  // 编译错误：IntSet 缺少 String 方法
```

Section 12.8 includes a program that prints the methods of an arbitrary value, and the godoc -analysis type tool (§10.7.4) displays the methods of each type and the relationship between interfaces and concrete types. 

第12.8节包含一个打印任意值的方法的程序，而`godoc -analysis type`工具（§10.7.4）则显示了每种类型的方法以及接口和具体类型之间的关系 。[cite: 82]

Like an envelope that wraps and conceals the letter it holds, an interface wraps and conceals the concrete type and value that it holds. Only the methods revealed by the interface type may be called, even if the concrete type has others: 
```go
os.Stdout.Write([]byte("hello")) // OK: *os.File has Write method
os.Stdout.Close()                // OK: *os.File has Close method

var w io.Writer
w = os.Stdout
w.Write([]byte("hello")) // OK: io.Writer has Write method
// w.Close()                // compile error: io.Writer lacks Close method
```
就像一个信封包装并隐藏了它所装的信件一样，接口包装并隐藏了它所持有的具体类型和值 。[cite: 82] 只有接口类型所暴露的方法可以被调用，即使具体类型拥有其他方法：[cite: 83]
```go
// os.Stdout.Write([]byte("hello")) // OK: *os.File has Write method
// os.Stdout.Close()                // OK: *os.File has Close method

var w io.Writer
w = os.Stdout
// w.Write([]byte("hello")) // OK: io.Writer has Write method
// w.Close()                // compile error: io.Writer lacks Close method
```

An interface with more methods, such as io.ReadWriter, tells us more about the values it contains, and places greater demands on the types that implement it, than does an interface with fewer methods such as io.Reader. So what does the type interface{}, which has no methods at all, tell us about the concrete types that satisfy it? That's right: nothing. This may seem useless, but in fact the type interface{}, which is called the empty interface type, is indispensable. Because the empty interface type places no demands on the types that satisfy it, we can assign any value to the empty interface. 
```go
var any interface{}
any = true
any = 12.34
any = "hello"
any = map[string]int{"one": 1}
any = new(bytes.Buffer)
```
一个拥有更多方法的接口，例如`io.ReadWriter`，比一个拥有较少方法的接口（例如`io.Reader`）能告诉我们更多关于它所包含的值的信息，并且对实现它的类型提出了更高的要求 。[cite: 85] 那么，没有任何方法的`interface{}`类型，它能告诉我们关于满足它的具体类型的什么信息呢？[cite: 86] 没错：什么也没有（nothing） 。[cite: 87] 这看起来可能毫无用处，但实际上，被称为**空接口类型**（empty interface type）的`interface{}`是不可或缺的（indispensable） 。[cite: 87] 因为空接口类型对其满足的类型没有任何要求，所以我们可以将任何值赋给空接口 。[cite: 88]
```go
var any interface{}
any = true
any = 12.34
any = "hello"
any = map[string]int{"one": 1}
any = new(bytes.Buffer)
```

Although it wasn't obvious, we've been using the empty interface type since the very first example in this book, because it is what allows functions like fmt.Println, or errorf in Section 5.7, to accept arguments of any type. Of course, having created an interface{} value containing a boolean, float, string, map, pointer, or any other type, we can do nothing directly to the value it holds since the interface has no methods. We need a way to get the value back out again. We'll see how to do that using a type assertion in Section 7.10. 

虽然不那么明显，但从本书的第一个例子开始，我们就一直在使用空接口类型，因为它允许像`fmt.Println`这样的函数，或者第5.7节中的`errorf`函数，接受任何类型的参数 。[cite: 89] 当然，一旦我们创建了一个包含布尔值、浮点数、字符串、map、指针或任何其他类型的`interface{}`值，由于该接口没有任何方法，我们无法直接对其持有的值进行任何操作 。[cite: 90] 我们需要一种方法将其值取回（get the value back out again） 。[cite: 91] 我们将在§7.10中看到如何使用类型断言来实现这一点 。[cite: 92]

Since interface satisfaction depends only on the methods of the two types involved, there is no need to declare the relationship between a concrete type and the interfaces it satisfies. That said, it is occasionally useful to document and assert the relationship when it is intended but not otherwise enforced by the program. The declaration below asserts at compile time that a value of type *bytes.Buffer satisfies io.Writer: 
```go
// *bytes.Buffer must satisfy io.Writer
var w io.Writer = new(bytes.Buffer)
```
由于接口的满足仅取决于所涉及的两种类型的方法，因此无需声明具体类型与其满足的接口之间的关系 。[cite: 93] 尽管如此，当这种关系是预期的但程序并未以其他方式强制执行时，记录并断言这种关系有时是很有用的 。[cite: 94] 下面的声明在编译时断言`*bytes.Buffer`类型的值满足`io.Writer`：[cite: 95]
```go
// *bytes.Buffer must satisfy io.Writer
var w io.Writer = new(bytes.Buffer)
```

We needn't allocate a new variable since any value of type *bytes.Buffer will do, even nil, which we write as (*bytes.Buffer)(nil) using an explicit conversion. And since we never intend to refer to w, we can replace it with the blank identifier. Together, these changes give us this more frugal variant: 
```go
// *bytes.Buffer must satisfy io.Writer
var _ io.Writer = (*bytes.Buffer)(nil)
```
我们不必分配一个新的变量，因为任何`*bytes.Buffer`类型的值都可以，即使是`nil`，我们使用显式转换将其写作`(*bytes.Buffer)(nil)` 。[cite: 97] 而且由于我们从不打算引用`w`，我们可以用空白标识符替换它 。[cite: 98] 综合这些改变，我们得到了这个更为 frugal（节俭的，简洁的）变体：[cite: 99]
```go
// *bytes.Buffer must satisfy io.Writer
var _ io.Writer = (*bytes.Buffer)(nil)
```

Non-empty interface types such as io.Writer are most often satisfied by a pointer type, particularly when one or more of the interface methods implies some kind of mutation to the receiver, as the Write method does. A pointer to a struct is an especially common method-bearing type. But pointer types are by no means the only types that satisfy interfaces, and even interfaces with mutator methods may be satisfied by one of Go's other reference types. We've seen examples of slice types with methods (geometry.Path, §6.1) and map types with methods (url.Values, §6.2.1), and later we'll see a function type with methods (http.HandlerFunc, §7.7). Even basic types may satisfy interfaces; as we saw in Section 7.4, time.Duration satisfies fmt.Stringer. 

像`io.Writer`这样的非空接口类型通常由指针类型来满足，特别是当一个或多个接口方法暗示对接收者进行某种修改（mutation）时，正如`Write`方法所做的那样 。[cite: 100] 指向结构体的指针是一种特别常见的拥有方法的类型（method-bearing type） 。[cite: 101] 但指针类型绝不是唯一满足接口的类型，即使是带有修改器方法（mutator methods）的接口也可能由Go的其他引用类型满足 。[cite: 102] 我们已经见过带有方法的切片类型（`geometry.Path`，§6.1）和带有方法的map类型（`url.Values`，§6.2.1）的例子，稍后我们将看到一个带有方法的函数类型（`http.HandlerFunc`，§7.7） 。[cite: 103] 即使是基本类型也可以满足接口；正如我们在§7.4看到的，`time.Duration`满足`fmt.Stringer` 。[cite: 104]

A concrete type may satisfy many unrelated interfaces. Consider a program that organizes or sells digitized cultural artifacts like music, films, and books. It might define the following set of concrete types: 
Album Book Movie Magazine Podcast TVEpisode Track
We can express each abstraction of interest as an interface. Some properties are common to all artifacts, such as a title, a creation date, and a list of creators (authors or artists). 
```go
type Artifact interface {
    Title() string
    Creators() []string
    Created() time.Time
}
```
一个具体类型可以满足许多不相关的接口 。[cite: 105] 考虑一个组织或销售数字化文化制品（如音乐、电影和书籍）的程序 。[cite: 106] 它可能会定义以下具体类型集合：[cite: 107]
Album Book Movie Magazine Podcast TVEpisode Track
我们可以将每个感兴趣的抽象表示为一个接口 。[cite: 108] 某些属性是所有制品共有的，例如标题、创作日期和创作者列表（作者或艺术家） 。[cite: 109]
```go
type Artifact interface {
    Title() string
    Creators() []string
    Created() time.Time
}
```

Other properties are restricted to certain types of artifacts. Properties of the printed word are relevant only to books and magazines, whereas only movies and TV episodes have a screen resolution. 
```go
type Text interface {
    Pages() int
    Words() int
    PageSize() int
}

type Audio interface {
    Stream() (io.ReadCloser, error)
    RunningTime() time.Duration
    Format() string // e.g., "MP3", "WAV"
}

type Video interface {
    Stream() (io.ReadCloser, error)
    RunningTime() time.Duration
    Format() string // e.g., "MP4", "WMV"
    Resolution() (x, y int)
}
```
其他属性则仅限于特定类型的制品 。[cite: 110] 印刷品的属性仅与书籍和杂志相关，而只有电影和电视剧集才有屏幕分辨率 。[cite: 111]
```go
type Text interface {
    Pages() int
    Words() int
    PageSize() int
}

type Audio interface {
    Stream() (io.ReadCloser, error)
    RunningTime() time.Duration
    Format() string // e.g., "MP3", "WAV"
}

type Video interface {
    Stream() (io.ReadCloser, error)
    RunningTime() time.Duration
    Format() string // e.g., "MP4", "WMV"
    Resolution() (x, y int)
}
```

These interfaces are but one useful way to group related concrete types together and express the facets they share in common. We may discover other groupings later. For example, if we find we need to handle Audio and Video items in the same way, we can define a Streamer interface to represent their common aspects without changing any existing type declarations. 
```go
type Streamer interface {
    Stream() (io.ReadCloser, error)
    RunningTime() time.Duration
    Format() string
}
```
这些接口只是将相关的具体类型组合在一起并表达它们共同特性（facets）的一种有用方式 。[cite: 114] 我们稍后可能会发现其他分组 。[cite: 114] 例如，如果我们发现需要以相同的方式处理`Audio`和`Video`项目，我们可以定义一个`Streamer`接口来表示它们的共同方面，而无需更改任何现有的类型声明 。[cite: 115]
```go
type Streamer interface {
    Stream() (io.ReadCloser, error)
    RunningTime() time.Duration
    Format() string
}
```

Each grouping of concrete types based on their shared behaviors can be expressed as an interface type. Unlike class-based languages, in which the set of interfaces satisfied by a class is explicit, in Go we can define new abstractions or groupings of interest when we need them, without modifying the declaration of the concrete type. This is particularly useful when the concrete type comes from a package written by a different author. Of course, there do need to be underlying commonalities in the concrete types. 

每个基于共享行为的具体类型的分组都可以表示为一个接口类型 。[cite: 116] 与基于类的语言（其中一个类满足的接口集合是显式的）不同，在Go中，我们可以在需要时定义新的抽象或感兴趣的分组，而无需修改具体类型的声明 。[cite: 117] 当具体类型来自不同作者编写的包时，这尤其有用 。[cite: 118] 当然，具体类型之间确实需要存在底层的共性（commonalities） 。[cite: 119]

---

**7.4. 使用 flag.Value 解析命令行参数 (Parsing Flags with flag.Value)**

In this section, we'll see how another standard interface, flag.Value, helps us define new notations for command-line flags. Consider the program below, which sleeps for a specified period of time. 
_gopl.io/ch7/sleep_
```go
var period = flag.Duration("period", 1*time.Second, "sleep period")

func main() {
    flag.Parse()
    fmt.Printf("Sleeping for %v...\n", *period)
    time.Sleep(*period)
    fmt.Println()
}
```
在本节中，我们将看到另一个标准接口`flag.Value`如何帮助我们为命令行参数（command-line flags）定义新的表示法（notations） 。[cite: 121] 考虑下面的程序，它会休眠指定的时间 。[cite: 122]
_gopl.io/ch7/sleep_
```go
var period = flag.Duration("period", 1*time.Second, "sleep period")

func main() {
    flag.Parse()
    fmt.Printf("Sleeping for %v...\n", *period)
    time.Sleep(*period)
    fmt.Println()
}
```

Before it goes to sleep it prints the time period. The fmt package calls the time.Duration's String method to print the period not as a number of nanoseconds, but in a user-friendly notation: 
```bash
$ go build gopl.io/ch7/sleep
$ ./sleep
Sleeping for 1s...
```
在进入休眠之前，它会打印时间周期 。[cite: 123] `fmt`包调用`time.Duration`的`String`方法来打印周期，不是以纳秒为单位的数字，而是以用户友好的表示法：[cite: 123]
```bash
$ go build gopl.io/ch7/sleep
$ ./sleep
Sleeping for 1s...
```

By default, the sleep period is one second, but it can be controlled through the -period command-line flag. The flag.Duration function creates a flag variable of type time.Duration and allows the user to specify the duration in a variety of user-friendly formats, including the same notation printed by the String method. This symmetry of design leads to a nice user interface. 
```bash
$ ./sleep -period 50ms
Sleeping for 50ms...
$ ./sleep -period 2m30s
Sleeping for 2m30s...
$ ./sleep -period 1.5h
Sleeping for 1h30m0s...
$ ./sleep -period "1 day"
invalid value "1 day" for flag -period: time: invalid duration 1 day
```
默认情况下，休眠周期是一秒，但可以通过`-period`命令行参数进行控制 。[cite: 124] `flag.Duration`函数创建一个`time.Duration`类型的参数变量，并允许用户以多种用户友好的格式指定持续时间，包括`String`方法打印的相同表示法 。[cite: 125] 这种设计的对称性（symmetry of design）带来了良好的用户界面 。[cite: 126]
```bash
$ ./sleep -period 50ms
Sleeping for 50ms...
$ ./sleep -period 2m30s
Sleeping for 2m30s...
$ ./sleep -period 1.5h
Sleeping for 1h30m0s...
$ ./sleep -period "1 day"
invalid value "1 day" for flag -period: time: invalid duration 1 day
```

Because duration-valued flags are so useful, this feature is built into the flag package, but it's easy to define new flag notations for our own data types. We need only define a type that satisfies the flag.Value interface, whose declaration is below: 
```go
package flag

// Value is the interface to the value stored in a flag.
type Value interface {
    String() string
    Set(string) error
}
```
因为表示持续时间的参数非常有用，这个功能被内置到了`flag`包中，但为我们自己的数据类型定义新的参数表示法也很容易 。[cite: 127] 我们只需要定义一个满足`flag.Value`接口的类型，其声明如下：[cite: 128]
```go
package flag

// Value is the interface to the value stored in a flag.
type Value interface {
    String() string
    Set(string) error
}
```

The String method formats the flag's value for use in command-line help messages; thus every flag.Value is also a fmt.Stringer. The Set method parses its string argument and updates the flag value. In effect, the Set method is the inverse of the String method, and it is good practice for them to use the same notation. 

`String`方法格式化参数的值，用于命令行帮助信息；[cite: 130] 因此，每个`flag.Value`也是一个`fmt.Stringer` 。[cite: 130] `Set`方法解析其字符串参数并更新参数值 。[cite: 131] 实际上，`Set`方法是`String`方法的逆过程，良好的实践是让它们使用相同的表示法 。[cite: 131]

Let's define a celsiusFlag type that allows a temperature to be specified in Celsius, or in Fahrenheit with an appropriate conversion. Notice that celsiusFlag embeds a Celsius (§2.5), thereby getting a String method for free. To satisfy flag.Value, we need only declare the Set method: 
_gopl.io/ch7/tempconv_
```go
// *celsiusFlag satisfies the flag.Value interface.
type celsiusFlag struct{ Celsius }

func (f *celsiusFlag) Set(s string) error {
    var unit string
    var value float64
    fmt.Sscanf(s, "%f%s", &value, &unit) // no error check needed
    switch unit {
    case "C", "°C":
        f.Celsius = Celsius(value)
        return nil
    case "F", "°F":
        f.Celsius = FToC(Fahrenheit(value))
        return nil
    }
    return fmt.Errorf("invalid temperature %q", s)
}
```
让我们定义一个`celsiusFlag`类型，允许以摄氏度指定温度，或者以华氏度指定并进行适当转换 。[cite: 132] 注意`celsiusFlag`嵌入了一个`Celsius`类型（§2.5），从而免费获得了`String`方法 。[cite: 133] 为了满足`flag.Value`接口，我们只需要声明`Set`方法：[cite: 134]
_gopl.io/ch7/tempconv_
```go
// *celsiusFlag satisfies the flag.Value interface.
type celsiusFlag struct{ Celsius }

func (f *celsiusFlag) Set(s string) error {
    var unit string
    var value float64
    fmt.Sscanf(s, "%f%s", &value, &unit) // no error check needed
    switch unit {
    case "C", "°C":
        f.Celsius = Celsius(value)
        return nil
    case "F", "°F":
        f.Celsius = FToC(Fahrenheit(value))
        return nil
    }
    return fmt.Errorf("invalid temperature %q", s)
}
```

The call to fmt.Sscanf parses a floating-point number (value) and a string (unit) from the input s. Although one must usually check Sscanf's error result, in this case we don't need to because if there was a problem, no switch case will match. The CelsiusFlag function below wraps it all up. To the caller, it returns a pointer to the Celsius field embedded within the celsiusFlag variable f. The Celsius field is the variable that will be updated by the Set method during flags processing. The call to Var adds the flag to the application's set of command-line flags, the global variable flag.CommandLine. Programs with unusually complex command-line interfaces may have several variables of this type. The call to Var assigns a *celsiusFlag argument to a flag.Value parameter, causing the compiler to check that *celsiusFlag has the necessary methods. 
```go
// CelsiusFlag defines a Celsius flag with the specified name,
// default value, and usage, and returns the address of the flag variable.
// The flag argument must have a quantity and a unit, e.g., "100C".
func CelsiusFlag(name string, value Celsius, usage string) *Celsius {
    f := celsiusFlag{value}
    flag.CommandLine.Var(&f, name, usage)
    return &f.Celsius
}
```
对`fmt.Sscanf`的调用从输入字符串`s`中解析一个浮点数(`value`)和一个字符串(`unit`) 。[cite: 136] 虽然通常必须检查`Sscanf`的错误结果，但在这种情况下我们不需要，因为如果存在问题，`switch`的任何一个`case`都不会匹配 。[cite: 137] 下面的`CelsiusFlag`函数将所有这些封装起来 。[cite: 138] 它向调用者返回一个指向嵌入在`celsiusFlag`变量`f`中的`Celsius`字段的指针 。[cite: 138] `Celsius`字段是在参数处理期间将由`Set`方法更新的变量 。[cite: 139] 对`Var`的调用将该参数添加到应用程序的命令行参数集，即全局变量`flag.CommandLine` 。[cite: 140] 具有异常复杂命令行界面的程序可能拥有此类型的多个变量 。[cite: 141] 对`Var`的调用将一个`*celsiusFlag`参数赋给一个`flag.Value`形参，这会使编译器检查`*celsiusFlag`是否具有必要的方法 。[cite: 142, 143]
```go
// CelsiusFlag defines a Celsius flag with the specified name,
// default value, and usage, and returns the address of the flag variable.
// The flag argument must have a quantity and a unit, e.g., "100C".
func CelsiusFlag(name string, value Celsius, usage string) *Celsius {
    f := celsiusFlag{value}
    flag.CommandLine.Var(&f, name, usage)
    return &f.Celsius
}
```

Now we can start using the new flag in our programs: 
_gopl.io/ch7/tempflag_
```go
var temp = tempconv.CelsiusFlag("temp", 20.0, "the temperature")
func main() {
    flag.Parse()
    fmt.Println(*temp)
}
```
现在我们可以在我们的程序中使用这个新的标志了：[cite: 147]
_gopl.io/ch7/tempflag_
```go
var temp = tempconv.CelsiusFlag("temp", 20.0, "the temperature")
func main() {
    flag.Parse()
    fmt.Println(*temp)
}
```

Here's a typical session: 
```bash
$ go build gopl.io/ch7/tempflag
$ ./tempflag
20°C
$ ./tempflag -temp -18C
-18°C
$ ./tempflag -temp 212°F
100°C
$ ./tempflag -temp 273.15K
invalid value "273.15K" for flag -temp: invalid temperature "273.15K"
Usage of ./tempflag:
  -temp value
    	the temperature (default 20°C)
$ ./tempflag -help
Usage of ./tempflag:
  -temp value
    	the temperature (default 20°C)
```
这是一个典型的会话：[cite: 148]
```bash
$ go build gopl.io/ch7/tempflag
$ ./tempflag
20°C
$ ./tempflag -temp -18C
-18°C
$ ./tempflag -temp 212°F
100°C
$ ./tempflag -temp 273.15K
invalid value "273.15K" for flag -temp: invalid temperature "273.15K"
Usage of ./tempflag:
  -temp value
    	the temperature (default 20°C)
$ ./tempflag -help
Usage of ./tempflag:
  -temp value
    	the temperature (default 20°C)
```

Exercise 7.6: Add support for Kelvin temperatures to tempflag.
Exercise 7.7: Explain why the help message contains °C when the default value of 20.0 does not.

练习 7.6：为 tempflag 添加对开尔文温度的支持。
练习 7.7：解释为什么当默认值 20.0（一个 float64）不包含 °C 时，帮助信息中却显示 20°C。 (提示：思考 flag.Value 的 String() 方法是如何被调用的，以及 celsiusFlag 类型的 String() 方法是如何获得的。)

---

**7.5. 接口值 (Interface Values)**

Conceptually, a value of an interface type, or interface value, has two components, a concrete type and a value of that type. These are called the interface's dynamic type and dynamic value. 

从概念上讲，接口类型的值，即**接口值**（interface value），有两个组成部分：一个具体类型（concrete type）和该类型的一个值 。[cite: 149] 这两个部分分别被称为接口的**动态类型**（dynamic type）和**动态值**（dynamic value） 。[cite: 149]

For a statically typed language like Go, types are a compile-time concept, so a type is not a value. In our conceptual model, a set of values called type descriptors provide information about each type, such as its name and methods. In an interface value, the type component is represented by the appropriate type descriptor. 

对于像Go这样的静态类型语言，类型是编译时的概念，因此类型不是一个值 。[cite: 150] 在我们的概念模型中，一组称为**类型描述符**（type descriptors）的值提供了关于每种类型的信息，例如其名称和方法 。[cite: 151] 在接口值中，类型组件由相应的类型描述符表示 。[cite: 152]

In the four statements below, the variable w takes on three different values. (The initial and final values are the same.) 
```go
var w io.Writer
w = os.Stdout
w = new(bytes.Buffer)
w = nil
```
在下面的四个语句中，变量`w`依次取了三个不同的值（初始值和最终值相同） 。[cite: 153, 154]
```go
var w io.Writer
w = os.Stdout
w = new(bytes.Buffer)
w = nil
```

Let's take a closer look at the value and dynamic behavior of w after each statement. The first statement declares w: 
`var w io.Writer`
In Go, variables are always initialized to a well-defined value, and interfaces are no exception. The zero value for an interface has both its type and value components set to nil (Figure 7.1). 
```
W
type    value
nil     nil
```
Figure 7.1. A nil interface value. 

让我们仔细观察每个语句执行后`w`的值和动态行为 。[cite: 155] 第一个语句声明了`w`：[cite: 155]
`var w io.Writer`
在Go中，变量总是被初始化为一个明确定义的值，接口也不例外 。[cite: 156] 接口的零值，其类型和值组件都设置为`nil` (图 7.1) 。[cite: 157]
```
w
类型 (type)   值 (value)
nil           nil
```
_图 7.1. 一个 nil 接口值。_[cite: 157]

An interface value is described as nil or non-nil based on its dynamic type, so this is a nil interface value. You can test whether an interface value is nil using `w == nil` or `w != nil`. Calling any method of a nil interface value causes a panic: 
`w.Write([]byte("hello")) // panic: nil pointer dereference`

一个接口值根据其动态类型是否为`nil`来判断其自身是否为`nil`或非`nil`。[cite: 158] 因此，这是一个`nil`接口值 。[cite: 158] 你可以使用`w == nil`或`w != nil`来测试一个接口值是否为`nil` 。[cite: 159] 调用一个`nil`接口值的任何方法都会导致 panic：[cite: 159]
`w.Write([]byte("hello")) // panic: nil pointer dereference`

The second statement assigns a value of type *os.File to w: 
`w = os.Stdout`
This assignment involves an implicit conversion from a concrete type to an interface type, and is equivalent to the explicit conversion `io.Writer(os.Stdout)`. A conversion of this kind, whether explicit or implicit, captures the type and the value of its operand. The interface value's dynamic type is set to the type descriptor for the pointer type *os.File, and its dynamic value holds a copy of os.Stdout, which is a pointer to the os.File variable representing the standard output of the process (Figure 7.2). 
```
W
type        value
*os.File    (os.File struct for stdout, fd=1)
```
Figure 7.2. An interface value containing an *os.File pointer. 

第二个语句将一个*os.File类型的值赋给w：[cite: 160]
`w = os.Stdout`
此赋值涉及从具体类型到接口类型的隐式转换，等价于显式转换`io.Writer(os.Stdout)` 。[cite: 161] 这种转换，无论是显式的还是隐式的，都会捕获其操作数的类型和值 。[cite: 162] 接口值的动态类型被设置为指针类型`*os.File`的类型描述符，其动态值持有`os.Stdout`的副本，`os.Stdout`是一个指向代表进程标准输出的`os.File`变量的指针 (图 7.2) 。[cite: 163, 164]
```
w
类型 (type)      值 (value)
*os.File         (指向 os.File 结构体的指针，该结构体代表 fd=1 即标准输出)
```
图 7.2. 一个包含*os.File指针的接口值。[cite: 165]

Calling the Write method on an interface value containing an *os.File pointer causes the (*os.File).Write method to be called. The call prints "hello". 
`w.Write([]byte("hello")) // "hello"`
In general, we cannot know at compile time what the dynamic type of an interface value will be, so a call through an interface must use dynamic dispatch. Instead of a direct call, the compiler must generate code to obtain the address of the method named Write from the type descriptor, then make an indirect call to that address. The receiver argument for the call is a copy of the interface's dynamic value, os.Stdout. The effect is as if we had made this call directly: 
`os.Stdout.Write([]byte("hello")) // "hello"`

在一个包含*os.File指针的接口值上调用Write方法，会导致(*os.File).Write方法被调用 。[cite: 166, 167] 该调用会打印 "hello" 。[cite: 167]
`w.Write([]byte("hello")) // "hello"`
通常，我们在编译时无法知道接口值的动态类型是什么，因此通过接口进行的调用必须使用**动态派发**（dynamic dispatch） 。[cite: 168, 169] 编译器不会生成直接调用，而是必须生成代码以从类型描述符中获取名为`Write`的方法的地址，然后对该地址进行间接调用 。[cite: 169] 调用的接收者参数是接口动态值`os.Stdout`的副本 。[cite: 170] 其效果就如同我们直接进行了如下调用：[cite: 171]
`os.Stdout.Write([]byte("hello")) // "hello"`

The third statement assigns a value of type *bytes.Buffer to the interface value: 
`w = new(bytes.Buffer)`
The dynamic type is now *bytes.Buffer and the dynamic value is a pointer to the newly allocated buffer (Figure 7.3). 
```
W
type            value
*bytes.Buffer   (bytes.Buffer struct)
```
Figure 7.3. An interface value containing a *bytes.Buffer pointer. 

第三条语句将一个*bytes.Buffer类型的值赋给接口值：[cite: 172]
`w = new(bytes.Buffer)`
动态类型现在是`*bytes.Buffer`，动态值是指向新分配的缓冲区的指针 (图 7.3) 。[cite: 173]
```
w
类型 (type)         值 (value)
*bytes.Buffer       (指向新分配的 bytes.Buffer 结构体的指针)
```
图 7.3. 一个包含*bytes.Buffer指针的接口值。[cite: 174]

A call to the Write method uses the same mechanism as before: 
`w.Write([]byte("hello")) // writes "hello" to the bytes.Buffer`
This time, the type descriptor is *bytes.Buffer, so the (*bytes.Buffer).Write method is called, with the address of the buffer as the value of the receiver parameter. The call appends "hello" to the buffer. 

对`Write`方法的调用使用与之前相同的机制：[cite: 174]
`w.Write([]byte("hello")) // writes "hello" to the bytes.Buffer`
这一次，类型描述符是`*bytes.Buffer`，所以`(*bytes.Buffer).Write`方法被调用，缓冲区的地址作为接收者参数的值 。[cite: 175] 该调用将 "hello" 追加到缓冲区 。[cite: 176]

Finally, the fourth statement assigns nil to the interface value: 
`w = nil`
This resets both its components to nil, restoring w to the same state as when it was declared, which was shown in Figure 7.1. 

最后，第四条语句将nil赋给接口值：[cite: 176]
`w = nil`
这将它的两个组件都重置为`nil`，使`w`恢复到声明时的状态，如图 7.1 所示 。[cite: 177]

An interface value can hold arbitrarily large dynamic values. For example, the time.Time type, which represents an instant in time, is a struct type with several unexported fields. If we create an interface value from it, 
`var x interface{} = time.Now()`
the result might look like Figure 7.4. Conceptually, the dynamic value always fits inside the interface value, no matter how large its type. (This is only a conceptual model; a realistic implementation is quite different.) 
```
x
type        value
time.Time   sec: 635...
            nsec: 689...
            loc: "UTC"
```
Figure 7.4. An interface value holding a time.Time struct. 

接口值可以持有任意大的动态值 。[cite: 177] 例如，代表时间瞬间的time.Time类型是一个包含若干未导出字段的结构体类型 。[cite: 178] 如果我们用它创建一个接口值：[cite: 179]
`var x interface{} = time.Now()`
结果可能如图7.4所示 。[cite: 180] 从概念上讲，无论其类型有多大，动态值总是能容纳在接口值之内 。[cite: 180] （这只是一个概念模型；实际的实现则大相径庭。）[cite: 181]
```
x
类型 (type)     值 (value)
time.Time       (time.Time 结构体，包含如 sec, nsec, loc 等字段)
```
图 7.4. 一个持有time.Time结构体的接口值。[cite: 182]

Interface values may be compared using `==` and `!=`. Two interface values are equal if both are nil, or if their dynamic types are identical and their dynamic values are equal according to the usual behavior of `==` for that type. Because interface values are comparable, they may be used as the keys of a map or as the operand of a switch statement. However, if two interface values are compared and have the same dynamic type, but that type is not comparable (a slice, for instance), then the comparison fails with a panic: 
```go
var x interface{} = []int{1, 2, 3}
fmt.Println(x == x) // panic: comparing uncomparable type []int
```
接口值可以使用`==`和`!=`进行比较 。[cite: 183] 两个接口值相等，当且仅当它们都为`nil`，或者它们的动态类型相同且它们的动态值根据该类型的`==`的通常行为也相等 。[cite: 183] 由于接口值是可比较的，它们可以用作map的键或`switch`语句的操作数 。[cite: 184] 然而，如果比较两个动态类型相同但该类型本身不可比较（例如，切片）的接口值，则比较操作会引发 panic：[cite: 185]
```go
var x interface{} = []int{1, 2, 3}
fmt.Println(x == x) // panic: comparing uncomparable type []int
```

In this respect, interface types are unusual. Other types are either safely comparable (like basic types and pointers) or not comparable at all (like slices, maps, and functions), but when comparing interface values or aggregate types that contain interface values, we must be aware of the potential for a panic. A similar risk exists when using interfaces as map keys or switch operands. Only compare interface values if you are certain that they contain dynamic values of comparable types. 

在这方面，接口类型比较特殊（unusual） 。[cite: 186] 其他类型要么是安全可比较的（如基本类型和指针），要么是完全不可比较的（如切片、map和函数），但是在比较接口值或包含接口值的聚合类型时，我们必须意识到潜在的panic风险 。[cite: 187] 当使用接口作为map键或`switch`操作数时，也存在类似的风险 。[cite: 188] 仅当确信接口值包含可比较类型的动态值时，才应比较它们 。[cite: 188]

When handling errors, or during debugging, it is often helpful to report the dynamic type of an interface value. For that, we use the fmt package's %T verb: 
```go
var w io.Writer
fmt.Printf("%T\n", w) // "<nil>"
w = os.Stdout
fmt.Printf("%T\n", w) // "*os.File"
w = new(bytes.Buffer)
fmt.Printf("%T\n", w) // "*bytes.Buffer"
```
在处理错误或调试时，报告接口值的动态类型通常很有帮助 。[cite: 189] 为此，我们使用`fmt`包的`%T`格式化动词：[cite: 190]
```go
var w io.Writer
fmt.Printf("%T\n", w) // "<nil>"
w = os.Stdout
fmt.Printf("%T\n", w) // "*os.File"
w = new(bytes.Buffer)
fmt.Printf("%T\n", w) // "*bytes.Buffer"
```

Internally, fmt uses reflection to obtain the name of the interface's dynamic type. We'll look at reflection in Chapter 12. 

在内部，`fmt`使用反射来获取接口动态类型的名称 。[cite: 191] 我们将在第12章探讨反射 。[cite: 191]

---

**7.5.1. 注意事项：包含nil指针的接口其本身并非nil (Caveat: An Interface Containing a Nil Pointer Is Non-Nil)**

A nil interface value, which contains no value at all, is not the same as an interface value containing a pointer that happens to be nil. This subtle distinction creates a trap into which every Go programmer has stumbled. 

一个`nil`接口值（完全不包含任何值）与一个包含碰巧为`nil`的指针的接口值是不同的 。[cite: 192] 这种微妙的区别（subtle distinction）是一个常见的陷阱（trap），几乎每个Go程序员都曾遇到过 。[cite: 192]

Consider the program below. With debug set to true, the main function collects the output of the function f in a bytes.Buffer. 
```go
const debug = true

func main() {
    var buf *bytes.Buffer
    if debug {
        buf = new(bytes.Buffer) // enable collection of output
    }
    f(buf) // NOTE: subtly incorrect!
    if debug {
        //...use buf...
    }
}

// If out is non-nil, output will be written to it.
func f(out io.Writer) {
    //...do something...
    if out != nil {
        out.Write([]byte("done!\n"))
    }
}
```
考虑下面的程序。[cite: 193] 当`debug`设置为`true`时，`main`函数将函数`f`的输出收集到一个`bytes.Buffer`中 。[cite: 193]
```go
const debug = true

func main() {
    var buf *bytes.Buffer
    if debug {
        buf = new(bytes.Buffer) // enable collection of output
    }
    f(buf) // NOTE: subtly incorrect!
    if debug {
        //...use buf...
    }
}

// If out is non-nil, output will be written to it.
func f(out io.Writer) {
    //...do something...
    if out != nil {
        out.Write([]byte("done!\n"))
    }
}
```

We might expect that changing debug to false would disable the collection of the output, but in fact it causes the program to panic during the out.Write call: 
```go
// if out != nil {
//     out.Write([]byte("done!\n")) // panic: nil pointer dereference
// }
```
我们可能期望将`debug`改为`false`会禁用输出收集，但实际上它会导致程序在`out.Write`调用期间发生panic：[cite: 197]
```go
// if out != nil {
//     out.Write([]byte("done!\n")) // panic: nil pointer dereference
// }
```

When main calls f, it assigns a nil pointer of type *bytes.Buffer to the out parameter, so the dynamic value of out is nil. However, its dynamic type is *bytes.Buffer, meaning that out is a non-nil interface containing a nil pointer value (Figure 7.5), so the defensive check out != nil is still true. 
```
out (was W in PDF)
type            value
*bytes.Buffer   nil
```
Figure 7.5. A non-nil interface containing a nil pointer. 

当`main`调用`f`时，它将一个类型为`*bytes.Buffer`的`nil`指针赋给了`out`参数，因此`out`的动态值是`nil` 。[cite: 198] 然而，其动态类型是`*bytes.Buffer`，这意味着`out`是一个**非nil的接口，其内部包含一个nil指针值**（图 7.5），所以防御性检查`out != nil`仍然为真 。[cite: 199, 200]
```
out
类型 (type)      值 (value)
*bytes.Buffer    nil
```
_图 7.5. 一个包含nil指针的非nil接口。_[cite: 200]

As before, the dynamic dispatch mechanism determines that (*bytes.Buffer).Write must be called but this time with a receiver value that is nil. For some types, such as *os.File, nil is a valid receiver (§6.2.1), but *bytes.Buffer is not among them. The method is called, but it panics as it tries to access the buffer. 

与之前一样，动态派发机制确定必须调用`(*bytes.Buffer).Write`方法，但这次的接收者值是`nil` 。[cite: 201] 对于某些类型，例如`*os.File`，`nil`是一个有效的接收者（§6.2.1），但`*bytes.Buffer`并非如此 。[cite: 202] 该方法被调用，但在尝试访问缓冲区时会发生panic 。[cite: 203]

The problem is that although a nil *bytes.Buffer pointer has the methods needed to satisfy the interface, it doesn't satisfy the behavioral requirements of the interface. In particular, the call violates the implicit precondition of (*bytes.Buffer).Write that its receiver is not nil, so assigning the nil pointer to the interface was a mistake. The solution is to change the type of buf in main to io.Writer, thereby avoiding the assignment of the dysfunctional value to the interface in the first place: 
```go
var buf io.Writer
if debug {
    buf = new(bytes.Buffer) // enable collection of output
}
f(buf) // OK
```
问题在于，尽管一个`nil`的`*bytes.Buffer`指针拥有满足该接口所需的方法，但它并不满足该接口的行为要求（behavioral requirements） 。[cite: 204] 特别是，该调用违反了`(*bytes.Buffer).Write`的隐式前提条件（implicit precondition），即其接收者不为`nil`，因此将`nil`指针赋给该接口是一个错误 。[cite: 206] 解决方案是将`main`函数中`buf`的类型更改为`io.Writer`，从而从一开始就避免将这个功能失常的（dysfunctional）值赋给接口：[cite: 207]
```go
var buf io.Writer
if debug {
    buf = new(bytes.Buffer) // enable collection of output
}
f(buf) // OK
```

Now that we've covered the mechanics of interface values, let's take a look at some more important interfaces from Go's standard library. In the next three sections, we'll see how interfaces are used for sorting, web serving, and error handling. 

既然我们已经介绍了接口值的机制，接下来让我们看一些Go标准库中更重要的接口 。[cite: 209] 在接下来的三个小节中，我们将看到接口如何用于排序、Web服务和错误处理 。[cite: 209]