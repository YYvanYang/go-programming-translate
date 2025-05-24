# Preface

# 前言

"Go is an open source programming language that makes it easy to build simple, reliable, and efficient software." (From the Go web site at golang.org)

"Go是一种开源编程语言，它让构建简单、可靠、高效的软件变得容易。"（摘自Go官网golang.org）

Go was conceived in September 2007 by Robert Griesemer, Rob Pike, and Ken Thompson, all at Google, and was announced in November 2009. The goals of the language and its accompanying tools were to be expressive, efficient in both compilation and execution, and effective in writing reliable and robust programs.

Go语言由Robert Griesemer、Rob Pike和Ken Thompson于2007年9月在Google构思，并于2009年11月对外发布。该语言及其配套工具的目标是：富有表现力、编译和执行都高效，以及能有效地编写可靠和健壮的程序。

Go bears a surface similarity to C and, like C, is a tool for professional programmers, achieving maximum effect with minimum means. But it is much more than an updated version of C. It borrows and adapts good ideas from many other languages, while avoiding features that have led to complexity and unreliable code. Its facilities for concurrency are new and efficient, and its approach to data abstraction and object-oriented programming is unusually flexible. It has automatic memory management or garbage collection.

Go在表面上与C相似，并且像C一样，是专业程序员的工具，以最少的手段实现最大的效果。但它远不止是C的更新版本。它借鉴并改进了许多其他语言的优秀理念，同时避免了那些导致复杂性和不可靠代码的特性。它的并发机制新颖而高效，其数据抽象和面向对象编程的方法异常灵活。它具有自动内存管理或垃圾回收功能。

Go is especially well suited for building infrastructure like networked servers, and tools and systems for programmers, but it is truly a general-purpose language and finds use in domains as diverse as graphics, mobile applications, and machine learning. It has become popular as a replacement for untyped scripting languages because it balances expressiveness with safety: Go programs typically run faster than programs written in dynamic languages and suffer far fewer crashes due to unexpected type errors.

Go特别适合构建网络服务器等基础设施，以及面向程序员的工具和系统，但它确实是一种通用语言，在图形处理、移动应用和机器学习等各种领域都有应用。它已成为无类型脚本语言的流行替代品，因为它在表现力和安全性之间取得了平衡：Go程序通常比用动态语言编写的程序运行得更快，而且由于意外的类型错误导致的崩溃要少得多。

Go is an open-source project, so source code for its compiler, libraries, and tools is freely available to anyone. Contributions to the project come from an active worldwide community. Go runs on Unix-like systems—Linux, FreeBSD, OpenBSD, Mac OS X—and on Plan 9 and Microsoft Windows. Programs written in one of these environments generally work without modification on the others.

Go是一个开源项目，因此其编译器、库和工具的源代码对任何人都是免费开放的。项目的贡献来自活跃的全球社区。Go可以运行在类Unix系统上——Linux、FreeBSD、OpenBSD、Mac OS X——以及Plan 9和Microsoft Windows上。在其中一个环境中编写的程序通常无需修改即可在其他环境中运行。

This book is meant to help you start using Go effectively right away and to use it well, taking full advantage of Go's language features and standard libraries to write clear, idiomatic, and efficient programs.

本书旨在帮助你立即有效地开始使用Go，并充分利用Go的语言特性和标准库来编写清晰、地道和高效的程序。

## The Origins of Go

## Go的起源

Like biological species, successful languages beget offspring that incorporate the advantages of their ancestors; interbreeding sometimes leads to surprising strengths; and, very occasionally, a radical new feature arises without precedent. We can learn a lot about why a language is the way it is and what environment it has been adapted for by looking at these influences.

就像生物物种一样，成功的语言会产生后代，这些后代融合了其祖先的优势；杂交有时会带来惊人的优势；而且，偶尔会出现没有先例的全新特性。通过研究这些影响，我们可以深入了解一种语言为何如此设计，以及它适应了什么样的环境。

The figure below shows the most important influences of earlier programming languages on the design of Go.

下图展示了早期编程语言对Go设计的最重要影响。

