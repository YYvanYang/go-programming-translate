# 7 Interfaces

# 7 接口

Interface types express generalizations or abstractions about the behaviors of other types. By generalizing, interfaces let us write functions that are more flexible and adaptable because they are not tied to the details of one particular implementation.

**接口类型（Interface types）** 表达了关于其他类型行为的概括或抽象。通过概括，接口让我们能够编写更灵活和适应性更强的函数，因为它们不依赖于某个特定实现的细节。

Many object-oriented languages have some notion of interfaces, but what makes Go's interfaces so distinctive is that they are satisfied implicitly. In other words, there's no need to declare all the interfaces that a given concrete type satisfies; simply possessing the necessary methods is enough. This design lets you create new interfaces that are satisfied by existing concrete types without changing the existing types, which is particularly useful for types defined in packages that you don't control.

许多面向对象语言都有某种接口概念，但Go的接口如此与众不同的原因在于它们是**隐式满足（implicitly satisfied）** 的。换句话说，无需声明给定的**具体类型（concrete type）** 满足的所有接口；仅仅拥有必要的方法就足够了。这种设计让您能够创建新的接口，这些接口可以被现有的具体类型满足，而无需更改现有类型，这对于您无法控制的包中定义的类型特别有用。

In this chapter, we'll start by looking at the basic mechanics of interface types and their values. Along the way, we'll study several important interfaces from the standard library. Many Go programs make as much use of standard interfaces as they do of their own ones. Finally, we'll look at type assertions (§7.10) and type switches (§7.13) and see how they enable a different kind of generality.

在本章中，我们将首先了解接口类型及其值的基本机制。在此过程中，我们将学习标准库中的几个重要接口。许多Go程序对标准接口的使用程度与对自定义接口的使用程度相当。最后，我们将了解**类型断言（type assertion）**（§7.10）和**类型开关（type switch）**（§7.13），并看看它们如何实现不同类型的通用性。

## 7.1. Interfaces as Contracts

## 7.1. 接口作为契约

All the types we've looked at so far have been concrete types. A concrete type specifies the exact representation of its values and exposes the intrinsic operations of that representation, such as arithmetic for numbers, or indexing, append, and range for slices. A concrete type may also provide additional behaviors through its methods. When you have a value of a concrete type, you know exactly what it is and what you can do with it.

到目前为止，我们看到的所有类型都是具体类型。具体类型指定其值的确切表示，并暴露该表示的内在操作，例如数字的算术运算，或切片的索引、追加和范围操作。具体类型还可以通过其方法提供额外的行为。当您有一个具体类型的值时，您确切地知道它是什么以及可以用它做什么。

There is another kind of type in Go called an interface type. An interface is an abstract type. It doesn't expose the representation or internal structure of its values, or the set of basic operations they support; it reveals only some of their methods. When you have a value of an interface type, you know nothing about what it is; you know only what it can do, or more precisely, what behaviors are provided by its methods.

Go中还有另一种类型叫做接口类型。接口是一种抽象类型。它不暴露其值的表示或内部结构，也不暴露它们支持的基本操作集合；它只显示一些方法。当您有一个接口类型的值时，您对它是什么一无所知；您只知道它能做什么，或者更准确地说，它的方法提供了什么行为。

Throughout the book, we've been using two similar functions for string formatting: fmt.Printf, which writes the result to the standard output (a file), and fmt.Sprintf, which returns the result as a string. It would be unfortunate if the hard part, formatting the result, had to be duplicated because of these superficial differences in how the result is used. Thanks to interfaces, it does not. Both of these functions are, in effect, wrappers around a third function, fmt.Fprintf, that is agnostic about what happens to the result it computes:

在整本书中，我们一直在使用两个类似的字符串格式化函数：`fmt.Printf`（将结果写入标准输出（一个文件）），和`fmt.Sprintf`（将结果作为字符串返回）。如果困难的部分——格式化结果——因为结果使用方式的这些表面差异而必须重复实现，那将是不幸的。感谢接口，它不需要重复实现。这两个函数实际上都是第三个函数`fmt.Fprintf`的包装器，该函数对其计算结果的处理方式是不可知的：

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

The F prefix of Fprintf stands for file and indicates that the formatted output should be written to the file provided as the first argument. In the Printf case, the argument, os.Stdout, is an *os.File. In the Sprintf case, however, the argument is not a file, though it superficially resembles one: &buf is a pointer to a memory buffer to which bytes can be written.

`Fprintf`的F前缀代表文件（file），表示格式化输出应该写入作为第一个参数提供的文件。在`Printf`情况下，参数`os.Stdout`是一个`*os.File`。然而在`Sprintf`情况下，参数不是文件，尽管表面上类似：`&buf`是一个指向内存缓冲区的指针，可以向其写入字节。

The first parameter of Fprintf is not a file either. It's an io.Writer, which is an interface type with the following declaration:

`Fprintf`的第一个参数也不是文件。它是一个`io.Writer`，这是一个接口类型，具有以下声明：

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

`io.Writer`接口定义了`Fprintf`与其调用者之间的契约。一方面，契约要求调用者提供一个具体类型的值，如`*os.File`或`*bytes.Buffer`，该值具有一个名为`Write`的方法，且具有适当的签名和行为。另一方面，契约保证`Fprintf`将在给定任何满足`io.Writer`接口的值时完成其工作。`Fprintf`不能假设它正在写入文件或内存，只能假设它可以调用`Write`。

Because fmt.Fprintf assumes nothing about the representation of the value and relies only on the behaviors guaranteed by the io.Writer contract, we can safely pass a value of any concrete type that satisfies io.Writer as the first argument to fmt.Fprintf. This freedom to substitute one type for another that satisfies the same interface is called substitutability, and is a hallmark of object-oriented programming.

因为`fmt.Fprintf`对值的表示不做任何假设，只依赖于`io.Writer`契约保证的行为，我们可以安全地将任何满足`io.Writer`的具体类型的值作为第一个参数传递给`fmt.Fprintf`。这种用满足相同接口的另一种类型替换一种类型的自由称为可替换性（substitutability），是面向对象编程的标志。

Let's test this out using a new type. The Write method of the *ByteCounter type below merely counts the bytes written to it before discarding them. (The conversion is required to make the types of len(p) and *c match in the += assignment statement.)

让我们使用一个新类型来测试这一点。下面`*ByteCounter`类型的`Write`方法仅仅在丢弃字节之前计算写入的字节数。（需要转换以使`len(p)`和`*c`的类型在`+=`赋值语句中匹配。）

```go
// gopl.io/ch7/bytecounter
type ByteCounter int

func (c *ByteCounter) Write(p []byte) (int, error) {
    *c += ByteCounter(len(p)) // convert int to ByteCounter
    return len(p), nil
}
```

Since *ByteCounter satisfies the io.Writer contract, we can pass it to Fprintf, which does its string formatting oblivious to this change; the ByteCounter correctly accumulates the length of the result.

由于`*ByteCounter`满足`io.Writer`契约，我们可以将其传递给`Fprintf`，`Fprintf`在不知道这种变化的情况下执行字符串格式化；`ByteCounter`正确地累积结果的长度。

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

除了`io.Writer`，还有另一个对`fmt`包非常重要的接口。`Fprintf`和`Fprintln`为类型提供了一种控制其值如何打印的方法。在第2.5节中，我们为`Celsius`类型定义了一个`String`方法，以便温度打印为"100°C"，在第6.5节中，我们为`*IntSet`配备了一个`String`方法，以便集合使用传统的集合表示法如"{1 2 3}"来呈现。声明`String`方法使类型满足最广泛使用的接口之一——`fmt.Stringer`：

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

我们将在第7.10节解释`fmt`包如何发现哪些值满足这个接口。

**Exercise 7.1:** Using the ideas from ByteCounter, implement counters for words and for lines. You will find bufio.ScanWords useful.

**练习7.1：** 使用来自`ByteCounter`的思想，实现单词和行的计数器。您会发现`bufio.ScanWords`很有用。

**Exercise 7.2:** Write a function CountingWriter with the signature below that, given an io.Writer, returns a new Writer that wraps the original, and a pointer to an int64 variable that at any moment contains the number of bytes written to the new Writer.

**练习7.2：** 编写一个具有下面签名的函数`CountingWriter`，给定一个`io.Writer`，返回一个包装原始Writer的新Writer，以及一个指向`int64`变量的指针，该变量在任何时刻都包含写入新Writer的字节数。

```go
func CountingWriter(w io.Writer) (io.Writer, *int64)
```

**Exercise 7.3:** Write a String method for the *tree type in gopl.io/ch4/treesort (§4.4) that reveals the sequence of values in the tree.

**练习7.3：** 为`gopl.io/ch4/treesort`（§4.4）中的`*tree`类型编写一个`String`方法，该方法显示树中值的序列。

## 7.2. Interface Types

## 7.2. 接口类型

An interface type specifies a set of methods that a concrete type must possess to be considered an instance of that interface.

接口类型指定了具体类型必须拥有的一组方法，以便被认为是该接口的实例。

The io.Writer type is one of the most widely used interfaces because it provides an abstraction of all the types to which bytes can be written, which includes files, memory buffers, network connections, HTTP clients, archivers, hashers, and so on. The io package defines many other useful interfaces. A Reader represents any type from which you can read bytes, and a Closer is any value that you can close, such as a file or a network connection. (By now you've probably noticed the naming convention for many of Go's single-method interfaces.)

`io.Writer`类型是最广泛使用的接口之一，因为它提供了所有可以写入字节的类型的抽象，包括文件、内存缓冲区、网络连接、HTTP客户端、归档器、哈希器等等。`io`包定义了许多其他有用的接口。`Reader`代表任何可以从中读取字节的类型，`Closer`是任何可以关闭的值，如文件或网络连接。（到现在您可能已经注意到了Go许多单方法接口的命名约定。）

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

进一步看，我们发现新接口类型的声明是现有接口的组合。这里有两个例子：

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

上面使用的语法类似于结构体嵌入，让我们可以将另一个接口命名为写出其所有方法的简写。这称为嵌入接口。我们可以不使用嵌入来编写`io.ReadWriter`，尽管不那么简洁，如下所示：

```go
type ReadWriter interface {
    Read(p []byte) (n int, err error)
    Write(p []byte) (n int, err error)
}
```

or even using a mixture of the two styles:

甚至可以使用两种风格的混合：

```go
type ReadWriter interface {
    Read(p []byte) (n int, err error)
    Writer
}
```

All three declarations have the same effect. The order in which the methods appear is immaterial. All that matters is the set of methods.

这三种声明具有相同的效果。方法出现的顺序无关紧要。重要的只是方法的集合。

**Exercise 7.4:** The strings.NewReader function returns a value that satisfies the io.Reader interface (and others) by reading from its argument, a string. Implement a simple version of NewReader yourself, and use it to make the HTML parser (§5.2) take input from a string.

**练习7.4：** `strings.NewReader`函数返回一个通过从其参数（一个字符串）读取来满足`io.Reader`接口（以及其他接口）的值。自己实现一个简单版本的`NewReader`，并使用它让HTML解析器（§5.2）从字符串获取输入。

**Exercise 7.5:** The LimitReader function in the io package accepts an io.Reader r and a number of bytes n, and returns another Reader that reads from r but reports an end-of-file condition after n bytes. Implement it.

**练习7.5：** `io`包中的`LimitReader`函数接受一个`io.Reader` r和字节数n，并返回另一个Reader，该Reader从r读取但在n个字节后报告文件结束条件。请实现它。

```go
func LimitReader(r io.Reader, n int64) io.Reader
```

## 7.3. Interface Satisfaction

## 7.3. 接口满足

A type satisfies an interface if it possesses all the methods the interface requires. For example, an *os.File satisfies io.Reader, Writer, Closer, and ReadWriter. A *bytes.Buffer satisfies Reader, Writer, and ReadWriter, but does not satisfy Closer because it does not have a Close method. As a shorthand, Go programmers often say that a concrete type ''is a'' particular interface type, meaning that it satisfies the interface. For example, a *bytes.Buffer is an io.Writer; an *os.File is an io.ReadWriter.

如果一个类型拥有接口要求的所有方法，则该类型满足该接口。例如，`*os.File`满足`io.Reader`、`Writer`、`Closer`和`ReadWriter`。`*bytes.Buffer`满足`Reader`、`Writer`和`ReadWriter`，但不满足`Closer`，因为它没有`Close`方法。作为简写，Go程序员经常说具体类型"是"某个特定的接口类型，意思是它满足该接口。例如，`*bytes.Buffer`是一个`io.Writer`；`*os.File`是一个`io.ReadWriter`。

The assignability rule (§2.4.2) for interfaces is very simple: an expression may be assigned to an interface only if its type satisfies the interface. So:

接口的可赋值规则（§2.4.2）非常简单：只有当表达式的类型满足接口时，才能将该表达式赋值给接口。因此：

```go
var w io.Writer
w = os.Stdout                 // OK: *os.File has Write method
w = new(bytes.Buffer)         // OK: *bytes.Buffer has Write method
w = time.Second               // compile error: time.Duration lacks Write method

var rwc io.ReadWriteCloser
rwc = os.Stdout               // OK: *os.File has Read, Write, Close methods
rwc = new(bytes.Buffer)       // compile error: *bytes.Buffer lacks Close method
```

This rule applies even when the right-hand side is itself an interface:

即使右侧本身是接口，此规则也适用：

```go
w = rwc        // OK: io.ReadWriteCloser has Write method
rwc = w        // compile error: io.Writer lacks Close method
```

Because ReadWriter and ReadWriteCloser include all the methods of Writer, any type that satisfies ReadWriter or ReadWriteCloser necessarily satisfies Writer.

因为`ReadWriter`和`ReadWriteCloser`包含`Writer`的所有方法，任何满足`ReadWriter`或`ReadWriteCloser`的类型必然满足`Writer`。

Before we go further, we should explain one subtlety in what it means for a type to have a method. Recall from Section 6.2 that for each named concrete type T, some of its methods have a receiver of type T itself whereas others require a *T pointer. Recall also that it is legal to call a *T method on an argument of type T so long as the argument is a variable; the compiler implicitly takes its address. But this is mere syntactic sugar: a value of type T does not possess all the methods that a *T pointer does, and as a result it might satisfy fewer interfaces.

在我们继续之前，我们应该解释类型拥有方法的含义中的一个微妙之处。回想第6.2节，对于每个命名的具体类型T，它的一些方法具有类型T本身的接收者，而另一些需要`*T`指针。还要记住，只要参数是变量，就可以合法地在类型T的参数上调用`*T`方法；编译器隐式获取其地址。但这只是语法糖：类型T的值并不拥有`*T`指针拥有的所有方法，因此它可能满足更少的接口。

An example will make this clear. The String method of the IntSet type from Section 6.5 requires a pointer receiver, so we cannot call that method on a non-addressable IntSet value:

一个例子将使这一点清楚。第6.5节中`IntSet`类型的`String`方法需要指针接收者，所以我们不能在不可寻址的`IntSet`值上调用该方法：

```go
type IntSet struct { /* ... */ }
func (*IntSet) String() string

var _ = IntSet{}.String() // compile error: String requires *IntSet receiver
```

but we can call it on an IntSet variable:

但我们可以在`IntSet`变量上调用它：

```go
var s IntSet
var _ = s.String() // OK: s is a variable and &s has a String method
```

However, since only *IntSet has a String method, only *IntSet satisfies the fmt.Stringer interface:

然而，由于只有`*IntSet`有`String`方法，只有`*IntSet`满足`fmt.Stringer`接口：

```go
var _ fmt.Stringer = &s  // OK
var _ fmt.Stringer = s   // compile error: IntSet lacks String method
```

Section 12.8 includes a program that prints the methods of an arbitrary value, and the godoc -analysis=typetool (§10.7.4) displays the methods of each type and the relationship between interfaces and concrete types.

第12.8节包含一个打印任意值方法的程序，`godoc -analysis=typetool`（§10.7.4）显示每种类型的方法以及接口和具体类型之间的关系。

Like an envelope that wraps and conceals the letter it holds, an interface wraps and conceals the concrete type and value that it holds. Only the methods revealed by the interface type may be called, even if the concrete type has others:

