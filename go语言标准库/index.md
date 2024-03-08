---
author: 吴生有崖
title: go语言标准库
date: 2023-02-25
description: 
image: image.jpg
categories:
  - 编程
tags:
  - go
---
## bufio,os包读取控制台输入
```go
//创建读bufio
reader := bufio.NewReader(os.Stdin)
fmt.Println("请输入内容:")
//读取
text, _ := reader.ReadString('\n')
text = strings.TrimSpace(text)
fmt.Println(text)
//指定输出io
fmt.Fprintln(os.Stdout, "向标准输出写入")
//输出到文件
f, err := os.OpenFile("./1.txt", os.O_CREATE|os.O_WRONLY|os.O_APPEND, 0644)
defer f.Close()
name := "刘华"
fmt.Fprintf(f, "文件加入信息,%s", name)
```
## 读取文件的三种方式
1. 读取整个文件
```go
// 读取整个文件
func readFile() {
	context, err := os.ReadFile("1.txt")
	if err != nil {
		log.Fatal(err)
	}
	fmt.Println(string(context))
}
```
2. 逐行读取文件
```go 
func readFileLine() {
	log.Fatal("log-按行读取文件")
	f1, err := os.Open("./2.txt") //打开文件
	if err != nil {
		log.Fatal("1", err)
	}
	defer f1.Close() //关闭文件

	//按行读取文件
	scanner := bufio.NewScanner(f1) //bufio.NewReader(f).ReadString('\n')//bufio读取用一行

	for scanner.Scan() {
		fmt.Printf("line:%s\n", scanner.Text())
	}
	if err := scanner.Err(); err != nil {
		log.Fatal("2", err)
	}
	log.Fatal("end-按行读取文件")
}
```
3. 分块读取
```go
func readFileBlock() {
	f, err := os.Open("1.txt")
	if err != nil {
		log.Fatal(err)
	}
	defer f.Close()
	s, _ := bufio.NewReader(f).ReadString('\n') //bufio读取用一行
	fmt.Println("bufid读取一行，", s)
	buf := make([]byte, bufSize)
	for {
		n, err := f.Read(buf)
		if err != nil && err != io.EOF {
			log.Fatal(err)
		}
		if err == io.EOF {
			break
		}
		fmt.Println(string(buf[0:n]))
	}

}
```
## 时间触发器
```go
//创建时间触发器 每间隔一秒
ticker := time.Tick(time.Second)
for i := range ticker {
    fmt.Println(i)
}
```
## flag包
```go
//定义命令行参数方式1
	var name string
	var age int
	var married bool
	var delay time.Duration

	flag.StringVar(&name, "name", "张三", "姓名")
	flag.IntVar(&age, "age", 18, "年龄")
	flag.BoolVar(&married, "married", false, "婚否")
	flag.DurationVar(&delay, "d", 0, "延迟时间间隔")

	//打印默认
	flag.PrintDefaults()
	//解析命令行参数
	flag.Parse() //必须要执行解析才有返回值
	fmt.Println(name, age, married, delay)
	//返回命令行参数后的其他参数
	fmt.Println(flag.Args())

	//返回命令行参数后的其他参数个数
	fmt.Println(flag.NArg())
	//返回使用的命令行参数个数
	fmt.Println(flag.NFlag())
```
## Log
```go
//[mikasa]2024/02/25 22:46:22.055007 D:/GoProjects/GoTest/8-默认log.go:20: 这是log1
//这是Log标记
log.SetFlags(log.Llongfile | log.Lmicroseconds | log.Ldate)

//设置前缀
log.SetPrefix("[mikasa]")

//设置将log输出到文件
f, err := os.OpenFile("1.txt", os.O_CREATE|os.O_APPEND, 0644)
defer f.Close()
if err != nil {
    log.Println(err)
}
log.SetOutput(f)
log.Println("这是log1")
log.Panicln("出现错误2")
```
## io包总结
```txt
文件io参数包  ioutil bufio  os
linux文件权限:
参数2 打开模式
参数3 权限控制
w 2 r 4 x 1 -rwxrwxrwx 0777 -rw-rw-rw 0666
```
```go
f,_:=os.OpenFile("1.txt")//打开创建文件,返回f 支持读写
f:=os.Open("1.txt")//仅打开文件 返回f 支持读写
reader:=bufio.NewReader(f)//传入f缓冲读写
writer:=bufio.NewWriter(f)
//写完后刷新缓冲区
writer.Flush()
```
## strcov类型转换
```go
//字符串转整型
strconv.Atoi("123")

//整型转字符串
//int to string  a是c语言遗留问题，C语言没有string 用array代替
	s := strconv.Itoa(i32)

//字符串转bool
b, _ := strconv.ParseBool("true")

//参数一 数据   参数二 进制  参数三 最大溢出进制
i64, _ := strconv.ParseInt("100", 10, 64)
fmt.Printf("%T-%d\n", i64, i64)

s1 := strconv.FormatInt(int64(i32), 10)
```
## sync
1. sync.WaitGroup 等待协程
```go
wg := &sync.WaitGroup{}
wg.Add(1)
go func(){
    log.Println("111")
    wg.Done()
}
wg.Wait()
//协程执行完后，接下来要执行的内容
...
```
2. sync.Mutex 互斥锁  读写锁
```go
//互斥锁
mutx := &sync.Mutex{}
mutx.Lock()
//要加锁的内容..
//...
mutx.Unlock()

//读写锁
rwMutx := sync.RWMutex{}
rwMutx.RLock()
//...
rwMutx.Unlock()
```
3.sync.Cond 同步通知
```go
mutx := sync.Mutex{}
//创建cond传入锁的指针
cond := sync.NewCond(&mutx)
cond.Wait()//等待
cond.Signal()//通知单个g
```
## 原子操作
```go
var count int
atomic.AddInt64(&count, 1) //原子的增加值
atomic.StoreInt64(&count,2)//原子地将一个 `int64` 类型的值存储到指定的内存地址中
atomic.LoadInt64(&count)//原子地加载指定内存地址的值
```
## 反射
```go
var c = "3"
a1 := reflect.TypeOf(a)
a2 := reflect.ValueOf(a)
fmt.Println("d2-canset:", d2.CanSet())//判断是否可以设置值
fmt.Println("d2-type:", d2.Type())
fmt.Println("d2-kind:", d2.Kind())//获取源类型
d3 := d1.Elem()//获取指针的值

```
## 泛型
增加代码的复用，不用类型需要处理相同逻辑时，比较适用泛型

