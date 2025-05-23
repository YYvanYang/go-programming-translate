**7.8. The error Interface**

Since the beginning of this book, we've been using and creating values of the mysterious predeclared error type without explaining what it really is. In fact, it's just an interface type with a single method that returns an error message:

从本书一开始，我们就一直在使用和创建预声明的（predeclared）`error`类型的值，但并未解释它究竟是什么。实际上，它只是一个接口类型，只有一个返回错误消息的方法：

Go

```
type error interface {
    Error() string
}
```

Go

```
type error interface {
    Error() string
}
```

The simplest way to create an error is by calling errors.New, which returns a new error for a given error message. The entire errors package is only four lines long:

创建错误的最简单方法是调用`errors.New`，它为给定的错误消息返回一个新的错误。整个`errors`包仅有四行代码：

Go

```
package errors

func New(text string) error { return &errorString{text} }

type errorString struct { text string }

func (e *errorString) Error() string { return e.text }
```

Go

```
package errors

func New(text string) error { return &errorString{text} }

type errorString struct{ text string }

func (e *errorString) Error() string { return e.text }
```

The underlying type of errorString is a struct, not a string, to protect its representation from inadvertent (or premeditated) updates. And the reason that the pointer type *errorString, not errorString alone, satisfies the error interface is so that every call to New allocates a distinct error instance that is equal to no other. We would not want a distinguished error such as io.EOF to compare equal to one that merely happened to have the same message.

`errorString`的底层类型是一个结构体，而不是字符串，这是为了保护其表示免遭无意（或有意）的更新。指针类型`*errorString`（而非`errorString`本身）满足`error`接口的原因在于，这样每次调用`New`都会分配一个与其他任何错误实例都不相等的、独特的（distinct）错误实例。我们不希望像`io.EOF`这样的特定错误（distinguished error）与一个仅仅碰巧具有相同消息的错误比较时相等。

Go

```
fmt.Println(errors.New("EOF") == errors.New("EOF")) // "false"
```

Go

```
fmt.Println(errors.New("EOF") == errors.New("EOF")) // "false"
```

Calls to errors.New are relatively infrequent because there's a convenient wrapper function, fmt.Errorf, that does string formatting too. We used it several times in Chapter 5.

对`errors.New`的调用相对不频繁，因为有一个方便的包装函数`fmt.Errorf`，它同时也进行字符串格式化。我们在第5章中多次使用它。

Go

```
package fmt

import "errors"

func Errorf(format string, args...interface{}) error {
    return errors.New(Sprintf(format, args...))
}
```

Go

```
package fmt

import "errors"

func Errorf(format string, args ...interface{}) error {
    return errors.New(Sprintf(format, args...))
}
```

Although *errorString may be the simplest type of error, it is far from the only one. For example, the syscall package provides Go's low-level system call API. On many platforms, it defines a numeric type Errno that satisfies error, and on Unix platforms, Errno's Error method does a lookup in a table of strings, as shown below:

虽然`*errorString`可能是最简单的错误类型，但它远非唯一的错误类型。例如，`syscall`包提供了Go的底层系统调用API。在许多平台上，它定义了一个满足`error`接口的数字类型`Errno`；在Unix平台上，`Errno`的`Error`方法会在一个字符串表中查找错误描述，如下所示：

Go

```
package syscall

type Errno uintptr // operating system error code
```

Go

```
package syscall

type Errno uintptr // operating system error code
```

Go

```
var errors = [...]string{
    1: "operation not permitted", // EPERM
    2: "no such file or directory", // ENOENT
    3: "no such process", // ESRCH
// ...
}

func (e Errno) Error() string {
    if 0 <= int(e) && int(e) < len(errors) {
        return errors[e]
    }
    return fmt.Sprintf("errno %d", e)
}
```

Go

