---
title: 闭包
date: 2021-10-15 15:51:20
tags:
- GoLang
categories:
---
-
<!-- more -->

## FunctionValue

 FunctionValue是个指针，但是他不是函数指针，它指向了一个结构体，结构体里有函数指针【函数指针指向了入口地址】，如果用FunctionValue调用函数，则FunctionValue先找到结构体，结构体通过函数指针调用函数。

其实只要有函数指针就可以完成调用的函数，FunctionValue的意义事什么呢？

答案就是：闭包

## 闭包

我个人对闭包的理解是，一个函数绑定了一个在外面定义的变量【这里简称绑定变量】。FunctionValue指向的结构体的作用不仅保存了函数指针，同时保存绑定变量的信息，把函数运行的上下文环境包裹起来。【这样看来闭包这个词还是很形象的】

### 闭包的几种情况

#### 捕获不变的变量

如果闭包函数绑定的变量不会改变，则直接在FunctionValue指向的结构体里，复制变量的拷贝即可。

<img src="/images/func1.png" width="60%"  />

#### 捕获改变的局部变量

如果闭包函数需要绑定一个会改变的局部变量，则会把这个局部变量分配至堆【内存逃逸】，然后FunctionValue指向的结构体里复制这个堆变量的指针。

<img src="/images/func2.png" width="70%"  />

#### 捕获参数

如果需要绑定传递的参数，则会在栈和堆上都分配 参数，但是会绑定堆上的那个参数。

## 参考资料

[幼麟实验室](https://www.bilibili.com/video/BV1hv411x7we?p=7)

