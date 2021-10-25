---
title: nil
date: 2021-10-24 15:27:38
tags:
categories:
---
-
<!-- more -->

转载自--

Golang中的nil，没有人比我更懂nil！ - 无敌的CF的文章 - 知乎 https://zhuanlan.zhihu.com/p/151140497

## 从代码开始

```go
type A interface{}
type B struct{}
var a *B

print(a == nil)            //true
print(a == (*B)(nil))      //true
print((A)(a) == (*B)(nil)) //true

print((A)(a) == nil)       //false
```

上面是一段原创的极度反直觉的代码。为什么前面3个都相等，最后一个就不相等了呢？

其实在我之前[关于interface的文章](https://zhuanlan.zhihu.com/p/86420182)中，是有部分解答的。但是只能回答为什么最后的等式不成立。原因简单说一下，是因为只有当一个interface的value和type都unset的时候，它才等于nil，而上述代码中的interface `(A)a` 的T是`*B`, 不是unset.

比较难以理解的其实是第三个等式。前两个式子左右两边都是nil，相等情有可原。第三个式子左边明明不等于nil的，为什么会相等呢？下面马上就会回答。

## 不同的nil

`nil`其实甚至不是golang的关键词，只是一个变量名。定义在 buildin/buildin.go 中

```go
// nil is a predeclared identifier representing the zero value for a
// pointer, channel, func, interface, map, or slice type.
var nil Type // Type must be a pointer, channel, func, interface, map, or slice type

// Type is here for the purposes of documentation only. It is a stand-in
// for any Go type, but represents the same type for any given function
// invocation.
type Type int
```

换句话说，我们也可以自己声明一个nil，就会把预定义的nil覆盖了。自己试试就好了， 这肯定是不推荐的。根据这里的定义，也可以看出，在golang中nil代表了`pointer`, `channel`, `func`, `interface`, `map` 或者 `slice` 的zero value.

而这里就出现一个问题，这些类型之间千差万别，一个nil怎么可以代表这么多类型呢？其实一个nil确实有些表达不了，这也是很多误会产生的原因。简单来说，nil也是有类型的，`(*int)(nil)`和`(interface{})(nil)`就是两个不同的变量，它们也不相等。更有些类型比如`([]int)(nil)`因为是slice类型的缘故，根本就是不能比较的。

回到上文。当interface与一个值做比较的时候，会同时比较type和value，都相等的时候，才会认为相等。上文中的`(*B)(nil)`的type与`(A)(a)`相同，都是`*B`，值也相同，都是`nil`，所以他们就相等了。

## 特别的nil

尝试一下一行很简单的代码

```go
var a = nil
```

报错"use of untyped nil in variable declaration". 这很好理解，任何变量都应该有类型的，但是a的类型是什么呢，编译器百思不得其解，于是它生气了。哄一下应该没用，试着这样改一下就没问题了。

```go
var a = (*int)(nil)
```

不过上文的错误信息中出现了一个特殊的nil，"untyped nil". 当我们直接写一个nil出来的时候，它是没有类型的。没有类型的nil虽然不能直接赋值给变量，但是可以与一些特定类型的变量进行比较，比如上面出现过的

```go
var a *B
print(a == nil)
```

这是合法的。这是`untyped nil`的特别之处，当它被拿来与一个变量进行比较的时候，根据不同的变量，就会有不同的逻辑。

## 展开说说

为了证明我没有骗人，下面展开说说不同变量类型在什么情况下可以等于nil

### pointer

nil pointer就是一个没有指向任何值的指针，它的值是 0x0. 做个小实验

```go
var a = (*int)(unsafe.Pointer(uintptr(0x0)))
print(a == nil)  //true
```

恭喜我们人工创造了一个 nil pointer !

### slice

一个slice由3部分组成，pointer，len和cap. 这句话其实展开来说很长。如果能看懂下面的代码，那就是大约理解了。

当pointer是nil，len和cap都是0的时候，这个slice等于nil. 下面做个实验

```go
var a = []int{}
print(a==nil) //false 
type aa struct {
    ptr unsafe.Pointer
    len int
    cap int
}
aaa := (*aa)(unsafe.Pointer(&a))
aaa.ptr = nil
print(a==nil) //true
```

略微有点黑科技。简单来说，我们原本声明了一个`empty slice`, empty slice是不等于nil。但是我们把这个slice结构体中的ptr改成了nil，于是这个slice就变成了`nil slice`.

话说，关于empty slice和nil slice取舍，golang的官方是推荐大多数情况下都应该用的nil slice的，除了是encoding JSON object等特殊情况。有点跑题，就不展开说了，具体可以参考这里的[官方文档](https://link.zhihu.com/?target=https%3A//github.com/golang/go/wiki/CodeReviewComments%23declaring-empty-slices)。

### chanel & map & func

这3位大哥每个都够讲一年的。但是简单来说，它们都是一个指针指向**一堆implementation**. 所以就可以把它们看成指针了，这个指针是nil，那就是nil了。

### interface

这个已经说过，当一个interface的type和value都是nil的时候，这个interface才等于nil. 这真的是个坑人无数的golang陷阱，这里就再举一个小栗子好了。

```go
type A interface{}
type B struct{}
var a A = (*B)(nil)
print(a == nil) //false
a = nil
print(a == nil) //true
```

## 一个神奇之处

当方法接收者为nil时，仍然可以访问对应的方法。偶尔可以帮忙减少代码量。虽然根据方法的写法，是有可能panic的。 比如

```go
type A []int

func (a A) Get() int {
    return a[0]
}

func main() {
    var a A
    a.Get()
}
```

这个代码是可以编译通过的，但是运行时会panic, runtime error: index out of range [0] with length 0.

## 小感想

从上面nil slice和empty slice的区别引出，其实empty slice也可以作为slice的zero value。特地发明一个nil值，应该是golang出于对性能的考虑。nil pointer其实是一切nil值的根本形态，我理解背后的思想就是能不分配的内存就先不分配，pointer就先让它nil着。

名字都是nil，细想之下区别非常之大，golang围绕了nil制定了了很多固定的特殊用法。因此，在大部分情况下，nil的使用是非常自然的。这样的设计，究竟是好是坏呢？