就像信封包装并隐藏它所装的信件一样，接口包装并隐藏它所持有的具体类型和值。只能调用接口类型所显示的方法，即使具体类型有其他方法：

```go
os.Stdout.Write([]byte("hello")) // OK: *os.File has Write method
os.Stdout.Close()                // OK: *os.File has Close method

var w io.Writer
w = os.Stdout
w.Write([]byte("hello"))         // OK: io.Writer has Write method
w.Close()                        // compile error: io.Writer lacks Close method
```

An interface with more methods, such as io.ReadWriter, tells us more about the values it contains, and places greater demands on the types that implement it, than does an interface with fewer methods such as io.Reader. So what does the type interface{}, which has no methods at all, tell us about the concrete types that satisfy it?

具有更多方法的接口（如`io.ReadWriter`）比具有较少方法的接口（如`io.Reader`）告诉我们更多关于它包含的值的信息，并对实现它的类型提出更高的要求。那么完全没有方法的类型`interface{}`告诉我们什么关于满足它的具体类型的信息呢？

That's right: nothing. This may seem useless, but in fact the type interface{}, which is called the empty interface type, is indispensable. Because the empty interface type places no demands on the types that satisfy it, we can assign any value to the empty interface.

没错：什么都不告诉我们。这可能看起来无用，但实际上类型`interface{}`（称为**空接口类型（empty interface type）**）是不可或缺的。因为空接口类型对满足它的类型没有任何要求，我们可以将任何值赋给空接口。

```go
var any interface{}
any = true
any = 12.34
any = "hello"
any = map[string]int{"one": 1}
any = new(bytes.Buffer)
```

Although it wasn't obvious, we've been using the empty interface type since the very first example in this book, because it is what allows functions like fmt.Println, or errorf in Section 5.7, to accept arguments of any type.

虽然这不明显，但我们从本书的第一个例子开始就一直在使用空接口类型，因为它是允许像`fmt.Println`或第5.7节中的`errorf`这样的函数接受任何类型参数的原因。

Of course, having created an interface{} value containing a boolean, float, string, map, pointer, or any other type, we can do nothing directly to the value it holds since the interface has no methods. We need a way to get the value back out again. We'll see how to do that using a type assertion in Section 7.10.

当然，创建了包含布尔值、浮点数、字符串、映射、指针或任何其他类型的`interface{}`值后，我们无法直接对它持有的值做任何事情，因为接口没有方法。我们需要一种方法将值重新取出。我们将在第7.10节中看到如何使用类型断言来做到这一点。

Since interface satisfaction depends only on the methods of the two types involved, there is no need to declare the relationship between a concrete type and the interfaces it satisfies. That said, it is occasionally useful to document and assert the relationship when it is intended but not otherwise enforced by the program. The declaration below asserts at compile time that a value of type *bytes.Buffer satisfies io.Writer:

由于接口满足只依赖于所涉及的两种类型的方法，因此无需声明具体类型与它满足的接口之间的关系。也就是说，当有意图但程序未强制执行这种关系时，记录和断言这种关系有时是有用的。下面的声明在编译时断言类型`*bytes.Buffer`的值满足`io.Writer`：

```go
// *bytes.Buffer must satisfy io.Writer
var w io.Writer = new(bytes.Buffer)
```

We needn't allocate a new variable since any value of type *bytes.Buffer will do, even nil, which we write as (*bytes.Buffer)(nil) using an explicit conversion. And since we never intend to refer to w, we can replace it with the blank identifier. Together, these changes give us this more frugal variant:

我们不需要分配新变量，因为任何类型`*bytes.Buffer`的值都可以，甚至是nil，我们使用显式转换将其写为`(*bytes.Buffer)(nil)`。由于我们从不打算引用w，我们可以用空白标识符替换它。这些更改一起给我们这个更节俭的变体：

```go
// *bytes.Buffer must satisfy io.Writer
var _ io.Writer = (*bytes.Buffer)(nil)
```

Non-empty interface types such as io.Writer are most often satisfied by a pointer type, particularly when one or more of the interface methods implies some kind of mutation to the receiver, as the Write method does. A pointer to a struct is an especially common method-bearing type.

非空接口类型（如`io.Writer`）最常由指针类型满足，特别是当接口方法中的一个或多个暗示对接收者进行某种修改时，如`Write`方法所做的那样。指向结构体的指针是一种特别常见的承载方法的类型。

But pointer types are by no means the only types that satisfy interfaces, and even interfaces with mutator methods may be satisfied by one of Go's other reference types. We've seen examples of slice types with methods (geometry.Path, §6.1) and map types with methods (url.Values, §6.2.1), and later we'll see a function type with methods (http.HandlerFunc, §7.7). Even basic types may satisfy interfaces; as we saw in Section 7.4, time.Duration satisfies fmt.Stringer.

但指针类型绝不是满足接口的唯一类型，甚至具有修改器方法的接口也可能由Go的其他引用类型之一满足。我们已经看到了带有方法的切片类型（`geometry.Path`，§6.1）和带有方法的映射类型（`url.Values`，§6.2.1）的例子，稍后我们将看到带有方法的函数类型（`http.HandlerFunc`，§7.7）。甚至基本类型也可能满足接口；正如我们在第7.4节中看到的，`time.Duration`满足`fmt.Stringer`。

A concrete type may satisfy many unrelated interfaces. Consider a program that organizes or sells digitized cultural artifacts like music, films, and books. It might define the following set of concrete types:

一个具体类型可能满足许多不相关的接口。考虑一个组织或销售数字化文化产品（如音乐、电影和书籍）的程序。它可能定义以下一组具体类型：

```
Album
Book
Movie
Magazine
Podcast
TVEpisode
Track
```

We can express each abstraction of interest as an interface. Some properties are common to all artifacts, such as a title, a creation date, and a list of creators (authors or artists).

我们可以将每个感兴趣的抽象表达为接口。一些属性对所有产品都是通用的，如标题、创建日期和创作者列表（作者或艺术家）。

```go
type Artifact interface {
    Title() string
    Creators() []string
    Created() time.Time
}
```

Other properties are restricted to certain types of artifacts. Properties of the printed word are relevant only to books and magazines, whereas only movies and TV episodes have a screen resolution.

其他属性仅限于某些类型的产品。印刷文字的属性只与书籍和杂志相关，而只有电影和电视剧集有屏幕分辨率。

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

这些接口只是将相关具体类型组合在一起并表达它们共同方面的一种有用方式。我们可能稍后发现其他分组。例如，如果我们发现需要以相同的方式处理`Audio`和`Video`项目，我们可以定义一个`Streamer`接口来表示它们的共同方面，而不需要更改任何现有的类型声明。

```go
type Streamer interface {
    Stream() (io.ReadCloser, error)
    RunningTime() time.Duration
    Format() string
}
```

Each grouping of concrete types based on their shared behaviors can be expressed as an interface type. Unlike class-based languages, in which the set of interfaces satisfied by a class is explicit, in Go we can define new abstractions or groupings of interest when we need them, without modifying the declaration of the concrete type. This is particularly useful when the concrete type comes from a package written by a different author. Of course, there do need to be underlying commonalities in the concrete types.

基于共同行为的每个具体类型分组都可以表达为接口类型。与基于类的语言（其中类满足的接口集合是明确的）不同，在Go中，我们可以在需要时定义新的抽象或感兴趣的分组，而不需要修改具体类型的声明。当具体类型来自不同作者编写的包时，这特别有用。当然，具体类型中确实需要有潜在的共同点。

## 7.4. Parsing Flags with flag.Value

## 7.4. 使用flag.Value解析标志

In this section, we'll see how another standard interface, flag.Value, helps us define new notations for command-line flags. Consider the program below, which sleeps for a specified period of time.

在本节中，我们将看到另一个标准接口`flag.Value`如何帮助我们为命令行标志定义新的表示法。考虑下面的程序，它休眠指定的时间段。

```go
// gopl.io/ch7/sleep
var period = flag.Duration("period", 1*time.Second, "sleep period")

func main() {
    flag.Parse()
    fmt.Printf("Sleeping for %v...", *period)
    time.Sleep(*period)
    fmt.Println()
}
```

Before it goes to sleep it prints the time period. The fmt package calls the time.Duration's String method to print the period not as a number of nanoseconds, but in a user-friendly notation:

在休眠之前，它打印时间段。`fmt`包调用`time.Duration`的`String`方法，不是将时间段打印为纳秒数，而是以用户友好的表示法：

```
$ go build gopl.io/ch7/sleep
$ ./sleep
Sleeping for 1s...
```

By default, the sleep period is one second, but it can be controlled through the -period command-line flag. The flag.Duration function creates a flag variable of type time.Duration and allows the user to specify the duration in a variety of user-friendly formats, including the same notation printed by the String method. This symmetry of design leads to a nice user interface.

默认情况下，休眠时间为一秒，但可以通过`-period`命令行标志进行控制。`flag.Duration`函数创建一个类型为`time.Duration`的标志变量，并允许用户以各种用户友好的格式指定持续时间，包括`String`方法打印的相同表示法。这种设计的对称性带来了良好的用户界面。

```
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

因为持续时间值标志如此有用，这个特性被内置到`flag`包中，但为我们自己的数据类型定义新的标志表示法很容易。我们只需要定义一个满足`flag.Value`接口的类型，其声明如下：

```go
package flag

// Value is the interface to the value stored in a flag.
type Value interface {
    String() string
    Set(string) error
}
```

The String method formats the flag's value for use in command-line help messages; thus every flag.Value is also a fmt.Stringer. The Set method parses its string argument and updates the flag value. In effect, the Set method is the inverse of the String method, and it is good practice for them to use the same notation.

`String`方法格式化标志的值以用于命令行帮助消息；因此每个`flag.Value`也是一个`fmt.Stringer`。`Set`方法解析其字符串参数并更新标志值。实际上，`Set`方法是`String`方法的逆操作，它们使用相同的表示法是良好的实践。

Let's define a celsiusFlag type that allows a temperature to be specified in Celsius, or in Fahrenheit with an appropriate conversion. Notice that celsiusFlag embeds a Celsius (§2.5), thereby getting a String method for free. To satisfy flag.Value, we need only declare the Set method:

让我们定义一个`celsiusFlag`类型，允许以摄氏度指定温度，或以华氏度指定并进行适当转换。注意`celsiusFlag`嵌入了一个`Celsius`（§2.5），从而免费获得了一个`String`方法。为了满足`flag.Value`，我们只需要声明`Set`方法：

```go
// gopl.io/ch7/tempconv
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

The call to fmt.Sscanf parses a floating-point number (value) and a string (unit) from the input s. Although one must usually check Sscanf's error result, in this case we don't need to because if there was a problem, no switch case will match.

对`fmt.Sscanf`的调用从输入s中解析一个浮点数（value）和一个字符串（unit）。尽管通常必须检查`Sscanf`的错误结果，但在这种情况下我们不需要，因为如果有问题，没有switch case会匹配。

The CelsiusFlag function below wraps it all up. To the caller, it returns a pointer to the Celsius field embedded within the celsiusFlag variable f. The Celsius field is the variable that will be updated by the Set method during flags processing. The call to Var adds the flag to the application's set of command-line flags, the global variable flag.CommandLine. Programs with unusually complex command-line interfaces may have several variables of this type. The call to Var assigns a *celsiusFlag argument to a flag.Value parameter, causing the compiler to check that *celsiusFlag has the necessary methods.

下面的`CelsiusFlag`函数将这一切包装起来。对调用者来说，它返回一个指向嵌入在`celsiusFlag`变量f中的`Celsius`字段的指针。`Celsius`字段是将在标志处理期间由`Set`方法更新的变量。对`Var`的调用将标志添加到应用程序的命令行标志集合中，即全局变量`flag.CommandLine`。具有异常复杂命令行界面的程序可能有几个这种类型的变量。对`Var`的调用将`*celsiusFlag`参数分配给`flag.Value`参数，导致编译器检查`*celsiusFlag`是否具有必要的方法。

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

现在我们可以开始在程序中使用新标志：

```go
// gopl.io/ch7/tempflag
var temp = tempconv.CelsiusFlag("temp", 20.0, "the temperature")

func main() {
    flag.Parse()
    fmt.Println(*temp)
}
```

Here's a typical session:

这是一个典型的会话：

