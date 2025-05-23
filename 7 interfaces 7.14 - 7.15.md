**7.14. Example: Token-Based XML Decoding**

Section 4.5 showed how to decode JSON documents into Go data structures with the Marshal and Unmarshal functions from the encoding/json package. The encoding/xml package provides a similar API. This approach is convenient when we want to construct a representation of the document tree, but that's unnecessary for many programs. The encoding/xml package also provides a lower-level token-based API for decoding XML. In the token-based style, the parser consumes the input and produces a stream of tokens, primarily of four kinds—StartElement, EndElement, CharData, and Comment—each being a concrete type in the encoding/xml package. Each call to (*xml.Decoder).Token returns a token.

**7.14. 示例：基于Token的XML解码 (Example: Token-Based XML Decoding)**

第4.5节展示了如何使用`encoding/json`包中的`Marshal`和`Unmarshal`函数将JSON文档解码为Go数据结构 。`encoding/xml`包提供了类似的API 。当我们想要构建文档树的表示时，这种方法很方便，但对于许多程序来说这是不必要的 。`encoding/xml`包还提供了一个更低级的基于**Token**（标记）的API用于解码XML 。在基于Token的风格中，解析器消费输入并产生一个Token流，主要有四种类型——`StartElement`、`EndElement`、`CharData`和`Comment`——每一种都是`encoding/xml`包中的具体类型 。每次调用`(*xml.Decoder).Token`都会返回一个Token 。

The relevant parts of the API are shown here:

encoding/xml

Go

````
package xml

type Name struct {
    Local string // e.g., "Title" or "id"
    // Space string // 命名空间 URI，原文未列出但通常存在
}

type Attr struct { // e.g., name="value"
    Name  Name
    Value string
}

// Token 是一个接口，涵盖了 StartElement, EndElement, CharData,
// Comment, ProcInst, 和 Directive 等类型。
// 原文只列举了部分，这里遵循原文。
type Token interface{} 

type StartElement struct { // e.g., <name>
    Name Name
    Attr []Attr
}
type EndElement struct{ Name Name } // e.g., </name>
type CharData []byte              // e.g., <p>CharData</p>
type Comment []byte               // e.g., ```

API的相关部分如下所示：

encoding/xml
```go
package xml

type Name struct {
    Local string // 例如 "Title" 或 "id"
    // Space string // 命名空间 URI，原文未列出但通常存在
}

type Attr struct { // 例如 name="value"
    Name  Name
    Value string
}

// Token 是一个接口，涵盖了 StartElement, EndElement, CharData,
// Comment, ProcInst, 和 Directive 等类型。
// 原文只列举了部分，这里遵循原文。
type Token interface{} 

type StartElement struct { // 例如 <name>
    Name Name
    Attr []Attr
}
type EndElement struct{ Name Name } // 例如 </name>
type CharData []byte              // 例如 <p>CharData</p>
type Comment []byte               // 例如 ```

```go
func NewDecoder(r io.Reader) *Decoder      // io.Reader is in package io
func (dec *Decoder) Token() (Token, error) // returns next Token in sequence
````

Go

```
func NewDecoder(r io.Reader) *Decoder      // io.Reader 在 io 包中
func (dec *Decoder) Token() (Token, error) // 返回序列中的下一个 Token
```

The Token interface, which has no methods, is also an example of a discriminated union. The purpose of a traditional interface like io.Reader is to hide details of the concrete types that satisfy it so that new implementations can be created; each concrete type is treated uniformly. By contrast, the set of concrete types that satisfy a discriminated union is fixed by the design and exposed, not hidden. Discriminated union types have few methods; functions that operate on them are expressed as a set of cases using a type switch, with different logic in each case.

`Token`接口没有任何方法，它也是可辨识联合的一个例子 。像`io.Reader`这样的传统接口的目的是隐藏满足它的具体类型的细节，以便可以创建新的实现；每个具体类型都被统一对待 。相比之下，满足可辨识联合的具体类型集合是由设计固定的并且是暴露的，而不是隐藏的 。可辨识联合类型的方法很少；操作它们的功能通常表示为使用类型选择 (type switch) 的一组`case`，每个`case`中具有不同的逻辑 。

The xmlselect program below extracts and prints the text found beneath certain elements in an XML document tree. Using the API above, it can do its job in a single pass over the input without ever materializing the tree.

下面的`xmlselect`程序提取并打印XML文档树中特定元素下的文本内容 。使用上述API，它可以在对输入进行单次遍历的情况下完成其工作，而无需将整个树物化（materializing）到内存中 。

_gopl.io/ch7/xmlselect_

Go

```
// xmlselect prints the text of selected elements of an XML document.
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
                fmt.Printf("%s: %s\n", strings.Join(stack, " "), tok) // tok is []byte
            }
        }
    }
}
```

_gopl.io/ch7/xmlselect_

Go

```
// xmlselect prints the text of selected elements of an XML document.
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
                fmt.Printf("%s: %s\n", strings.Join(stack, " "), tok) // tok 是 []byte
            }
        }
    }
}
```

Go

```
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