```
var errors = [...]string{
    1: "operation not permitted", // EPERM
    2: "no such file or directory", // ENOENT
    3: "no such process", // ESRCH
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

以下语句创建一个接口值，其中包含`Errno`值`2`，表示POSIX `ENOENT`（文件不存在）条件：

Go

```
var err error = syscall.Errno(2)
fmt.Println(err.Error()) // "no such file or directory"
fmt.Println(err) // "no such file or directory"
```

Go

```
var err error = syscall.Errno(2)
fmt.Println(err.Error()) // "no such file or directory"
fmt.Println(err)         // "no such file or directory"
```

The value of err is shown graphically in Figure 7.6.

`err`的值如图 7.6 所示。

```
err
type          value
syscall.Errno  2
```

_图 7.6. 一个持有 `syscall.Errno` 整数的接口值。_

```
err
类型 (type)      值 (value)
syscall.Errno    2
```

_图 7.6. 一个持有 `syscall.Errno` 整数的接口值。_

Errno is an efficient representation of system call errors drawn from a finite set, and it satisfies the standard error interface. We'll see other types that satisfy this interface in Section 7.11.

`Errno`是从一个有限集合中提取的系统调用错误的有效表示，并且它满足标准的`error`接口。我们将在§7.11中看到其他满足此接口的类型。

---

**我的解读 (7.8)**

本节介绍了 Go 语言内置的 `error` 接口类型。其核心在于它的简洁性——只有一个 `Error() string` 方法。这种设计哲学鼓励创建自定义的错误类型，只要它们实现了这个方法，就能在 Go 的错误处理机制中无缝协作。

**核心要点总结：**

- **`error` 是接口**：任何实现了 `Error() string` 方法的类型都满足 `error` 接口。
- **`errors.New` 和 `fmt.Errorf`**：是创建简单错误值的标准方式。`errors.New` 创建的每个错误实例都是唯一的，即使错误消息相同（比较时为 `false`）。
    
- **自定义错误类型**：Go 鼓励定义具有更丰富结构或行为的自定义错误类型。`syscall.Errno` 就是一个例子，它是一个数字类型，但通过实现 `Error()` 方法来提供人类可读的错误信息。 这种方式允许错误携带更多上下文信息，而不仅仅是一个字符串。
    
- **错误值的比较**：直接比较错误实例（如 `err1 == err2`）通常是检查它们是否为同一个实例，而不是基于错误消息。对于特定的、可预期的错误（如 `io.EOF`），应该直接与这些哨兵错误值（sentinel errors）比较。

Go 的错误处理机制强调显式处理，并且通过接口的灵活性，允许开发者根据需要创建不同粒度和复杂度的错误类型。

---

**7.9. Example: Expression Evaluator**

In this section, we'll build an evaluator for simple arithmetic expressions. We'll use an interface, Expr, to represent any expression in this language. For now, this interface needs no methods, but we'll add some later.

在本节中，我们将构建一个简单算术表达式的求值器。我们将使用一个接口`Expr`来表示该语言中的任何表达式。目前，此接口不需要任何方法，但我们稍后会添加一些。

Go

```
// An Expr is an arithmetic expression.
type Expr interface{}
```

Go

```
// An Expr is an arithmetic expression.
type Expr interface{}
```

Our expression language consists of floating-point literals; the binary operators +, -, *, and /; the unary operators -x and +x; function calls pow(x,y), sin(x), and sqrt(x); variables such as x and pi; and of course parentheses and standard operator precedence. All values are of type float64. Here are some example expressions:

我们的表达式语言包含：浮点数字面量（floating-point literals）；二元运算符`+`、`-`、`*`和`/`；一元运算符`-x`和`+x`；函数调用`pow(x,y)`、`sin(x)`和`sqrt(x)`；变量（如`x`和`pi`）；当然还有括号和标准的运算符优先级。所有值均为`float64`类型。以下是一些表达式示例：

```
sqrt(A/pi)
pow(x,3)+pow(y,3)
(F-32)*5/9
```

```
sqrt(A/pi)
pow(x,3)+pow(y,3)
(F-32)*5/9
```

The five concrete types below represent particular kinds of expression. A Var represents a reference to a variable. (We'll soon see why it is exported.) A literal represents a floating-point constant. The unary and binary types represent operator expressions with one or two operands, which can be any kind of Expr. A call represents a function call; we'll restrict its fn field to pow, sin, or sqrt.

下面的五个具体类型代表特定种类的表达式。`Var`代表对变量的引用。（我们很快就会看到为什么它是导出的。）`literal`代表一个浮点常量。`unary`和`binary`类型代表带有一个或两个操作数的操作符表达式，操作数可以是任何类型的`Expr`。`call`代表一个函数调用；我们将其`fn`字段限制为`"pow"`、`"sin"`或`"sqrt"`。

gopl.io/ch7/eval

_gopl.io/ch7/eval_

Go

```
// A Var identifies a variable, e.g., x.
type Var string