```
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

**Exercise 7.6:** Add support for Kelvin temperatures to tempflag.

**练习7.6：** 为tempflag添加开尔文温度支持。

**Exercise 7.7:** Explain why the help message contains °C when the default value of 20.0 does not.

**练习7.7：** 解释为什么当默认值20.0不包含°C时，帮助消息却包含°C。

## 7.5. Interface Values

Conceptually, a value of an interface type, or interface value, has two components, a concrete type and a value of that type. These are called the interface's dynamic type and dynamic value.

从概念上讲，接口类型的值（或称为接口值）有两个组成部分：一个具体类型和该类型的一个值。这两个部分分别称为接口的**动态类型**（dynamic type）和**动态值**（dynamic value）。

For a statically typed language like Go, types are a compile-time concept, so a type is not a value. In our conceptual model, a set of values called type descriptors provide information about each type, such as its name and methods. In an interface value, the type component is represented by the appropriate type descriptor.

对于像Go这样的静态类型语言，类型是一个编译时概念，因此类型本身不是一个值。在我们的概念模型中，一组称为类型描述符（type descriptors）的值提供了关于每个类型的信息，比如类型的名称和方法。在接口值中，类型组件由相应的类型描述符表示。

In the four statements below, the variable w takes on three different values. (The initial and final values are the same.)

在下面的四条语句中，变量w被赋予了三个不同的值。（初始值和最终值是相同的。）

```go
var w io.Writer
w = os.Stdout
w = new(bytes.Buffer)
w = nil
```

Let's take a closer look at the value and dynamic behavior of w after each statement. The first statement declares w:

让我们仔细看看每条语句执行后w的值和动态行为。第一条语句声明了w：

```go
var w io.Writer
```

In Go, variables are always initialized to a well-defined value, and interfaces are no exception. The zero value for an interface has both its type and value components set to nil (Figure 7.1).

在Go中，变量总是被初始化为一个明确定义的值，接口也不例外。接口的零值将其类型和值组件都设置为nil（图7.1）。

**Figure 7.1. A nil interface value.**

**图7.1. 一个nil接口值。**

An interface value is described as nil or non-nil based on its dynamic type, so this is a nil interface value. You can test whether an interface value is nil using w == nil or w != nil. Calling any method of a nil interface value causes a panic:

接口值根据其动态类型被描述为nil或非nil，所以这是一个nil接口值。你可以使用`w == nil`或`w != nil`来测试接口值是否为nil。调用nil接口值的任何方法都会引起panic：

```go
w.Write([]byte("hello")) // panic: nil pointer dereference
```

The second statement assigns a value of type *os.File to w:

第二条语句将一个*os.File类型的值赋给w：

```go
w = os.Stdout
```

This assignment involves an implicit conversion from a concrete type to an interface type, and is equivalent to the explicit conversion io.Writer(os.Stdout). A conversion of this kind, whether explicit or implicit, captures the type and the value of its operand. The interface value's dynamic type is set to the type descriptor for the pointer type *os.File, and its dynamic value holds a copy of os.Stdout, which is a pointer to the os.File variable representing the standard output of the process (Figure 7.2).

这个赋值涉及从具体类型到接口类型的隐式转换，等价于显式转换`io.Writer(os.Stdout)`。这种转换，无论是显式的还是隐式的，都会捕获其操作数的类型和值。接口值的动态类型被设置为指针类型*os.File的类型描述符，其动态值保存os.Stdout的副本，它是指向表示进程标准输出的os.File变量的指针（图7.2）。

**Figure 7.2. An interface value containing an *os.File pointer.**

**图7.2. 包含*os.File指针的接口值。**

Calling the Write method on an interface value containing an *os.File pointer causes the (*os.File).Write method to be called. The call prints "hello".

在包含*os.File指针的接口值上调用Write方法会导致调用(*os.File).Write方法。该调用打印"hello"。

```go
w.Write([]byte("hello")) // "hello"
```

In general, we cannot know at compile time what the dynamic type of an interface value will be, so a call through an interface must use dynamic dispatch. Instead of a direct call, the compiler must generate code to obtain the address of the method named Write from the type descriptor, then make an indirect call to that address. The receiver argument for the call is a copy of the interface's dynamic value, os.Stdout. The effect is as if we had made this call directly:

一般来说，我们无法在编译时知道接口值的动态类型是什么，所以通过接口的调用必须使用动态分发（dynamic dispatch）。编译器必须生成代码从类型描述符中获取名为Write的方法的地址，然后对该地址进行间接调用，而不是直接调用。调用的接收者参数是接口动态值的副本，即os.Stdout。效果就好像我们直接进行了这个调用：

```go
os.Stdout.Write([]byte("hello")) // "hello"
```

The third statement assigns a value of type *bytes.Buffer to the interface value:

第三条语句将一个*bytes.Buffer类型的值赋给接口值：

```go
w = new(bytes.Buffer)
```

The dynamic type is now *bytes.Buffer and the dynamic value is a pointer to the newly allocated buffer (Figure 7.3).

现在动态类型是*bytes.Buffer，动态值是指向新分配缓冲区的指针（图7.3）。

**Figure 7.3. An interface value containing a *bytes.Buffer pointer.**

**图7.3. 包含*bytes.Buffer指针的接口值。**

A call to the Write method uses the same mechanism as before:

对Write方法的调用使用与之前相同的机制：

```go
w.Write([]byte("hello")) // writes "hello" to the bytes.Buffer
```

This time, the type descriptor is *bytes.Buffer, so the (*bytes.Buffer).Write method is called, with the address of the buffer as the value of the receiver parameter. The call appends "hello" to the buffer.

这次，类型描述符是*bytes.Buffer，所以调用(*bytes.Buffer).Write方法，缓冲区的地址作为接收者参数的值。调用将"hello"追加到缓冲区中。

Finally, the fourth statement assigns nil to the interface value:

最后，第四条语句将nil赋给接口值：

```go
w = nil
```

This resets both its components to nil, restoring w to the same state as when it was declared, which was shown in Figure 7.1.

这将其两个组件都重置为nil，将w恢复到声明时的相同状态，如图7.1所示。

An interface value can hold arbitrarily large dynamic values. For example, the time.Time type, which represents an instant in time, is a struct type with several unexported fields. If we create an interface value from it,

接口值可以保存任意大的动态值。例如，表示时间瞬间的time.Time类型是一个包含多个未导出字段的结构体类型。如果我们从它创建一个接口值：

```go
var x interface{} = time.Now()
```

the result might look like Figure 7.4. Conceptually, the dynamic value always fits inside the interface value, no matter how large its type. (This is only a conceptual model; a realistic implementation is quite different.)

结果可能看起来像图7.4。从概念上讲，无论其类型多大，动态值总是适合在接口值内部。（这只是一个概念模型；实际的实现相当不同。）

**Figure 7.4. An interface value holding a time.Time struct.**

**图7.4. 保存time.Time结构体的接口值。**

Interface values may be compared using == and !=. Two interface values are equal if both are nil, or if their dynamic types are identical and their dynamic values are equal according to the usual behavior of == for that type. Because interface values are comparable, they may be used as the keys of a map or as the operand of a switch statement.

接口值可以使用==和!=进行比较。如果两个接口值都是nil，或者它们的动态类型相同且动态值根据该类型的==的常规行为相等，则这两个接口值相等。因为接口值是可比较的，所以它们可以用作映射的键或switch语句的操作数。

However, if two interface values are compared and have the same dynamic type, but that type is not comparable (a slice, for instance), then the comparison fails with a panic:

但是，如果比较两个接口值时它们具有相同的动态类型，但该类型不可比较（例如切片），那么比较会失败并引起panic：

```go
var x interface{} = []int{1, 2, 3}
fmt.Println(x == x) // panic: comparing uncomparable type []int
```

In this respect, interface types are unusual. Other types are either safely comparable (like basic types and pointers) or not comparable at all (like slices, maps, and functions), but when comparing interface values or aggregate types that contain interface values, we must be aware of the potential for a panic. A similar risk exists when using interfaces as map keys or switch operands. Only compare interface values if you are certain that they contain dynamic values of comparable types.

在这方面，接口类型是不寻常的。其他类型要么是安全可比较的（如基本类型和指针），要么完全不可比较（如切片、映射和函数），但是当比较接口值或包含接口值的聚合类型时，我们必须意识到可能出现panic的风险。在将接口用作映射键或switch操作数时也存在类似的风险。只有在确定接口值包含可比较类型的动态值时才比较接口值。

When handling errors, or during debugging, it is often helpful to report the dynamic type of an interface value. For that, we use the fmt package's %T verb:

在处理错误或调试时，报告接口值的动态类型通常很有帮助。为此，我们使用fmt包的%T动词：

```go
var w io.Writer
fmt.Printf("%T\n", w) // "<nil>"
w = os.Stdout
fmt.Printf("%T\n", w) // "*os.File"
w = new(bytes.Buffer)
fmt.Printf("%T\n", w) // "*bytes.Buffer"
```

Internally, fmt uses reflection to obtain the name of the interface's dynamic type. We'll look at reflection in Chapter 12.

在内部，fmt使用反射（reflection）来获取接口动态类型的名称。我们将在第12章讨论反射。

### 7.5.1. Caveat: An Interface Containing a Nil Pointer Is Non-Nil

### 7.5.1. 注意事项：包含nil指针的接口是非nil的

A nil interface value, which contains no value at all, is not the same as an interface value containing a pointer that happens to be nil. This subtle distinction creates a trap into which every Go programmer has stumbled.

一个完全不包含任何值的nil接口值，与包含一个恰好为nil的指针的接口值是不同的。这种微妙的区别造成了一个陷阱，每个Go程序员都曾经踩过。

Consider the program below. With debug set to true, the main function collects the output of the function f in a bytes.Buffer.

考虑下面的程序。当debug设置为true时，main函数将函数f的输出收集到bytes.Buffer中。

```go
const debug = true

func main() {
    var buf *bytes.Buffer
    if debug {
        buf = new(bytes.Buffer) // enable collection of output
    }
    f(buf) // NOTE: subtly incorrect!
    if debug {
        // ...use buf...
    }
}

// If out is non-nil, output will be written to it.
func f(out io.Writer) {
    // ...do something...
    if out != nil {
        out.Write([]byte("done!\n"))
    }
}
```

We might expect that changing debug to false would disable the collection of the output, but in fact it causes the program to panic during the out.Write call:

我们可能期望将debug改为false会禁用输出收集，但实际上它会导致程序在out.Write调用时panic：

```go
if out != nil {
    out.Write([]byte("done!\n")) // panic: nil pointer dereference
}
```

When main calls f, it assigns a nil pointer of type *bytes.Buffer to the out parameter, so the dynamic value of out is nil. However, its dynamic type is *bytes.Buffer, meaning that out is a non-nil interface containing a nil pointer value (Figure 7.5), so the defensive check out != nil is still true.

当main调用f时，它将一个*bytes.Buffer类型的nil指针赋给out参数，所以out的动态值是nil。但是，它的动态类型是*bytes.Buffer，这意味着out是一个包含nil指针值的非nil接口（图7.5），所以防御性检查`out != nil`仍然为真。

**Figure 7.5. A non-nil interface containing a nil pointer.**

**图7.5. 包含nil指针的非nil接口。**

As before, the dynamic dispatch mechanism determines that (*bytes.Buffer).Write must be called but this time with a receiver value that is nil. For some types, such as *os.File, nil is a valid receiver (§6.2.1), but *bytes.Buffer is not among them. The method is called, but it panics as it tries to access the buffer.

如前所述，动态分发机制确定必须调用(*bytes.Buffer).Write，但这次接收者值是nil。对于某些类型，如*os.File，nil是一个有效的接收者（§6.2.1），但*bytes.Buffer不在其中。方法被调用，但在尝试访问缓冲区时panic。

The problem is that although a nil *bytes.Buffer pointer has the methods needed to satisfy the interface, it doesn't satisfy the behavioral requirements of the interface. In particular, the call violates the implicit precondition of (*bytes.Buffer).Write that its receiver is not nil, so assigning the nil pointer to the interface was a mistake. The solution is to change the type of buf in main to io.Writer, thereby avoiding the assignment of the dysfunctional value to the interface in the first place:

问题在于，虽然nil的*bytes.Buffer指针具有满足接口所需的方法，但它不满足接口的行为要求。特别是，该调用违反了(*bytes.Buffer).Write的隐式前提条件，即其接收者不为nil，所以将nil指针赋给接口是一个错误。解决方案是将main中buf的类型改为io.Writer，从而首先避免将功能失常的值赋给接口：

```go
var buf io.Writer
if debug {
    buf = new(bytes.Buffer) // enable collection of output
}
f(buf) // OK
```

Now that we've covered the mechanics of interface values, let's take a look at some more important interfaces from Go's standard library. In the next three sections, we'll see how interfaces are used for sorting, web serving, and error handling.

现在我们已经介绍了接口值的机制，让我们看看Go标准库中一些更重要的接口。在接下来的三节中，我们将看到接口如何用于排序、Web服务和错误处理。

## 7.6. Sorting with sort.Interface

## 7.6. 使用sort.Interface进行排序

Like string formatting, sorting is a frequently used operation in many programs. Although a minimal Quicksort can be written in about 15 lines, a robust implementation is much longer, and it is not the kind of code we should wish to write anew or copy each time we need it.

与字符串格式化一样，排序是许多程序中经常使用的操作。虽然一个最小的快速排序可以用大约15行代码编写，但一个健壮的实现要长得多，这不是我们每次需要时都希望重新编写或复制的那种代码。

Fortunately, the sort package provides in-place sorting of any sequence according to any ordering function. Its design is rather unusual. In many languages, the sorting algorithm is associated with the sequence data type, while the ordering function is associated with the type of the elements. By contrast, Go's sort.Sort function assumes nothing about the representation of either the sequence or its elements. Instead, it uses an interface, sort.Interface, to specify the contract between the generic sort algorithm and each sequence type that may be sorted. An implementation of this interface determines both the concrete representation of the sequence, which is often a slice, and the desired ordering of its elements.

幸运的是，sort包提供了根据任何排序函数对任何序列进行原地排序的功能。它的设计相当不寻常。在许多语言中，排序算法与序列数据类型相关联，而排序函数与元素的类型相关联。相比之下，Go的sort.Sort函数对序列或其元素的表示不做任何假设。相反，它使用一个接口sort.Interface来指定通用排序算法与每个可能被排序的序列类型之间的约定。这个接口的实现既确定了序列的具体表示（通常是切片），也确定了其元素的期望排序。

An in-place sort algorithm needs three things—the length of the sequence, a means of comparing two elements, and a way to swap two elements—so they are the three methods of sort.Interface:

原地排序算法需要三样东西——序列的长度、比较两个元素的方法和交换两个元素的方法——所以它们是sort.Interface的三个方法：

```go
package sort

type Interface interface {
    Len() int
    Less(i, j int) bool // i, j are indices of sequence elements
    Swap(i, j int)
}
```

To sort any sequence, we need to define a type that implements these three methods, then apply sort.Sort to an instance of that type. As perhaps the simplest example, consider sorting a slice of strings. The new type StringSlice and its Len, Less, and Swap methods are shown below.

要对任何序列进行排序，我们需要定义一个实现这三个方法的类型，然后将sort.Sort应用于该类型的实例。作为可能最简单的例子，考虑对字符串切片进行排序。新类型StringSlice及其Len、Less和Swap方法如下所示。

```go
type StringSlice []string
func (p StringSlice) Len() int           { return len(p) }
func (p StringSlice) Less(i, j int) bool { return p[i] < p[j] }
func (p StringSlice) Swap(i, j int)      { p[i], p[j] = p[j], p[i] }
```

Now we can sort a slice of strings, names, by converting the slice to a StringSlice like this:

现在我们可以通过将切片转换为StringSlice来对字符串切片names进行排序：

```go
sort.Sort(StringSlice(names))
```

The conversion yields a slice value with the same length, capacity, and underlying array as names but with a type that has the three methods required for sorting.

这个转换产生一个与names具有相同长度、容量和底层数组的切片值，但类型具有排序所需的三个方法。

Sorting a slice of strings is so common that the sort package provides the StringSlice type, as well as a function called Strings so that the call above can be simplified to sort.Strings(names).

对字符串切片排序非常常见，所以sort包提供了StringSlice类型，以及一个名为Strings的函数，这样上面的调用可以简化为`sort.Strings(names)`。

The technique here is easily adapted to other sort orders, for instance, to ignore capitalization or special characters. (The Go program that sorts index terms and page numbers for this book does this, with extra logic for Roman numerals.) For more complicated sorting, we use the same idea, but with more complicated data structures or more complicated implementations of the sort.Interface methods.

这里的技术很容易适用于其他排序顺序，例如，忽略大小写或特殊字符。（为本书的索引术语和页码排序的Go程序就是这样做的，还有处理罗马数字的额外逻辑。）对于更复杂的排序，我们使用相同的思想，但使用更复杂的数据结构或更复杂的sort.Interface方法实现。

Our running example for sorting will be a music playlist, displayed as a table. Each track is a single row, and each column is an attribute of that track, like artist, title, and running time. Imagine that a graphical user interface presents the table, and that clicking the head of a column causes the playlist to be sorted by that attribute; clicking the same column head again reverses the order. Let's look at what might happen in response to each click.

我们排序的运行示例将是一个音乐播放列表，显示为表格。每个曲目是一行，每列是该曲目的一个属性，如艺术家、标题和播放时间。想象一个图形用户界面呈现这个表格，点击列标题会导致播放列表按该属性排序；再次点击同一列标题会反转顺序。让我们看看响应每次点击可能发生什么。

The variable tracks below contains a playlist. (One of the authors apologizes for the other author's musical tastes.) Each element is indirect, a pointer to a Track. Although the code below would work if we stored the Tracks directly, the sort function will swap many pairs of elements, so it will run faster if each element is a pointer, which is a single machine word, instead of an entire Track, which might be eight words or more.

下面的变量tracks包含一个播放列表。（其中一位作者为另一位作者的音乐品味道歉。）每个元素都是间接的，即指向Track的指针。虽然如果我们直接存储Track，下面的代码也能工作，但排序函数会交换许多对元素，所以如果每个元素是一个指针（单个机器字），而不是整个Track（可能是八个字或更多），它会运行得更快。

```go
gopl.io/ch7/sorting
type Track struct {
    Title  string
    Artist string
    Album  string
    Year   int
    Length time.Duration
}

var tracks = []*Track{
    {"Go", "Delilah", "From the Roots Up", 2012, length("3m38s")},
    {"Go", "Moby", "Moby", 1992, length("3m37s")},
    {"Go Ahead", "Alicia Keys", "As I Am", 2007, length("4m36s")},
    {"Ready 2 Go", "Martin Solveig", "Smash", 2011, length("4m24s")},
}

func length(s string) time.Duration {
    d, err := time.ParseDuration(s)
    if err != nil {
        panic(s)
    }
    return d
}
```

The printTracks function prints the playlist as a table. A graphical display would be nicer, but this little routine uses the text/tabwriter package to produce a table whose columns are neatly aligned and padded as shown below. Observe that *tabwriter.Writer satisfies io.Writer. It collects each piece of data written to it; its Flush method formats the entire table and writes it to os.Stdout.

printTracks函数将播放列表打印为表格。图形显示会更好，但这个小例程使用text/tabwriter包生成一个列整齐对齐和填充的表格，如下所示。注意*tabwriter.Writer满足io.Writer。它收集写入它的每一块数据；它的Flush方法格式化整个表格并将其写入os.Stdout。

```go
func printTracks(tracks []*Track) {
    const format = "%v\t%v\t%v\t%v\t%v\t\n"
    tw := new(tabwriter.Writer).Init(os.Stdout, 0, 8, 2, ' ', 0)
    fmt.Fprintf(tw, format, "Title", "Artist", "Album", "Year", "Length")
    fmt.Fprintf(tw, format, "-----", "------", "-----", "----", "------")
    for _, t := range tracks {
        fmt.Fprintf(tw, format, t.Title, t.Artist, t.Album, t.Year, t.Length)
    }
    tw.Flush() // calculate column widths and print table
}
```

To sort the playlist by the Artist field, we define a new slice type with the necessary Len, Less, and Swap methods, analogous to what we did for StringSlice.

要按Artist字段对播放列表排序，我们定义一个新的切片类型，具有必要的Len、Less和Swap方法，类似于我们为StringSlice所做的。

```go
type byArtist []*Track