Go

```
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

Each time the loop in main encounters a StartElement, it pushes the element's name onto a stack, and for each EndElement it pops the name from the stack. The API guarantees that the sequence of StartElement and EndElement tokens will be properly matched, even in ill-formed documents. Comments are ignored. When xmlselect encounters a CharData, it prints the text only if the stack contains all the elements named by the command-line arguments, in order.

每次主循环中的`for`循环遇到一个`StartElement`时，它会将元素的名称压入栈中 ；对于每个`EndElement`，它会从栈中弹出名称 。API保证`StartElement`和`EndElement` Token的序列将正确匹配，即使在格式错误的（ill-formed）文档中也是如此 。注释会被忽略 。当`xmlselect`遇到`CharData`时，它仅在栈按顺序包含命令行参数指定的所有元素时才打印文本 。

The command below prints the text of any h2 elements appearing beneath two levels of div elements. Its input is the XML specification, itself an XML document.

Bash

```
$ go build gopl.io/ch1/fetch # assume fetch program already compiled
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

以下命令打印出现在两级`div`元素下的任何`h2`元素的文本 。其输入是XML规范，它本身就是一个XML文档 。

Bash

```
$ go build gopl.io/ch1/fetch # assume fetch program already compiled
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

Exercise 7.17: Extend xmlselect so that elements may be selected not just by name, but by their attributes too, in the manner of CSS, so that, for instance, an element like `<div id="page" class="wide">` could be selected by a matching id or class as well a1s its name.

练习 7.17：扩展xmlselect，使其不仅可以通过名称选择元素，还可以通过其属性进行选择，方式类似于CSS选择器。例如，像&lt;div id="page" class="wide">这样的元素可以通过匹配的id或class属性及其名称来选择 。

Exercise 7.18: Using the token-based decoder API, write a program that will read an arbitrary XML document and construct a tree of generic nodes that represents it. Nodes are of two kinds: CharData nodes represent text strings, and Element nodes represent named elements and their attributes. Each element node has a slice of child nodes.

You may find the following declarations helpful.

Go

```
import "encoding/xml" 

type Node interface{} // CharData or *Element 

type CharData string 

type Element struct { 
    Type     xml.Name   
    Attr     []xml.Attr 
    Children []Node     
}
```

练习 7.18：使用基于Token的解码器API，编写一个程序，该程序将读取任意XML文档并构造一个表示该文档的通用节点树 。节点有两种类型：CharData节点表示文本字符串，Element节点表示命名元素及其属性 。每个元素节点都有一个子节点切片 。

你可能会发现以下声明很有帮助。

Go

```
import "encoding/xml" 

type Node interface{} // CharData or *Element 

type CharData string 

