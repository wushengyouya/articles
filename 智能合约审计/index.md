---
author: 吴生有涯
title: 智能合约审计
date: 2024-09-12
description: 
image: 
categories:
  - 编程
tags:
  - solidity
  - foundry
---

# 智能合约审计
- Foundry Fuzzing = Stateless fuzzing

- Foundry Invariant = Stateful fuzzing
## 有状态模糊测试(Foundry Invariant)
- 在多次测试运行之间保持合同状态
- 允许在一系列操作中测试复杂场景
- 适用于检查跨多个操作应该始终保持不变的属性
- 需要更多的设置，但可以发现更深层次的问题
## 无状态模糊测试(Foundry fuzzing)
- 每次测试运行都会创建一个新的合约实例
- 不保留任何状态
- 适用于单独测试各个函数
## 不变性测试
1. 确定不变量
2. 
不变性是应该在模糊测试活动期间始终成立的条件表达式。
- 对于Uniswap,x*y=k 公式始终成立
- 对于ERC-20代币，所有用户的余额等于 totalSupply()总供应量
## 模糊测试
```solidity
    // SPDX-License-Identifier: MIT
    pragma solidity ^0.8.23;

    import "@thirdweb-dev/contracts/base/ERC721Drop.sol";

    contract NFTDrop is ERC721Drop {
        constructor(
            string memory _name,
            string memory _symbol,
            address _royaltyRecipient,
            uint128 _royaltyBps,
            address _primarySaleRecipient
        )
            ERC721Drop(
                _name,
                _symbol,
                _royaltyRecipient,
                _royaltyBps,
                _primarySaleRecipient
            )
        {}

        function mint(address _to, uint256 _amount) external {
            require(_amount > 0, "You must mint at least one token!");
            _safeMint(_to, _amount);
        }
    }

    /************************************************************/
 

```
```soldity
   // SPDX-License-Identifier: MIT
pragma solidity ^0.8.23;

import "forge-std/Test.sol";
import "./NFTDrop.sol";

contract NFTDropTest is Test {
    NFTDrop drop;
    address testAddr;

    function setUp() public {
        drop = new NFTDrop("TestToken", "TEST", address(this), 0, address(this));
        testAddr = address(this);
    }

    function testMint(uint16 amount) public {
        vm.expectRevert("You must mint at least one token!");
        drop.mint(testAddr, amount);
        assertEq(drop.balanceOf(testAddr), amount);
    }
}

```

## The Rekt Test
1. Do you have all actors,roles,and privileges documented?
2. Do you keep documentation of all the external services,contracts,and oracles you rely on?
3. Do you have a written and tested incident response plan？
4. Do you document the best ways to attack your system?
5. Do you perform identity verification and backgroud checks on all employees?
6. Do you have a team member with security defined in their role?
7. Do you require hardware security keys for production systems?
8. Does your key management system require multiple humans and physical steps?
9. Do you define key invariants for your system and test them on every commit?
10. Do you use the best automated tools to discover security issues in you code?
11. Do you undergo external audits and maintain a vulnerability disclosure or bug bounty program?
12. Have you considerd and mitigated avenues for abusing users of you system?