func (x byArtist) Len() int           { return len(x) }
func (x byArtist) Less(i, j int) bool { return x[i].Artist < x[j].Artist }
func (x byArtist) Swap(i, j int)      { x[i], x[j] = x[j], x[i] }
```

To call the generic sort routine, we must first convert tracks to the new type, byArtist, that defines the order:

要调用通用排序例程，我们必须首先将tracks转换为定义顺序的新类型byArtist：

```go
sort.Sort(byArtist(tracks))
```

After sorting the slice by artist, the output from printTracks is

按艺术家排序切片后，printTracks的输出是：

```
Title        Artist       Album              Year  Length
-----        ------       -----              ----  ------
Go Ahead     Alicia Keys  As I Am            2007  4m36s
Go           Delilah      From the Roots Up  2012  3m38s
Ready 2 Go   Martin Solveig Smash           2011  4m24s
Go           Moby         Moby               1992  3m37s
```

If the user requests ''sort by artist'' a second time, we'll sort the tracks in reverse. We needn't define a new type byReverseArtist with an inverted Less method, however, since the sort package provides a Reverse function that transforms any sort order to its inverse.

如果用户第二次请求"按艺术家排序"，我们将按相反顺序对曲目排序。但是，我们不需要定义一个具有倒置Less方法的新类型byReverseArtist，因为sort包提供了一个Reverse函数，它将任何排序顺序转换为其逆序。

```go
sort.Sort(sort.Reverse(byArtist(tracks)))
```

After reverse-sorting the slice by artist, the output from printTracks is

按艺术家逆序排序切片后，printTracks的输出是：

```
Title        Artist        Album              Year  Length
-----        ------        -----              ----  ------
Go           Moby          Moby               1992  3m37s
Ready 2 Go   Martin Solveig Smash            2011  4m24s
Go           Delilah       From the Roots Up  2012  3m38s
Go Ahead     Alicia Keys   As I Am            2007  4m36s
```

The sort.Reverse function deserves a closer look since it uses composition (§6.3), which is an important idea. The sort package defines an unexported type reverse, which is a struct that embeds a sort.Interface. The Less method for reverse calls the Less method of the embedded sort.Interface value, but with the indices flipped, reversing the order of the sort results.

sort.Reverse函数值得仔细看看，因为它使用了组合（§6.3），这是一个重要的思想。sort包定义了一个未导出的类型reverse，它是一个嵌入sort.Interface的结构体。reverse的Less方法调用嵌入的sort.Interface值的Less方法，但交换了索引，从而逆转了排序结果的顺序。

```go
package sort

type reverse struct{ Interface } // that is, sort.Interface

func (r reverse) Less(i, j int) bool { return r.Interface.Less(j, i) }
func Reverse(data Interface) Interface { return reverse{data} }
```

Len and Swap, the other two methods of reverse, are implicitly provided by the original sort.Interface value because it is an embedded field. The exported function Reverse returns an instance of the reverse type that contains the original sort.Interface value.

reverse的另外两个方法Len和Swap由原始的sort.Interface值隐式提供，因为它是一个嵌入字段。导出的函数Reverse返回包含原始sort.Interface值的reverse类型的实例。

To sort by a different column, we must define a new type, such as byYear:

要按不同的列排序，我们必须定义一个新类型，比如byYear：

```go
type byYear []*Track

func (x byYear) Len() int           { return len(x) }
func (x byYear) Less(i, j int) bool { return x[i].Year < x[j].Year }
func (x byYear) Swap(i, j int)      { x[i], x[j] = x[j], x[i] }
```

After sorting tracks by year using sort.Sort(byYear(tracks)), printTracks shows a chronological listing:

使用`sort.Sort(byYear(tracks))`按年份排序tracks后，printTracks显示按时间顺序的列表：

```
Title        Artist        Album              Year  Length
-----        ------        -----              ----  ------
Go           Moby          Moby               1992  3m37s
Go Ahead     Alicia Keys   As I Am            2007  4m36s
Ready 2 Go   Martin Solveig Smash            2011  4m24s
Go           Delilah       From the Roots Up  2012  3m38s
```

For every slice element type and every ordering function we need, we declare a new implementation of sort.Interface. As you can see, the Len and Swap methods have identical definitions for all slice types. In the next example, the concrete type customSort combines a slice with a function, letting us define a new sort order by writing only the comparison function. Incidentally, the concrete types that implement sort.Interface are not always slices; customSort is a struct type.

对于我们需要的每个切片元素类型和每个排序函数，我们都声明一个新的sort.Interface实现。如你所见，对于所有切片类型，Len和Swap方法都有相同的定义。在下一个例子中，具体类型customSort将切片与函数结合，让我们只需编写比较函数就能定义新的排序顺序。顺便说一下，实现sort.Interface的具体类型并不总是切片；customSort是一个结构体类型。

```go
type customSort struct {
    t    []*Track
    less func(x, y *Track) bool
}

func (x customSort) Len() int           { return len(x.t) }
func (x customSort) Less(i, j int) bool { return x.less(x.t[i], x.t[j]) }
func (x customSort) Swap(i, j int)      { x.t[i], x.t[j] = x.t[j], x.t[i] }
```

Let's define a multi-tier ordering function whose primary sort key is the Title, whose secondary key is the Year, and whose tertiary key is the running time, Length. Here's the call to Sort using an anonymous ordering function:

让我们定义一个多层排序函数，其主要排序键是Title，次要键是Year，第三键是播放时间Length。以下是使用匿名排序函数调用Sort的代码：

```go
sort.Sort(customSort{tracks, func(x, y *Track) bool {
    if x.Title != y.Title {
        return x.Title < y.Title
    }
    if x.Year != y.Year {
        return x.Year < y.Year
    }
    if x.Length != y.Length {
        return x.Length < y.Length
    }
    return false
}})
```

And here's the result. Notice that the tie between the two tracks titled ''Go'' is broken in favor of the older one.

结果如下。注意两个标题为"Go"的曲目之间的平局被打破，偏向于较早的那个。

```
Title        Artist        Album              Year  Length
-----        ------        -----              ----  ------
Go           Moby          Moby               1992  3m37s
Go           Delilah       From the Roots Up  2012  3m38s
Go Ahead     Alicia Keys   As I Am            2007  4m36s
Ready 2 Go   Martin Solveig Smash            2011  4m24s
```

Although sorting a sequence of length n requires O(n log n) comparison operations, testing whether a sequence is already sorted requires at most n−1 comparisons. The IsSorted function from the sort package checks this for us. Like sort.Sort, it abstracts both the sequence and its ordering function using sort.Interface, but it never calls the Swap method: This code demonstrates the IntsAreSorted and Ints functions and the IntSlice type:

虽然对长度为n的序列排序需要O(n log n)次比较操作，但测试序列是否已排序最多需要n−1次比较。sort包中的IsSorted函数为我们检查这一点。像sort.Sort一样，它使用sort.Interface抽象序列及其排序函数，但它从不调用Swap方法：这段代码演示了IntsAreSorted和Ints函数以及IntSlice类型：

```go
values := []int{3, 1, 4, 1}
fmt.Println(sort.IntsAreSorted(values)) // "false"
sort.Ints(values)
fmt.Println(values)                     // "[1 1 3 4]"
fmt.Println(sort.IntsAreSorted(values)) // "true"
sort.Sort(sort.Reverse(sort.IntSlice(values)))
fmt.Println(values)                     // "[4 3 1 1]"
fmt.Println(sort.IntsAreSorted(values)) // "false"
```

For convenience, the sort package provides versions of its functions and types specialized for []int, []string, and []float64 using their natural orderings. For other types, such as []int64 or []uint, we're on our own, though the path is short.

为了方便，sort包提供了针对[]int、[]string和[]float64使用其自然排序的函数和类型的专门版本。对于其他类型，如[]int64或[]uint，我们需要自己处理，尽管路径很短。

**Exercise 7.8:** Many GUIs provide a table widget with a stateful multi-tier sort: the primary sort key is the most recently clicked column head, the secondary sort key is the second-most recently clicked column head, and so on. Define an implementation of sort.Interface for use by such a table. Compare that approach with repeated sorting using sort.Stable.

**练习7.8：** 许多GUI提供一个具有状态多层排序的表格小部件：主要排序键是最近点击的列标题，次要排序键是第二个最近点击的列标题，依此类推。为这样的表格定义一个sort.Interface的实现。将该方法与使用sort.Stable的重复排序进行比较。

**Exercise 7.9:** Use the html/template package (§4.6) to replace printTracks with a function that displays the tracks as an HTML table. Use the solution to the previous exercise to arrange that each click on a column head makes an HTTP request to sort the table.

**练习7.9：** 使用html/template包（§4.6）用一个将曲目显示为HTML表格的函数替换printTracks。使用前一个练习的解决方案来安排每次点击列标题都发出HTTP请求来对表格排序。

**Exercise 7.10:** The sort.Interface type can be adapted to other uses. Write a function IsPalindrome(s sort.Interface) bool that reports whether the sequence s is a palindrome, in other words, reversing the sequence would not change it. Assume that the elements at indices i and j are equal if !s.Less(i, j) && !s.Less(j, i).

**练习7.10：** sort.Interface类型可以适用于其他用途。编写一个函数`IsPalindrome(s sort.Interface) bool`，报告序列s是否是回文，换句话说，反转序列不会改变它。假设如果`!s.Less(i, j) && !s.Less(j, i)`，则索引i和j处的元素相等。

## 7.7. The http.Handler Interface

## 7.7. http.Handler 接口

In Chapter 1, we saw a glimpse of how to use the net/http package to implement web clients (§1.5) and servers (§1.7). In this section, we'll look more closely at the server API, whose foundation is the http.Handler interface:

在第1章中，我们简要了解了如何使用 net/http 包实现 Web 客户端（§1.5）和服务器（§1.7）。在本节中，我们将更深入地研究服务器 API，其基础是 http.Handler 接口：

```go
package http

type Handler interface {
    ServeHTTP(w ResponseWriter, r *Request)
}

func ListenAndServe(address string, h Handler) error
```

The ListenAndServe function requires a server address, such as "localhost:8000", and an instance of the Handler interface to which all requests should be dispatched. It runs forever, or until the server fails (or fails to start) with an error, always non-nil, which it returns.

ListenAndServe 函数需要一个服务器地址，比如 "localhost:8000"，以及一个 Handler 接口的实例，所有请求都将分发给它。该函数会一直运行，直到服务器失败（或启动失败）并返回一个错误，返回值总是非 nil。

Imagine an e-commerce site with a database mapping the items for sale to their prices in dollars. The program below shows the simplest imaginable implementation. It models the inventory as a map type, database, to which we've attached a ServeHTTP method so that it satisfies the http.Handler interface. The handler ranges over the map and prints the items.

想象一个电子商务网站，其数据库将待售商品映射到以美元计价的价格。下面的程序展示了最简单的实现。它将库存建模为一个 map 类型 database，我们为其附加了一个 ServeHTTP 方法，使其满足 http.Handler 接口。该处理器遍历 map 并打印商品信息。

```go
gopl.io/ch7/http1

func main() {
    db := database{"shoes": 50, "socks": 5}
    log.Fatal(http.ListenAndServe("localhost:8000", db))
}
```

```go
type dollars float32

func (d dollars) String() string { return fmt.Sprintf("$%.2f", d) }

type database map[string]dollars

func (db database) ServeHTTP(w http.ResponseWriter, req *http.Request) {
    for item, price := range db {
        fmt.Fprintf(w, "%s: %s\n", item, price)
    }
}
```

If we start the server,

如果我们启动服务器，

```
$ go build gopl.io/ch7/http1
$ ./http1 &
```

then connect to it with the fetch program from Section 1.5 (or a web browser if you prefer), we get the following output:

然后使用第1.5节的 fetch 程序连接到它（或者使用你喜欢的 Web 浏览器），我们会得到以下输出：

```
$ go build gopl.io/ch1/fetch
$ ./fetch http://localhost:8000
shoes: $50.00
socks: $5.00
```

So far, the server can only list its entire inventory and will do this for every request, regardless of URL. A more realistic server defines multiple different URLs, each triggering a different behavior. Let's call the existing one /list and add another one called /price that reports the price of a single item, specified as a request parameter like /price?item=socks.

到目前为止，服务器只能列出其全部库存，并且无论 URL 是什么，每个请求都会这样做。一个更实际的服务器会定义多个不同的 URL，每个 URL 触发不同的行为。让我们把现有的称为 /list，并添加另一个名为 /price 的，它报告单个商品的价格，通过请求参数指定，如 /price?item=socks。

```go
gopl.io/ch7/http2

func (db database) ServeHTTP(w http.ResponseWriter, req *http.Request) {
    switch req.URL.Path {
    case "/list":
        for item, price := range db {
            fmt.Fprintf(w, "%s: %s\n", item, price)
        }
    case "/price":
        item := req.URL.Query().Get("item")
        price, ok := db[item]
        if !ok {
            w.WriteHeader(http.StatusNotFound) // 404
            fmt.Fprintf(w, "no such item: %q\n", item)
            return
        }
        fmt.Fprintf(w, "%s\n", price)
    default:
        w.WriteHeader(http.StatusNotFound) // 404
        fmt.Fprintf(w, "no such page: %s\n", req.URL)
    }
}
```

Now the handler decides what logic to execute based on the path component of the URL, req.URL.Path. If the handler doesn't recognize the path, it reports an HTTP error to the client by calling w.WriteHeader(http.StatusNotFound); this must be done before writing any text to w. (Incidentally, http.ResponseWriter is another interface. It augments io.Writer with methods for sending HTTP response headers.) Equivalently, we could use the http.Error utility function:

现在处理器根据 URL 的路径部分 req.URL.Path 来决定执行什么逻辑。如果处理器不识别该路径，它会通过调用 w.WriteHeader(http.StatusNotFound) 向客户端报告 HTTP 错误；这必须在向 w 写入任何文本之前完成。（顺便说一下，http.ResponseWriter 是另一个接口。它增强了 io.Writer，添加了发送 HTTP 响应头的方法。）等效地，我们可以使用 http.Error 实用函数：

```go
msg := fmt.Sprintf("no such page: %s\n", req.URL)
http.Error(w, msg, http.StatusNotFound) // 404
```

The case for /price calls the URL's Query method to parse the HTTP request parameters as a map, or more precisely, a multimap of type url.Values (§6.2.1) from the net/url package. It then finds the first item parameter and looks up its price. If the item wasn't found, it reports an error.

/price 的 case 调用 URL 的 Query 方法来将 HTTP 请求参数解析为一个 map，或者更准确地说，是来自 net/url 包的 url.Values 类型的多重映射（§6.2.1）。然后它找到第一个 item 参数并查询其价格。如果找不到该商品，它会报告一个错误。

Here's an example session with the new server:

这是新服务器的一个示例会话：

```
$ go build gopl.io/ch7/http2
$ go build gopl.io/ch1/fetch
$ ./http2 &
$ ./fetch http://localhost:8000/list
shoes: $50.00
socks: $5.00
$ ./fetch http://localhost:8000/price?item=socks
$5.00
$ ./fetch http://localhost:8000/price?item=shoes
$50.00
$ ./fetch http://localhost:8000/price?item=hat
no such item: "hat"
$ ./fetch http://localhost:8000/help
no such page: /help
```

Obviously we could keep adding cases to ServeHTTP, but in a realistic application, it's convenient to define the logic for each case in a separate function or method. Furthermore, related URLs may need similar logic; several image files may have URLs of the form /images/*.png, for instance. For these reasons, net/http provides ServeMux, a request multiplexer, to simplify the association between URLs and handlers. A ServeMux aggregates a collection of http.Handlers into a single http.Handler. Again, we see that different types satisfying the same interface are substitutable: the web server can dispatch requests to any http.Handler, regardless of which concrete type is behind it.

显然我们可以继续向 ServeHTTP 添加 case，但在实际应用中，为每个 case 在单独的函数或方法中定义逻辑会更方便。此外，相关的 URL 可能需要类似的逻辑；例如，几个图像文件可能有 /images/*.png 形式的 URL。出于这些原因，net/http 提供了 ServeMux，一个请求多路复用器，来简化 URL 和处理器之间的关联。ServeMux 将一组 http.Handler 聚合成一个单一的 http.Handler。我们再次看到，满足相同接口的不同类型是可以替换的：Web 服务器可以将请求分发给任何 http.Handler，而不管它背后是哪种具体类型。

For a more complex application, several ServeMuxes may be composed to handle more intricate dispatching requirements. Go doesn't have a canonical web framework analogous to Ruby's Rails or Python's Django. This is not to say that such frameworks don't exist, but the building blocks in Go's standard library are flexible enough that frameworks are often unnecessary. Furthermore, although frameworks are convenient in the early phases of a project, their additional complexity can make longer-term maintenance harder.

对于更复杂的应用程序，可以组合多个 ServeMux 来处理更复杂的分发需求。Go 没有类似于 Ruby 的 Rails 或 Python 的 Django 的规范化 Web 框架。这并不是说这样的框架不存在，而是 Go 标准库中的构建块足够灵活，使得框架通常是不必要的。此外，虽然框架在项目的早期阶段很方便，但它们的额外复杂性可能会使长期维护变得更加困难。

## 7.8. The error Interface

## 7.8. error 接口

Since the beginning of this book, we've been using and creating values of the mysterious predeclared error type without explaining what it really is. In fact, it's just an interface type with a single method that returns an error message:

自本书开始以来，我们一直在使用和创建神秘的预声明 error 类型的值，却没有解释它到底是什么。事实上，它只是一个具有单个方法的接口类型，该方法返回一个错误消息：

```go
type error interface {
    Error() string
}
```

The simplest way to create an error is by calling errors.New, which returns a new error for a given error message. The entire errors package is only four lines long:

创建错误的最简单方法是调用 errors.New，它为给定的错误消息返回一个新的错误。整个 errors 包只有四行代码：

```go
package errors