Go is sometimes described as a "C-like language," or as "C for the 21st century." From C, Go inherited its expression syntax, control-flow statements, basic data types, call-by-value parameter passing, pointers, and above all, C's emphasis on programs that compile to efficient machine code and cooperate naturally with the abstractions of current operating systems.

Go有时被描述为"类C语言"，或"21世纪的C"。从C语言中，Go继承了表达式语法、控制流语句、基本数据类型、按值传递参数、指针，以及最重要的——C语言强调程序应编译成高效的机器码并与当前操作系统的抽象自然协作的理念。

But there are other ancestors in Go's family tree. One major stream of influence comes from languages by Niklaus Wirth, beginning with Pascal. Modula-2 inspired the package concept. Oberon eliminated the distinction between module interface files and module implementation files. Oberon-2 influenced the syntax for packages, imports, and declarations, and Object Oberon provided the syntax for method declarations.

但Go的家族树中还有其他祖先。一个主要的影响来源是Niklaus Wirth的语言系列，从Pascal开始。Modula-2启发了包（package）的概念。Oberon消除了模块接口文件和模块实现文件之间的区别。Oberon-2影响了包、导入和声明的语法，而Object Oberon提供了方法声明的语法。

Another lineage among Go's ancestors, and one that makes Go distinctive among recent programming languages, is a sequence of little-known research languages developed at Bell Labs, all inspired by the concept of communicating sequential processes (CSP) from Tony Hoare's seminal 1978 paper on the foundations of concurrency. In CSP, a program is a parallel composition of processes that have no shared state; the processes communicate and synchronize using channels. But Hoare's CSP was a formal language for describing the fundamental concepts of concurrency, not a programming language for writing executable programs.

Go祖先中的另一个谱系，也是使Go在近期编程语言中独具特色的一个谱系，是贝尔实验室开发的一系列鲜为人知的研究语言，它们都受到Tony Hoare 1978年关于并发基础的开创性论文中通信顺序进程（CSP）概念的启发。在CSP中，程序是没有共享状态的进程的并行组合；进程使用通道（channel）进行通信和同步。但Hoare的CSP是一种用于描述并发基本概念的形式语言，而不是用于编写可执行程序的编程语言。

Rob Pike and others began to experiment with CSP implementations as actual languages. The first was called Squeak ("A language for communicating with mice"), which provided a language for handling mouse and keyboard events, with statically created channels. This was followed by Newsqueak, which offered C-like statement and expression syntax and Pascal-like type notation. It was a purely functional language with garbage collection, again aimed at managing keyboard, mouse, and window events. Channels became first-class values, dynamically created and storable in variables.

Rob Pike等人开始尝试将CSP作为实际语言来实现。第一个被称为Squeak（"与鼠标通信的语言"），它提供了一种处理鼠标和键盘事件的语言，使用静态创建的通道。随后是Newsqueak，它提供了类似C的语句和表达式语法以及类似Pascal的类型表示法。它是一种带有垃圾回收的纯函数式语言，同样旨在管理键盘、鼠标和窗口事件。通道成为一等值，可以动态创建并存储在变量中。

The Plan 9 operating system carried these ideas forward in a language called Alef. Alef tried to make Newsqueak a viable system programming language, but its omission of garbage collection made concurrency too painful.

Plan 9操作系统在一种名为Alef的语言中推进了这些理念。Alef试图使Newsqueak成为一种可行的系统编程语言，但由于缺少垃圾回收，使得并发编程变得过于痛苦。

Other constructions in Go show the influence of non-ancestral genes here and there; for example iota is loosely from APL, and lexical scope with nested functions is from Scheme (and most languages since). Here too we find novel mutations. Go's innovative slices provide dynamic arrays with efficient random access but also permit sophisticated sharing arrangements reminiscent of linked lists. And the defer statement is new with Go.

Go中的其他构造在各处显示了非祖先基因的影响；例如，iota大致来自APL，而带有嵌套函数的词法作用域来自Scheme（以及此后的大多数语言）。在这里我们也发现了新颖的变异。Go创新的切片（slice）提供了具有高效随机访问的动态数组，但也允许类似链表的复杂共享安排。而defer语句是Go的新创造。