// A literal is a numeric constant, e.g., 3.141.
type literal float64

// A unary represents a unary operator expression, e.g., х.
type unary struct {
    op rune // one of '+', '-'
    x Expr
}

// A binary represents a binary operator expression, e.g., x+y.
type binary struct {
    op rune // one of '+', '-', '*', '/'
    x, y Expr
}

// A call represents a function call expression, e.g., sin(x).
type call struct {
    fn string // one of "pow", "sin", "sqrt"
    args []Expr
}
```

Go

```
// A Var identifies a variable, e.g., x.
type Var string

// A literal is a numeric constant, e.g., 3.141.
type literal float64

// A unary represents a unary operator expression, e.g., х.
type unary struct {
    op rune // one of '+', '-'
    x Expr
}

// A binary represents a binary operator expression, e.g., x+y.
type binary struct {
    op rune // one of '+', '-', '*', '/'
    x, y Expr
}

// A call represents a function call expression, e.g., sin(x).
type call struct {
    fn string // one of "pow", "sin", "sqrt"
    args []Expr
}
```

To evaluate an expression containing variables, we'll need an environment that maps variable names to values:

要对包含变量的表达式求值，我们需要一个将变量名映射到值的环境（environment）：

Go

```
type Env map[Var]float64
```

Go

```
type Env map[Var]float64
```

We'll also need each kind of expression to define an Eval method that returns the expression's value in a given environment. Since every expression must provide this method, we add it to the Expr interface. The package exports only the types Expr, Env, and Var; clients can use the evaluator without access to the other expression types.

我们还需要每种表达式都定义一个`Eval`方法，该方法在给定环境中返回表达式的值。由于每个表达式都必须提供此方法，我们将其添加到`Expr`接口中。该包仅导出`Expr`、`Env`和`Var`类型；客户端无需访问其他表达式类型即可使用求值器。

Go

```
type Expr interface {
    // Eval returns the value of this Expr in the environment env.
    Eval(env Env) float64
}
```

Go

```
type Expr interface {
    // Eval returns the value of this Expr in the environment env.
    Eval(env Env) float64
}
```

The concrete Eval methods are shown below. The method for Var performs an environment lookup, which returns zero if the variable is not defined, and the method for literal simply returns the literal value.

具体的`Eval`方法如下所示。`Var`的方法执行环境查找，如果变量未定义则返回零；`literal`的方法仅返回字面量值。

Go

```
func (v Var) Eval(env Env) float64 {
    return env[v]
}

func (l literal) Eval(_ Env) float64 {
    return float64(l)
}
```

Go

```
func (v Var) Eval(env Env) float64 {
    return env[v]
}

func (l literal) Eval(_ Env) float64 {
    return float64(l)
}
```

The Eval methods for unary and binary recursively evaluate their operands, then apply the operation op to them. We don't consider divisions by zero or infinity to be errors, since they produce a result, albeit non-finite. Finally, the method for call evaluates the arguments to the pow, sin, or sqrt function, then calls the corresponding function in the math package.

`unary`和`binary`的`Eval`方法递归地对其操作数求值，然后将操作`op`应用于它们。我们不认为除以零或无穷大是错误，因为它们会产生结果，尽管是非有限的（non-finite）。最后，`call`的方法对传递给`pow`、`sin`或`sqrt`函数的参数求值，然后调用`math`包中相应的函数。

Go

```
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

Go

```
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

Several of these methods can fail. For example, a call expression could have an unknown function or the wrong number of arguments. It's also possible to construct a unary or binary expression with an invalid operator such as ! or < (although the Parse function mentioned below will never do this). These errors cause Eval to panic. Other errors, like evaluating a Var not present in the environment, merely cause Eval to return the wrong result. All of these errors could be detected by inspecting the Expr before evaluating it. That will be the job of the Check method, which we will show soon, but first let's test Eval.

其中一些方法可能会失败。例如，一个`call`表达式可能有一个未知的函数或错误的参数数量。也有可能构造一个带无效运算符（如`!`或`<`）的`unary`或`binary`表达式（尽管下面提到的`Parse`函数永远不会这样做）。这些错误会导致`Eval`引发panic。其他错误，比如对环境中不存在的`Var`求值，仅导致`Eval`返回错误的结果（对于`map`查找失败，返回零值）。所有这些错误都可以在对表达式求值之前通过检查`Expr`来检测。这将是`Check`方法的工作，我们稍后会展示它，但首先让我们测试`Eval`。

The TestEval function below is a test of the evaluator. It uses the testing package, which we'll explain in Chapter 11, but for now it's enough to know that calling t.Errorf reports an error. The function loops over a table of inputs that defines three expressions and different environments for each one. The first expression computes the radius of a circle given its area A, the second computes the sum of the cubes of two variables x and y, and the third converts a Fahrenheit temperature F to Celsius.

下面的`TestEval`函数是对求值器的一个测试。它使用了`testing`包，我们将在第11章中解释它，但现在只需知道调用`t.Errorf`会报告一个错误。该函数遍历一个输入表，该表定义了三个表达式和每个表达式的不同环境。第一个表达式根据面积`A`计算圆的半径，第二个表达式计算两个变量`x`和`y`的立方和，第三个表达式将华氏温度`F`转换为摄氏温度。

Go

```
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

