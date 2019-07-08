---
title: "获取随机数"
weight: 1000
---

区块链上的合约经常会需要产生随机值，目前的区块链一般采用交易hash或者区块hash产生随机值，但是这些都不是完全独立的随机值，可以被矿工或者交易发起者进行攻击或者利用。

Kaleido提供了真正不可预测的安全可靠的随机数机制。Kaleido的随机性主要有以下几个来源：

1. **区块种子** 受益于VRF算法和Algorand共识算法机制，每个区块中的随机数种子是无法被任何人（包括提案人自己）提前预测的
2. **交易Hash** 发起获取随机数的交易hash
3. **内部序列号** 区块内部使用种子的次数，除了区块构造者，其他人无法提取预测
4. **用户数据** 由调用者传入的任意数据，可选

因此，Kaleido所产生的随机数，是一个真正不可预测的安全可靠的随机数。 


你可以通过内置函数直接获取随机数，代码如下：

```solidity
function random() public view returns(uint256) {
    uint256[1] memory output;
    assembly {
        if iszero(staticcall(not(0), 101, 0x0, 0x0, output, 0x20)) {
            revert(0, 0)
        }
    }
    return output[0];
}
```

也可以获取过去某个区块的随机种子（限制最多能取到最近2048个区块）：

```solidity
function getSeed(uint height) public view returns(uint256) {
    uint256[1] memory input;
    uint256[1] memory output;
	input[0] = height;
    assembly {
        if iszero(staticcall(not(0), 100, input, 0x20, output, 0x20)) {
            revert(0, 0)
        }
    }
    return output[0];
}
```

以上函数已经在有封装好的函数库可以直接调用，具体可参考`SysContract.sol`。
