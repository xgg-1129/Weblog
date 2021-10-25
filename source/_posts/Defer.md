---
title: Defer
date: 2021-10-15 15:42:07
tags:
categories:
---
-
<!-- more -->

## 延迟调用

GoLang的return会被编程成两条语句，一条是给返回值传值【defer可以修改返回值】，第二条是退出函数。Defer能够向当前函数注册一些稍后执行的函数，这些函数在退出函数前执行，作为一个编程语言中的关键字，`defer的实现一定是由编译器和运行时共同完成的。

## Defer链表栈

defer关键字在 Go 语言源代码中对应的数据结构：

```go
type _defer struct {
	siz       int32
	started   bool
	openDefer bool
	sp        uintptr
	pc        uintptr
	fn        *funcval   //注册的函数
	_panic    *_panic
	link      *_defer   //结点相互连接成一个链表
}
```

GoLang是使用头插法在链表前面添加defer结点，然后从头节点开始执行注册的defer结点函数【链表栈】，所以defer的调用顺序是先进后出的顺序。

## Defer参数

- 函数的参数会被预先计算；
  - 调用 [`runtime.deferproc`](https://draveness.me/golang/tree/runtime.deferproc) 函数创建新的延迟调用时就会立刻拷贝函数的参数，函数的参数不会等到真正执行时计算；

## 参考资料

[Draveness](https://draveness.me/golang/docs/part2-foundation/ch05-keyword/golang-defer/#53-defer)

[幼麟实验室](https://www.bilibili.com/video/BV1hv411x7we?p=10)