func New(text string) error { return &errorString{text} }

type errorString struct { text string }

func (e *errorString) Error() string { return e.text }
```

The underlying type of errorString is a struct, not a string, to protect its representation from inadvertent (or premeditated) updates. And the reason that the pointer type *errorString, not errorString alone, satisfies the error interface is so that every call to New allocates a distinct error instance that is equal to no other. We would not want a distinguished error such as io.EOF to compare equal to one that merely happened to have the same message.

errorString 的底层类型是结构体而不是字符串，这是为了保护其表示形式免受无意（或蓄意）的更新。而指针类型 *errorString（而不是 errorString 本身）满足 error 接口的原因是，每次调用 New 都会分配一个独特的错误实例，它不等于任何其他实例。我们不希望像 io.EOF 这样的特定错误与仅仅碰巧具有相同消息的错误相等。

```go
fmt.Println(errors.New("EOF") == errors.New("EOF")) // "false"
```

Calls to errors.New are relatively infrequent because there's a convenient wrapper function, fmt.Errorf, that does string formatting too. We used it several times in Chapter 5.

对 errors.New 的调用相对较少，因为有一个方便的包装函数 fmt.Errorf，它还能进行字符串格式化。我们在第5章中多次使用了它。

```go
package fmt

import "errors"

func Errorf(format string, args ...interface{}) error {
    return errors.New(Sprintf(format, args...))
}
```

Although *errorString may be the simplest type of error, it is far from the only one. For example, the syscall package provides Go's low-level system call API. On many platforms, it defines a numeric type Errno that satisfies error, and on Unix platforms, Errno's Error method does a lookup in a table of strings, as shown below:

虽然 *errorString 可能是最简单的错误类型，但它远不是唯一的。例如，syscall 包提供了 Go 的低级系统调用 API。在许多平台上，它定义了一个满足 error 的数值类型 Errno，在 Unix 平台上，Errno 的 Error 方法在字符串表中进行查找，如下所示：

```go
package syscall

type Errno uintptr // operating system error code
```

```go
var errors = [...]string{
    1:   "operation not permitted",   // EPERM
    2:   "no such file or directory", // ENOENT
    3:   "no such process",           // ESRCH
    // ...
}

func (e Errno) Error() string {
    if 0 <= int(e) && int(e) < len(errors) {
        return errors[e]
    }
    return fmt.Sprintf("errno %d", e)
}
```

The following statement creates an interface value holding the Errno value 2, signifying the POSIX ENOENT condition:

下面的语句创建了一个包含 Errno 值 2 的接口值，表示 POSIX ENOENT 条件：

```go
var err error = syscall.Errno(2)
fmt.Println(err.Error()) // "no such file or directory"
fmt.Println(err)         // "no such file or directory"
```

The value of err is shown graphically in Figure 7.6.

err 的值在图 7.6 中以图形方式显示。

Figure 7.6. An interface value holding a syscall.Errno integer.

图 7.6. 持有 syscall.Errno 整数的接口值。

Errno is an efficient representation of system call errors drawn from a finite set, and it satisfies the standard error interface. We'll see other types that satisfy this interface in Section 7.11.

Errno 是从有限集合中提取的系统调用错误的高效表示，它满足标准的 error 接口。我们将在 7.11 节中看到满足此接口的其他类型。

当然，我已经仔细回顾了之前的翻译，并根据您提供的最新指南（特别是关于引用的严格要求）进行了修正。以下是第7.9节和第7.10节的更新翻译：

## 7.9. 示例：表达式求值器 (Example: Expression Evaluator)

In this section, we'll build an evaluator for simple arithmetic expressions. We'll use an interface, `Expr`, to represent any expression in this language[cite: 350, 351]. For now, this interface needs no methods, but we'll add some later.

本节中，我们将构建一个简单算术表达式的求值器。 我们将使用一个接口 `Expr` 来表示该语言中的任何表达式 [cite: 350, 351]。 目前，这个接口不需要任何方法，但我们稍后会添加一些。

```go
// An Expr is an arithmetic expression.
type Expr interface{}
```

Our expression language consists of floating-point literals; the binary operators `+`, `-`, `*`, and `/`; the unary operators `-x` and `+x`; function calls `pow(x,y)`, `sin(x)`, and `sqrt(x)`; variables such as `x` and `pi`; and of course parentheses and standard operator precedence. All values are of type `float64`. Here are some example expressions:

我们的表达式语言包含浮点数字面量；二元运算符 `+`、`-`、`*` 和 `/`；一元运算符 `-x` 和 `+x`；函数调用 `pow(x,y)`、`sin(x)` 和 `sqrt(x)`；变量（例如 `x` 和 `pi`）；当然还有括号和标准的运算符优先级。 所有值都是 `float64` 类型。以下是一些表达式示例：

```
sqrt(A/pi)
pow(x,3)+pow(y,3)
(F-32)*5/9
```

The five concrete types below represent particular kinds of expression. A `Var` represents a reference to a variable. (We'll soon see why it is exported.) A `literal` represents a floating-point constant. The `unary` and `binary` types represent operator expressions with one or two operands, which can be any kind of `Expr`. A `call` represents a function call; we'll restrict its `fn` field to `pow`, `sin`, or `sqrt`.

下面的五个**具体类型** (concrete types) 代表特定种类的表达式。 `Var` 代表对变量的引用。 （我们很快就会明白为什么它是导出的。）`literal` 代表一个浮点常量。 `unary` 和 `binary` 类型代表带有一个或两个操作数的操作符表达式，操作数可以是任何类型的 `Expr`。`call` 代表一个函数调用；我们将其 `fn` 字段限制为 `pow`、`sin` 或 `sqrt`。

```go
gopl.io/ch7/eval

// A Var identifies a variable, e.g., x.
type Var string

// A literal is a numeric constant, e.g., 3.141.
type literal float64

// A unary represents a unary operator expression, e.g., -x.
type unary struct {
    op rune // one of '+', '-'
    x Expr
}

// A binary represents a binary operator expression, e.g., x+y.
type binary struct {
    op    rune // one of '+', '-', '*', '/'
    x, y Expr
}

// A call represents a function call expression, e.g., sin(x).
type call struct {
    fn    string // one of "pow", "sin", "sqrt"
    args []Expr
}
```

To evaluate an expression containing variables, we'll need an `environment` that maps variable names to values:

为了对包含变量的表达式求值，我们需要一个将变量名映射到值的 `environment`（环境）：
*[译注：原文此处及下一代码块没有源标签。]*

```go
type Env map[Var]float64
```

We'll also need each kind of expression to define an `Eval` method that returns the expression's value in a given environment. Since every expression must provide this method, we add it to the `Expr` interface. The package exports only the types `Expr`, `Env`, and `Var`; clients can use the evaluator without access to the other expression types.

我们还需要每种表达式都定义一个 `Eval` 方法，该方法在给定环境中返回表达式的值。 由于每个表达式都必须提供此方法，我们将其添加到 `Expr` 接口中。 该包仅导出 `Expr`、`Env` 和 `Var` 类型；客户端可以在不访问其他表达式类型的情况下使用求值器。

```go
type Expr interface {
    // Eval returns the value of this Expr in the environment env.
Eval(env Env) float64
}
```

The concrete `Eval` methods are shown below. The method for `Var` performs an environment lookup, which returns zero if the variable is not defined, and the method for `literal` simply returns the literal value.

具体的 `Eval` 方法如下所示。`Var` 的方法执行环境查找，如果变量未定义，则返回零；`literal` 的方法仅返回字面量值。

```go
func (v Var) Eval(env Env) float64 {
    return env[v]
}

func (l literal) Eval(_ Env) float64 {
    return float64(l)
}
```

The `Eval` methods for `unary` and `binary` recursively evaluate their operands, then apply the operation `op` to them. We don't consider divisions by zero or infinity to be errors, since they produce a result, albeit non-finite. Finally, the method for `call` evaluates the arguments to the `pow`, `sin`, or `sqrt` function, then calls the corresponding function in the `math` package.

`unary` 和 `binary` 的 `Eval` 方法会递归地对其操作数求值，然后将操作 `op` 应用于它们。 我们不认为除以零或无穷大是错误，因为它们会产生结果，尽管是非有限的。 最后，`call` 的方法会对 `pow`、`sin` 或 `sqrt` 函数的参数求值，然后调用 `math` 包中相应的函数。

```go
func (u unary) Eval(env Env) float64 {
    switch u.op {
    case '+':
        return +u.x.Eval(env)
    case '-':
        return -u.x.Eval(env)
    }
    panic(fmt.Sprintf("unsupported unary operator: %q", u.op))
}

func (b binary) Eval(env Env) float64 {
    switch b.op {
    case '+':
        return b.x.Eval(env) + b.y.Eval(env)
    case '-':
        return b.x.Eval(env) - b.y.Eval(env)
    case '*':
        return b.x.Eval(env) * b.y.Eval(env)
    case '/':
        return b.x.Eval(env) / b.y.Eval(env)
    }
    panic(fmt.Sprintf("unsupported binary operator: %q", b.op))
}

func (c call) Eval(env Env) float64 {
    switch c.fn {
    case "pow":
        return math.Pow(c.args[0].Eval(env), c.args[1].Eval(env))
    case "sin":
        return math.Sin(c.args[0].Eval(env))
    case "sqrt":
        return math.Sqrt(c.args[0].Eval(env))
    }
    panic(fmt.Sprintf("unsupported function call: %s", c.fn))
}
```

Several of these methods can fail. For example, a `call` expression could have an unknown function or the wrong number of arguments. It's also possible to construct a `unary` or `binary` expression with an invalid operator such as `!` or `<` (although the `Parse` function mentioned below will never do this)[cite: 377, 378]. These errors cause `Eval` to panic. Other errors, like evaluating a `Var` not present in the environment, merely cause `Eval` to return the wrong result. All of these errors could be detected by inspecting the `Expr` before evaluating it. That will be the job of the `Check` method, which we will show soon, but first let's test `Eval`.

其中一些方法可能会失败。 例如，一个 `call` 表达式可能有一个未知的函数或错误数量的参数。 还有可能构造一个带有无效运算符（如 `!` 或 `<`）的 `unary` 或 `binary` 表达式（尽管下面提到的 `Parse` 函数永远不会这样做）[cite: 377, 378]。 这些错误会导致 `Eval` 发生 panic。 其他错误，比如对环境中不存在的 `Var` 求值，只会导致 `Eval` 返回错误的结果。 所有这些错误都可以在对 `Expr` 求值之前通过检查它来检测到。这将是 `Check` 方法的工作，我们很快就会展示它，但首先让我们测试一下 `Eval`。

The `TestEval` function below is a test of the evaluator. It uses the `testing` package, which we'll explain in Chapter 11, but for now it's enough to know that calling `t.Errorf` reports an error[cite: 384, 385]. The function loops over a table of inputs that defines three expressions and different environments for each one. The first expression computes the radius of a circle given its area `A`, the second computes the sum of the cubes of two variables `x` and `y`, and the third converts a Fahrenheit temperature `F` to Celsius.

下面的 `TestEval` 函数是求值器的一个测试。 它使用了 `testing` 包，我们将在第11章中解释它，但现在只需要知道调用 `t.Errorf` 会报告一个错误 [cite: 384, 385]。 该函数遍历一个输入表，该表定义了三个表达式以及每个表达式的不同环境。 第一个表达式根据给定面积 `A` 计算圆的半径，第二个表达式计算两个变量 `x` 和 `y` 的立方和，第三个表达式将华氏温度 `F` 转换为摄氏温度。

```go
func TestEval(t *testing.T) {
    tests := []struct {
        expr string
        env Env
        want string
    }{
        {"sqrt(A/pi)", Env{"A": 87616, "pi": math.Pi}, "167"},
        {"pow(x,3)+pow(y,3)", Env{"x": 12, "y": 1}, "1729"},
        {"pow(x,3)+pow(y,3)", Env{"x": 9, "y": 10}, "1729"},
        {"5/9*(F-32)", Env{"F": -40}, "-40"},
        {"5/9*(F-32)", Env{"F": 32}, "0"},
        {"5/9*(F-32)", Env{"F": 212}, "100"},
    }
    var prevExpr string
    for _, test := range tests {
        // Print expr only when it changes.
        if test.expr != prevExpr {
            fmt.Printf("\n%s\n", test.expr)
            prevExpr = test.expr
        }
        expr, err := Parse(test.expr)
        if err != nil {
            t.Error(err) // parse error
            continue
        }
        got := fmt.Sprintf("%.6g", expr.Eval(test.env))
        fmt.Printf("\t%v => %s\n", test.env, got)
        if got != test.want {
            t.Errorf("%s.Eval() in %s = %q, want %q\n",
                test.expr, test.env, got, test.want)
        }
    }
}
```

For each entry in the table, the test parses the expression, evaluates it in the environment, and prints the result. We don't have space to show the `Parse` function here, but you'll find it if you download the package using `go get`.

对于表中的每个条目，测试都会解析表达式，在环境中对其进行求值，并打印结果。 这里我们没有足够的空间来展示 `Parse` 函数，但是如果你使用 `go get` 下载该包，你会找到它。

The `go test` command (§11.1) runs a package's tests:

`go test` 命令（§11.1）运行一个包的测试：

```
$ go test -v gopl.io/ch7/eval
```

The `-v` flag lets us see the printed output of the test, which is normally suppressed for a successful test like this one. Here is the output of the test's `fmt.Printf` statements:

`-v` 标志让我们能够看到测试的打印输出，对于像这样成功的测试，通常会抑制输出。 以下是测试中 `fmt.Printf` 语句的输出：

```
sqrt(A/pi)
    map[A:87616 pi:3.141592653589793] => 167

pow(x,3)+pow(y,3)
    map[x:12 y:1] => 1729
    map[x:9 y:10] => 1729

