+++
title = "Defer, Panic, and Recover"
description = ""
toc = true
authors = []
tags = ["学习笔记"]
categories = ["官方博客翻译"]
series = ["Golang学习"]
date =  "2021-02-26T15:37:48+08:00"
lastmod = "2021-02-26T15:37:48+08:00"
featuredImage = ""
draft = false
+++

## 1. defer

语句有以下三条简单的规则。

### 1. 在defer语句调用的地方，参数值是取得defer语句执行的值

在下面这个例子中，“i”的值在fmt.Println语句被压入defer栈的时候就决定了。当函数a返回侯，将会打印“0”。

```go
func a() {
    i := 0
    defer fmt.Println(i)
    i++
    return
}
```

### 2. defer语句的执行顺序是先入后出的。
下面的函数将会打印“3210”

```go
func b() {
    for i := 0; i < 4; i++ {
        defer fmt.Print(i)
    }
}
```

### 3. defer语句可以读取和赋值函数命名的返回值
在下面的例子中，defer语句在上层函数执行完，并return侯，将返回值i加一，因此，函数的返回值是2

```go
func c() {
    defer func() {i++}()
    return i
}
```
这是一个修改函数的错误返回值的有效的方法，我们很快会看到一个例子。

## 2. panic

panic是一个中断正常控制流并且开始 <em>panicking</em> 的内置函数。当函数F调用panic，函数F的执行被中断，所有已经被压入defer栈的行数正常执行，随后F将程序控制权还给调用者。对于调用者来说，调用F就相当于调用了panic。这个流程一直运行到当前协程（goroutine）函数调用栈中所以函数都已返回为止，这个时候程序崩溃了。除了在代码里直接调用外，像数组越界等运行时异常也可以引起Panics。

## 3. recover

recover是一个重新获得panicking goroutine控制权的内置函数。Recover只在defer的函数里面才有用，在函数正常执行的过程中调用recover，recover将会返回nil值，并且没有任何效果。如果当前goroutine在panicking，调用recover会捕获panic的参数，并且继续正常的执行。

下面的例子演示panic和defer的用法
```go
package main

import "fmt"

func main() {
    f()
    fmt.Println("Returned normally from f.")
}

func f() {
    defer func() {
        if r := recover(); r != nil {
            fmt.Println("Recovered in f", r)
        }
    }()
    fmt.Println("Calling g.")
    g(0)
    fmt.Println("Returned normally from g.")
}

func g(i int) {
    if i > 3 {
        fmt.Println("Panicking!")
        panic(fmt.Sprintf("%v", i))
    }
    defer fmt.Println("Defer in g", i)
    fmt.Println("Printing in g", i)
    g(i + 1)
}
```

运行此程序将会得到以下输出：
```console
Calling g.
Printing in g 0
Printing in g 1
Printing in g 2
Printing in g 3
Panicking!
Defer in g 3
Defer in g 2
Defer in g 1
Defer in g 0
Recovered in f 4
Returned normally from f.
```

如果我们将f中的defer函数删除，panic就不会被捕获，运行到最后程序会异常退出。修改过的程序会有以下输出：
```console
Calling g.
Printing in g 0
Printing in g 1
Printing in g 2
Printing in g 3
Panicking!
Defer in g 3
Defer in g 2
Defer in g 1
Defer in g 0
panic: 4

panic PC=0x2a9cd8
[stack trace omitted]
```

[1] https://blog.golang.org/defer-panic-and-recover