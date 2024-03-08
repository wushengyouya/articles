---
author: 吴生有崖
title: go语言-4
date: 2023-01-29
description: 
image: image.jpg
categories:
  - 编程
tags:
  - go
---

## rpc
0. rpc封装
```go
// rpc服务端封装
type MyInterface interface {
	HelloWorld(string, *string) error
}

func RegisterService(i MyInterface) {
	rpc.RegisterName("World", i)
}

// rpc客服端封装,像调用本地函数一样调用远程函数
type MyClient struct {
	c *rpc.Client
}

func InitClient(addr string) MyClient {
	conn, _ := jsonrpc.Dial("tcp", addr)
	return MyClient{conn}
}

func (this *MyClient) HelloWorld(a string, b *string) error {
	return this.c.Call("World.HelloWorld", a, b)
}
```
1. rpc服务端
```go
type World struct {
}
func (w *World) HelloWorld(name string, resp *string) error {
	*resp = name + "你好"
	return nil
}
//1.注册服务
err := rpc.RegisterName("World", new(World))
if err != nil {
    fmt.Println("register err>", err)
}
//2.设置监听地址
listener, err := net.Listen("tcp", "127.0.0.1:8089")
if err != nil {
    fmt.Println("listen err:", err)
}
//3.监听连接
fmt.Println("开始监听...")
conn, err := listener.Accept()
if err != nil {
    fmt.Println("accept err>", err)
}
//4.绑定连接
jsonrpc.ServeConn(conn)
```
2. rpc客服端
```go
//1.连接服务
conn, err := jsonrpc.Dial("tcp", "127.0.0.1:8089")
//2.调用服务器方法
var resp string
err = conn.Call("World.HelloWorld", "李白", &resp)
//err := client.HelloWorld("李白 ", &resp)
```
