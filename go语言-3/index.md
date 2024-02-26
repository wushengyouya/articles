---
author: 吴生有崖
title: go语言-3
date: 2024-02-25
description: 
image: image.jpg
categories:
  - 编程
tags:
  - go
---
## chan
用在go协程之间的通信
- 创建一个有缓冲区的管道,有点类似于数组但不能用下标进行访问
- 无缓冲区读写堵塞，nil管道读写堵塞,缓冲区无数据读堵塞，缓冲区写满时写堵塞
- 关闭已经关闭的管道会崩溃，向已经关闭的管道写数据会崩溃,管道读写必须对等,
堵塞在主go程会崩溃，子go程会内存泄漏

```go
//1.创建chan
numChan := make(chan int, 6) //有缓冲区
//map slice  chan 使用时都需要先make,默认创建初始状态都是nil
numChan2 := make(chan int) //无缓冲的管道

var testChan chan int
testChan = make(chan int, 1)

//关闭通道
close(testChan)
v, ok := <-testChan//判断是否已关闭,实际是判断缓冲区里面是否还有数据

//2.读写
//写
numChan2 <- 100
//读
go func() {
		fmt.Println("numChan2有缓冲区的管道读取数据,>>", <-numChan2)
}()
```
### 遍历chan
```go
    numChan := make(chan int, 2)
	go func() {
		for i := 0; i < 10; i++ {
			numChan <- i
		}
		fmt.Println("数据写入完毕")
		close(numChan) //关闭管道放在写入端,读取端不知道什么时候会没有数据，已经关闭的关闭可以读取数据
		fmt.Println("关闭管道")
	}()
	for v := range numChan {
		fmt.Println(v)
		if v == 0 {
			fmt.Println(111)
		}
	}
```

### 单向chan
```go
numChan:=make(chan int,5) //双向通道
var numChan <- chan int //只读管道
var numChan chan <- int //只写管道

//只写chan
func producer(out chan<- int) {
	for i := 0; i < 5; i++ {
		//data := <-out 只写管道不支持读取操作
		out <- i
		//fmt.Println(data)
	}
	fmt.Println("数据写入完毕")
}
//只读chan
func consumer(in <-chan int) {
	for i := 0; i < 5; i++ {
		data := <-in
		//in<-i 不支持写入操作
		fmt.Println("读取到数据-", data)
	}
	fmt.Println("读取数据完成")
}
```

### select监听chan
```go
for {
    select {
    case data1 := <-numChan1:
        fmt.Println("numChan1读取数据，", data1)
        time.Sleep(1 * time.Second)
    case data2 := <-numChan2:
        fmt.Println("numChan2读取数据,", data2)
        time.Sleep(1 * time.Second)
    default:
        fmt.Println("其他管道")
        time.Sleep(1 * time.Second)
    }
}
```
## Socket
1. 服务端
```go
//1.定义监听端口Ip
ip := "127.0.0.1"
port := "8089"
addr := fmt.Sprintf("%s:%s", ip, port)   //拼接字符串
//2.开始监听
listener, err := net.Listen("tcp", addr)
fmt.Println("监听中...")
if err != nil {
    fmt.Println("listen err>", err)
}
//3.获取连接
for {
    conn, err := listener.Accept() //2.建立连接
    fmt.Println("连接建立成功")
    if err != nil {
        fmt.Println("accept err>", err)
    }
    go handle(conn)
}
//4.读取数据
func handle(conn net.Conn) {
	buf := make([]byte, 1024) //make([]byte,1024)
	n, err := conn.Read(buf)  //3.读取数据

	if err != nil {
		fmt.Println("read err>", err)
	}
	fmt.Println("读取数据client>>>server,len=", n, "data=", string(buf[0:n]))
	msg := strings.ToUpper(string(buf[0:n]))
	_, err = conn.Write([]byte(msg)) //5.写入数据
	if err != nil {
		fmt.Println("写入失败,", err)
	}
}

```
2. 客户端
```go
ip := "127.0.0.1"
port := "8089"
addr := fmt.Sprintf("%s:%s", ip, port)
conn, err := net.Dial("tcp", addr)
```
## httpClient
```go
//1.创建http客服端
httpclient := http.Client{}
//2.配置请求地址
resp, err := httpclient.Get("http://localhost:8089/user")
if err != nil {
    fmt.Println("get err-", err)
}
//3.获取响应体
body := resp.Body
//4.读取数据
respStr, err := io.ReadAll(body)
if err != nil {
    fmt.Println("readall err>", err)
}
//获取响应头
server := resp.Header.Get("server")
date := resp.Header.Get("Date")
//http协议参数
fmt.Println(resp.StatusCode, resp.Request.URL, resp.Request.Method, resp.Request.ContentLength, resp.Status)
```
## httpServer
```go
//1.设置监听路由
http.HandleFunc("/user", func(writer http.ResponseWriter, request *http.Request) {
		_, _ = io.WriteString(writer, "<html><body><div>你好</div></body></html>")
})
//2.开始监听
err := http.ListenAndServe("127.0.0.1:8089", nil) //监听并且提供服务ListenAndServe
if err != nil {
    fmt.Println("server start err>", err)
}
```