5/9*(F-32)
    map[F:-40] => -40
    map[F:32] => 0
    map[F:212] => 100
```

Fortunately the inputs so far have all been well formed, but our luck is unlikely to last. Even in interpreted languages, it is common to check the syntax for static errors, that is, mistakes that can be detected without running the program. By separating the static checks from the dynamic ones, we can detect errors sooner and perform many checks only once instead of each time an expression is evaluated.

幸运的是，到目前为止，所有输入都是格式良好的，但我们的运气不太可能一直持续下去。 即使在解释型语言中，检查语法的静态错误也很常见，也就是说，这些错误可以在不运行程序的情况下被检测到。 通过将静态检查与动态检查分开，我们可以更早地检测到错误，并且许多检查只需要执行一次，而不是每次对表达式求值时都执行。

Let's add another method to the `Expr` interface. The `Check` method checks for static errors in an expression syntax tree. We'll explain its `vars` parameter in a moment.

让我们向 `Expr` 接口添加另一个方法。`Check` 方法检查表达式语法树中的静态错误。 我们稍后会解释它的 `vars` 参数。

```go
type Expr interface {
    Eval(env Env) float64
    // Check reports errors in this Expr and adds its Vars to the set.
Check(vars map[Var]bool) error
}
```

The concrete `Check` methods are shown below. Evaluation of `literal` and `Var` cannot fail, so the `Check` methods for these types return `nil`. The methods for `unary` and `binary` first check that the operator is valid, then recursively check the operands. Similarly, the method for `call` first checks that the function is known and has the right number of arguments, then recursively checks each argument.

具体的 `Check` 方法如下所示。 `literal` 和 `Var` 的求值不会失败，因此这些类型的 `Check` 方法返回 `nil`。 `unary` 和 `binary` 的方法首先检查运算符是否有效，然后递归地检查操作数。 类似地，`call` 的方法首先检查函数是否已知并且具有正确数量的参数，然后递归地检查每个参数。

```go
func (v Var) Check(vars map[Var]bool) error {
    vars[v] = true
    return nil
}

func (literal) Check(vars map[Var]bool) error {
    return nil
}

func (u unary) Check(vars map[Var]bool) error {
    if !strings.ContainsRune("+-", u.op) {
        return fmt.Errorf("unexpected unary op %q", u.op)
    }
    return u.x.Check(vars)
}

func (b binary) Check(vars map[Var]bool) error {
    if !strings.ContainsRune("+-*/", b.op) {
        return fmt.Errorf("unexpected binary op %q", b.op)
    }
    if err := b.x.Check(vars); err != nil {
        return err
    }
    return b.y.Check(vars)
}

func (c call) Check(vars map[Var]bool) error {
    arity, ok := numParams[c.fn]
    if !ok {
        return fmt.Errorf("unknown function %q", c.fn)
    }
    if len(c.args) != arity {
        return fmt.Errorf("call to %s has %d args, want %d",
            c.fn, len(c.args), arity)
    }
    for _, arg := range c.args {
        if err := arg.Check(vars); err != nil {
            return err
        }
    }
    return nil
}

var numParams = map[string]int{"pow": 2, "sin": 1, "sqrt": 1}
```

We've listed a selection of flawed inputs and the errors they elicit, in two groups. The `Parse` function (not shown) reports syntax errors and the `Check` function reports semantic errors.

我们列出了一些有缺陷的输入以及它们引发的错误，分为两组。 `Parse` 函数（未显示）报告语法错误，`Check` 函数报告语义错误。

The following table:
| 输入          | 错误信息 (Parse 或 Check)                 |
|---------------|-------------------------------------------|
| `x % 2`       | `unexpected '%'` (解析错误)             |
| `math.Pi`     | `unexpected '.'` (解析错误)             |
| `!true`       | `unexpected '!'` (解析错误)             |
| `"hello"`     | `unexpected` (解析错误，具体取决于解析器实现) |
| `log(10)`     | `unknown function "log"` (检查错误)       |
| `sqrt(1, 2)`  | `call to sqrt has 2 args, want 1` (检查错误) |
*[译注：表格内容根据原文整理。]*

`Check`'s argument, a set of `Var`s, accumulates the set of variable names found within the expression. Each of these variables must be present in the environment for evaluation to succeed. This set is logically the result of the call to `Check`, but because the method is recursive, it is more convenient for `Check` to populate a set passed as a parameter. The client must provide an empty set in the initial call.

`Check` 的参数是一个 `Var` 的集合，它累积表达式中找到的变量名集合。 为了求值成功，这些变量中的每一个都必须存在于环境中。 这个集合在逻辑上是调用 `Check` 的结果，但由于该方法是递归的，因此让 `Check` 填充作为参数传递的集合更为方便。 客户端必须在初始调用中提供一个空集合。

In Section 3.2, we plotted a function $f(x,y)$ that was fixed at compile time. Now that we can parse, check, and evaluate expressions in strings, we can build a web application that receives an expression at run time from the client and plots the surface of that function. We can use the `vars` set to check that the expression is a function of only two variables, `x` and `y`—three, actually, since we'll provide `r`, the radius, as a convenience. And we'll use the `Check` method to reject ill-formed expressions before evaluation begins so that we don't repeat those checks during the 40,000 evaluations ($100 \times 100$ cells, each with four corners) of the function that follow.

在第3.2节中，我们绘制了一个在编译时固定的函数 $f(x,y)$。 现在我们可以解析、检查和求值字符串中的表达式，我们可以构建一个Web应用程序，该应用程序在运行时从客户端接收表达式并绘制该函数的曲面。 我们可以使用 `vars` 集合来检查表达式是否仅是两个变量 `x` 和 `y` 的函数——实际上是三个，因为为了方便，我们将提供半径 `r`。 而且，我们将在求值开始之前使用 `Check` 方法来拒绝格式错误的表达式，这样我们就不会在随后的40,000次函数求值（$100 \times 100$ 个单元格，每个单元格有四个角）中重复这些检查。

The `parseAndCheck` function combines these parsing and checking steps:

`parseAndCheck` 函数结合了这些解析和检查步骤：

```go
gopl.io/ch7/surface

import "gopl.io/ch7/eval"

func parseAndCheck(s string) (eval.Expr, error) {
    if s == "" {
        return nil, fmt.Errorf("empty expression")
    }
    expr, err := eval.Parse(s)
    if err != nil {
        return nil, err
    }
    vars := make(map[eval.Var]bool)
    if err := expr.Check(vars); err != nil {
        return nil, err
    }
    for v := range vars {
        if v != "x" && v != "y" && v != "r" {
            return nil, fmt.Errorf("undefined variable: %s", v)
        }
    }
    return expr, nil
}
```

To make this a web application, all we need is the `plot` function below, which has the familiar signature of an `http.HandlerFunc`:

要将其构建成一个Web应用程序，我们只需要下面的 `plot` 函数，它具有我们熟悉的 `http.HandlerFunc` 的签名：

```go
func plot(w http.ResponseWriter, r *http.Request) {
    r.ParseForm()
    expr, err := parseAndCheck(r.Form.Get("expr"))
    if err != nil {
        http.Error(w, "bad expr: "+err.Error(), http.StatusBadRequest)
        return
    }
    w.Header().Set("Content-Type", "image/svg+xml")
    surface(w, func(x, y float64) float64 {
        r := math.Hypot(x, y) // distance from (0,0)
        return expr.Eval(eval.Env{"x": x, "y": y, "r": r})
    })
}
```

(Image of three 3D surface plots labeled (a), (b), and (c), with URLs in the browser bar showing the expressions used for plotting)

*[译注：图7.7显示了三个由程序生成的曲面图。]*
Figure 7.7. The surfaces of three functions: (a) $sin(-x) \cdot pow(1.5,-r)$; (b) $pow(2,sin(y)) \cdot pow(2,sin(x))/12$; (c) $sin(x \cdot y/10)/10$.

图7.7. 三个函数的曲面图：(a) $sin(-x) \cdot pow(1.5,-r)$；(b) $pow(2,sin(y)) \cdot pow(2,sin(x))/12$；(c) $sin(x \cdot y/10)/10$。

The `plot` function parses and checks the expression specified in the HTTP request and uses it to create an anonymous function of two variables. The anonymous function has the same signature as the fixed function `f` from the original surface-plotting program, but it evaluates the user-supplied expression. The environment defines `x`, `y`, and the radius `r`. Finally, `plot` calls `surface`, which is just the `main` function from `gopl.io/ch3/surface`, modified to take the function to plot and the output `io.Writer` as parameters, instead of using the fixed function `f` and `os.Stdout`. Figure 7.7 shows three surfaces produced by the program.

`plot` 函数解析并检查HTTP请求中指定的表达式，并用它创建一个包含两个变量的匿名函数。 该匿名函数与原始曲面绘制程序中的固定函数 `f` 具有相同的签名，但它会对用户提供的表达式进行求值。 环境定义了 `x`、`y` 和半径 `r`。 最后，`plot` 调用 `surface`，它只是 `gopl.io/ch3/surface` 中的 `main` 函数修改而成，使其接受要绘制的函数和输出 `io.Writer` 作为参数，而不是使用固定的函数 `f` 和 `os.Stdout`。 图7.7显示了程序生成的三个曲面。

Exercise 7.13: Add a `String` method to `Expr` to pretty-print the syntax tree. Check that the results, when parsed again, yield an equivalent tree.

练习7.13：向 `Expr` 添加一个 `String` 方法来优美地打印语法树。 检查结果再次解析时是否能产生等价的树。

Exercise 7.14: Define a new concrete type that satisfies the `Expr` interface and provides a new operation such as computing the minimum value of its operands. Since the `Parse` function does not create instances of this new type, to use it you will need to construct a syntax tree directly (or extend the parser).

练习7.14：定义一个新的**具体类型**，使其满足 `Expr` 接口并提供一个新的操作，例如计算其操作数的最小值。 由于 `Parse` 函数不会创建此新类型的实例，要使用它，你需要直接构造语法树（或扩展解析器）。

Exercise 7.15: Write a program that reads a single expression from the standard input, prompts the user to provide values for any variables, then evaluates the expression in the resulting environment. Handle all errors gracefully.

练习7.15：编写一个程序，从标准输入读取单个表达式，提示用户为任何变量提供值，然后在结果环境中对表达式求值。 优雅地处理所有错误。

Exercise 7.16: Write a web-based calculator program.

练习7.16：编写一个基于Web的计算器程序。

## 7.10. 类型断言 (Type Assertions)

A type assertion is an operation applied to an interface value. Syntactically, it looks like `x.(T)`, where `x` is an expression of an interface type and `T` is a type, called the "asserted" type. A type assertion checks that the dynamic type of its operand matches the asserted type.

**类型断言** (type assertion) 是应用于接口值的一种操作。在语法上，它看起来像 `x.(T)`，其中 `x` 是一个**接口类型** (interface type) 的表达式，`T` 是一个类型，称为“断言的”类型。 类型断言检查其操作数的**动态类型** (dynamic type) 是否与断言的类型匹配。

There are two possibilities. First, if the asserted type `T` is a concrete type, then the type assertion checks whether `x`'s dynamic type is identical to `T`. If this check succeeds, the result of the type assertion is `x`'s dynamic value, whose type is of course `T`. In other words, a type assertion to a concrete type extracts the concrete value from its operand. If the check fails, then the operation panics. For example:

有两种可能性。首先，如果断言的类型 `T` 是一个**具体类型** (concrete type)，那么类型断言会检查 `x` 的**动态类型**是否与 `T` 相同。 如果检查成功，类型断言的结果是 `x` 的**动态值** (dynamic value)，其类型当然是 `T`。换句话说，对**具体类型**的类型断言会从其操作数中提取具体值。 如果检查失败，则操作会引发 panic。例如：

```go
var w io.Writer
w = os.Stdout
f := w.(*os.File) // success: f == os.Stdout
c := w.(*bytes.Buffer) // panic: interface holds *os.File, not *bytes.Buffer
```

Second, if instead the asserted type `T` is an interface type, then the type assertion checks whether `x`'s dynamic type satisfies `T`. If this check succeeds, the dynamic value is not extracted; the result is still an interface value with the same type and value components, but the result has the interface type `T`. In other words, a type assertion to an interface type changes the type of the expression, making a different (and usually larger) set of methods accessible, but it preserves the dynamic type and value components inside the interface value.

其次，如果断言的类型 `T` 是一个**接口类型**，那么类型断言会检查 `x` 的**动态类型**是否满足 `T`。 如果此检查成功，则不会提取**动态值**；结果仍然是一个具有相同类型和值组件的接口值，但结果具有**接口类型** `T`。换句话说，对**接口类型**的类型断言会更改表达式的类型，从而使一组不同（通常更大）的方法可访问，但它会保留接口值内部的**动态类型**和值组件。

After the first type assertion below, both `w` and `rw` hold `os.Stdout` so each has a dynamic type of `*os.File`, but `w`, an `io.Writer`, exposes only the file's `Write` method, whereas `rw` exposes its `Read` method too[cite: 442, 443, 444].

在下面的第一个类型断言之后，`w` 和 `rw` 都持有 `os.Stdout`，因此每个都具有 `*os.File` 的**动态类型** [cite: 442, 443]，但是 `w`（一个 `io.Writer`）仅公开文件的 `Write` 方法，而 `rw` 也公开其 `Read` 方法。

```go
var w io.Writer
w = os.Stdout
rw := w.(io.ReadWriter) // success: *os.File has both Read and Write
w = new(ByteCounter)
rw = w.(io.ReadWriter) // panic: *ByteCounter has no Read method
```

No matter what type was asserted, if the operand is a `nil` interface value, the type assertion fails. A type assertion to a less restrictive interface type (one with fewer methods) is rarely needed, as it behaves just like an assignment, except in the `nil` case.

无论断言什么类型，如果操作数是 `nil` 接口值，类型断言都会失败。 对限制较少的**接口类型**（方法较少的**接口类型**）进行类型断言通常是不必要的，因为它的行为就像赋值一样，除非在 `nil` 的情况下。

```go
w = rw // io.ReadWriter is assignable to io.Writer
w = rw.(io.Writer) // fails only if rw == nil
```

Often we're not sure of the dynamic type of an interface value, and we'd like to test whether it is some particular type. If the type assertion appears in an assignment in which two results are expected, such as the following declarations, the operation does not panic on failure but instead returns an additional second result, a boolean indicating success:

通常我们不确定接口值的**动态类型**，并且我们想测试它是否是某个特定类型。 如果类型断言出现在期望有两个结果的赋值中，例如以下声明，则该操作在失败时不会引发 panic，而是返回一个额外的第二个结果，一个布尔值，指示成功：

```go
var w io.Writer = os.Stdout
f, ok := w.(*os.File) // success: ok, f == os.Stdout
b, ok := w.(*bytes.Buffer) // failure: !ok, b == nil
```

The second result is conventionally assigned to a variable named `ok`. If the operation failed, `ok` is `false`, and the first result is equal to the zero value of the asserted type, which in this example is a `nil *bytes.Buffer`[cite: 450, 451].

第二个结果通常赋给名为 `ok` 的变量。 如果操作失败，`ok` 为 `false`，并且第一个结果等于断言类型的零值，在此示例中为 `nil *bytes.Buffer` [cite: 450, 451]。

The `ok` result is often immediately used to decide what to do next. The extended form of the `if` statement makes this quite compact:

`ok` 结果通常立即用于决定下一步做什么。 `if` 语句的扩展形式使其非常紧凑：

```go
if f, ok := w.(*os.File); ok {
    // ...use f...
}
```

When the operand of a type assertion is a variable, rather than invent another name for the new local variable, you'll sometimes see the original name reused, shadowing the original, like this:

当类型断言的操作数是一个变量时，你有时会看到原始名称被重用，而不是为新的局部变量发明另一个名称，从而遮蔽了原始变量，如下所示：

```go
if w, ok := w.(*os.File); ok {
    // ...use w...
}
```

### 7.11. Discriminating Errors with Type Assertions
### 7.11. 使用类型断言区分错误

Consider the set of errors returned by file operations in the os package. I/O can fail for any number of reasons, but three kinds of failure often must be handled differently: file already exists (for create operations), file not found (for read operations), and permission denied.
考虑 `os` 包中文件操作返回的错误集合。I/O（输入/输出）可能由于多种原因失败，但有三种类型的失败通常必须区别对待：文件已存在（针对创建操作）、文件未找到（针对读取操作）以及权限被拒绝。

The os package provides these three helper functions to classify the failure indicated by a given error value:
`os` 包提供了以下三个辅助函数，用于对给定错误值所指示的故障进行分类：

```go
package os