## The Go Project

## Go项目

All programming languages reflect the programming philosophy of their creators, which often includes a significant component of reaction to the perceived shortcomings of earlier languages. The Go project was borne of frustration with several software systems at Google that were suffering from an explosion of complexity. (This problem is by no means unique to Google.)

所有编程语言都反映了其创造者的编程哲学，这通常包括对早期语言感知缺陷的重要反应。Go项目诞生于对Google几个软件系统复杂性爆炸的挫败感。（这个问题绝不是Google独有的。）

As Rob Pike put it, "complexity is multiplicative": fixing a problem by making one part of the system more complex slowly but surely adds complexity to other parts. With constant pressure to add features and options and configurations, and to ship code quickly, it's easy to neglect simplicity, even though in the long run simplicity is the key to good software.

正如Rob Pike所说，"复杂性是乘法式的"：通过使系统的一部分变得更复杂来解决问题，会缓慢但肯定地给其他部分增加复杂性。在不断增加功能、选项和配置以及快速发布代码的压力下，很容易忽视简单性，尽管从长远来看，简单性是优秀软件的关键。

Simplicity requires more work at the beginning of a project to reduce an idea to its essence and more discipline over the lifetime of a project to distinguish good changes from bad or pernicious ones. With sufficient effort, a good change can be accommodated without compromising what Fred Brooks called the "conceptual integrity" of the design but a bad change cannot, and a pernicious change trades simplicity for its shallow cousin, convenience. Only through simplicity of design can a system remain stable, secure, and coherent as it grows.

简单性需要在项目开始时付出更多努力来将想法提炼到本质，并在项目的整个生命周期中保持更多纪律来区分好的变更和坏的或有害的变更。通过充分的努力，好的变更可以在不损害Fred Brooks所说的设计"概念完整性"的情况下被采纳，但坏的变更则不能，而有害的变更则是用简单性换取其肤浅的表亲——便利性。只有通过设计的简单性，系统才能在成长过程中保持稳定、安全和连贯。

The Go project includes the language itself, its tools and standard libraries, and last but not least, a cultural agenda of radical simplicity. As a recent high-level language, Go has the benefit of hindsight, and the basics are done well: it has garbage collection, a package system, first-class functions, lexical scope, a system call interface, and immutable strings in which text is generally encoded in UTF-8. But it has comparatively few features and is unlikely to add more. For instance, it has no implicit numeric conversions, no constructors or destructors, no operator overloading, no default parameter values, no inheritance, no generics, no exceptions, no macros, no function annotations, and no thread-local storage. The language is mature and stable, and guarantees backwards compatibility: older Go programs can be compiled and run with newer versions of compilers and standard libraries.

Go项目包括语言本身、它的工具和标准库，最后但同样重要的是，一种激进简单的文化议程。作为一种较新的高级语言，Go具有后见之明的优势，基础工作做得很好：它有垃圾回收、包系统、一等函数、词法作用域、系统调用接口，以及通常以UTF-8编码文本的不可变字符串。但它的特性相对较少，而且不太可能增加更多。例如，它没有隐式数值转换、没有构造函数或析构函数、没有运算符重载、没有默认参数值、没有继承、没有泛型、没有异常、没有宏、没有函数注解，也没有线程局部存储。该语言成熟且稳定，并保证向后兼容：较早的Go程序可以用较新版本的编译器和标准库编译并运行。

Go has enough of a type system to avoid most of the careless mistakes that plague programmers in dynamic languages, but it has a simpler type system than comparable typed languages. This approach can sometimes lead to isolated pockets of "untyped" programming within a broader framework of types, and Go programmers do not go to the lengths that C++ or Haskell programmers do to express safety properties as type-based proofs. But in practice Go gives programmers much of the safety and run-time performance benefits of a relatively strong type system without the burden of a complex one.