type Element struct { 
    Type     xml.Name   
    Attr     []xml.Attr 
    Children []Node     
}
```

---

核心要点总结 (7.14):

本节介绍了 encoding/xml 包中一种更底层的、基于Token的XML解码方式。与 Unmarshal 直接将整个XML文档映射到Go数据结构不同，基于Token的API允许开发者逐个处理XML的构成单元（如起始标签、结束标签、字符数据、注释等）。

- **Token流**：解码器 (`xml.Decoder`) 逐次调用其 `Token()` 方法，从输入流中解析并返回下一个XML Token 。Token本身是一个空接口 (`interface{}`)，实际返回的类型是如 `xml.StartElement`, `xml.EndElement`, `xml.CharData` 等具体类型 。
    
- **可辨识联合 (Discriminated Union)**：`xml.Token` 接口是可辨识联合的一个例子 。与传统接口隐藏具体实现不同，可辨识联合的类型集合是固定的、已知的，通常通过类型选择 (type switch) 来处理不同的Token类型 。
    
- **内存效率**：基于Token的解码避免了将整个XML文档加载到内存中构建完整的DOM树，这对于处理大型XML文件或只需要提取部分信息的场景非常高效 。`xmlselect` 示例程序就展示了这一点，它通过维护一个元素路径栈，在单次遍历中提取所需数据 。
    
- **`xmlselect` 示例**：该程序通过栈来追踪当前的XML元素路径 ，当遇到 `CharData` 时，检查当前路径是否与用户指定的路径选择器匹配，如果匹配则打印文本内容 。`containsAll` 函数用于辅助判断路径匹配 。
    
- **练习题**：引导读者扩展 `xmlselect` 的功能（支持属性选择）以及使用Token API构建通用的XML节点树，进一步加深对Token API的理解和应用 。
    

Go语言的 `encoding/xml` 包同时提供了高阶（`Unmarshal`）和低阶（Token API）两种解码方式，体现了其灵活性，允许开发者根据具体需求选择合适的工具。低阶API虽然使用起来可能更繁琐，但提供了对解码过程更精细的控制和更高的性能潜力。

---

**7.15. A Few Words of Advice**

When designing a new package, novice Go programmers often start by creating a set of interfaces and only later define the concrete types that satisfy them. This approach results in many interfaces, each of which has only a single implementation. Don't do that. Such interfaces are unnecessary abstractions; they also have a run-time cost. You can restrict which methods of a type or fields of a struct are visible outside a package using the export mechanism (§6.6). Interfaces are only needed when there are two or more concrete types that must be dealt with in a uniform way.

**7.15. 一点建议 (A Few Words of Advice)**

在设计新包时，Go的新手程序员通常首先创建一组接口，然后再定义满足这些接口的具体类型 。这种方法会导致许多接口，而每个接口只有一个实现 。**不要这样做** 。这样的接口是不必要的抽象；它们还会带来运行时成本 。你可以使用导出机制（§6.6）来限制包外可见的类型方法或结构体字段 。**只有当存在两个或多个具体类型必须以统一方式处理时，才需要接口** 。

We make an exception to this rule when an interface is satisfied by a single concrete type but that type cannot live in the same package as the interface because of its dependencies. In that case, an interface is a good way to decouple two packages.

我们对这条规则有一个例外：当一个接口仅由单个具体类型满足，但由于依赖关系，该类型不能与接口位于同一个包中时 。在这种情况下，接口是解耦两个包的好方法 。

Because interfaces are used in Go only when they are satisfied by two or more types, they necessarily abstract away from the details of any particular implementation. The result is smaller interfaces with fewer, simpler methods, often just one as with io.Writer or fmt.Stringer. Small interfaces are easier to satisfy when new types come along. A good rule of thumb for interface design is ask only for what you need.

因为在Go中，接口仅在被两个或多个类型满足时使用，所以它们必然会从任何特定实现的细节中抽象出来 。结果是更小的接口，具有更少、更简单的方法，通常只有一个方法，如`io.Writer`或`fmt.Stringer` 。当新的类型出现时，小接口更容易被满足 。接口设计的一个好经验法则是：**只要求你需要的东西** (ask only for what you need) 。

This concludes our tour of methods and interfaces. Go has great support for the object-oriented style of programming, but this does not mean you need to use it exclusively. Not everything need be an object; standalone functions have their place, as do unencapsulated data types. Observe that together, the examples in the first five chapters of this book call no more than two dozen methods, like input.Scan, as opposed to ordinary function calls like fmt.Printf.

我们对方法和接口的探讨到此结束 。Go对面向对象编程风格提供了很好的支持，但这并不意味着你需要完全依赖它 。并非所有东西都需要是一个对象；独立的函数有其用武之地，未封装的数据类型也是如此 。值得注意的是，本书前五章中的示例总共调用的方法不超过二十几个（例如`input.Scan`），与普通函数调用（例如`fmt.Printf`）相比数量很少 。

---

核心要点总结 (7.15):

本节给出了关于在Go语言中设计和使用接口的重要建议，强调了Go接口设计的哲学——实用主义和简约主义。

- **避免不必要的接口**：不要一开始就为每个具体类型设计接口，尤其是当接口只有一个实现时 。这样的接口是过度抽象，并且会带来不必要的运行时开销 。
    
- **接口的真正用途**：接口的核心价值在于处理多个具体类型需要被统一对待的场景 。 只有当确实存在这种多态需求时，才应该引入接口。
    
- **导出机制控制可见性**：如果只是想限制类型或结构体字段的包外可见性，应使用Go的导出机制（首字母大写/小写），而不是接口 。
    
- **解耦包的例外**：一个接口只有一个实现是可以接受的特殊情况是，当接口用于解耦两个包，且由于依赖关系，具体类型不能与接口定义在同一个包中时 。
    
- **小接口原则 (Ask only for what you need)**：因为接口通常被多个类型实现，它们自然会抽象掉具体实现的细节，从而倾向于定义更少、更简单的方法 。 小接口（如 `io.Writer`只有一个 `Write` 方法）更容易被新类型满足，也更易于组合和理解 。 "只要求你需要的东西"是接口设计的黄金法则 。
    
- **Go的面向对象并非唯一范式**：虽然Go通过方法和接口很好地支持了面向对象的编程风格，但这并不意味着所有代码都必须遵循这种风格 。独立的函数和未封装的数据类型在Go中同样有其重要地位 。 Go鼓励混合范式编程，选择最适合问题的方式。
    

总而言之，Go语言的接口设计鼓励开发者在真正需要抽象和多态时才使用接口，并倾向于定义小而专注的接口。这种设计哲学有助于保持代码的清晰、高效和可维护性。