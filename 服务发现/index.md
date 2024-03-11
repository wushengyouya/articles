---
author: 吴生有崖
title: 服务发现
date: 2024-01-29
description: 
image: image.png
categories:
  - 编程
tags:
  - 服务发现
---

## consul
Consul是一个服务网格解决方案，提供了一个功能齐全的控制平面，具有服务发现、配置和分段功能。这些功能中的每一项都可以根据需要单独使用，也可以一起使用来构建一个完整的服务网格。
- 服务发现
- 健康检查
- KV存储
- 安全服务通信
- 多数据中心

### 常用命令
```bash
consul agent --dev # 开启开发者模式
```
### 服务端consul

引入consul包
```go
import "github.com/hashicorp/consul/api"
```
### 初始化服务发现consul
```go
defaultConfig := api.DefaultConfig()
client, err := api.NewClient(defaultConfig)
if err != nil {
    log.Println(err)
}
```
### 注册服务
```go
client.Agent().ServiceRegister(&api.AgentServiceRegistration{
        //需要注册的服务信息
		ID:      "1",
		Name:    "HelloService",
		Tags:    []string{"test1", "tag"},
		Address: "127.0.0.1",
		Port:    8810,
        //检查服务
		Check: &api.AgentServiceCheck{
			CheckID:  "consul check id",
			TCP:      "127.0.0.1:8810",
			Timeout:  "10s",//超时
			Interval: "10s",
		},
	})
```
### 绑定grpc
```go
    //1.创建grpc服务
	grpcServer := grpc.NewServer()
	//2.注册服务
	pb.RegisterHelloServer(grpcServer, new(Student))
	//3.监听
	listen, err := net.Listen("tcp", "127.0.0.1:8810")
	if err != nil {
		log.Println(err)
	}
	defer listen.Close()
	fmt.Println("开始监听....")
	//4.绑定监听
	grpcServer.Serve(listen)
```
## 客户端consul
### 初始化consul
```go
//初始化consul
defaultConfig := api.DefaultConfig()
c, err := api.NewClient(defaultConfig)
if err != nil {
    log.Println(err)
}
```
### 获取健康的服务
```go
//获取健康的服务
sevices, _, err := c.Health().Service("HelloService", "test1", true, nil)
if err != nil {
    log.Println(err)
}
```
### 获取服务发现已注册的ip、port
```go
//获取服务发现上的ip port
addr := fmt.Sprintf("%s:%d", sevices[0].Service.Address, sevices[0].Service.Port)
fmt.Println(addr)
```
### 注册grpc
```go
//1.连接服务器
	clientConn, err := grpc.Dial(addr, grpc.WithTransportCredentials(insecure.NewCredentials()))
	if err != nil {
		log.Println(err)
	}
//2.初始化客户端
hc := pb.NewHelloClient(clientConn)
p := &pb.Person{
    Name: "李白",
    Age:  11,
}
//3.调用
p2, err2 := hc.SayHello(context.Background(), p)
if err2 != nil {
    log.Println(err2)
}
fmt.Println(p2)
```
## etcd
etcd 是一种开源的分布式键值存储库，用于保存和管理分布式系统保持运行所需的关键信息。
```bash
#docker安装
docker run --name etcd -d -p 2379:2379  -e ALLOW_NONE_AUTHENTICATION=yes bitnami/etcd
```
### 常用命令
```bash
#设置或者更新值
etcdctl put name 张三
#获取值
etcdctl get name
#只要value
etcdctl get name --print-value-only
#获取name前缀的键值对
etcdctl get --prefix name
#删除键值对
etcdctl del name
#监听键的变化
etcdctl watch name
```
## etcd与redis区别
- redis是一种内存中的数据存储库，可用作数据库、缓存、消息代理。redis支持的数据类型要比etcd多，其读写性能也快得多
- etcd具有超强的容错能力以及更强的故障转移和持续数据可用性能力
- etcd将所有数据存储在硬盘上
- redis更实用用作分布式缓存系统，而不是存储分布式配置信息