Go有足够的类型系统来避免动态语言程序员常犯的大多数粗心错误，但它的类型系统比同类的类型化语言更简单。这种方法有时会在更广泛的类型框架内导致孤立的"无类型"编程区域，Go程序员不会像C++或Haskell程序员那样费尽心思将安全属性表达为基于类型的证明。但在实践中，Go为程序员提供了相对强大的类型系统的大部分安全性和运行时性能优势，而没有复杂类型系统的负担。

Go encourages an awareness of contemporary computer system design, particularly the importance of locality. Its built-in data types and most library data structures are crafted to work naturally without explicit initialization or implicit constructors, so relatively few memory allocations and memory writes are hidden in the code. Go's aggregate types (structs and arrays) hold their elements directly, requiring less storage and fewer allocations and pointer indirections than languages that use indirect fields. And since the modern computer is a parallel machine, Go has concurrency features based on CSP, as mentioned earlier. The variable-size stacks of Go's lightweight threads or goroutines are initially small enough that creating one goroutine is cheap and creating a million is practical.

Go鼓励对当代计算机系统设计的认识，特别是局部性的重要性。它的内置数据类型和大多数库数据结构都被精心设计为无需显式初始化或隐式构造函数即可自然工作，因此代码中隐藏的内存分配和内存写入相对较少。Go的聚合类型（结构体和数组）直接持有它们的元素，与使用间接字段的语言相比，需要更少的存储空间、更少的分配和指针间接引用。由于现代计算机是并行机器，Go具有基于CSP的并发特性，如前所述。Go的轻量级线程或goroutine的可变大小栈最初很小，因此创建一个goroutine很便宜，创建一百万个也是可行的。

Go's standard library, often described as coming with "batteries included," provides clean building blocks and APIs for I/O, text processing, graphics, cryptography, networking, and distributed applications, with support for many standard file formats and protocols. The libraries and tools make extensive use of convention to reduce the need for configuration and explanation, thus simplifying program logic and making diverse Go programs more similar to each other and thus easier to learn. Projects built using the go tool use only file and identifier names and an occasional special comment to determine all the libraries, executables, tests, benchmarks, examples, platform-specific variants, and documentation for a project; the Go source itself contains the build specification.

Go的标准库，通常被描述为"内置电池"，为I/O、文本处理、图形、密码学、网络和分布式应用提供了清晰的构建块和API，支持许多标准文件格式和协议。库和工具广泛使用约定来减少配置和解释的需要，从而简化程序逻辑，使不同的Go程序彼此更相似，因此更容易学习。使用go工具构建的项目仅使用文件和标识符名称以及偶尔的特殊注释来确定项目的所有库、可执行文件、测试、基准测试、示例、特定平台变体和文档；Go源代码本身包含构建规范。

## Organization of the Book

## 本书的组织结构

We assume that you have programmed in one or more other languages, whether compiled like C, C++, and Java, or interpreted like Python, Ruby, and JavaScript, so we won't spell out everything as if for a total beginner. Surface syntax will be familiar, as will variables and constants, expressions, control flow, and functions.

我们假设你已经用一种或多种其他语言编程，无论是像C、C++和Java这样的编译型语言，还是像Python、Ruby和JavaScript这样的解释型语言，所以我们不会像对完全初学者那样详细说明一切。表面语法会很熟悉，变量和常量、表达式、控制流和函数也是如此。

Chapter 1 is a tutorial on the basic constructs of Go, introduced through a dozen programs for everyday tasks like reading and writing files, formatting text, creating images, and communicating with Internet clients and servers.

第1章是Go基本构造的教程，通过十几个日常任务程序来介绍，如读写文件、格式化文本、创建图像以及与Internet客户端和服务器通信。

Chapter 2 describes the structural elements of a Go program—declarations, variables, new types, packages and files, and scope. Chapter 3 discusses numbers, booleans, strings, and constants, and explains how to process Unicode. Chapter 4 describes composite types, that is, types built up from simpler ones using arrays, maps, structs, and slices, Go's approach to dynamic lists. Chapter 5 covers functions and discusses error handling, panic and recover, and the defer statement.

