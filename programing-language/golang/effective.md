介绍

虽然go借鉴了其它编程语言的特性，但其实go是一门全新的编程语言，它具有很多独特的特性，使得其在构建代码的时候区别于其它编程语言。java和c++代码是很难直接翻译成go代码的，但如果我们从go的角度去考虑问题，我们可以构建一个完全不一样的程序。换句话说，写出符合go风格的代码对于理解其风格和特性是非常重要的。了解go的编程规范很重要，比如命名、格式、代码结构等等，这样写出来的程序才便于其它开发人员理解。

这篇文档会给出很多技巧，让你能编写出干练、地道的go代码。



格式

格式是争议最大但影响最小的部分，虽然每个人都可以使用不同的格式，但如果大家使用统一的格式，就能节省很多讨论争议的时间。问题是如果没有一个规范的使用手册如何达到风格统一呢？在go中，大多数的格式问题都可以用机器解决，gofmt（也可以使用go fmt，这个工具是包级别的，会自动搜索所有的包）可以读取go代码并自动缩进，和垂直对齐，保留并且重新格式化注释，然后输出最新的代码。如果你想知道怎么格式化代码，你可以运行gofmt，如果答案看起来不正确，你可以重新调整格式，这可能是gofmt的bug导致的。

举个例子，我们不需要花时间去排列注释的格式，gofmt会帮我们格式化，下面是gofmt格式化之前的代码：

```go
type T struct {
    name string // name of the object
    value int // its value
}
```

gofmt会将注释自动对齐，如下：

```go
type T struct {
    name    string // name of the object
    value   int    // its value
}
```

所有标准库都通过gofmt格式化过。

要注意的一些格式细节，非常简单：

缩进

​	非常必要不要使用空格缩进，要使用tab

每行长度

​	go中每行长度没有限制，如果一行太长，可以使用制表符换行，并适当缩进

括号

​	相比java、c，go在很if、for、switch语句都不需要括号，去处符优先级结构和层次更加清晰，比如：

```go
x<<8 + y<<16
```

与其它语言不同，表示空格的含义。



注释

go提供了c风格的/**/多行注释和//单行注释，一般单行注释比较常用，多行注释一般用于包级别的注释，但如果有大段的注释也可以使用多行注释。

godoc进程可以提取包级别的注释来生成文档，出现在顶级声明之前的注释（没有中间的换行符）将与声明一起提取，作为该项的解释性文本。这些评论的性质和风格决定了godoc制作的文档的质量。

在package语句之前，每个包都应该有包级别的注释，对于一个包有多个文件，只需要写在其中一个文件里就可以了，包注释应该包含包整体的的介绍以及包提供了哪些功能。它会出现在文档的最开始。

```go
/*
Package regexp implements a simple library for regular expressions.

The syntax of the regular expressions accepted is:

    regexp:
        concatenation { '|' concatenation }
    concatenation:
        { closure }
    closure:
        term [ '*' | '+' | '?' ]
    term:
        '^'
        '$'
        '.'
        character
        '[' [ '^' ] character-ranges ']'
        '(' regexp ')'
*/
package regexp
```

如果包比较简单，注释也可以比较简短

```go
// Package path implements utility routines for
// manipulating slash-separated filename paths.
```

注释不需要额外的格式，比如使用*号分隔，不保证以相同宽度字体输出。所以，不要依赖空格对齐，godoc会和gofmt一样自己处理。注释为纯文本，不能解析比如html这类格式化文本。godoc做的一件事情就是以固定宽度字体显示文档内容，以适配段落。可以参考go fmt包的注释。

godoc工作取决于上下文，所以要确保注释格式是正确的，使用正确的拼写、标点、段落格式、长行换行等等。

在包内，顶级声明前面的内容都将被当作文档注释，每个输出都应该包含一个文档注释

文档注释最好是一段连贯句子，第一句应该是一句摘要，以所声明的内容为准。

```go
// Compile parses a regular expression and returns, if successful,
// a Regexp that can be used to match against text.
func Compile(str string) (*Regexp, error) {
```

