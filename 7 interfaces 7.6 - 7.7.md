Okay, I have updated the Chinese translation based on the previous proofreading comments, primarily ensuring that comments within code blocks are identical to the English original (and not translated or added), and removing any "校对注".

Here is the revised output:

## 7.6 Sorting with sort.Interface

Like string formatting, sorting is a frequently used operation in many programs.  Although a minimal Quicksort can be written in about 15 lines, a robust implementation is much longer, and it is not the kind of code we should wish to write anew or copy each time we need it.  Fortunately, the sort package provides in-place sorting of any sequence according to any ordering function.  Its design is rather unusual.  In many languages, the sorting algorithm is associated with the sequence data type, while the ordering function is associated with the type of the elements.  By contrast, Go's sort.Sort function assumes nothing about the representation of either the sequence or its elements.  Instead, it uses an interface, sort.Interface, to specify the contract between the generic sort algorithm and each sequence type that may be sorted.  An implementation of this interface determines both the concrete representation of the sequence, which is often a slice, and the desired ordering of its elements. 

与字符串格式化类似，排序是许多程序中频繁执行的操作。  尽管一个最小化的快速排序可以用大约15行代码实现，但一个健壮的（robust）实现要长得多，而且它不是我们希望每次需要时都重新编写或复制的那种代码。  幸运的是，`sort`包提供了根据任意排序函数对任意序列进行原地（in-place）排序的功能。  其设计颇为不寻常（unusual）。  在许多语言中，排序算法与序列数据类型相关联，而排序函数则与元素的类型相关联。  相比之下，Go的`sort.Sort`函数对序列或其元素的表示不做任何假设。  取而代之的是，它使用一个接口`sort.Interface`来指定通用排序算法与每个可排序序列类型之间的合约。  该接口的实现既确定了序列的具体表示（通常是切片），也确定了其元素的期望排序顺序。 

An in-place sort algorithm needs three things-the length of the sequence, a means of comparing two elements, and a way to swap two elements so they are the three methods of sort.Interface: 

```go
package sort

type Interface interface {
    Len() int
    Less(i, j int) bool // i, j are indices of sequence elements
    Swap(i, j int)
}
```


一个原地排序算法需要三样东西——序列的长度、比较两个元素的方式以及交换两个元素的方式——这正是`sort.Interface`的三个方法： 
```go
package sort

type Interface interface {
    Len() int
    Less(i, j int) bool // i, j are indices of sequence elements
    Swap(i, j int)
}
```


To sort any sequence, we need to define a type that implements these three methods, then apply sort.Sort to an instance of that type.  As perhaps the simplest example, consider sorting a slice of strings.  The new type StringSlice and its Len, Less, and Swap methods are shown below. 

```go
type StringSlice []string

func (p StringSlice) Len() int           { return len(p) }
func (p StringSlice) Less(i, j int) bool { return p[i] < p[j] }
func (p StringSlice) Swap(i, j int)      { p[i], p[j] = p[j], p[i] }
```


要对任何序列进行排序，我们需要定义一个实现这三个方法的类型，然后将`sort.Sort`应用于该类型的实例。  作为一个可能最简单的例子，考虑对字符串切片进行排序。  新类型`StringSlice`及其`Len`、`Less`和`Swap`方法如下所示。 
```go
type StringSlice []string

func (p StringSlice) Len() int           { return len(p) }
func (p StringSlice) Less(i, j int) bool { return p[i] < p[j] }
func (p StringSlice) Swap(i, j int)      { p[i], p[j] = p[j], p[i] }
```


Now we can sort a slice of strings, names, by converting the slice to a StringSlice like this: 

sort.Sort(StringSlice(names))

The conversion yields a slice value with the same length, capacity, and underlying array as names but with a type that has the three methods required for sorting.  Sorting a slice of strings is so common that the sort package provides the StringSlice type, as well as a function called Strings so that the call above can be simplified to sort.Strings(names).  The technique here is easily adapted to other sort orders, for instance, to ignore capitalization or special characters.  (The Go program that sorts index terms and page numbers for this book does this, with extra logic for Roman numerals.) For more complicated sorting, we use the same idea, but with more complicated data structures or more complicated implementations of the sort.Interface methods. 

现在我们可以通过将切片names转换为StringSlice来对其进行排序，如下所示： 
```go
sort.Sort(StringSlice(names))
```
该转换产生一个切片值，其长度、容量和底层数组与`names`相同，但其类型拥有排序所需的三个方法。  对字符串切片进行排序非常常见，因此`sort`包提供了`StringSlice`类型以及一个名为`Strings`的函数，这样上述调用可以简化为`sort.Strings(names)`。 
这里的技术很容易适应其他排序规则，例如，忽略大小写或特殊字符。  （本书用于对索引术语和页码进行排序的Go程序就是这样做的，并带有处理罗马数字的额外逻辑。）对于更复杂的排序，我们使用相同的思想，但采用更复杂的数据结构或更复杂的`sort.Interface`方法实现。 

