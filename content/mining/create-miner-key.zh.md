---
title: "创建挖矿凭证"
date: 2019-05-23T11:58:34+08:00
weight: 1
---

我们首先需要创建挖矿公私钥对，得到挖矿凭证`minerkey`。

假设当前区块高度为12065，通常情况下，你想尽快开始挖矿，那么你应该选择比当前区块高度略大或相等的高度值，这里我们选取12100，来生成从12100高度开始挖矿的minerkey，这个minerkey属于12100高度所在区间。这个开始高度由参数`--begin`指定。此外，在生成minerkey的时候，还要通过`--miner`指明矿工账号，`--coinbase`指明受益人的地址（即接收区块奖励的地址）。

如下命令就是用来生成一个属于`12100`高度所在区间的挖矿凭证minerkey，矿工账号`miner`和受益人`coinbase`都是`0x958DE277Cde7f5808a910dBf6f7854DF52C25833`。

```bash
docker run --rm -v $KALEIDO_HOME:/root/.kaleido \
kaleidochain/kalgo --testnet makeminerkey \
--begin 12100 \
--miner 0x958DE277Cde7f5808a910dBf6f7854DF52C25833 \
--coinbase 0x958DE277Cde7f5808a910dBf6f7854DF52C25833
```

以上命令执行需要一定的时间，完成后输出结果最后内容如下，这里给出了生成的minerkey，其内部包含了受益人coinbase的地址、起始高度begin、区间的结束高度end等信息。

```
Using following params:
	contract address: 0x1000000000000000000000000000000000000002
	miner: 0x958DE277Cde7f5808a910dBf6f7854DF52C25833
	coinbase: 0x958DE277Cde7f5808a910dBf6f7854DF52C25833
	begin: 12100
	end: 1000000
	Generate minerkey: 0x2c91689a000000000000000000000000958de277cde7f5808a910dbf6f7854df52c258330000000000000000000000000000000000000000000000000000000000000002c0bd0fe9db297d84f28da5ff851e8c7f30dab5f45ee7891311d78d7bbda6e5d878c98aab5aa50746b052e423385c497ba38066c2f6f6dc05c600a5eab6110b02f9e77403c5becfc80251c4d038e50d3a6b99bd1db88ca7547b8abc134e8235610000000000000000000000000000000000000000000000000000000000002f44
```

以上输出的最后一行中的`0x2c9168...`就是挖矿凭证`minerkey`，其作用相当于公钥。对应的私钥数据存储在节点的数据目录`$KALEIDO_HOME/testnet/kalgo/`下以`ephemeralkey`为前缀的目录，该数据通常比较大。