func IsExist(err error) bool
func IsNotExist(err error) bool
func IsPermission(err error) bool
```

A naïve implementation of one of these predicates might check that the error message contains a certain substring,
这些谓词函数中，一个比较初级的实现可能会检查错误消息是否包含某个特定的子字符串，

```go
func IsNotExist(err error) bool {
    // NOTE: not robust!
    return strings.Contains(err.Error(), "file does not exist")
}
```

but because the logic for handling I/O errors can vary from one platform to another, this approach is not robust and the same failure may be reported with a variety of different error messages. Checking for substrings of error messages may be useful during testing to ensure that functions fail in the expected manner, but it's inadequate for production code.
但是由于处理 I/O 错误的逻辑可能因平台而异，这种方法并不健壮，并且相同的故障可能会以各种不同的错误消息报告。检查错误消息的子字符串在测试期间可能有助于确保函数按预期方式失败，但对于生产代码而言，这是不够的。

A more reliable approach is to represent structured error values using a dedicated type. The os package defines a type called PathError to describe failures involving an operation on a file path, like Open or Delete, and a variant called LinkError to describe failures of operations involving two file paths, like Symlink and Rename. Here's os. PathError:
一种更可靠的方法是使用专用类型来表示结构化的错误值。`os` 包定义了一个名为 `PathError` 的类型，用于描述涉及文件路径操作（如 `Open` 或 `Delete`）的失败，以及一个名为 `LinkError` 的变体，用于描述涉及两个文件路径的操作（如 `Symlink` 和 `Rename`）的失败。以下是 `os.PathError` 的定义：

```go
package os

// PathError records an error and the operation and file path that caused it.
type PathError struct {
    Op   string
    Path string
    Err  error
}

func (e *PathError) Error() string {
    return e.Op + " " + e.Path + ": " + e.Err.Error()
}
```

Most clients are oblivious to PathError and deal with all errors in a uniform way by calling their Error methods. Although PathError's Error method forms a message by simply concatenating the fields, PathError's structure preserves the underlying components of the error. Clients that need to distinguish one kind of failure from another can use a type assertion to detect the specific type of the error; the specific type provides more detail than a simple string.
大多数客户端并不知道 `PathError` 的存在，它们通过调用错误实例的 `Error` 方法以统一的方式处理所有错误。尽管 `PathError` 的 `Error` 方法通过简单地连接字段来形成消息，但 `PathError` 的结构保留了错误的底层组件。需要区分不同类型故障的客户端可以使用**类型断言 (type assertion)** 来检测错误的特定类型；特定类型比简单字符串提供了更多细节。

```go
_, err := os.Open("/no/such/file")
fmt.Println(err) // "open /no/such/file: No such file or directory"
fmt.Printf("%#v\n", err)
// Output:
// &os.PathError{Op:"open", Path:"/no/such/file", Err:0x2}
```

That's how the three helper functions work. For example, IsNotExist, shown below, reports whether an error is equal to syscall.ENOENT (§7.8) or to the distinguished error os.ErrNotExist (see io.EOF in §5.4.2), or is a *PathError whose underlying error is one of those two.
这三个辅助函数就是这样工作的。例如，下面显示的 `IsNotExist` 函数报告一个错误是否等于 `syscall.ENOENT`（§7.8）或特定的错误 `os.ErrNotExist`（参见 §5.4.2 中的 `io.EOF`），或者它是否是一个 `*PathError`，其底层错误是这两者之一。

```go
import (
    "errors"
    "syscall"
)

var ErrNotExist = errors.New("file does not exist")

// IsNotExist returns a boolean indicating whether the error is known to
// report that a file or directory does not exist. It is satisfied by
// ErrNotExist as well as some syscall errors.
func IsNotExist(err error) bool {
    if pe, ok := err.(*PathError); ok {
        err = pe.Err
    }
    return err == syscall.ENOENT || err == ErrNotExist
}
```

And here it is in action:
下面是它的实际应用：

```go
err := os.Open("/no/such/file")
fmt.Println(os.IsNotExist(err)) // "true"
```

Of course, PathError's structure is lost if the error message is combined into a larger string, for instance by a call to fmt.Errorf. Error discrimination must usually be done immediately after the failing operation, before an error is propagated to the caller.
当然，如果错误消息通过调用 `fmt.Errorf` 等方式组合成一个更大的字符串，`PathError` 的结构就会丢失。错误区分通常必须在失败操作之后立即进行，在错误传播给调用者之前。

### 7.12. Querying Behaviors with Interface Type Assertions
### 7.12. 使用接口类型断言查询行为

The logic below is similar to the part of the net/http web server responsible for writing HTTP header fields such as "Content-type: text/html". The io.Writer w represents the HTTP response; the bytes written to it are ultimately sent to someone's web browser.
下面的逻辑类似于 `net/http` Web 服务器中负责写入 HTTP 头部字段（如 "Content-type: text/html"）的部分。`io.Writer w` 代表 HTTP 响应；写入它的字节最终会发送到某个用户的 Web 浏览器。

```go
func writeHeader(w io.Writer, contentType string) error {
    if err := w.Write([]byte("Content-Type: ")); err != nil {
        return err
    }
    if err := w.Write([]byte(contentType)); err != nil {
        return err
    }
    // ...
}
```

Because the Write method requires a byte slice, and the value we wish to write is a string, a []byte(...) conversion is required. This conversion allocates memory and makes a copy, but the copy is thrown away almost immediately after.
由于 `Write` 方法需要一个字节切片，而我们希望写入的值是一个字符串，因此需要进行 `[]byte(...)` 转换。这种转换会分配内存并进行复制，但复制的内容几乎在之后立即被丢弃。

Let's pretend that this is a core part of the web server and that our profiling has revealed that this memory allocation is slowing it down. Can we avoid allocating memory here?
让我们假设这是 Web 服务器的核心部分，并且我们的性能分析显示此内存分配正在拖慢其速度。我们能否在此处避免内存分配？

The io.Writer interface tells us only one fact about the concrete type that w holds: that bytes may be written to it. If we look behind the curtains of the net/http package, we see that the dynamic type that w holds in this program also has a WriteString method that allows strings to be efficiently written to it, avoiding the need to allocate a temporary copy. (This may seem like a shot in the dark, but a number of important types that satisfy io.Writer also have a WriteString method, including *bytes.Buffer, *os.File and *bufio.Writer.)
`io.Writer` 接口只告诉我们关于 `w` 所持有的**具体类型 (concrete type)** 的一个事实：可以将字节写入其中。如果我们深入了解 `net/http` 包的内部，我们会发现 `w` 在此程序中持有的**动态类型 (dynamic type)** 也具有 `WriteString` 方法，该方法允许有效地向其写入字符串，从而避免了分配临时副本的需要。（这可能看起来像是在碰运气，但是许多满足 `io.Writer` 的重要类型也具有 `WriteString` 方法，包括 `*bytes.Buffer`、`*os.File` 和 `*bufio.Writer`。）

We cannot assume that an arbitrary io.Writer w also has the WriteString method. But we can define a new interface that has just this method and use a type assertion to test whether the dynamic type of w satisfies this new interface.
我们不能假设任意的 `io.Writer w` 也具有 `WriteString` 方法。但是我们可以定义一个只包含此方法的新接口，并使用**类型断言 (type assertion)** 来测试 `w` 的**动态类型 (dynamic type)** 是否满足这个新接口。

```go
// writeString writes s to w.
// If w has a WriteString method, it is invoked instead of w.Write.
func writeString(w io.Writer, s string) (n int, err error) {
    type stringWriter interface {
        WriteString(string) (n int, err error)
    }
    if sw, ok := w.(stringWriter); ok {
        return sw.WriteString(s) // avoid a copy
    }
    return w.Write([]byte(s)) // allocate temporary copy
}

func writeHeader(w io.Writer, contentType string) error {
    if _, err := writeString(w, "Content-Type: "); err != nil {
        return err
    }
    if _, err := writeString(w, contentType); err != nil {
        return err
    }
    // ...
}
```

To avoid repeating ourselves, we've moved the check into the utility function writeString, but it is so useful that the standard library provides it as io.WriteString. It is the recommended way to write a string to an io.Writer.
为了避免重复，我们已将检查移至辅助函数 `writeString` 中，但它非常有用，以至于标准库将其作为 `io.WriteString` 提供。这是向 `io.Writer` 写入字符串的推荐方法。

What's curious in this example is that there is no standard interface that defines the WriteString method and specifies its required behavior. Furthermore, whether or not a concrete type satisfies the stringWriter interface is determined only by its methods, not by any declared relationship between it and the interface type. What this means is that the technique above relies on the assumption that if a type satisfies the interface below, then WriteString(s) must have the same effect as Write([]byte(s)).
这个例子中奇怪的是，没有标准接口定义 `WriteString` 方法并指定其必需的行为。此外，一个**具体类型 (concrete type)** 是否满足 `stringWriter` 接口仅由其方法确定，而不是由它与**接口类型 (interface type)** 之间的任何声明关系确定。这意味着上述技术依赖于这样一个假设：如果一个类型满足下面的接口，那么 `WriteString(s)` 必须与 `Write([]byte(s))` 具有相同的效果。

```go
interface {
    io.Writer
    WriteString(s string) (n int, err error)
}
```

Although io.WriteString documents its assumption, few functions that call it are likely to document that they too make the same assumption. Defining a method of a particular type is taken as an implicit assent for a certain behavioral contract.
尽管 `io.WriteString` 文档化了它的假设，但很少有调用它的函数会文档化它们也做了同样的假设。为特定类型定义方法被视为对特定行为契约的默许。

Newcomers to Go, especially those from a background in strongly typed languages, may find this lack of explicit intention unsettling, but it is rarely a problem in practice. With the exception of the empty interface interface{}, interface types are seldom satisfied by unintended coincidence.
Go 的新手，尤其是那些有强类型语言背景的人，可能会觉得这种缺乏明确意图的做法令人不安，但这在实践中很少成为问题。除了空接口 `interface{}`，**接口类型 (interface types)** 很少会因为意外的巧合而被满足。

The writeString function above uses a type assertion to see whether a value of a general interface type also satisfies a more specific interface type, and if so, it uses the behaviors of the specific interface. This technique can be put to good use whether or not the queried interface is standard like io.ReadWriter or user-defined like stringWriter.
上面的 `writeString` 函数使用**类型断言 (type assertion)** 来查看通用**接口类型 (interface type)** 的值是否也满足更具体的**接口类型 (interface type)**，如果是，则使用特定接口的行为。无论被查询的接口是像 `io.ReadWriter` 这样的标准接口，还是像 `stringWriter` 这样的用户定义接口，这种技术都可以得到很好的应用。

It's also how fmt.Fprintf distinguishes values that satisfy error or fmt.Stringer from all other values. Within fmt.Fprintf, there is a step that converts a single operand to a string, something like this:
`fmt.Fprintf` 也是通过这种方式来区分满足 `error` 或 `fmt.Stringer` 接口的值与所有其他值的。在 `fmt.Fprintf` 内部，有一个步骤将单个操作数转换为字符串，大致如下：

```go
package fmt

func formatOneValue(x interface{}) string {
    if err, ok := x.(error); ok {
        return err.Error()
    }
    if str, ok := x.(Stringer); ok {
        return str.String()
    }
    // ...all other types...
}
```

If x satisfies either of the two interfaces, that determines the formatting of the value. If not, the default case handles all other types more or less uniformly using reflection; we'll find out how in Chapter 12.
如果 `x` 满足这两个接口中的任何一个，则该接口决定了值的格式化方式。如果不满足，则默认情况使用反射以或多或少统一的方式处理所有其他类型；我们将在第 12 章中了解其工作原理。

Again, this makes the assumption that any type with a String method satisfies the behavioral contract of fmt.Stringer, which is to return a string suitable for printing.
同样，这基于一个假设，即任何具有 `String` 方法的类型都满足 `fmt.Stringer` 的行为契约，即返回一个适合打印的字符串。

好的，我已经仔细回顾了之前的翻译，并根据您的所有指南进行了检查和微调。以下是修订后的完整翻译内容：

### 7.13. Type Switches

Interfaces are used in two distinct styles. In the first style, exemplified by io. Reader, io.Writer, fmt. Stringer, sort. Interface, http. Handler, and error, an interface's meth- ods express the similarities of the concrete types that satisfy the interface but hide the rep- resentation details and intrinsic operations of those concrete types. The emphasis is on the methods, not on the concrete types. 

### 7.13. 类型开关 (Type Switches)

接口（interfaces）有两种不同的使用风格。 第一种风格以 `io.Reader`、`io.Writer`、`fmt.Stringer`、`sort.Interface`、`http.Handler` 和 `error` 为例，接口的方法表达了满足该接口的具体类型 (concrete types) 之间的相似性，但隐藏了这些具体类型的表示细节和固有操作。 这种风格的重点在于方法，而不是具体类型。 

The second style exploits the ability of an interface value to hold values of a variety of concrete types and considers the interface to be the union of those types. Type assertions are used to discriminate among these types dynamically and treat each case differently. In this style, the emphasis is on the concrete types that satisfy the interface, not on the interface's methods (if indeed it has any), and there is no hiding of information. We'll describe interfaces used this way as discriminated unions. 

第二种风格利用接口值持有多种具体类型值的能力，并将接口视为这些类型的联合体 (union)。 **类型断言 (type assertion)** 用于动态地区分这些类型，并对每种情况进行不同处理。 在这种风格中，重点在于满足接口的具体类型，而不是接口的方法（如果它确实有任何方法的话），并且没有信息隐藏。 我们将以这种方式使用的接口称为**可辨识联合体 (discriminated unions)**。 

If you're familiar with object-oriented programming, you may recognize these two styles as subtype polymorphism and ad hoc polymorphism, but you needn't remember those terms. For the remainder of this chapter, we'll present examples of the second style. Go's API for querying an SQL database, like those of other languages, lets us cleanly separate the fixed part of a query from the variable parts. An example client might look like this: 

如果你熟悉面向对象编程，你可能会认出这两种风格分别是子类型多态 (subtype polymorphism) 和特设多态 (ad hoc polymorphism)，但你不需要记住这些术语。 在本章的其余部分，我们将展示第二种风格的示例。 Go语言用于查询SQL数据库的API，与其他语言类似，允许我们清晰地分离查询的固定部分和可变部分。 一个示例客户端可能如下所示： 

```go
import "database/sql"

