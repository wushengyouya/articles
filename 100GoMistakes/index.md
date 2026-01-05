---
author: 吴生有崖
title: 100GoMistakes
date: 2026-01-05
description: 100 Go Mistakes And How To Avoid Them笔记
image: 
categories:
  - 编程
tags:
  - go
---
## 100 Go Mistakes And How To Avoid Them

1、避免使用遮挡变量
```go
var i
func get(){
    i := 1
}
```

2、通过避免嵌套层级并将主路径保持在左侧

3、init函数不能处理error，当有多个init函数先加载依赖项的init(按照文件名字母表的顺序)

4、抽象是被发现的，而不是创建的。

5、接口尽量在使用端，而不是在生产端

6、函数不应该返回抽象而是具体的实现，而参数尽可能的使用接口作为参数

7、尽量不使用`any`类型，因为它提供有意思的信息，尽在需要进行序列化的使用`any`

8、当有多个参数需要配置时，可以采用功能选项模式。(function option)

9、创建包名时应该可以直观的明白这个包时做什么的，避免使用泛化包名如util、base、common

10、对于可以在外部访问的变量与函数都需要注释
- 变量和常量: 注释用途和内容
- 函数和方法：注释函数用途而非实现方式

11、使用静态代码分析工具检测代码如[vet](https://golang.org/cmd/vet/)、[golangci-lint](https://golangci-lint.run/docs/linters/)

12、在go中`0o`代表8进制,`0B | 0b`代表二进制,`0x`代表十六进制,`i`后缀代表虚数

13、go中整数上溢和下溢都是隐式的，可以使用`math.MaxInt`,`math.MinInt`,`math.MaxUint`构建函数检测

14、浮点数在加减运算时应将数量级相近的操作分组以提高准确性，乘除运算应在加减运算前进行以提高准确性

15、切片
- 切片的长度是当前可以获取的元素数，容量是可以存储的元素数。当元素存储满时底层会创建一个新的数组，容量在元素在1024以内成倍扩容，超过后每次扩容1/4
- 当创建切片时如果已知长度，make时指定长度，减少内存分配提升性能。同样适用于创建map的时候
- 复制一个切片到另一个切片使用`copy`函数，复制的元素数取决于两个切片中最小的长度
- `str[1:5]`使用切片表达式缩减切片，底层都是指向同一个数组，如果缩减一个大切片时可能会发生内存泄漏。可以使用`copy`或   
完整的切片表达式`str[1:5:2]`缩减大切片必须使用`copy`
- 在处理切片时需要记住，如果元素是指针或者是包含指针资源的结构体，这些元素将不会被gc回收。  
  清除方法: 1.手动置为nil 2.使用`copy`而不是reslice
```go
// 示例1：指针切片
// 创建包含指针的切片
items := make([]*Item, 0, 100)

// 添加一些指针元素
items = append(items, &Item{data: "data1"})
items = append(items, &Item{data: "data2"})

// 假设我们"删除"第一个元素
// 手动清除方法 items[0] = nil
// 使用copy
copy(item[0:],item[1:])
items = items[1:]  // 现在 items 只有 data2
// 问题：底层数组仍然引用着 data1 的指针！
// data1 指向的内存不会被GC回收

///////////////////////////////////////////////////////
// 示例2：包含指针字段的结构体
// 问题：底层数组仍然引用着 data1 的指针！
// data1 指向的内存不会被GC回收
type User struct {
    Name string
    Data *Data  // 指针字段！
}

users := make([]User, 0, 10)
users = append(users, User{Name: "Alice", Data: &Data{value: 100}})

// 删除元素后，Data指针仍然被底层数组引用
users = users[1:]
```




