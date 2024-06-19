---
author: 吴生有涯
title: npm版本bug
date: 2024-6-19
description: 
image: image-1.png
categories:
  - 编程
tags:
  - 其它
---
# npm install 执行出现的问题
## Error: error:0308010C:digital envelope routines::unsupported
[npm版本问题，解决方案一般是这两个](https://stackoverflow.com/questions/69692842/error-message-error0308010cdigital-envelope-routinesunsupported)

1. 执行命令
```bash
# linux:
set NODE_OPTIONS=--openssl-legacy-provider  
# windows
export NODE_OPTIONS=--openssl-legacy-provider
```
2. 修改package.json文件
```json
//加入这两句
"scripts":{
    "serve": "SET NODE_OPTIONS=--openssl-legacy-provider && vue-cli-service serve",
    "build": "SET NODE_OPTIONS=--openssl-legacy-provider && vue-cli-service build"
}
```
3. 安装命令
```bash
    npm install --registry https://registry.npmmirror.com
```
 