Our running example for sorting will be a music playlist, displayed as a table.  Each track is a single row, and each column is an attribute of that track, like artist, title, and running time.  Imagine that a graphical user interface presents the table, and that clicking the head of a column causes the playlist to be sorted by that attribute; clicking the same column head again reverses the order.  Let's look at what might happen in response to each click. 
The variable tracks below contains a playlist.  (One of the authors apologizes for the other author's musical tastes.) Each element is indirect, a pointer to a Track.  Although the code below would work if we stored the Tracks directly, the sort function will swap many pairs of elements, so it will run faster if each element is a pointer, which is a single machine word, instead of an entire Track, which might be eight words or more. 
gopl.io/ch7/sorting
```go
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
```


我们贯穿排序的示例是一个音乐播放列表，以表格形式显示。  每个音轨（track）是一行，每列是该音轨的一个属性，如艺术家、标题和播放时长。  想象一个图形用户界面呈现该表格，点击列标题会导致播放列表按该属性排序；再次点击同一列标题则反转顺序。  让我们看看每次点击后可能发生什么。 
下面的变量`tracks`包含一个播放列表。  （其中一位作者为另一位作者的音乐品味致歉。）每个元素都是间接的（indirect），是一个指向`Track`的指针。  虽然如果我们直接存储`Track`结构体，下面的代码也能工作，但排序函数会交换许多元素对，因此如果每个元素都是指针（一个机器字大小），而不是整个`Track`结构体（可能需要八个或更多机器字），排序会运行得更快。 
_gopl.io/ch7/sorting_
```go
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
```


```go
func length(s string) time.Duration {
    d, err := time.ParseDuration(s)
    if err != nil {
        panic(s)
    }
    return d
}
```


```go
func length(s string) time.Duration {
    d, err := time.ParseDuration(s)
    if err != nil {
        panic(s)
    }
    return d
}
```


The printTracks function prints the playlist as a table.  A graphical display would be nicer, but this little routine uses the text/tabwriter package to produce a table whose columns are neatly aligned and padded as shown below.  Observe that *tabwriter.Writer satisfies io.Writer.  It collects each piece of data written to it; its Flush method formats the entire table and writes it to os.Stdout. 
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


`printTracks`函数将播放列表打印为表格。  图形显示会更好，但这个小程序使用`text/tabwriter`包来生成一个表格，其列被整齐地对齐和填充，如下所示。  注意`*tabwriter.Writer`满足`io.Writer`接口。  它收集写入它的每一条数据；其`Flush`方法格式化整个表格并将其写入`os.Stdout`。 
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
```go
type byArtist []*Track

func (x byArtist) Len() int           { return len(x) }
func (x byArtist) Less(i, j int) bool { return x[i].Artist < x[j].Artist }
func (x byArtist) Swap(i, j int)      { x[i], x[j] = x[j], x[i] }
```


To call the generic sort routine, we must first convert tracks to the new type, byArtist, that defines the order: 
```go
sort.Sort(byArtist(tracks))
```


After sorting the slice by artist, the output from printTracks is: 
```text
Title       Artist          Album              Year  Length
-----       ------          -----              ----  ------
Go Ahead    Alicia Keys     As I Am            2007  4m36s
Go          Delilah         From the Roots Up  2012  3m38s
Ready 2 Go  Martin Solveig  Smash              2011  4m24s
Go          Moby            Moby               1992  3m37s
```

要按`Artist`字段对播放列表进行排序，我们定义一个新的切片类型，并使其拥有必要的`Len`、`Less`和`Swap`方法，这与我们为`StringSlice`所做的类似。 
```go
type byArtist []*Track

func (x byArtist) Len() int           { return len(x) }
func (x byArtist) Less(i, j int) bool { return x[i].Artist < x[j].Artist }
func (x byArtist) Swap(i, j int)      { x[i], x[j] = x[j], x[i] }
```

要调用通用排序例程，我们必须首先将tracks转换为定义了排序顺序的新类型byArtist： 
```go
sort.Sort(byArtist(tracks))
```

按艺术家对切片排序后，printTracks的输出是： 
```text
Title       Artist          Album              Year  Length
-----       ------          -----              ----  ------
Go Ahead    Alicia Keys     As I Am            2007  4m36s
Go          Delilah         From the Roots Up  2012  3m38s
Ready 2 Go  Martin Solveig  Smash              2011  4m24s
Go          Moby            Moby               1992  3m37s
```

If the user requests "sort by artist" a second time, we'll sort the tracks in reverse.  We needn't define a new type byReverseArtist with an inverted Less method, however, since the sort package provides a Reverse function that transforms any sort order to its inverse. 
```go
sort.Sort(sort.Reverse(byArtist(tracks)))
```

After reverse-sorting the slice by artist, the output from printTracks is: 
```text
Title       Artist          Album              Year  Length
-----       ------          -----              ----  ------
Go          Moby            Moby               1992  3m37s
Ready 2 Go  Martin Solveig  Smash              2011  4m24s
Go          Delilah         From the Roots Up  2012  3m38s
Go Ahead    Alicia Keys     As I Am            2007  4m36s
```

如果用户再次请求“按艺术家排序”，我们将按相反的顺序对曲目进行排序。  然而，我们不必定义一个具有反转`Less`方法的新类型`byReverseArtist`，因为`sort`包提供了一个`Reverse`函数，它可以将任何排序顺序转换为其逆序。 
```go
sort.Sort(sort.Reverse(byArtist(tracks)))
```

按艺术家对切片进行反向排序后，`printTracks`的输出是： 
```text
Title       Artist          Album              Year  Length
-----       ------          -----              ----  ------
Go          Moby            Moby               1992  3m37s
Ready 2 Go  Martin Solveig  Smash              2011  4m24s
Go          Delilah         From the Roots Up  2012  3m38s
Go Ahead    Alicia Keys     As I Am            2007  4m36s
```

The sort.Reverse function deserves a closer look since it uses composition (§6.3), which is an important idea.  The sort package defines an unexported type reverse, which is a struct that embeds a sort.Interface.  The Less method for reverse calls the Less method of the embedded sort.Interface value, but with the indices flipped, reversing the order of the sort results. 
```go
package sort

type reverse struct{ Interface } // that is, sort.Interface
                                  // Interface is sort.Interface 

func (r reverse) Less(i, j int) bool { return r.Interface.Less(j, i) }

// Len and Swap are implicitly provided by r.Interface. 
func Reverse(data Interface) Interface { return reverse{data} }
```

Len and Swap, the other two methods of reverse, are implicitly provided by the original sort.Interface value because it is an embedded field.  The exported function Reverse returns an instance of the reverse type that contains the original sort.Interface value. 

`sort.Reverse`函数值得仔细研究，因为它使用了组合（composition）（§6.3），这是一个重要的概念。  `sort`包定义了一个未导出的类型`reverse`，它是一个嵌入了`sort.Interface`的结构体。  `reverse`类型的`Less`方法调用嵌入的`sort.Interface`值的`Less`方法，但交换了索引`i`和`j`，从而反转了排序结果的顺序。 
```go
package sort

type reverse struct{ Interface } // that is, sort.Interface
                                  // Interface is sort.Interface 

func (r reverse) Less(i, j int) bool { return r.Interface.Less(j, i) }

// Len and Swap are implicitly provided by r.Interface. 
func Reverse(data Interface) Interface { return reverse{data} }
```

`reverse`类型的另外两个方法，`Len`和`Swap`，由原始的`sort.Interface`值隐式提供，因为它是嵌入字段。  导出的`Reverse`函数返回一个`reverse`类型的实例，该实例包含原始的`sort.Interface`值。 

To sort by a different column, we must define a new type, such as byYear: 
```go
type byYear []*Track

func (x byYear) Len() int           { return len(x) }
func (x byYear) Less(i, j int) bool { return x[i].Year < x[j].Year }
func (x byYear) Swap(i, j int)      { x[i], x[j] = x[j], x[i] }
```

After sorting tracks by year using sort.Sort(byYear(tracks)), printTracks shows a chronological listing: 
```text
Title       Artist          Album              Year  Length
-----       ------          -----              ----  ------
Go          Moby            Moby               1992  3m37s
Go Ahead    Alicia Keys     As I Am            2007  4m36s
Ready 2 Go  Martin Solveig  Smash              2011  4m24s
Go          Delilah         From the Roots Up  2012  3m38s
```


要按不同的列排序，我们必须定义一个新的类型，例如`byYear`： 
```go
type byYear []*Track

func (x byYear) Len() int           { return len(x) }
func (x byYear) Less(i, j int) bool { return x[i].Year < x[j].Year }
func (x byYear) Swap(i, j int)      { x[i], x[j] = x[j], x[i] }
```

使用`sort.Sort(byYear(tracks))`按年份对曲目进行排序后，`printTracks`显示一个按时间顺序排列的列表： 
```text
Title       Artist          Album              Year  Length
-----       ------          -----              ----  ------
Go          Moby            Moby               1992  3m37s
Go Ahead    Alicia Keys     As I Am            2007  4m36s
Ready 2 Go  Martin Solveig  Smash              2011  4m24s
Go          Delilah         From the Roots Up  2012  3m38s
```


For every slice element type and every ordering function we need, we declare a new implementation of sort.Interface.  As you can see, the Len and Swap methods have identical definitions for all slice types.  In the next example, the concrete type customSort combines a slice with a function, letting us define a new sort order by writing only the comparison function.  Incidentally, the concrete types that implement sort.Interface are not always slices; customSort is a struct type. 
```go
type customSort struct {
    t    []*Track
    less func(x, y *Track) bool
}

func (x customSort) Len() int           { return len(x.t) }
func (x customSort) Less(i, j int) bool { return x.less(x.t[i], x.t[j]) }
func (x customSort) Swap(i, j int)      { x.t[i], x.t[j] = x.t[j], x.t[i] }
```


对于每一种切片元素类型和每一种我们需要的排序函数，我们都声明了一个新的`sort.Interface`实现。  如你所见，`Len`和`Swap`方法对于所有切片类型都有相同的定义。  在下一个示例中，具体类型`customSort`将一个切片与一个函数结合起来，使我们仅通过编写比较函数就能定义新的排序顺序。  顺便提一下，实现`sort.Interface`的具体类型并不总是切片；`customSort`是一个结构体类型。 
```go
type customSort struct {
    t    []*Track
    less func(x, y *Track) bool
}

func (x customSort) Len() int           { return len(x.t) }
func (x customSort) Less(i, j int) bool { return x.less(x.t[i], x.t[j]) }
func (x customSort) Swap(i, j int)      { x.t[i], x.t[j] = x.t[j], x.t[i] }
```


Let's define a multi-tier ordering function whose primary sort key is the Title, whose secondary key is the Year, and whose tertiary key is the running time, Length.  Here's the call to Sort using an anonymous ordering function: 
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

And here's the result. Notice that the tie between the two tracks titled "Go" is broken in favor of the older one. 
```text
Title       Artist          Album              Year  Length
-----       ------          -----              ----  ------
Go          Moby            Moby               1992  3m37s
Go          Delilah         From the Roots Up  2012  3m38s
Go Ahead    Alicia Keys     As I Am            2007  4m36s
Ready 2 Go  Martin Solveig  Smash              2011  4m24s
```


让我们定义一个多级排序函数，其主排序键是`Title`，次排序键是`Year`，三级排序键是播放时长`Length`。  以下是使用匿名排序函数调用`Sort`的示例： 
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

结果如下。注意，两个标题为 "Go" 的曲目之间的平局被打破，优先选择了年代较早的那一个。 
```text
Title       Artist          Album              Year  Length
-----       ------          -----              ----  ------
Go          Moby            Moby               1992  3m37s
Go          Delilah         From the Roots Up  2012  3m38s
Go Ahead    Alicia Keys     As I Am            2007  4m36s
Ready 2 Go  Martin Solveig  Smash              2011  4m24s
```


Although sorting a sequence of length n requires $O(n \log n)$ comparison operations, testing whether a sequence is already sorted requires at most $n-1$ comparisons.  The IsSorted function from the sort package checks this for us.  Like sort.Sort, it abstracts both the sequence and its ordering function using sort.Interface, but it never calls the Swap method: This code demonstrates the IntsAreSorted and Ints functions and the IntSlice type: 
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


虽然对长度为 n 的序列进行排序需要 $O(n \log n)$ 次比较操作，但测试一个序列是否已经排序最多需要 $n-1$ 次比较。  `sort`包中的`IsSorted`函数为我们执行此检查。  与`sort.Sort`类似，它使用`sort.Interface`来抽象序列及其排序函数，但它从不调用`Swap`方法：以下代码演示了`IntsAreSorted`和`Ints`函数以及`IntSlice`类型： 
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


For convenience, the sort package provides versions of its functions and types specialized for []int, []string, and []float64 using their natural orderings.  For other types, such as []int64 or []uint, we're on our own, though the path is short. 

为方便起见，`sort`包提供了针对`[]int`、`[]string`和`[]float64`类型的专用版本函数和类型，它们使用这些类型的自然排序。  对于其他类型，如`[]int64`或`[]uint`，我们需要自行实现，尽管过程不长。 

Exercise 7.8: Many GUIs provide a table widget with a stateful multi-tier sort: the primary sort key is the most recently clicked column head, the secondary sort key is the second-most recently clicked column head, and so on.  Define an implementation of sort.Interface for use by such a table.  Compare that approach with repeated sorting using sort.Stable. 

练习 7.8：许多GUI提供一个带有状态的多级排序表格控件：主排序键是最近点击的列标题，次排序键是次近点击的列标题，以此类推。  为此类表格定义一个`sort.Interface`的实现。  将该方法与使用`sort.Stable`重复排序的方法进行比较。 

Exercise 7.9: Use the html/template package (§4.6) to replace printTracks with a function that displays the tracks as an HTML table.  Use the solution to the previous exercise to arrange that each click on a column head makes an HTTP request to sort the table. 

练习 7.9：使用`html/template`包（§4.6）替换`printTracks`函数，使其将曲目显示为HTML表格。  使用上一个练习的解决方案，实现每次点击列标题时都向服务器发送HTTP请求以对表格进行排序。 

Exercise 7.10: The sort.Interface type can be adapted to other uses.  Write a function IsPalindrome(s sort.Interface) bool that reports whether the sequence s is a palindrome, in other words, reversing the sequence would not change it.  Assume that the elements at indices i and j are equal if !s.Less(i, j) && !s.Less(j, i). 

练习 7.10：`sort.Interface`类型可以适用于其他用途。  编写一个函数`IsPalindrome(s sort.Interface) bool`，用于报告序列s是否为回文序列，换言之，反转序列不会改变它。  假设当`!s.Less(i, j) && !s.Less(j, i)`时，索引i和j处的元素相等。 

### 我的解读/核心要点总结
本节（7.6）详细介绍了Go语言标准库中 `sort` 包的用法，其核心在于 `sort.Interface` 接口。
* **接口驱动排序**：Go的排序机制不是将排序算法绑定到特定的数据结构上，而是定义了一个通用的 `sort.Interface`，它包含 `Len()`、`Less(i, j int) bool` 和 `Swap(i, j int)` 三个方法。  任何类型只要实现了这三个方法，就可以使用通用的 `sort.Sort()` 函数进行排序。  这种设计体现了Go接口的强大之处：通过行为（方法）来定义能力，而不是通过继承或类型本身。
* **灵活性与可扩展性**：
    * 用户可以为任何自定义数据类型（通常是切片，但也可以是结构体等）实现 `sort.Interface`，从而实现自定义排序逻辑。 
    * 通过实现不同的 `Less` 方法，可以轻松改变排序规则（例如，按不同字段排序、升序/降序、忽略大小写等）。 
* **辅助工具**：
    * `sort` 包提供了一些预定义的辅助类型（如 `StringSlice`、`IntSlice`）和函数（如 `Strings()`、`Ints()`）来简化常见类型的排序。 
    * `sort.Reverse` 是一个巧妙的适配器（adapter），它通过包装一个已有的 `sort.Interface` 并反转其 `Less` 方法的逻辑，来实现反向排序，而无需重新实现整个接口。  这展示了组合优于继承的思想。 
* **自定义复杂排序**：通过 `customSort` 结构体的例子，展示了如何将排序逻辑（`less` 函数）与数据分离，使得可以动态提供比较函数，实现多级排序等复杂场景。 
* **性能考量**：在排序大数据集时，如果元素本身较大（如 `Track` 结构体），排序指针切片（`[]*Track`）通常比排序结构体切片（`[]Track`）更高效，因为指针的交换成本远低于整个结构体的交换成本。 
* **不仅仅是排序**：`sort.Interface` 的抽象概念不仅限于排序，还可以用于其他需要比较和交换元素序列的场景，如练习7.10中判断回文序列的例子。 

总的来说，`sort.Interface` 是Go语言接口强大灵活性和组合思想的一个典范，它使得排序功能既通用又高度可定制。

## 7.7 The http.Handler Interface

In Chapter 1, we saw a glimpse of how to use the net/http package to implement web clients (§1.5) and servers (§1.7).  In this section, we'll look more closely at the server API, whose foundation is the http.Handler interface: 
_net/http_
```go
package http

type Handler interface {
    ServeHTTP(w ResponseWriter, r *Request)
}

func ListenAndServe(address string, h Handler) error
```

The ListenAndServe function requires a server address, such as "localhost:8000", and an instance of the Handler interface to which all requests should be dispatched.  It runs forever, or until the server fails (or fails to start) with an error, always non-nil, which it returns. 

在第一章中，我们简要介绍了如何使用`net/http`包来实现Web客户端（§1.5）和服务器（§1.7）。  在本节中，我们将更深入地探讨服务器API，其核心是`http.Handler`接口： 
_net/http_
```go
package http

type Handler interface {
    ServeHTTP(w ResponseWriter, r *Request)
}

func ListenAndServe(address string, h Handler) error
```

`ListenAndServe`函数需要一个服务器地址（例如`"localhost:8000"`）和一个`Handler`接口的实例，所有请求都将分派给该实例。  它会持续运行，直到服务器发生故障（或启动失败）并返回一个错误（该错误总是非`nil`的）。 

Imagine an e-commerce site with a database mapping the items for sale to their prices in dollars.  The program below shows the simplest imaginable implementation.  It models the inventory as a map type, database, to which we've attached a ServeHTTP method so that it satisfies the http.Handler interface.  The handler ranges over the map and prints the items. 
_gopl.io/ch7/http1_
```go
func main() {
    db := database{"shoes": 50, "socks": 5}
    log.Fatal(http.ListenAndServe("localhost:8000", db))
}
```

```go
type dollars float32

func (d dollars) String() string { return fmt.Sprintf("$%.2f", d) }
```

```go
type database map[string]dollars

func (db database) ServeHTTP(w http.ResponseWriter, req *http.Request) {
    for item, price := range db {
        fmt.Fprintf(w, "%s: %s\n", item, price)
    }
}
```


设想一个电子商务网站，其数据库将待售商品映射到以美元为单位的价格。  下面的程序展示了最简单的可能实现。  它将库存建模为一个`map`类型`database`，我们为该类型附加了一个`ServeHTTP`方法，使其满足`http.Handler`接口。  该处理程序遍历`map`并打印商品。 
_gopl.io/ch7/http1_
```go
func main() {
    db := database{"shoes": 50, "socks": 5}
    log.Fatal(http.ListenAndServe("localhost:8000", db))
}
```

```go
type dollars float32

func (d dollars) String() string { return fmt.Sprintf("$%.2f", d) }
```

```go
type database map[string]dollars

func (db database) ServeHTTP(w http.ResponseWriter, req *http.Request) {
    for item, price := range db {
        fmt.Fprintf(w, "%s: %s\n", item, price)
    }
}
```


If we start the server,
```bash
$ go build gopl.io/ch7/http1
$ ./http1 &
```
then connect to it with the fetch program from Section 1.5 (or a web browser if you prefer), we get the following output: 
```bash
$ go build gopl.io/ch1/fetch
$ ./fetch http://localhost:8000
shoes: $50.00
socks: $5.00
```

如果我们启动服务器：
```bash
$ go build gopl.io/ch7/http1
$ ./http1 &
```
然后使用第1.5节中的`Workspace`程序（或者你喜欢的Web浏览器）连接到它，我们会得到以下输出： 
```bash
$ go build gopl.io/ch1/fetch
$ ./fetch http://localhost:8000
shoes: $50.00
socks: $5.00
```

So far, the server can only list its entire inventory and will do this for every request, regardless of URL.  A more realistic server defines multiple different URLs, each triggering a different behavior.  Let's call the existing one /list and add another one called /price that reports the price of a single item, specified as a request parameter like /price?item=socks. 
_gopl.io/ch7/http2_
```go
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


到目前为止，该服务器只能列出其全部库存，并且会对每个请求都这样做，无论URL是什么。  一个更现实的服务器会定义多个不同的URL，每个URL触发不同的行为。  让我们将现有的称为`/list`，并添加另一个名为`/price`的URL，它报告单个商品的价格，该商品通过请求参数指定，例如`/price?item=socks`。 
_gopl.io/ch7/http2_
```go
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


Now the handler decides what logic to execute based on the path component of the URL, req.URL.Path.  If the handler doesn't recognize the path, it reports an HTTP error to the client by calling w.WriteHeader(http.StatusNotFound); this must be done before writing any text to w.  (Incidentally, http.ResponseWriter is another interface. It augments io.Writer with methods for sending HTTP response headers.)  Equivalently, we could use the http.Error utility function: 
```go
msg := fmt.Sprintf("no such page: %s\n", req.URL)
http.Error(w, msg, http.StatusNotFound) // 404
```

The case for /price calls the URL's Query method to parse the HTTP request parameters as a map, or more precisely, a multimap of type url.Values (§6.2.1) from the net/url package.  It then finds the first item parameter and looks up its price.  If the item wasn't found, it reports an error. 

现在，处理程序根据URL的路径组件`req.URL.Path`来决定执行什么逻辑。  如果处理程序无法识别该路径，它会通过调用`w.WriteHeader(http.StatusNotFound)`向客户端报告一个HTTP错误；这必须在向`w`写入任何文本之前完成。  （顺便提一下，`http.ResponseWriter`是另一个接口。它通过提供发送HTTP响应头的方法来增强`io.Writer`。）  等效地，我们可以使用`http.Error`实用函数： 
```go
msg := fmt.Sprintf("no such page: %s\n", req.URL)
http.Error(w, msg, http.StatusNotFound) // 404
```

`/price`的处理逻辑调用URL的`Query`方法，将HTTP请求参数解析为一个`map`，或者更准确地说，是来自`net/url`包的`url.Values`类型（§6.2.1）的多值`map`。  然后它查找第一个名为`item`的参数并检索其价格。  如果未找到该商品，则报告错误。 

Here's an example session with the new server: 
```bash
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

以下是与新服务器交互的一个示例会话： 
```bash
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

Obviously we could keep adding cases to ServeHTTP, but in a realistic application, it's convenient to define the logic for each case in a separate function or method.  Furthermore, related URLs may need similar logic; several image files may have URLs of the form /images/*.png, for instance.  For these reasons, net/http provides ServeMux, a request multiplexer, to simplify the association between URLs and handlers.  A ServeMux aggregates a collection of http.Handlers into a single http.Handler.  Again, we see that different types satisfying the same interface are substitutable: the web server can dispatch requests to any http.Handler, regardless of which concrete type is behind it.  For a more complex application, several ServeMuxes may be composed to handle more intricate dispatching requirements.  Go doesn't have a canonical web framework analogous to Ruby's Rails or Python's Django.  This is not to say that such frameworks don't exist, but the building blocks in Go's standard library are flexible enough that frameworks are often unnecessary.  Furthermore, although frameworks are convenient in the early phases of a project, their additional complexity can make longer-term maintenance harder. 

显然，我们可以继续向`ServeHTTP`添加`case`分支，但在实际应用中，将每个`case`的逻辑定义在单独的函数或方法中更为方便。  此外，相关的URL可能需要相似的逻辑；例如，多个图片文件可能具有形如`/images/*.png`的URL。  出于这些原因，`net/http`包提供了一个`ServeMux`，即请求多路复用器（request multiplexer），以简化URL与处理程序之间的关联。  `ServeMux`将一组`http.Handler`聚合为一个单一的`http.Handler`。  再次，我们看到满足相同接口的不同类型是可替换的：Web服务器可以将请求分派给任何`http.Handler`，无论其背后的具体类型是什么。  对于更复杂的应用程序，可以组合多个`ServeMux`来处理更复杂的调度需求。  Go没有一个像Ruby的Rails或Python的Django那样堪称典范（canonical）的Web框架。  这并非说这类框架不存在，而是Go标准库中的构建模块足够灵活，以至于框架通常并非必需。  此外，虽然框架在项目初期很方便，但它们额外的复杂性可能会使长期维护更加困难。 

In the program below, we create a ServeMux and use it to associate the URLs with the corresponding handlers for the /list and /price operations, which have been split into separate methods.  We then use the ServeMux as the main handler in the call to ListenAndServe. 
_gopl.io/ch7/http3_
```go
func main() {
    db := database{"shoes": 50, "socks": 5}
    mux := http.NewServeMux()
    mux.Handle("/list", http.HandlerFunc(db.list))
    mux.Handle("/price", http.HandlerFunc(db.price))
    log.Fatal(http.ListenAndServe("localhost:8000", mux))
}

// type database map[string]dollars // already defined

func (db database) list(w http.ResponseWriter, req *http.Request) {
    for item, price := range db {
        fmt.Fprintf(w, "%s: %s\n", item, price)
    }
}

func (db database) price(w http.ResponseWriter, req *http.Request) {
    item := req.URL.Query().Get("item")
    price, ok := db[item]
    if !ok {
        w.WriteHeader(http.StatusNotFound) // 404
        fmt.Fprintf(w, "no such item: %q\n", item)
        return
    }
    fmt.Fprintf(w, "%s\n", price)
}
```


在下面的程序中，我们创建了一个`ServeMux`，并用它来关联`/list`和`/price`操作的URL与相应的处理程序，这些操作已被拆分为单独的方法。  然后，我们将`ServeMux`作为主处理程序用在对`ListenAndServe`的调用中。 
_gopl.io/ch7/http3_
```go
func main() {
    db := database{"shoes": 50, "socks": 5}
    mux := http.NewServeMux()
    mux.Handle("/list", http.HandlerFunc(db.list))
    mux.Handle("/price", http.HandlerFunc(db.price))
    log.Fatal(http.ListenAndServe("localhost:8000", mux))
}

// type database map[string]dollars // 已定义

func (db database) list(w http.ResponseWriter, req *http.Request) {
    for item, price := range db {
        fmt.Fprintf(w, "%s: %s\n", item, price)
    }
}

func (db database) price(w http.ResponseWriter, req *http.Request) {
    item := req.URL.Query().Get("item")
    price, ok := db[item]
    if !ok {
        w.WriteHeader(http.StatusNotFound) // 404
        fmt.Fprintf(w, "no such item: %q\n", item)
        return
    }
    fmt.Fprintf(w, "%s\n", price)
}
```


Let's focus on the two calls to mux.Handle that register the handlers.  In the first one, db.list is a method value (§6.4), that is, a value of type `func(w http.ResponseWriter, req *http.Request)` that, when called, invokes the database.list method with the receiver value db.  So db.list is a function that implements handler-like behavior, but since it has no methods, it doesn't satisfy the http.Handler interface and can't be passed directly to mux.Handle. 
The expression http.HandlerFunc(db.list) is a conversion, not a function call, since http.HandlerFunc is a type.  It has the following definition: 
_net/http_
```go
package http

type HandlerFunc func(w ResponseWriter, r *Request)

// ServeHTTP calls f(w, r).
func (f HandlerFunc) ServeHTTP(w ResponseWriter, r *Request) {
    f(w, r)
}
```


让我们关注注册处理程序的两个`mux.Handle`调用。  在第一个调用中，`db.list`是一个**方法值**（method value）（§6.4），即一个类型为`func(w http.ResponseWriter, req *http.Request)`的值，当它被调用时，会以接收者值`db`来调用`database.list`方法。  因此，`db.list`是一个实现了类似处理程序行为的函数，但由于它没有方法，它不满足`http.Handler`接口，也不能直接传递给`mux.Handle`。 
表达式`http.HandlerFunc(db.list)`是一个类型转换，而不是函数调用，因为`http.HandlerFunc`是一个类型。  它具有以下定义： 
_net/http_
```go
package http

type HandlerFunc func(w ResponseWriter, r *Request)

// ServeHTTP calls f(w, r).
func (f HandlerFunc) ServeHTTP(w ResponseWriter, r *Request) {
    f(w, r)
}
```


HandlerFunc demonstrates some unusual features of Go's interface mechanism.  It is a function type that has methods and satisfies an interface, http.Handler.  The behavior of its ServeHTTP method is to call the underlying function.  HandlerFunc is thus an adapter that lets a function value satisfy an interface, where the function and the interface's sole method have the same signature.  In effect, this trick lets a single type such as database satisfy the http.Handler interface several different ways: once through its list method, once through its price method, and so on. 
Because registering a handler this way is so common, ServeMux has a convenience method called HandleFunc that does it for us, so we can simplify the handler registration code to this: 
_gopl.io/ch7/http3a_
```go
mux.HandleFunc("/list", db.list)
mux.HandleFunc("/price", db.price)
```


`HandlerFunc`展示了Go接口机制的一些不寻常特性。  它是一个函数类型，却拥有方法并满足一个接口（`http.Handler`）。  其`ServeHTTP`方法的行为是调用其底层的函数。  因此，`HandlerFunc`是一个**适配器**（adapter），它允许一个函数值满足一个接口，其中该函数和接口的唯一方法具有相同的签名。  实际上，这种技巧使得像`database`这样的单一类型可以通过多种不同方式满足`http.Handler`接口：一次通过其`list`方法，一次通过其`price`方法，等等。 
由于以这种方式注册处理程序非常普遍，`ServeMux`有一个名为`HandleFunc`的便捷方法可以为我们完成此操作，因此我们可以将处理程序注册代码简化为： 
_gopl.io/ch7/http3a_
```go
mux.HandleFunc("/list", db.list)
mux.HandleFunc("/price", db.price)
```


It's easy to see from the code above how one would construct a program in which there are two different web servers, listening on different ports, defining different URLs, and dispatching to different handlers.  We would just construct another ServeMux and make another call to ListenAndServe, perhaps concurrently. 
But in most programs, one web server is plenty.  Also, it's typical to define HTTP handlers across many files of an application, and it would be a nuisance if they all had to be explicitly registered with the application's ServeMux instance.  So, for convenience, net/http provides a global ServeMux instance called DefaultServeMux and package-level functions called http.Handle and http.HandleFunc.  To use DefaultServeMux as the server's main handler, we needn't pass it to ListenAndServe; nil will do.  The server's main function can then be simplified to: 
_gopl.io/ch7/http4_
```go
func main() {
    db := database{"shoes": 50, "socks": 5}
    http.HandleFunc("/list", db.list)
    http.HandleFunc("/price", db.price)
    log.Fatal(http.ListenAndServe("localhost:8000", nil))
}
```


从上面的代码很容易看出，如何构建一个程序，其中包含两个不同的Web服务器，它们侦听不同的端口，定义不同的URL，并分派到不同的处理程序。  我们只需构建另一个`ServeMux`并再次调用`ListenAndServe`，或许可以并发执行。 
但在大多数程序中，一个Web服务器就足够了。  此外，通常会在应用程序的许多文件中定义HTTP处理程序，如果它们都必须显式注册到应用程序的`ServeMux`实例，那将是一件麻烦事。  因此，为方便起见，`net/http`提供了一个名为`DefaultServeMux`的全局`ServeMux`实例，以及包级别的函数`http.Handle`和`http.HandleFunc`。  要将`DefaultServeMux`用作服务器的主处理程序，我们无需将其传递给`ListenAndServe`；传递`nil`即可。  服务器的`main`函数可以简化为： 
_gopl.io/ch7/http4_
```go
func main() {
    db := database{"shoes": 50, "socks": 5}
    http.HandleFunc("/list", db.list)
    http.HandleFunc("/price", db.price)
    log.Fatal(http.ListenAndServe("localhost:8000", nil))
}
```


Finally, an important reminder: as we mentioned in Section 1.7, the web server invokes each handler in a new goroutine, so handlers must take precautions such as locking when accessing variables that other goroutines, including other requests to the same handler, may be accessing.  We'll talk about concurrency in the next two chapters. 

最后，一个重要的提醒：正如我们在§1.7中提到的，Web服务器在新的goroutine中调用每个处理程序，因此处理程序在访问可能被其他goroutine（包括对同一处理程序的其他请求）访问的变量时，必须采取诸如加锁之类的预防措施。  我们将在接下来的两章中讨论并发。 

Exercise 7.11: Add additional handlers so that clients can create, read, update, and delete database entries.  For example, a request of the form /update?item=socks&price=6 will update the price of an item in the inventory and report an error if the item does not exist or if the price is invalid.  (Warning: this change introduces concurrent variable updates.) 

练习 7.11：添加额外的处理程序，使客户端能够创建、读取、更新和删除数据库条目。  例如，形如/update?item=socks&price=6的请求将更新库存中商品的售价，如果商品不存在或价格无效，则报告错误。  （警告：此更改引入了并发变量更新。） 

Exercise 7.12: Change the handler for /list to print its output as an HTML table, not text.  You may find the html/template package (§4.6) useful. 

练习 7.12：修改/list的处理程序，使其输出打印为HTML表格，而非文本。  你可能会发现`html/template`包（§4.6）很有用。 

### 我的解读/核心要点总结
本节（7.7）重点介绍了 `net/http` 包中的核心接口 `http.Handler` 及其在构建 Web 服务器中的应用。
* **`http.Handler` 接口**：
    * 定义了处理 HTTP 请求的契约，只有一个方法：`ServeHTTP(ResponseWriter, *Request)`。 
    * 任何实现了此接口的类型都可以处理 HTTP 请求。 
* **`ListenAndServe` 函数**：
    * 启动 HTTP 服务器，监听指定地址，并将接收到的请求分派给提供的 `http.Handler`。 
* **基本 Handler 实现**：
    * 可以直接为一个自定义类型（如 `database` map）实现 `ServeHTTP` 方法，使其成为一个 `Handler`。 
    * 在 `ServeHTTP` 方法内部，可以通过 `req.URL.Path` 来区分不同的请求路径，并执行相应的逻辑（路由）。 
* **`http.ServeMux` 请求多路复用器**：
    * 当处理多个 URL 路由时，直接在单个 `ServeHTTP` 方法中使用 `switch` 会变得复杂。  `ServeMux` (或 `http.NewServeMux()`) 提供了一种更结构化的方式来注册不同 URL 路径对应的 `Handler`。 
    * `mux.Handle(path, handler)` 用于注册。 
* **`http.HandlerFunc` 适配器**：
    * 这是一个非常巧妙的设计：`HandlerFunc` 本身是一个函数类型 `func(ResponseWriter, *Request)`，但它也实现了 `http.Handler` 接口。  它的 `ServeHTTP` 方法就是直接调用其自身（该函数）。 
    * 这使得普通的函数（只要签名匹配）可以通过类型转换 `http.HandlerFunc(myFunction)` 轻松地用作 `http.Handler`，而无需显式定义一个新类型并为其实现 `ServeHTTP` 方法。  这常用于将一个类型的方法（如 `db.list`）适配为 `Handler`。 
    * `mux.HandleFunc(path, function)` 是 `mux.Handle(path, http.HandlerFunc(function))` 的便捷写法。 
* **`DefaultServeMux` 和包级函数**：
    * `net/http` 包提供了一个全局的 `DefaultServeMux` 实例。 
    * 包级别的 `http.Handle()` 和 `http.HandleFunc()` 函数默认会将处理器注册到 `DefaultServeMux`。 
    * 调用 `http.ListenAndServe(address, nil)` 时，如果第二个参数为 `nil`，则会使用 `DefaultServeMux` 作为处理器。  这简化了在小型应用或当所有处理器都在一个中心位置注册时的代码。 
* **并发**：
    * 一个关键点是，Web 服务器会为每个接收到的请求启动一个新的 goroutine 来执行对应的 `Handler`。  这意味着 `Handler` 的实现必须是并发安全的，尤其是在访问共享数据时（如示例中的 `db` map），需要适当的同步机制（如互斥锁），这一点将在后续章节讨论。 

本节展示了Go标准库如何通过简洁的接口和巧妙的适配器模式，提供强大而灵活的Web服务构建块，鼓励组合和自定义，而非依赖庞大的框架。 