Go

```
func TestEval(t *testing.T) {
    tests := []struct {
        expr string
        env  Env
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

For each entry in the table, the test parses the expression, evaluates it in the environment, and prints the result. We don't have space to show the Parse function here, but you'll find it if you download the package using go get.

对于表中的每个条目，测试会解析表达式，在环境中对其求值，并打印结果。我们没有足够的空间来展示`Parse`函数，但如果你使用`go get`下载该包，你会找到它。

The go test command (§11.1) runs a package's tests:

$ go test -v gopl.io/ch7/eval

The -v flag lets us see the printed output of the test, which is normally suppressed for a successful test like this one. Here is the output of the test's fmt.Printf statements:

go test命令（§11.1）运行包的测试：

$ go test -v gopl.io/ch7/eval

-v标志使我们能够看到测试的打印输出，对于像这样的成功测试，输出通常会被抑制。以下是测试的fmt.Printf语句的输出：

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

幸运的是，到目前为止所有的输入都是格式良好的（well formed），但我们的运气不太可能一直这么好。即使在解释型语言中，检查语法的**静态错误**（static errors）（即无需运行程序即可检测到的错误）也是常见的。通过将静态检查与动态检查分离，我们可以更快地检测到错误，并且许多检查只需执行一次，而不是每次对表达式求值时都执行。

Let's add another method to the Expr interface. The Check method checks for static errors in an expression syntax tree. We'll explain its vars parameter in a moment.

让我们向`Expr`接口添加另一个方法。`Check`方法检查表达式语法树中的静态错误。我们稍后会解释它的`vars`参数。

Go

```
type Expr interface {
    Eval(env Env) float64
    // Check reports errors in this Expr and adds its Vars to the set.
    Check(vars map[Var]bool) error
}
```

Go

```
type Expr interface {
    Eval(env Env) float64
    // Check reports errors in this Expr and adds its Vars to the set.
    Check(vars map[Var]bool) error
}
```

The concrete Check methods are shown below. Evaluation of literal and Var cannot fail, so the Check methods for these types return nil. The methods for unary and binary first check that the operator is valid, then recursively check the operands. Similarly, the method for call first checks that the function is known and has the right number of arguments, then recursively checks each argument.

具体的`Check`方法如下所示。`literal`和`Var`的求值不会失败，因此这些类型的`Check`方法返回`nil`。`unary`和`binary`的方法首先检查操作符是否有效，然后递归地检查操作数。类似地，`call`的方法首先检查函数是否已知并且具有正确数量的参数，然后递归地检查每个参数。

Go

```
func (v Var) Check(vars map[Var]bool) error {
    vars[v] = true
    return nil
}

func (l literal) Check(vars map[Var]bool) error {
    return nil
}
```

Go

```
func (v Var) Check(vars map[Var]bool) error {
    vars[v] = true
    return nil
}

func (l literal) Check(vars map[Var]bool) error {
    return nil
}
```

Go

```
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

Go

```
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

We've listed a selection of flawed inputs and the errors they elicit, in two groups. The Parse function (not shown) reports syntax errors and the Check function reports semantic errors.

我们列出了一些有缺陷的输入及其引发的错误，分为两组。`Parse`函数（未显示）报告语法错误，`Check`函数报告语义错误。

|   |   |
|---|---|
|**Input Expression**|**Error Message**|
|`x % 2`|`unexpected '%'`|
|`math.Pi`|`unexpected '.'`|
|`!true`|`unexpected '!'`|
|`"""hello"""`|`unexpected \"`|
|`log(10)`|`unknown function "log"`|
|`sqrt(1, 2)`|`call to sqrt has 2 args, want 1`|

