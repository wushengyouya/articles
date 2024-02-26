---
author: 吴生有崖
title: go语言-2
date: 2024-02-25
description: 
image: image.png
categories:
  - 编程
tags:
  - go
---
## switch
```go
    cmds := os.Args //获取控制台参数,cmds[0]存储的是该程序本身
    switch cmds[1] {
	case "hello":
		fmt.Println("hello")
	case "world":
		fmt.Println("world")
		fallthrough //go语言的switch默认加了break 如果需要穿透需要加关键字fallthrough
	case "1":
		fmt.Println("1")
		fallthrough
	case "2":
		fmt.Println("1,2")

	}
```
## LABEL
```go
func main() {
LABEL:
	for i := 0; i < 5; i++ {
		for j := 0; j < 5; j++ {
			if j == 3 {
				//goto LABEL //goto 不会保存i的状态，下次进入循环时重新开始
				//break LABEL //直接跳出指定位置的循环
				continue LABEL //跳到指定位置，但会保存状态
			}
			fmt.Println("i:", i, "j:", j)
		}
	}

}
```
## 结构体
```go
type student struct {
		name string
		age  int
        Sex  int
	}
type Myint int //对变量关键字重命名
s2 := student{
		"mikasa",
		10, //最后一行必须加上逗号，如果不想加逗号，大括号要与参数写在一行{"mikasa",10}
}
//匿名结构体
t1 := struct {
    name string
}{name: "赵今麦"}
```
## init
1. init在import时被调用
2. init函数不能被显示调用
3. 存在多个init函数时调用顺序是随机的
4. 如果只想调用某个包的init函数 使用下划线  _ import ..
5. 不管包被调用多少次init函数都只执行一次
```go
func init(){
	fmt.Println("第一个Init函数")
}
```
## defer
执行的情况
- 函数执行后用于资源的释放，如文件的关闭，网络连接的关闭，数据连接的关闭
- 多个defer采用后进先出(LIFO)模式执行，类似于栈存储
- defer执行在`return`之后
- 如果函数中发生了`panic`,在程序开始逐层向上抛出`panic`之前，会执行该函数中的所有`defer`语句
不执行的情况
- 程序正常退出或调用了`os.Exit`退出程序
- 执行了其他协程的`runtime.Goexit`函数
```go
func doSomething() {
    defer fmt.Println("deferred call in doSomething") // 将会在函数结束时执行
    fmt.Println("doing some work")
    // ...做一些工作...
    return // 此时会触发defer执行
}
```
## 类
```go
//1. go语言里面没有类，使用结构体来模拟
type Student struct {
	name   string
	age    int
	adress string//访问修饰采用大小写的方式,大写则是对外开放
	Sex int
}
//2. 绑定方法
func (stu Student) eat() {
	fmt.Println(stu.name, "学生吃饭")
	stu.name = "sakura"
	//fmt.Println("修改后:", stu.name)
}
func (this *Student) Eat2() {
	fmt.Println(this.name, "eat2学生吃法")
	this.name = "sakura"
	//fmt.Println("修改后:", this.name)
}
```
## 组合
go不支持继承采用组合的方式来实现继承，组合是通过在一个结构体中嵌入其他结构体或者接口来实现的，
嵌入的结构体或接口的方法会被提升到外层结构体中，就好像是外层结构体自己的方法一样。
```go
type People struct {
	name string
	age  int
}
func (this *People) eat() {
	fmt.Println("吃饭")
}
//组合
type Man struct {
	People
	gender string
}
```
## interface
Go语言的接口工作原理基于一种被称为“鸭子类型”的概念，即`如果它像鸭子一样走路，像鸭子一样叫，那么它就是一只鸭子`。
如果类型实现了接口中的所有方法那么，它就被认为实现了该接口，而且这一切都是隐式实现的。不需要显示声明。
- 实现了接口中的`所有方法`则是隐式实现了该接口
- 定义对象的行为
- 作为函数参数，接收任意类型的值

1. 作为参数
```go
//interface  实现多态.也可以接受任意数据类型
	var i, k interface{}
		//判断数据类型
	kvalue, ok := k.(int) //做类型的二次确认
	if !ok {
		fmt.Println("k不是Int")
	} else {
		fmt.Println("k是int,", kvalue)
	}

	//最常用的场景是作为参数，根据interface的数据类型执行不同的操作
	array := make([]interface{}, 3)
	array[0] = 1
	array[1] = "mikasa"
	array[2] = 3.14
	for _, value := range array {
		//这种方式的类型判断只能在switch中使用
		switch v := value.(type) {
		case int:
			fmt.Printf("当前为int,数据为%d\n", v)
		case string:
			fmt.Printf("当前为string,数据为%s\n", v)
		case float64:
			fmt.Printf("当前为folat,数据为%v\n", v) //%v自动推断数据类型
		default:
			fmt.Printf("不是合理的数据类型")
		}
	}
```
2. 定义行为
```go
type IAttack interface {
	Attack()
}
// 低等级
type HumanLowLevel struct {
	name  string
	level int
}
func (this *HumanLowLevel) Attack() {
	fmt.Println("我是", this.name, "等级为", this.level)
}
// 高等级
type HumanHighLevel struct {
	name  string
	level int
}
func (this *HumanHighLevel) Attack() {
	fmt.Println("我是", this.name, "等级为", this.level)

}
// 定义一个通用接口，通过传入不同的类型，调用同一个方法实现不同的效果
func DoAttack(a IAttack) {
	a.Attack()
}
lowLevel := HumanLowLevel{
		name:  "David",
		level: 1,
}
highLevel := HumanHighLevel{
	name:  "David",
	level: 10,
}
DoAttack(&lowLevel)
DoAttack(&highLevel)
```
## goroutine
```go
// return 返回当前函数
// exit 退出当前进程
// goexit 退出当前go程
func main() {
	go func() {
		go func() {
			func() {
				fmt.Println("这是子go程内部的函数!")
				//return //这是返回当前函数
				os.Exit(-1) //退出进程
				//runtime.Goexit() //退出当前go程
			}()

			fmt.Println("子go程结束!") //这句会打印吗？ 会1：  不打印2
			fmt.Println("go 2222222222 ")

		}()
		time.Sleep(2 * time.Second)
		fmt.Println("go 111111111111111")
	}()

	fmt.Println("这是主go程!")
	time.Sleep(3 * time.Second)
	fmt.Println("OVER!")
}

```