func listTracks (db sql.DB, artist string, minYear, maxYear int) {
 result, err := db. Exec(
 "SELECT * FROM tracks WHERE artist = ? AND ? <= year AND year <= ?",
 artist, minYear, maxYear)
 // ...
}
```
[No specific text to translate here, it's part of the code block and its lead-in]

The Exec method replaces each'?' in the query string with an SQL literal denoting the cor- responding argument value, which may be a boolean, a number, a string, or nil. Construct- ing queries this way helps avoid SQL injection attacks, in which an adversary takes control of the query by exploiting improper quotation of input data. Within Exec, we might find a func- tion like the one below, which converts each argument value to its literal SQL notation. 

`Exec` 方法会将查询字符串中的每个 '?' 替换为一个SQL字面量，该字面量表示相应的参数值，这个值可以是布尔型、数字、字符串或nil。 以这种方式构造查询有助于避免SQL注入攻击，在这种攻击中，攻击者通过利用输入数据的不当引用来控制查询。 在 `Exec` 内部，我们可能会找到类似下面这样的函数，它将每个参数值转换为其字面SQL表示法。 

```go
func sqlQuote (x interface{}) string {
 if x == nil {
  return "NULL"
 } else if _, ok := x.(int); ok {
  return fmt.Sprintf("%d", x)
 } else if _, ok := x.(uint); ok {
  return fmt.Sprintf("%d", x)
 } else if b, ok := x.(bool); ok {
  if b {
   return "TRUE"
  }
  return "FALSE"
 } else if s, ok := x.(string); ok {
  return sqlQuoteString(s) // (not shown)
 } else {
  panic(fmt.Sprintf("unexpected type %T: %v", x, x))
 }
}
```
[No specific text to translate here, it's part of the code block]

A switch statement simplifies an if-else chain that performs a series of value equality tests.

`switch` 语句简化了执行一系列值相等性测试的 `if-else` 链。

An analogous type switch statement simplifies an if-else chain of type assertions.

类似的，**类型开关 (type switch)** 语句简化了类型断言的 `if-else` 链。

In its simplest form, a type switch looks like an ordinary switch statement in which the oper- and is x. (type)-that's literally the keyword type-and each case has one or more types. A type switch enables a multi-way branch based on the interface value's dynamic type. The nil case matches if $x==ni1$, and the default case matches if no other case does. A type switch for sqlQuote would have these cases: 

在其最简单的形式中，类型开关看起来像一个普通的 `switch` 语句，其操作数是 `x.(type)`——这里 `type` 是一个关键字——并且每个 `case` 有一个或多个类型。 类型开关根据接口值的**动态类型 (dynamic type)** 实现多路分支。 如果 `x == nil`，则 `nil` case匹配；如果没有其他case匹配，则 `default` case匹配。 用于 `sqlQuote` 的类型开关将包含以下这些case： 

```go
switch x.(type) {
case nil:
 // ...
case int, uint:
 // ...
case bool:
 // ...
case string:
 // ...
default:
 // ...
}
```
[No specific text to translate here, it's part of the code block]

As with an ordinary switch statement (§1.8), cases are considered in order and, when a match is found, the case's body is executed. Case order becomes significant when one or more case types are interfaces, since then there is a possibility of two cases matching. The position of the default case relative to the others is immaterial. No fallthrough is allowed. 

与普通 `switch` 语句（§1.8）一样，`case` 按顺序进行判断，当找到匹配项时，执行该 `case` 的主体。 当一个或多个 `case` 类型是接口时，`case` 的顺序就变得很重要，因为此时可能有两个 `case` 同时匹配。 `default` case相对于其他case的位置无关紧要。 不允许 `fallthrough` (贯穿)。 

Notice that in the original function, the logic for the bool and string cases needs access to the value extracted by the type assertion. Since this is typical, the type switch statement has an extended form that binds the extracted value to a new variable within each case: 

请注意，在原始函数中，`bool` 和 `string` case的逻辑需要访问通过类型断言提取的值。 由于这种情况很典型，类型开关语句提供了一种扩展形式，可以在每个 `case` 内部将提取的值绑定到一个新变量上： 

```go
switch x := x.(type) { /* ... */ }
```


Here we've called the new variables x too; as with type assertions, reuse of variable names is common. Like a switch statement, a type switch implicitly creates a lexical block, so the dec- laration of the new variable called x does not conflict with a variable x in an outer block. Each case also implicitly creates a separate lexical block. 

这里我们也把新变量命名为 `x`； 与类型断言一样，重用变量名是很常见的。 与 `switch` 语句类似，类型开关隐式地创建了一个词法块，因此名为 `x` 的新变量的声明不会与外部块中的变量 `x` 冲突。 每个 `case` 也隐式地创建了一个单独的词法块。 

Rewriting sqlQuote to use the extended form of type switch makes it significantly clearer:

将 `sqlQuote` 重写为使用类型开关的扩展形式，会使其更加清晰：

```go
func sqlQuote (x interface{}) string {
 switch x := x.(type) {
 case nil:
  return "NULL"
 case int, uint:
  return fmt.Sprintf("%d", x) // x has type interface{} here.
 case bool:
  if x { // x has type bool here
   return "TRUE"
  }
  return "FALSE"
 case string:
  return sqlQuoteString(x) // x has type string here (not shown)
 default:
  // x has type interface{} here
  panic(fmt.Sprintf("unexpected type %T: %v", x, x))
 }
}
```
[No specific text to translate here, it's part of the code block]

In this version, within the block of each single-type case, the variable x has the same type as the case. For instance, x has type bool within the bool case and string within the string case. In all other cases, x has the (interface) type of the switch operand, which is inter- face{} in this example. When the same action is required for multiple cases, like int and uint, the type switch makes it easy to combine them. 

在这个版本中，在每个单一类型 `case` 的块内，变量 `x` 的类型与该 `case` 的类型相同。 例如，在 `bool` case中 `x` 的类型是 `bool`，在 `string` case中 `x` 的类型是 `string`。 在所有其他情况下（包括 `nil` case，以及 `int, uint` 这种联合了多种具体类型的 case，还有 `default` case），`x` 具有与 `switch` 操作数相同的（接口）类型，在本例中即为 `interface{}`。 当多个 `case` 需要执行相同的操作时，比如 `int` 和 `uint`，类型开关可以轻松地将它们组合起来。 

Although sqlQuote accepts an argument of any type, the function runs to completion only if the argument's type matches one of the cases in the type switch; otherwise it panics with an "unexpected type" message. Although the type of x is interface{}, we consider it a discriminated union of int, uint, bool, string, and nil. 

尽管 `sqlQuote` 接受任何类型的参数，但只有当参数的类型匹配类型开关中的某个 `case` 时，函数才能成功执行完毕； 否则它会因“意外类型 (unexpected type)”消息而发生 `panic`。 虽然 `x` 的类型是 `interface{}`，但我们将其视为 `int`、`uint`、`bool`、`string` 和 `nil` 的可辨识联合体。 

### 7.14. Example: Token-Based XML Decoding

Section 4.5 showed how to decode JSON documents into Go data structures with the Marshal and Unmarshal functions from the encoding/json package. The encoding/xml package provides a similar API. This approach is convenient when we want to construct a represen- tation of the document tree, but that's unnecessary for many programs. The encoding/xml package also provides a lower-level token-based API for decoding XML. In the token-based style, the parser consumes the input and produces a stream of tokens, primarily of four kinds-StartElement, EndElement, CharData, and Comment each being a concrete type in the encoding/xml package. Each call to (*xml.Decoder). Token returns a token. 

### 7.14. 示例：基于标记（Token-Based）的XML解码

4.5节展示了如何使用 `encoding/json` 包中的 `Marshal` 和 `Unmarshal` 函数将JSON文档解码到Go数据结构中。 `encoding/xml` 包提供了类似的API。 当我们想要构建文档树的表示时，这种方法很方便，但对于许多程序来说这是不必要的。 `encoding/xml` 包还提供了一个更底层的**基于标记 (token-based)** 的API用于解码XML。 在基于标记的风格中，解析器处理输入并产生一个标记流，主要有四种类型——`StartElement`、`EndElement`、`CharData` 和 `Comment`——每种都是 `encoding/xml` 包中的具体类型。 每次调用 `(*xml.Decoder).Token` 都会返回一个标记。 

The relevant parts of the API are shown here:

API的相关部分如下所示：

```go
encoding/xml
package xml

type Name struct {
 Local string // e.g., "Title" or "id"
}

type Attr struct { // e.g., name="value"
 Name Name
 Value string
}

// A Token includes StartElement, EndElement, CharData,
// and Comment, plus a few esoteric types (not shown).
type Token interface{} 

type StartElement struct { // e.g., <name>
 Name Name
 Attr []Attr
}
type EndElement struct { Name Name } // e.g., </name>
type CharData []byte // e.g., <p>CharData</p>
type Comment []byte // e.g., type Decoder struct{ /* ... */ }
```
[No specific text to translate here, it's part of the code block]

```go
func NewDecoder(io.Reader) *Decoder 
func (*Decoder) Token() (Token, error) // returns next Token in sequence 
```
[No specific text to translate here, it's part of the code block]

The Token interface, which has no methods, is also an example of a discriminated union. The purpose of a traditional interface like io. Reader is to hide details of the concrete types that satisfy it so that new implementations can be created; each concrete type is treated uniformly. By contrast, the set of concrete types that satisfy a discriminated union is fixed by the design and exposed, not hidden. Discriminated union types have few methods; functions that oper- ate on them are expressed as a set of cases using a type switch, with different logic in each case. 

`Token` 接口没有任何方法，它也是可辨识联合体的一个例子。 像 `io.Reader` 这样的传统接口的目的是隐藏满足它的具体类型的细节，以便可以创建新的实现； 每个具体类型都被统一对待。 相反，满足可辨识联合体的具体类型集合是由设计固定的并且是暴露的，而不是隐藏的。 可辨识联合体类型的方法很少；操作它们的功能通常使用类型开关表示为一组 `case`，每个 `case` 中有不同的逻辑。 

The xmlselect program below extracts and prints the text found beneath certain elements in an XML document tree. Using the API above, it can do its job in a single pass over the input without ever materializing the tree. 

下面的 `xmlselect` 程序提取并打印XML文档树中特定元素下的文本。 使用上述API，它可以在单遍扫描输入的情况下完成工作，而无需实际构建整个树。 

```go
gopl.io/ch7/xmlselect
// Xmlselect prints the text of selected elements of an XML document. 
package main

import (
 "encoding/xml"
 "fmt"
 "io"
 "os"
 "strings"
)

func main() {
 dec := xml.NewDecoder(os.Stdin)
 var stack []string // stack of element names
 for {
  tok, err := dec.Token() 
  if err == io.EOF {
   break
  } else if err != nil {
   fmt.Fprintf(os.Stderr, "xmlselect: %v\n", err)
   os.Exit(1)
  }
  switch tok := tok.(type) { 
  case xml.StartElement:
   stack = append(stack, tok.Name.Local) // push 
  case xml.EndElement:
   stack = stack[:len(stack)-1] // pop 
  case xml.CharData:
   if containsAll(stack, os.Args[1:]) { 
    fmt.Printf("%s: %s\n", strings.Join(stack, " "), tok) 
   }
  }
 }
}
```
[No specific text to translate here, it's part of the code block]

```go
// containsAll reports whether x contains the elements of y, in order. 
func containsAll(x, y []string) bool {
 for len(y) <= len(x) {
  if len(y) == 0 {
   return true
  }
  if x[0] == y[0] {
   y = y[1:]
  }
  x = x[1:]
 }
 return false
}
```
[No specific text to translate here, it's part of the code block]

Each time the loop in main encounters a StartElement, it pushes the element's name onto a stack, and for each EndElement it pops the name from the stack. The API guarantees that the sequence of StartElement and EndElement tokens will be properly matched, even in ill- formed documents. Comments are ignored. When xmlselect encounters a CharData, it prints the text only if the stack contains all the elements named by the command-line argu- ments, in order. 

`main` 函数中的循环每次遇到 `StartElement` 时，都会将其元素名称压入一个栈中；每次遇到 `EndElement` 时，都会从栈中弹出名称。 API保证即使在格式错误的文档中，`StartElement` 和 `EndElement` 标记的序列也将正确匹配。 注释会被忽略。 当 `xmlselect` 遇到 `CharData` 时，仅当栈中按顺序包含命令行参数指定的所有元素名称时，它才会打印文本。 

The command below prints the text of any h2 elements appearing beneath two levels of div elements. Its input is the XML specification, itself an XML document. 

下面的命令打印出现在两层 `div` 元素下的任何 `h2` 元素的文本。 其输入是XML规范，它本身也是一个XML文档。 

```bash
$ go build gopl.io/ch1/fetch
$ ./fetch http://www.w3.org/TR/2006/REC-xml11-20060816 | \
 ./xmlselect div div h2 
html body div div h2: 1 Introduction 
html body div div h2: 2 Documents 
html body div div h2: 3 Logical Structures 
html body div div h2: 4 Physical Structures 
html body div div h2: 5 Conformance 
html body div div h2: 6 Notation 
html body div div h2: A References 
html body div div h2: B Definitions for Character Normalization 
```
[No specific text to translate here, it's part of the command-line example and output]

Exercise 7.17: Extend xmlselect so that elements may be selected not just by name, but by their attributes too, in the manner of CSS, so that, for instance, an element like `<div id="page" class="wide">` could be selected by a matching id or class as well as its name. 

练习 7.17：扩展 `xmlselect`，使其不仅可以通过名称选择元素，还可以通过元素的属性进行选择，类似CSS的方式。例如，像 `<div id="page" class="wide">` 这样的元素可以通过匹配的 `id` 或 `class` 以及其名称来选择。 

Exercise 7.18: Using the token-based decoder API, write a program that will read an arbitrary XML document and construct a tree of generic nodes that represents it. Nodes are of two kinds: CharData nodes represent text strings, and Element nodes represent named elements and their attributes. Each element node has a slice of child nodes. 

练习 7.18：使用基于标记的解码器API，编写一个程序，该程序将读取任意XML文档并构造一个表示该文档的通用节点树。 节点有两种类型：`CharData` 节点表示文本字符串，`Element` 节点表示命名元素及其属性。 每个元素节点都有一个子节点切片。 

You may find the following declarations helpful.

以下声明可能会对你有所帮助。

```go
import "encoding/xml"
```
[No specific text to translate here, it's part of the code block]
```go
type Node interface{} // CharData or *Element 

type CharData string 

type Element struct {
 Type xml.Name
 Attr []xml.Attr
 Children []Node
} 
```
[No specific text to translate here, it's part of the code block]

### 7.15. A Few Words of Advice

When designing a new package, novice Go programmers often start by creating a set of inter- faces and only later define the concrete types that satisfy them. This approach results in many interfaces, each of which has only a single implementation. Don't do that. Such interfaces are unnecessary abstractions; they also have a run-time cost. You can restrict which methods of a type or fields of a struct are visible outside a package using the export mechanism (§6.6). Interfaces are only needed when there are two or more concrete types that must be dealt with in a uniform way. 

### 7.15. 一些建议

在设计新包时，Go语言新手程序员通常首先创建一组接口，然后再定义满足这些接口的具体类型。 这种方法会导致许多接口，而每个接口只有一个实现。 不要这样做。 此类接口是不必要的抽象；它们还会带来运行时成本。 你可以使用**导出机制 (export mechanism)**（§6.6）来限制类型的方法或结构体的字段在包外部的可见性。 只有当存在两个或更多具体类型需要以统一方式处理时，才需要接口。 

We make an exception to this rule when an interface is satisfied by a single concrete type but that type cannot live in the same package as the interface because of its dependencies. In that case, an interface is a good way to decouple two packages. 

当接口只有一个具体类型满足，但由于依赖关系，该类型不能与接口存在于同一个包中时，我们可以对此规则作一例外处理。 在这种情况下，接口是解耦两个包的好方法。 

Because interfaces are used in Go only when they are satisfied by two or more types, they necessarily abstract away from the details of any particular implementation. The result is smaller interfaces with fewer, simpler methods, often just one as with io.Writer or fmt.Stringer. Small interfaces are easier to satisfy when new types come along. A good rule of thumb for interface design is ask only for what you need. 

由于在Go中，接口仅在被两个或多个类型满足时才使用，因此它们必然会从任何特定实现的细节中抽象出来。 结果是接口更小，方法更少、更简单，通常只有一个方法，例如 `io.Writer` 或 `fmt.Stringer`。 当出现新类型时，小接口更容易满足。 接口设计的一个好经验法则是：只要求你所需要的。 

This concludes our tour of methods and interfaces. Go has great support for the object- oriented style of programming, but this does not mean you need to use it exclusively. Not everything need be an object; standalone functions have their place, as do unencapsulated data types. Observe that together, the examples in the first five chapters of this book call no more than two dozen methods, like input. Scan, as opposed to ordinary function calls like fmt.Printf. 

我们关于方法和接口的介绍到此结束。Go对面向对象风格的编程提供了很好的支持，但这并不意味着你需要完全依赖它。 并非所有东西都需要成为对象；独立函数有其用武之地，未封装的数据类型也是如此。 请注意，本书前五章中的示例总共调用的方法（如 `input.Scan`）不超过二十几个，而普通的函数调用（如 `fmt.Printf`）则更多。 