|   |   |
|---|---|
|**输入表达式**|**错误信息**|
|`x % 2`|`unexpected '%'`|
|`math.Pi`|`unexpected '.'`|
|`!true`|`unexpected '!'`|
|`"""hello"""`|`unexpected \"`|
|`log(10)`|`unknown function "log"`|
|`sqrt(1, 2)`|`call to sqrt has 2 args, want 1`|

Check's argument, a set of Vars, accumulates the set of variable names found within the expression. Each of these variables must be present in the environment for evaluation to succeed. This set is logically the result of the call to Check, but because the method is recursive, it is more convenient for Check to populate a set passed as a parameter. The client must provide an empty set in the initial call.

`Check`的参数是一个`Var`的`map`（用作集合），用于累积表达式中找到的变量名。这些变量中的每一个都必须存在于环境中，求值才能成功。这个集合在逻辑上是`Check`调用的结果，但由于该方法是递归的，因此让`Check`填充作为参数传递的集合更为方便。客户端必须在初始调用中提供一个空集合。

In Section 3.2, we plotted a function f(x,y) that was fixed at compile time. Now that we can parse, check, and evaluate expressions in strings, we can build a web application that receives an expression at run time from the client and plots the surface of that function. We can use the vars set to check that the expression is a function of only two variables, x and y—three, actually, since we'll provide r, the radius, as a convenience. And we'll use the Check method to reject ill-formed expressions before evaluation begins so that we don't repeat those checks during the 40,000 evaluations (100×100 cells, each with four corners) of the function that follow. The parseAndCheck function combines these parsing and checking steps:

在§3.2中，我们绘制了一个在编译时固定的函数 f(x,y)。现在我们可以解析、检查和计算字符串中的表达式，我们可以构建一个Web应用程序，在运行时从客户端接收表达式并绘制该函数的曲面。我们可以使用`vars`集合来检查表达式是否仅是两个变量`x`和`y`的函数——实际上是三个，因为为了方便，我们将提供`r`（半径）作为便捷变量。并且我们将使用`Check`方法在求值开始前拒绝格式错误的（ill-formed）表达式，这样在后续对函数进行的40,000次求值（100×100个单元格，每个单元格有四个角点）中，我们就不必重复这些检查。`parseAndCheck`函数结合了这些解析和检查步骤：

gopl.io/ch7/surface

Go

```
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

_gopl.io/ch7/surface_

Go

```
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

To make this a web application, all we need is the plot function below, which has the familiar signature of an http.HandlerFunc:

要使其成为一个Web应用程序，我们只需要下面的plot函数，它具有http.HandlerFunc的熟悉签名：

Go

```
func plot(w http.ResponseWriter, r *http.Request) {
    r.ParseForm()
    expr, err := parseAndCheck(r.Form.Get("expr"))
    if err != nil {
        http.Error(w, "bad expr: "+err.Error(), http.StatusBadRequest)
        return
    }
    w.Header().Set("Content-Type", "image/svg+xml")
    surface(w, func(x, y float64) float64 {
        r_val := math.Hypot(x, y) // distance from (0,0)
        return expr.Eval(eval.Env{"x": x, "y": y, "r": r_val})
    })
}
```

Go

```
func plot(w http.ResponseWriter, r *http.Request) {
    r.ParseForm()
    expr, err := parseAndCheck(r.Form.Get("expr"))
    if err != nil {
        http.Error(w, "bad expr: "+err.Error(), http.StatusBadRequest)
        return
    }
    w.Header().Set("Content-Type", "image/svg+xml")
    surface(w, func(x, y float64) float64 {
        r_val := math.Hypot(x, y) // distance from (0,0)
        return expr.Eval(eval.Env{"x": x, "y": y, "r": r_val})
    })
}
```

Figure 7.7. The surfaces of three functions: (a) sin(−x)∗pow(1.5,−r); (b) pow(2,sin(y))∗pow(2,sin(x))/12; (c) sin(x∗y/10)/10. (Note: PDF shows sin(x∗y/1θ)/1θ, which is taken as sin(x∗y/10)/10 based on context and typical examples)