泛型种类: 1.泛型函数 2.泛型类型 3.泛型结构 4.泛型结构体 5.泛型receiver

泛型限制:1.匿名结构体与匿名函数不支持泛型 2.不支持类型断言

不支持泛型方法，只能通过recevier来实现方法的泛型处理 4. ~后的类型必须为基本类型，不能为结构类型

定义一个泛型函数 泛型 1.函数名[T int|float](){}  2.函数名[T interface{int|float}](){}
```go
//泛型函数
func getMaxNum[T int | float64](a, b T) T { //比较最大值的泛型番薯
	if a > b {
		return a
	}
	return b
}
// 泛型结构体
type Student[T interface{ *int | *string }] struct {
	name string
	data T
}
// 泛型函数 any类型
func printAnyType[T any](a T) {
	fmt.Println("any:", a)
}
// 自定义泛型
type CustomT interface {
	//~  表示支持类型的衍生类型
	// | 表示取并集
	// 多行取交集
	int | ~int64
	uint8 | ~int32
}

//进行手动约束
fmt.Println(getMaxNum[int](1, 2))
//编译器推断
fmt.Println(getMaxNum(2, 3))

//any 泛型
printAnyType[int](1)
printAnyType[string]("avb")
printAnyType[bool](true)

var data string = "你好"
//泛型结构体
stu := &Student[*string]{name: "张三", data: &data}
```

## rpc
像调用本地函数一样，调用远程函数。
### 服务端
1. 注册rpc服务对象,给对象绑定方法（1.定义类 2.绑定类方法）
```go
rpc.RegisterName("服务名",回调对象)
```
2. 创建监听器
```go
listener,err:=net.Listen()
```
3. 建立连接
```go
conn,err:=listener.Accpet()
```
4. 将连接绑定rpc服务
```go
rpc.ServeConn(conn)
```
### 客户端
1. 用rpc连接服务器
```go
conn,err:=rpc.Dial()
```
2. 调用远程函数
```go
conn.Call("服务名.方法名",传入参数，传出参数)
```
### rpc相关函数
1. 注册服务
```go
func RegisterName(name string, rcvr interface{}) error
	参1：服务名。字符串类型
	参2：对应rpc对象。该对象绑定方法要满足如下条件：
		1）方法必须是导出的。 ----指包外可见。首字母大写
		2）方法必须有两个参数，都是导出类型、内奸类型
		3）方法第二个参数必须是“指针”（传出参数）
		4）方法只有一个error接口接口类型的返回值
	举例：
	type World struct{
	}
	func (w *World)HelloWorld(name string,resp *string)error{
	}
	注册服务:
	rpc.RegisterName("服务名",new(World))
```
2. 绑定服务
```go
func ServeConn(conn io.ReadWriteCloser)
	-- conn: 成功就建立好连接的socket
rpc.ServeConn(conn)
```
3. 调用远程函数
```go
func (client *Client) Call(serviceMethod string, args interface{}, reply interface{}) error
	serviceMethod:"服务名.方法名"
	args:传入参数。方法需要的数据
	reply:传出参数。定义var变量,&变量名  完成传参
```
### protobuf
1. message成员编号，可以不从1开始但是哦不能重复，编号不能使用19000-19999
2. 可以使用message嵌套
3. 定义数组、切片使用repeated关键字
4. 可以使用枚举，编号必须从0开始
5. 可以使用联合体。oneof关键字，成员编号不能重复。
```go
//定义包名
package pb;
//定义枚举类型
enum Week{
	Monday=0;//枚举值必须从 0 开始
	Turesday=1;
}
//定义消息体
message Student{
	int32 age=1;//可以不从 1 开始，但不能重复
	string name=2;
    People p=3;//消息体可以进行组合
	repeated int32 score=4;//定义数组类型
	
	//枚举
	Week w=5;
	//联合体
	oneof data{
		string teacher=6;
		string class=7;
	}
}
//消息体可以进行嵌套
message People{
	int32 weight=1;
}
```

