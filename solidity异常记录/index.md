---
author: 吴生有涯
title: solidity异常记录
date: 2025-01-07
description: 记录一些在编写solidity中遇到的异常
image: 
categories:
  - 编程
tags:
  - solidity
  - foundry
---

## 负数转为uint256
在`solidity`中将一个负数强制转为`uint256`时,solidity会直接将其`二进制补码`解释为一个`无符号整数`

- 例如：`-1` 的二进制补码是全 1，转为`uint256`后会变成`2^256-1`
- 将负数-0.008396714242162444 ether,转为`uint256`会得到下面这个非常大的数,实际上是`2^256 - 0.008396714242162444 ether `的结果
`115792089237316195423570985008687907853269984665640564039457584007913129639935`

```js
    int256 negative_1 = -1;
    int256 negative_2 = -0.008396714242162444 ether;
    uint256 a = uint256(negative_1);
    uint256 b = uint256(negative_2);
```
