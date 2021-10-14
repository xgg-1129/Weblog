---
title: GoLang字符串
date: 2021-10-11 11:50:26
tags: GoLang
categories: 学习
---
-
<!-- more -->

## 字符串

字符串的本质实际就是一片连续的内存空间，我们通常可以把他理解为一个字符数组，Go语言的字符串是一片只读的字节数组。

### 内存中的字符串

如果是代码中存在的字符串，编译器会将其标记成只读数据 `SRODATA`，假设我们有以下代码，其中包含了一个字符串，当我们将这段代码编译成汇编语言时，就能够看到 `hello` 字符串有一个 `SRODATA` 的标记

//禁止内联生成汇编代码

GOOS=linux GOARCH=amd64 go tool compile -S main.go

```go
func main() {
	str := "hello"
	println([]byte(str))
}

$ GOOS=linux GOARCH=amd64 go tool compile -S main.go
...
go.string."hello" SRODATA dupok size=5
	0x0000 68 65 6c 6c 6f  
```

只读只意味着字符串会分配到只读的内存空间【数据段+代码=制度空间】，我们不可以直接修改 `string`类型变量的内存空间。

### 字符串的数据结构

字符串在运行的时候用StringHeader表示

```go
type StringHeader struct {
	Data uintptr //数据指针
	Len  int	 //字符串长度
}
```

### 字符串拼接

字符串是只读空间，我们无法向字符串追加元素改变它的内存空间，所有字符串的修改操作都是通过拷贝实现的。字符串运行时调用copy将多个字符串拷贝到新的内存空间，这样的内存拷贝是比较消耗性能的。

### 字符串和[]byte的转换

字符串和[]byte底层内容都是一样的，但是[]byte是可读可写的【堆栈】，类型转换通常使用截断或者拷贝完成，像字符串和[]byte这样处于不同类型内存空间的数据转换时会产生内存拷贝。

### Len(String)

String底层是[]byte,它的len返回的是字节长度，所以len("我")!=1

### 总结

### 参考资料

[Golang的String解析](https://zhuanlan.zhihu.com/p/355082331)

[幼麟实验室](https://www.bilibili.com/video/BV1hv411x7we?spm_id_from=333.999.0.0)

[Draveness字符串解析](https://draveness.me/golang/docs/part2-foundation/ch03-datastructure/golang-string/#344-%E7%B1%BB%E5%9E%8B%E8%BD%AC%E6%8D%A2)