第2章描述了Go程序的结构元素——声明、变量、新类型、包和文件以及作用域。第3章讨论数字、布尔值、字符串和常量，并解释如何处理Unicode。第4章描述复合类型，即使用数组、映射、结构体和切片（Go处理动态列表的方法）从更简单的类型构建的类型。第5章涵盖函数，并讨论错误处理、panic和recover以及defer语句。

Chapters 1 through 5 are thus the basics, things that are part of any mainstream imperative language. Go's syntax and style sometimes differ from other languages, but most programmers will pick them up quickly. The remaining chapters focus on topics where Go's approach is less conventional: methods, interfaces, concurrency, packages, testing, and reflection.

因此，第1章到第5章是基础知识，这些是任何主流命令式语言的一部分。Go的语法和风格有时与其他语言不同，但大多数程序员会很快掌握它们。其余章节关注Go方法不太传统的主题：方法、接口、并发、包、测试和反射。

Go has an unusual approach to object-oriented programming. There are no class hierarchies, or indeed any classes; complex object behaviors are created from simpler ones by composition, not inheritance. Methods may be associated with any user-defined type, not just structures, and the relationship between concrete types and abstract types (interfaces) is implicit, so a concrete type may satisfy an interface that the type's designer was unaware of. Methods are covered in Chapter 6 and interfaces in Chapter 7.

Go对面向对象编程有一种不寻常的方法。没有类层次结构，实际上也没有任何类；复杂的对象行为是通过组合而不是继承从更简单的行为创建的。方法可以与任何用户定义的类型关联，而不仅仅是结构体，具体类型（concrete types）和抽象类型（接口）之间的关系是隐式的，因此具体类型可以满足类型设计者不知道的接口。方法在第6章中介绍，接口在第7章中介绍。

Chapter 8 presents Go's approach to concurrency, which is based on the idea of communicating sequential processes (CSP), embodied by goroutines and channels. Chapter 9 explains the more traditional aspects of concurrency based on shared variables.

第8章介绍Go的并发方法，它基于通信顺序进程（CSP）的思想，由goroutine和通道体现。第9章解释基于共享变量的更传统的并发方面。

Chapter 10 describes packages, the mechanism for organizing libraries. This chapter also shows how to make effective use of the go tool, which provides for compilation, testing, benchmarking, program formatting, documentation, and many other tasks, all within a single command.

第10章描述包，即组织库的机制。本章还展示了如何有效使用go工具，它在单个命令中提供编译、测试、基准测试、程序格式化、文档和许多其他任务。

Chapter 11 deals with testing, where Go takes a notably lightweight approach, avoiding abstraction-laden frameworks in favor of simple libraries and tools. The testing libraries provide a foundation atop which more complex abstractions can be built if necessary.

第11章涉及测试，Go在这方面采用了明显的轻量级方法，避免了充满抽象的框架，而是选择简单的库和工具。测试库提供了一个基础，如果需要，可以在其上构建更复杂的抽象。

Chapter 12 discusses reflection, the ability of a program to examine its own representation during execution. Reflection is a powerful tool, though one to be used carefully; this chapter explains finding the right balance by showing how it is used to implement some important Go libraries. Chapter 13 explains the gory details of low-level programming that uses the unsafe package to step around Go's type system, and when that is appropriate.

第12章讨论反射，即程序在执行期间检查自身表示的能力。反射是一个强大的工具，尽管需要谨慎使用；本章通过展示如何使用它来实现一些重要的Go库来解释如何找到正确的平衡。第13章解释了使用unsafe包绕过Go类型系统的低级编程的血腥细节，以及何时这样做是合适的。

Each chapter has a number of exercises that you can use to test your understanding of Go, and to explore extensions and alternatives to the examples from the book.

每章都有一些练习，你可以用来测试对Go的理解，并探索书中示例的扩展和替代方案。

All but the most trivial code examples in the book are available for download from the public Git repository at gopl.io. Each example is identified by its package import path and may be conveniently fetched, built, and installed using the go get command. You'll need to choose a directory to be your Go workspace and set the GOPATH environment variable to point to it. The go tool will create the directory if necessary. For example:

