---
title: go-极速入门
date: 2020-1-14 13:43:45
tags: go
categories: go
---

# 依赖包管理工具

- dep
  未完待续...
- go vendor
  未完待续...
- glide
  未完待续...
- **go modules**
  
  伴随Go 1.11出现，**官方推荐**

# go mod 命令
- `go mod tidy`
  
  拉取缺少的模块，移除不用的模块
- `go mod vandor`
  
  将依赖复制到vendor下
- `go mod download`
  
  下载依赖包
- `go mod verify`
  
  检测依赖
- `go mod graph`
  
  打印模块依赖图

# 切片

## 切片就像数组的引用

## 切片拥有 *长度* 和 *容量*
- 切片的长度就是它所包含的元素个数。
- 切片的容量是从它的第一个元素开始数，到其底层数组元素末尾的个数。
- 切片 s 的长度和容量可通过表达式 `len(s)` 和 `cap(s)` 来获取。

## 切片可以用内建函数 `make` 来创建，这也是创建动态数组的方式。`make` 函数会分配一个元素为零值的数组并返回一个引用了它的切片：
``` go
a := make([]int, 5)  // len(a)=5

// 要指定它的容量，需向 make 传入第三个参数：
b := make([]int, 0, 5) // len(b)=0, cap(b)=5
b = b[:cap(b)] // len(b)=5, cap(b)=5
b = b[1:]      // len(b)=4, cap(b)=4
```

## 向切片追加元素
append 的第一个参数 s 是一个元素类型为 T 的切片，其余类型为 T 的值将会追加到该切片的末尾。append 的结果是一个包含原切片所有元素加上新添加元素的切片。当 s 的底层数组太小，不足以容纳所有给定的值时，它就会分配一个(**cap值翻倍**)更大的数组。返回的切片会指向这个新分配的数组。
``` go
import "fmt"

func main() {
	var s []int
    printSlice(s)
    // len=0 cap=0 []

    // cap不够用——>cap翻倍——>cap=2
	s = append(s, 0)
    printSlice(s)
    // len=1 cap=2 [0]

    // cap够用——>保持不变——>cap=2
	s = append(s, 1)
    printSlice(s)
    // len=2 cap=2 [0 1]

    // cap不够用——>cap翻倍——>cap=8
	s = append(s, 2, 3, 4)
    printSlice(s)
    // len=5 cap=8 [0 1 2 3 4]

    // cap够用——>cap保持不变——>cap=8
    s = append(s, 5, 6, 7)
    printSlice(s)
    // len=8 cap=8 [0 1 2 3 4 5 6 7]
}

func printSlice(s []int) {
	fmt.Printf("len=%d cap=%d %v\n", len(s), cap(s), s)
}
```

# `map`-无序的 `key-value` 数据结构
map集合中的 `key/value` 可以是任意类型，但所有`key`必须同属于一种数据类型，所有的`value`必须同属于一种数据类型。`key` 和 `value` 的数据类型可以不相同

# `for`循环

``` go
person := [3]string { "tom", "john", "jerry" }

for k, v := range person {
  fmt.Printf("person[%d]: %s\n", k, v)
}
```

# `go`关键字
在go关键字后面加一个函数，就可以创建一个线程，函数可以为已经写好的函数，也可以是匿名函数
``` go
func main() {
  fmt.Printf("main start")
  go func() {
    fmt.Println("goroutine")
  }()
  time.sleep(1 * time.Second)
  fmt.Printf("main end")
}
```

# `chan`-类似于队列，遵循先进先出的规则
## 声明`chan`
``` go
// 声明不带缓冲的通道
ch1 := make(chan string)

// 声明带10个缓冲的通道
ch2 := make(chan string, 10)

// 声明只读通道
ch3 := make(<-chan string)

// 声明只写通道
ch4 := make(chan<- string)
```
不带缓冲的通道，进何处都会阻塞
带缓冲的通道，进一次长度+1，出一次长度-1，如果长度等于缓冲区长度时，再进就会阻塞

## 写入`chan`
``` go
ch1 := make(chan string, 10)
ch1 <- "a"
```

## 读取`chan`
``` go
val, ok := <- ch1
val := <- ch1
```

## 关闭`chan`
``` go
close(chan)
```

``` go
func productor(ch chan string) {
  fmt.Println("producer start")
  ch <- "a"
  ch <- "b"
  ch <- "c"
  ch <- "d"
  fmt.Println("producer end")
}

func main() {
  fmt.Println("main start")
  ch := make(chan string, 3)
  go producer(ch)

  time.Sleep(1 * time.Second)
  fmt.Println("main end")
}

// main start
// producer start  发生阻塞
// main end
```

# `defer`

主要应用场景有异常处理、记录日志、清理数据、释放资源 等等。
- `defer`在声明时不会立刻去执行，而是在函数 `return` 后去执行的。
- `defer`函数定义的顺序与实际执行的顺序是相反的，也就是最先声明的最后才执行

``` go
func main() {
  defer fmt.Println("a")
  defer fmt.Println("b")
  defer fmt.Println("c")
  defer fmt.Println("d")

  fmt.Println("main")
}

// main
// d
// c
// b
// a
```

``` go
func calc(index string, a, b int) int {
	ret := a + b
	fmt.Println(index, a, b, ret)
	return ret
}

func main() {
	x := 1
	y := 2
	defer calc("A", x, calc("B", x, y))
	x = 3
	defer calc("C", x, calc("D", x, y))
	y = 4
}

// B 1 2 3
// D 3 2 5
// C 3 5 8
// A 1 3 4
```

# `interface`
## 基本
- 一种**抽象**类型

``` go
package main

import "fmt"

type Sayer interface {
	say()
}

type Cat struct {
	Sayer
}

type Dog struct {
	Sayer
}

func (cat *Cat) say() {
	fmt.Println("miao miao miao~")
}

func (dog *Dog) say() {
	fmt.Println("wang wang wang~")
}

func main() {
	var dog Dog
	dog.say()
	var cat Cat
	cat.say()
}
```
## 空接口的应用
- 空接口作为函数的参数
- 空接口作为`map`的值
- 类型断言
``` go
func main() {
	var x interface{}
	x = "hello world"
	v, ok := x.(string)
	if ok {
		fmt.Println(v)
	} else {
		fmt.Println("类型断言失败")
	}
}
```

# 单元测试

Go语言中的测试依赖`go test`命令。编写测试代码和编写普通的`Go`代码过程是类似的。

`go test`命令是一个按照一定约定和组织的测试代码的驱动程序。在包目录内，所有以`_test.go`为后缀名的源代码文件都是`go test`测试的一部分，不会被`go build`编译到最终的可执行文件中。

在`*_test.go`文件中有三种类型的函数，单元测试函数、基准测试函数和示例函数。

类型|格式|作用
--|:--|--
测试函数|函数名前缀为`Test`|测试程序的一些逻辑行为是否正确
基准函数|函数名前缀为`Benchmark`|测试函数的性能
示例函数|函数名前缀为`Example`|为文档提供示例文档

测试函数实例
``` go
func TestName(t *testing.T){
  // ...
}
```