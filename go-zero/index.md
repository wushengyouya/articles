---
author: 吴生有涯
title: go-zero
date: 2024-6-19
description: 
image: go-zero调用关系.png
categories:
  - 编程
tags:
  - go
---
# 安装protobuf组件
- go-zero框架使用goctl命令安装  
```sh
    # 1.安装goctl
    go install github.com/zeromicro/go-zero/tools/goctl@latest
    ```sh
        ##执行goctl检查是否安装陈工
        goctl -v
    ```
    # 2.使用goctl命令 一键安装protoc，protoc-gen-go，protoc-gen-go-grpc
    goctl env check --install --verbose --force
    ## --force 静默安装
    ## --verbose 输出执行日志
    # 检查环境安装是否成功
    goctl env check --verbose
```

- goctl安装protoc不成功,从 [protoc-github仓库地址](https://github.com/protocolbuffers/protobuf)下载对应的系统版本，
放入gopath目录中
```sh
    #执行命令检查是否安装陈工
    protoc
```