书中除了最简单的代码示例外，所有示例都可以从gopl.io的公共Git仓库下载。每个示例都通过其包导入路径标识，并且可以使用go get命令方便地获取、构建和安装。你需要选择一个目录作为Go工作空间，并设置GOPATH环境变量指向它。如果需要，go工具会创建该目录。例如：

```
$ export GOPATH=$HOME/gobook    # choose workspace directory
$ go get gopl.io/ch1/helloworld # fetch, build, install
$ $GOPATH/bin/helloworld        # run
Hello, 世界
```

To run the examples, you will need at least version 1.5 of Go.

要运行这些示例，你需要至少Go 1.5版本。

```
$ go version
go version go1.5 linux/amd64
```

Follow the instructions at https://golang.org/doc/install if the go tool on your computer is older or missing.

如果你计算机上的go工具较旧或缺失，请按照https://golang.org/doc/install上的说明进行操作。

## Where to Find More Information

## 更多信息的获取途径

The best source for more information about Go is the official web site, https://golang.org, which provides access to the documentation, including the Go Programming Language Specification, standard packages, and the like. There are also tutorials on how to write Go and how to write it well, and a wide variety of online text and video resources that will be valuable complements to this book. The Go Blog at blog.golang.org publishes some of the best writing on Go, with articles on the state of the language, plans for the future, reports on conferences, and in-depth explanations of a wide variety of Go-related topics.

获取更多Go信息的最佳来源是官方网站https://golang.org，它提供了文档访问，包括Go编程语言规范、标准包等。还有关于如何编写Go以及如何编写好Go的教程，以及各种在线文本和视频资源，这些将是本书的宝贵补充。blog.golang.org的Go博客发布了一些关于Go的最佳文章，包括语言状态、未来计划、会议报告以及各种Go相关主题的深入解释。

One of the most useful aspects of online access to Go (and a regrettable limitation of a paper book) is the ability to run Go programs from the web pages that describe them. This functionality is provided by the Go Playground at play.golang.org, and may be embedded within other pages, such as the home page at golang.org or the documentation pages served by the godoc tool.

在线访问Go最有用的方面之一（也是纸质书的一个遗憾限制）是能够从描述它们的网页运行Go程序。这个功能由play.golang.org的Go Playground提供，并且可以嵌入到其他页面中，例如golang.org的主页或godoc工具提供的文档页面。

The Playground makes it convenient to perform simple experiments to check one's understanding of syntax, semantics, or library packages with short programs, and in many ways takes the place of a read-eval-print loop (REPL) in other languages. Its persistent URLs are great for sharing snippets of Go code with others, for reporting bugs or making suggestions.

Playground使得通过短程序进行简单实验来检查对语法、语义或库包的理解变得方便，在许多方面取代了其他语言中的读取-求值-打印循环（REPL）。它的持久URL非常适合与他人分享Go代码片段，用于报告错误或提出建议。

Built atop the Playground, the Go Tour at tour.golang.org is a sequence of short interactive lessons on the basic ideas and constructions of Go, an orderly walk through the language.

建立在Playground之上的Go Tour（tour.golang.org）是一系列关于Go基本思想和构造的简短交互式课程，是对该语言的有序浏览。

The primary shortcoming of the Playground and the Tour is that they allow only standard libraries to be imported, and many library features—networking, for example—are restricted for practical or security reasons. They also require access to the Internet to compile and run each program. So for more elaborate experiments, you will have to run Go programs on your own computer. Fortunately the download process is straightforward, so it should not take more than a few minutes to fetch the Go distribution from golang.org and start writing and running Go programs of your own.

Playground和Tour的主要缺点是它们只允许导入标准库，并且出于实用或安全原因，许多库功能（例如网络）受到限制。它们还需要访问Internet来编译和运行每个程序。因此，对于更复杂的实验，你必须在自己的计算机上运行Go程序。幸运的是，下载过程很简单，所以从golang.org获取Go发行版并开始编写和运行自己的Go程序应该不会花费超过几分钟的时间。

Since Go is an open-source project, you can read the code for any type or function in the standard library online at https://golang.org/pkg; the same code is part of the downloaded distribution. Use this to figure out how something works, or to answer questions about details, or merely to see how experts write really good Go.

