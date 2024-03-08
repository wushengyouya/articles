---
author: 吴生有崖
title: go语言-1
date: 2023-02-25
description: 
image: image.jpg
categories:
  - 编程
tags:
  - go
---
## 特性
- go的语法简洁明了，易于学习和阅读便于学习。
- 是一种编译型语言，go代码会被编译成机器码，执行效率高启动速度快
- go天然支持高并发提供了goroutine来处理并发操作，channel用于goroutine之间的通信。go并发模型
基于“不要通过共享内存来通信，而是通过通信来共享内存”
- 自带垃圾回收机制，降低了内存泄漏的风险
- go鼓励使用组合而不是继承来组织和复用代码。它的结构体（structs）可以嵌入其他类型。
## 关键字(25个)
| 关键字     | 描述                                       |
|------------|------------------------------------------|
| `break`    | 用于终止当前循环                         |
| `case`     | switch 语句中的一个分支                |
| `chan`     | 表示一个通道类型                         |
| `const`    | 定义常量                                 |
| `continue` | 跳过当前循环的剩余代码，开始下一次循环  |
| `default`  | switch 或 select 语句中的默认分支      |
| `defer`    | 延迟执行一个函数直到上层函数返回        |
| `else`     | if 语句中的否则分支                    |
| `fallthrough` | 在 switch 语句中强制执行下一个 case |
| `for`      | 循环语句                               |
| `func`     | 定义一个函数                             |
| `go`       | 启动一个新的 goroutine                 |
| `goto`     | 转到一个标签                             |
| `if`       | 条件语句                                 |
| `import`   | 引入包                                   |
| `interface`| 定义一个接口                             |
| `map`      | 表示一个映射类型                         |
| `package`  | 定义一个包                               |
| `range`    | 迭代数组、切片、字符串、映射或通道     |
| `return`   | 从函数中返回                             |
| `select`   | 选择不同类型的通讯                     |
| `struct`   | 定义一个结构体                           |
| `switch`   | 条件分支语句                             |
| `type`     | 类型定义或类型别名                       |
| `var`      | 定义一个变量                             |
## import
```go
//导入单个包
import "fmt"
//导入多个
import (
    "fmt"
    "sync"
)
```
## 包管理
go默认采用go mod进行包管理
```bash
go mod init temp.github.com #初始化
go mod tidy #添加缺少的模块，移除不再需要的模块
go mod download #下载模块
go mod list #列出模块
go mod graph #打印模块依赖图
```
## main函数
```go
    package main
    import "fmt"

    func main(){
        //执行内容
    }
```
## 变量与常量
```go
    //1.定义再赋值
	var name string
	name = "mikasa"
	//2.定义直接赋值
	var age = 10
	var gender = "man"
	fmt.Printf("name : %s,age : %d,gender:%s\n", name, age, gender)
    //3.自动推导赋值
	adress := "背景"
	fmt.Println("adress:", adress)
    //4.平行赋值
	i, j := 10, 20
	fmt.Println("i:", i, "j:", j)
	i, j = j, i//交换两个变量的位置
	fmt.Println("i:", i, "j:", j)
    //通过字符串创建了一个切片
	str11 := "abcdefljin"
	str22 := str11[2:4]
	fmt.Println(str22)
   //定义常量
	const conNum = 5
	const con string = "10"
	const (
		a = 1
		b = 2
	)
```
## 自增与自减
```go
    //go里面没有 --i  ++i
	//只有i++ i-- 且必须只能独占一行
    i := 2
	i++
	i--
```
## 数组及遍历
[for-range底层实现](https://draveness.me/golang/docs/part2-foundation/ch05-keyword/golang-for-range/#513-%E8%8C%83%E5%9B%B4%E5%BE%AA%E7%8E%AF)

for-range典型面试问题

1.
```go
func main() {
	arr := []int{1, 2, 3}
	for _, v := range arr {
		arr = append(arr, v)
	}
	fmt.Println(arr)
}

$ go run main.go
1 2 3 1 2 3
```
2.
```go
func main() {
	arr := []int{1, 2, 3}
	newArr := []*int{}
	for _, v := range arr {
		newArr = append(newArr, &v)
	}
	for _, v := range newArr {
		fmt.Println(*v)
	}
}

$ go run main.go
3 3 3
```


```go
    //定义一个长度为10的数组
    nums := [10]int{1, 2, 3, 4}

    //数组的遍历方法1
	for i := 0; i < len(nums); i++ {
		fmt.Println("i:", i, "nums", nums[i])
	}
	//数组的遍历方法2
    //所有的 for-range 循环都会被转换成不包含复杂结构、只包含基本表达式的for语句
    //对于所有的 range 循环，Go 语言都会在编译期将原切片或者数组赋值给一个新变量 ha，在赋值的
    //过程中就发生了拷贝，而我们又通过 len 关键字预先获取了切片的长度，所以在循环中追加新的元素
    //也不会改变循环执行的次数
	for key, value := range nums {
		fmt.Println("key:", key, "value:", value)
	}
    //3.
    //在go语言如果想省略一个值可以用下划线_
	//如果两个值都想省略不能:=而用=
	for _, value := range nums {
		value += 1 //··
		fmt.Println("省略Key的value:", value)
	}
	for _, _ = range nums {
		fmt.Println("省略两个值：")
	}

    //4.数组传递
    var arr [5]int = [5]int{1, 2, 3, 4, 5}
	printArr(&arr) //数组是值类型默认是值传递 ，此处采用引用传递，函数修改后能影响源数据
	fmt.Println(arr)
   
   //函数结构数组的指针
   func printArr(arr *[5]int) {
	arr[0] = 10
	for _, v := range arr {
		println(v)
	}
}
```

## 指针
```go
//go 语言也有指针 * 为取地址里面的值
//结构体成员调用 c语言 ptr->name  go语言 ptr.name
//go语言在使用指针时，会使用内部的垃圾回收机制gc(garbage collector)，开发人员不需要手动释放内存
//c语言不许返回栈上的指针，go语言可以返回，程序在编译的时候就确定的指针的位置
//go语言在编译的时候，如果发现有必须要会将变量分配到堆上
//1.第一种定义指针
	name := "mika"
	ptr := &name
	fmt.Println("name:", name, "ptr:", ptr, "*ptr:", *ptr)
	//2.第二种定义指针
	age := new(int)
	fmt.Println("age:", age)
	*age = 10
	fmt.Println("*age:", *age)
    //可以返回栈上面的指针，因为在编译的时候，会自动判断，将city这个变量分配到堆上面
	ptr = test()
	fmt.Println("ptr:", ptr)

    // 定义函数返回一个指针，go语言函数的返回值写在后面
func test() *string {
	//作为局部变量函数执行完后应该会被释放
	city := "beijing"
	return &city
}
```

## 不支持的运算符
```go
++i
--i
c=a>b?3:4 go语言不支持三目运算符三目运算符

0在go语言中不能表示逻辑 否
if 0{

    fmt.Println(1)
}
```
## 字符串
```go
    //1.定义
    var str string
    str = "aaa"
    name := "duke"
    //2.go语言按照原格式输入字符串加 ``
	usage := `
			sfds
		
			nsi
			`
    //3.字符串长度访问
	strLength := len(name)
    //4.拼接 使用strings.Builder 效率更高
	i := "mikasa : "
	j := "like"
	fmt.Println("字符串拼接：", i+j)
	//字符串截取 左闭右开
	fmt.Println("字符串截取:", i[2:])
	fmt.Println("字符串截取:", i[:2])
	fmt.Println("字符串截取:", i[1:3])
	//可直接使用字符串进行比较，使用ascii表
	if i > j {
		fmt.Println(2)
	}
```
## 枚举
```go
const {
  BEIJING = iota //iota默认等于0
  SHANGHAI //iota=1
  SHENZHEN //iota=2
}

```
## 切片
```go
//切片，它的底层也是数组，可动态的修改长度
	adress := []string{"北京", "上海", "广州", "深圳"}

	//追加元素
	adress1 := append(adress, "长沙")
    //2.对于切片不仅有长度的概念还有容量的概念  容量不够时为小于1024时翻倍增长
	for i := 0; i < 50; i++ {
		nums = append(nums, i)
		fmt.Println("len:", len(nums), "cap:", cap(nums))

	}
```
### 切片注意点
```go
names := []string{"beijing", "shanghai", "shenzhen", "changsha"}

	//基于names创建一个数组
	names1 := [3]string{}
	names1[0] = names[0]
	names1[1] = names[1]
	names1[2] = names[2]

	//修改
	names1[2] = "深圳"
	fmt.Println("names:", names)   //
	fmt.Println("names1:", names1) //深圳

	//基于切片创建一个新数组,属于引用之前的数组，如果修改数据之前的数据也会发生改变
	names2 := names1[0:3]

	fmt.Println("修改前names1:", names1, "修改前names2:", names2)
	names2[2] = "广州"
	fmt.Println("修改后names1:", names1, "修改后names2:", names2)
	//1.如果从0开始截取,: 左边可以省略
	names3 := names[:2]
	fmt.Println("names3[:2]:", names3)
	//2.如果截取数组最后一个元素，冒号右边可以省略
	names3 = names[1:]
	fmt.Println("截取到最后一个元素names3[1:]:", names3)

	//3.如果截取所有元素，冒号左右都可以省略
	names3 = names[:]
	fmt.Println("截取所有元素[:]:", names3)
	//4.对字符串截取
	str := "abcd"
	s1 := str[1:]
	fmt.Println("字符串截取:", s1)

	//5.可以在创建空切片的时候，明确指定容量，这样可以提高运行效率
	//创建一个容量为20的切片，当前长度为0的string
	//make的时候，初始值类型都是对应的零值类型  bool -> false  字符串 -> 空  数字->0
	str2 := make([]string, 10, 20) //第三个参数不是必须 ，如果未填写容量则与长度相等
	fmt.Println("str2:", str2[0])
	if str2[0] == "" {
		fmt.Println("为空")
	}
	fmt.Println("str2-len:", len(str2), "str2-cap:", cap(str2))
	str2[0] = "hello"
	str2[1] = "world"

	//6.如果想让切片完全独立于原始数组，可以用copy()函数
	str3 := make([]string, len(str2))
	copy(str3, str2[:]) //str3完全独立于原始数组，修改后不会对原始数组产生影响
	str3[0] = "mikasa"
	fmt.Println("复制的str3:", str3, "原str2:", str2)
```
## 字典
```go
    //1.定义一个字典，此时还不能赋值，因为还未初始化，它是空的
	var namesM map[int]string
    //2.分配空间，可以指定长度或者不指定长度，指定长度性能更好
	namesM = make(map[int]string)
	namesM[0] = "北京"
    //3.定义直接分配空间, nums:=make(map[int]string,10) 比较常用
    idNames := make(map[int]string)
    //4.字典遍历
	for k, v := range namesM {
		fmt.Println("k:", k, "v:", v)
	}
    //5.如何确定一个key 在mao中,mao不存在索引越界的问题
	fmt.Println("索引越界不会抛异常：", idNames[50]) //此句在go语言中不会抛异常
    value, ok := namesM[9] //如果当前map中存在该key，value的值为 0 ，ok返回true
    //6.删除map中的元素
	//使用自由函数delete在删除map中的元素
	fmt.Println("删除前idNames:", idNames)
	delete(idNames, 0)
```
- Map不是线程安全的,如果遇到并发访问需要加锁
- sync.Map 是并发安全的其内部默认加了互斥锁
## 函数
```go
//1.函数的返回值在参数列表之后
//2.如果有多个返回值用圆括号包裹，用逗号分割
num := test1()
d, e, _ := test2() //不需要接受的返回值可以用下划线省略
fmt.Println(num)

// 多个返回值
func test2() (a int, b int, c string) {
	return 10, 20, "北京"
}

// 局部变量内存逃逸
func test4(str string) *int {
	num := 10
	return &num
}
```