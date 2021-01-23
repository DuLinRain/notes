# Go泛型Part 01: 基本语法
泛型是很多语言（如C++, TypeScript）等都有的语法，但是在当前Go语言中是还没有的。本文将介绍有关Go泛型的草案相关内容。

### 介绍

在关于GO泛型的该系列文章中，我将通过代码呈现和讲述Go泛型草案的不同方面。https://go.googlesource.com/proposal/+/refs/heads/master/design/go2draft-type-parameters.md
。也会提供go2go(https://go2goplay.golang.org/) 的地址这样可以方便的在线尝试这些例子。这些例子也会在这个仓库中https://github.com/ardanlabs/gotraining/tree/master/topics/go/generics，
但是仓库中的例子可能会发生变化。

### 具体例子

假设你想写一个可以打印int类型slice所有元素的方法，你可能很容易就像这样写出来了：

**Listing 1**
    
    
    13 func printNumbers(numbers []int) {
    14     fmt.Print("Numbers: ")
    15     for _, num := range numbers {
    16         fmt.Print(num, " ")
    17     }
    18     fmt.Print("\n")
    19 }
    
    Main:
    numbers := []int{1, 2, 3}
    printNumbers(numbers)
    
    Output:
    Numbers: 1 2 3

**Listing 1**中的代码展示了使用5行代码打印出一个存储了整型slice的实现。我敢保证，大多数Go开发者都可以很轻松的读懂和
维护这段代码。

但是，假设要打印的是string类型的slice的话，又该怎么写呢？

**Listing 2**

    21 func printStrings(strings []string) {
    22     fmt.Print("Strings: ")
    23     for _, str := range strings {
    24         fmt.Print(str, " ")
    25     }
    26     fmt.Print("\n")
    27 }
    
    Main:
    strings := []string{"A", "B", "C"}
    printStrings(strings)
    
    Output:
    Strings: A B C
    
**Listing 2** 中的代码展示了使用5行代码打印出一个存储了string类型slice的实现。可以看到和
**Listing 1** 中的实现非常类似，唯一的差别就是<code>printStrings</code>函数接收的是string类型的slice
而不是int型的。以及第22行，打印的文案是Strings 而不是 Numbers。所以，结论就是，如果我想
接受string类型的slice作为参数的话，我不得不写一个新的函数。

### 使用空interface和类型断言解决
那要是想只是用一个函数来同时处理这2种类型的话该咋办呢？一种方案是采用空interface结合类型断言来实现。

**Listing 3**

    35 func printAssert(v interface{}) {// 空interface,即interface{}
    36     fmt.Print("Assert: ")
    37     switch list := v.(type) { // 断言
    38     case []int:
    39         for _, num := range list {
    40             fmt.Print(num, " ")
    41         }
    42     case []string:
    43         for _, str := range list {
    44             fmt.Print(str, " ")
    45         }
    46     }
    47     fmt.Print("\n")
    48 }
    
    Main:
    numbers := []int{1, 2, 3}
    strings := []string{"A", "B", "C"}
    printAssert(numbers)
    printAssert(strings)
    
    Output:
    Assert: 1 2 3
    Assert: A B C

**Listing 3** 中的代码展示了只使用一个函数就可以同时支持参数是string和int类型的slice，它的
原理是使用空interface和类型断言来实现的。

由于interface{}的特殊性，任何类型都可以传递给这个函数。在37行，使用了一个.(type)进行类型断言，并且结合switch语法
来判断传入进来的是一个int型slice还是string型slice。然后根据类型创建不同的case分支处理。

由于这里面针对每种类型都会创建一个case分支，所以这种实现并不是我们所理解的泛型。它目前只能处理int和string2中类型型的slice，
本质上就是把**Listing 1** 和 **Listing 2** 中的示例函数替换成了case语句。如果你想继续
支持float64类型的slice的话，你还得继续加case分支。

### 反射
那要是想只是用一个函数来同时处理任意类型slice的话该咋办呢？目前只能通过空interface和反射来实现。

**Listing 4**

    56 func printReflect(v interface{}) {
    57     fmt.Print("Reflect: ")
    58     val := reflect.ValueOf(v)// 反射
    59     if val.Kind() != reflect.Slice {
    60         return
    61     }
    62     for i := 0; i < val.Len(); i++ {
    63         fmt.Print(val.Index(i).Interface(), " ")
    64     }
    65     fmt.Print("\n")
    66 }
    
    Main:
    numbers := []int{1, 2, 3}
    strings := []string{"A", "B", "C"}
    floats := []float64{1.5, 2.9, 3.1}
    printReflect(numbers)
    printReflect(strings)
    printReflect(floats)
    
    Output:
    Reflect: 1 2 3
    Reflect: A B C
    Reflect: 1.5 2.9 3.1


**Listing 4** 中的代码展示了使用空interface和反射实现的支持任意类型slice参数的函数。通样，
这个函数的参数类似是interface，这样它可以接受任意类型的slice作为参数。并且，多亏reflect包，
使得可以通过对参数进行线性遍历而无需管参数具体是哪种类型的slice(如62，63行所示)。

但是这个实现并不完美，因为如果调用者没有传递slice类型，那么第60行将返回error值，这与最初
设计的API相悖。这个实现比上面的几个实现稍微复杂一些，你需要有关于reflect包的一些知识。最终，总的
来说，这可以称的上是一个泛型函数，因为它支持了打印任意类型的slice。所以，你可以这么认为，GO已经有泛型了。

但是，你可能会问自己，如果这个打印函数可以支持一种通用的参数类型，那么我们就可以复用最开始例子中的简洁的
实现，避免使用复杂的反射，这样是不是更有价值呢？

这其实就是真正的泛型，如果你使用过其它语言，你可能早就知道应该怎么样写了。

### 泛型
泛型可以让你写出满足上述要求的接受任意类型slice的函数，并且保持和最初的代码一样简洁并且不依赖于反射：

**Listing 5** 

    75 func printGeneric[type T](slice []T) {  |   13 func printNumbers(numbers []int) {
    76     fmt.Print("Generic: ")              |   14     fmt.Print("Numbers: ")
    77     for _, v := range slice {           |   15     for _, num := range numbers {
    78         fmt.Print(v, " ")               |   16         fmt.Print(num, " ")
    79     }                                   |   17     }
    80     fmt.Print("\n")                     |   18     fmt.Print("\n")
    81 }                                       |   19 }

**Listing 5** 的左边是采用草案中泛型的语法实现的支持任意类型slice的版本，右边是最初的只支持int类型slice的版本，
可以看出两者基本上一模一样，除了76行和14行文案的一丢丢差异。

### 理解泛型语法

### 泛型列表
在当前草案中，一种方法是使用一组方括号定义一组通用类型标识符。

>注意：也可以使用括号代替方括号，但是这里我会使用方括号，因为我觉得它更容易阅读。
>


**Listing 8**

    75 func printGeneric[type T](slice []T) {
    76     fmt.Print("Generic: ")
    77     for _, v := range slice {
    78         fmt.Print(v, " ")
    79     }
    80     fmt.Print("\n")
    81 }
    
**Listing 8**中代码展示了一个泛型函数，这里使用方括号定义了泛型标识符T，告诉编译器在编译之前，不会有这样的类型会声明，编译器需要在编译的时候弄清楚
具体对应的类型。

> 注意：你也可以在方括号中使用多个类型标识符，如[type T, S, R]，在本例中只使用了一个。

您可以将这些类型标识符命名成任何东西，只要您觉得对函数的可读性有所帮助即可。在这种情况下，我使用大写字母T来描述将传递某种类型T（在编译时确定）的slice。


### 调用泛型函数
那怎么调用这个泛型函数呢？

**Listing 9**

    numbers := []int{1, 2, 3}
    printGeneric[int](numbers)
    
    strings := []string{"A", "B", "C"}
    printGeneric[string](strings)
    
    floats := []float64{1.7, 2.2, 3.14}
    printGeneric[float64](floats)

### 尖括号


### 总结