由于Go是一个开源项目，你可以在https://golang.org/pkg在线阅读标准库中任何类型或函数的代码；相同的代码是下载发行版的一部分。使用它来弄清楚某些东西是如何工作的，或回答有关细节的问题，或仅仅看看专家如何编写真正优秀的Go代码。

## Acknowledgments

## 致谢

Rob Pike and Russ Cox, core members of the Go team, read the manuscript several times with great care; their comments on everything from word choice to overall structure and organization have been invaluable. While preparing the Japanese translation, Yoshiki Shibata went far beyond the call of duty; his meticulous eye spotted numerous inconsistencies in the English text and errors in the code. We greatly appreciate thorough reviews and critical comments on the entire manuscript from Brian Goetz, Corey Kosak, Arnold Robbins, Josh Bleecher Snyder, and Peter Weinberger.

Go团队的核心成员Rob Pike和Russ Cox非常仔细地多次阅读了手稿；他们对从措辞选择到整体结构和组织的所有评论都是无价的。在准备日文翻译时，Yoshiki Shibata的工作远远超出了职责范围；他细致的眼光发现了英文文本中的许多不一致之处和代码中的错误。我们非常感谢Brian Goetz、Corey Kosak、Arnold Robbins、Josh Bleecher Snyder和Peter Weinberger对整个手稿的全面审查和批判性评论。

We are indebted to Sameer Ajmani, Ittai Balaban, David Crawshaw, Billy Donohue, Jonathan Feinberg, Andrew Gerrand, Robert Griesemer, John Linderman, Minux Ma, Bryan Mills, Bala Natarajan, Cosmos Nicolaou, Paul Staniforth, Nigel Tao, and Howard Trickey for many helpful suggestions. We also thank David Brailsford and Raph Levien for typesetting advice.

我们感谢Sameer Ajmani、Ittai Balaban、David Crawshaw、Billy Donohue、Jonathan Feinberg、Andrew Gerrand、Robert Griesemer、John Linderman、Minux Ma、Bryan Mills、Bala Natarajan、Cosmos Nicolaou、Paul Staniforth、Nigel Tao和Howard Trickey提供的许多有用建议。我们还要感谢David Brailsford和Raph Levien的排版建议。

Our editor Greg Doench at Addison-Wesley got the ball rolling originally and has been continuously helpful ever since. The AW production team—John Fuller, Dayna Isley, Julie Nahil, Chuti Prasertsith, and Barbara Wood—has been outstanding; authors could not hope for better support.

我们在Addison-Wesley的编辑Greg Doench最初推动了这个项目，并且从那时起一直在持续提供帮助。AW制作团队——John Fuller、Dayna Isley、Julie Nahil、Chuti Prasertsith和Barbara Wood——表现出色；作者们不可能期望得到更好的支持。

Alan Donovan wishes to thank: Sameer Ajmani, Chris Demetriou, Walt Drummond, and Reid Tatge at Google for allowing him time to write; Stephen Donovan, for his advice and timely encouragement; and above all, his wife Leila Kazemi, for her unhesitating enthusiasm and unwavering support for this project, despite the long hours of distraction and absenteeism from family life that it entailed.

Alan Donovan要感谢：Google的Sameer Ajmani、Chris Demetriou、Walt Drummond和Reid Tatge允许他有时间写作；Stephen Donovan，感谢他的建议和及时的鼓励；最重要的是，他的妻子Leila Kazemi，尽管这个项目需要长时间的分心和缺席家庭生活，但她对这个项目表现出了毫不犹豫的热情和坚定不移的支持。

Brian Kernighan is deeply grateful to friends and colleagues for their patience and forbearance as he moved slowly along the path to understanding, and especially to his wife Meg, who has been unfailingly supportive of book-writing and so much else.

Brian Kernighan深深感谢朋友和同事们在他缓慢理解的道路上的耐心和宽容，特别是他的妻子Meg，她对写书和其他许多事情一直给予了坚定的支持。

New York  
October 2015

纽约  
2015年10月