+++
title = "循环"
description = "循环"
toc = true
authors = []
tags = ["没用的知识"]
categories = []
series = ["Golang学习"]
date =  "2021-02-26T10:30:48+08:00"
lastmod = "2021-02-26T10:30:48+08:00"
featuredImage = ""
draft = false
+++

## Golang的循环

go语言不像其它语言那样将循环分为for循环和while循环，而while又进一步可以分为do...while...和while两种。在go中只有for一种。并且for不光能用在整数区间内循环，也可以配合range用于chan、array、slice、map等的遍历。
### 1. 有限循环

```go
for i := 0; i < 10; i++ {

}
```
### 2. 无限循环

```go
for {
    
}
```

### 3. 循环提前退出

1. break

```go
for i := 0; i < 10; i++ {
    if i > 5 {
        break
    }
    fmt.Println(i)
}
```

2. continue

```go
for i := 0; i < 10; i++ {
    if i % 2 ==0 {
        continue
    }
    fmt.Println(i)
}
```

### 4. slice遍历

```go
nums := []int{1, 2, 3, 4}
for i, num := range nums {
    fmt.Println(i, num)
} 
```

### 5. map遍历

```go
m := map[string]interface{}{"name": "张三", "age": 32}
for k, v := range m {
    fmt.Println(k, v)
}
```

### 6. chan遍历

```go
ch := make(chan string)
go func() {
    ticker := time.NewTicker(time.Second)
    for i := 0; i < 5; i++ {
        ch <- fmt.Sprintf("Hello %d", i)
        time.Sleep(time.Second)
    } 
    close(ch)
}()
for msg := range ch {
    fmt.Println(msg)
}
```