图 7.7. 三个函数的曲面图：(a) sin(−x)∗pow(1.5,−r)； (b) pow(2,sin(y))∗pow(2,sin(x))/12； (c) sin(x∗y/10)/10。

The plot function parses and checks the expression specified in the HTTP request and uses it to create an anonymous function of two variables. The anonymous function has the same signature as the fixed function f from the original surface-plotting program, but it evaluates the user-supplied expression. The environment defines x, y, and the radius r. Finally, plot calls surface, which is just the main function from gopl.io/ch3/surface, modified to take the function to plot and the output io.Writer as parameters, instead of using the fixed function f and os.Stdout. Figure 7.7 shows three surfaces produced by the program.

`plot`函数解析并检查HTTP请求中指定的表达式，并用它来创建一个接受两个变量的匿名函数。该匿名函数与原始曲面绘制程序中的固定函数`f`具有相同的签名，但它对用户提供的表达式进行求值。环境定义了`x`、`y`和半径`r`。最后，`plot`调用`surface`，它只是_gopl.io/ch3/surface_中的`main`函数，修改为接受要绘制的函数和输出`io.Writer`作为参数，而不是使用固定的函数`f`和`os.Stdout`。图7.7显示了该程序生成的三个曲面。

Exercise 7.13: Add a String method to Expr to pretty-print the syntax tree. Check that the results, when parsed again, yield an equivalent tree.

练习 7.13：为Expr添加一个String方法以美化打印语法树。检查其结果再次解析后是否能产生等效的树。

Exercise 7.14: Define a new concrete type that satisfies the Expr interface and provides a new operation such as computing the minimum value of its operands. Since the Parse function does not create instances of this new type, to use it you will need to construct a syntax tree directly (or extend the parser).

练习 7.14：定义一个新的具体类型，该类型满足Expr接口并提供一个新的操作，例如计算其操作数的最小值。由于Parse函数不创建此新类型的实例，要使用它，你需要直接构造语法树（或扩展解析器）。

Exercise 7.15: Write a program that reads a single expression from the standard input, prompts the user to provide values for any variables, then evaluates the expression in the resulting environment. Handle all errors gracefully.

练习 7.15：编写一个程序，从标准输入读取单个表达式，提示用户为任何变量提供值，然后在结果环境中计算该表达式。妥善处理所有错误。

## Exercise 7.16: Write a web-based calculator program. 练习 7.16：编写一个基于Web的计算器程序。

**我的解读 (7.9)**

本节通过构建一个算术表达式求值器，生动地展示了接口在 Go 语言中的强大作用，特别是如何利用接口来表示和处理一组异构但相关的类型（各种表达式节点）。

**核心要点总结：**

- **接口定义行为契约**：`Expr` 接口最初定义为空接口 `interface{}` ，表示它可以持有任何类型的表达式节点。随后，为其添加了 `Eval(env Env) float64` 方法 ，要求所有具体的表达式类型都能对自己进行求值。之后又添加了 `Check(vars map[Var]bool) error` 方法 ，用于静态类型检查。
    
- **具体类型实现接口**：定义了多种具体类型（`Var`, `literal`, `unary`, `binary`, `call` ） 来表示表达式的不同组成部分。每种类型都实现了 `Expr` 接口的 `Eval` 和 `Check` 方法，提供了各自特定的逻辑。
    
- **递归处理**：`Eval` 和 `Check` 方法在处理复合表达式（如 `unary`, `binary`, `call`）时，通常会递归调用其操作数（子表达式）的相应方法。 这是处理树形结构的常见模式。
    
- **环境（Environment）**：通过 `Env` 类型（`map[Var]float64`）来存储变量的值，使得表达式可以在特定上下文中求值。
- **静态检查与动态求值分离**：引入 `Check` 方法实现了对表达式的静态分析（如操作符有效性 ，函数参数数量正确性 ，变量收集 等），这有助于在实际求值（`Eval`）之前捕获错误，提高健壮性和效率。
    
- **接口的灵活性**：这个例子展示了如何定义一个抽象（`Expr`），然后通过不同的具体实现来赋予这个抽象多种形态和行为。这种设计使得求值器的核心逻辑（如解析器、遍历器）可以针对 `Expr` 接口编程，而无需关心具体的表达式类型。

此示例是接口驱动设计的一个典型应用，通过接口将系统的不同部分解耦，并允许系统易于扩展（例如，通过添加新的表达式类型或操作）。