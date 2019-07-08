---
title: "合约创建人"
date: 2019-05-23T18:29:20+08:00
weight: 100
---

在Kaleido Chain中，系统内置了一个系统合约，记录了所有合约的创建人地址。创建人地址就是部署合约的外部账号地址。如果一个合约A是由另一个合约B部署的，合约A的创建人地址就自动继承合约B的创建人地址。该创建人地址在合约的构造函数中也可以成功获取。

合约创建人有多种用途。第一种用途就是合约抵押机制。合约要可执行，需要先抵押一定量的token到其创建人账户中。系统自动识别合约的创建人抵押量，来计算合约可被调用的次数。第二种用途就是权限管理。很多情况下，一个合约都需要与一个超级管理员关联，以限制部分接口的调用权限。合约创建人机制默认提供了一直超级管理员的可选项，简化了相关合约的开发。

要在Solidity中获取一个合约的创建人，需要使用以下代码：

```solidity
import 'lib/kaleido/SysContract.sol';

contract Demo {
    // WithDraw - only creator can get the token
    function WithDraw(uint amount) public {
        require(msg.sender == SysContract.getCreator(address(this)));
        msg.sender.transfer(amount);
    }
}
```




