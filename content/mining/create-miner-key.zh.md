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
--miner.stakeowner 0x958DE277Cde7f5808a910dBf6f7854DF52C25833 \
--minerkey.coinbase 0x958DE277Cde7f5808a910dBf6f7854DF52C25833 \
--minerkey.start 12100
```

以上命令执行需要一定的时间，完成后输出结果最后内容如下，这里给出了生成的minerkey，其内部包含了受益人coinbase的地址、起始高度begin、区间的结束高度end等信息。

```
MinerKey: 0x39fb25e90000000000000000000000000000000000000000000000000000000000002f440000000000000000000000000000000000000000000000000000000000000064000000000000000000000000958de277cde7f5808a910dbf6f7854df52c2583355094cc911dca3b0ff42e63e7d4d627d5131307fcd495b535de4fa627bb999bfb728eed47ed4b7426c46e8b57c4132dc89e4092463267040c7c6769a80e72c69
Details:
	miner = 0x958DE277Cde7f5808a910dBf6f7854DF52C25833
	coinbase = 0x958DE277Cde7f5808a910dBf6f7854DF52C25833
	start = 12100
	end = 1000000
	lifespan = 100
	vrfVerifier = 0x55094cc911dca3b0ff42e63e7d4d627d5131307fcd495b535de4fa627bb999bf
	voteVerfier = 0xb728eed47ed4b7426c46e8b57c4132dc89e4092463267040c7c6769a80e72c69
```

以上输出中的`0x39fb25...`就是挖矿凭证`minerkey`，其作用相当于公钥。对应的私钥数据存储在节点的数据目录`$KALEIDO_HOME/testnet/kalgo/minerkeys`下，注意不要公开或泄露该私钥数据。

生成minerkey有以下几点需要注意：

1. 必须提前100个区块注册：你如果想生成start=H的minerkey，就必须在H-100区块高度之前，完成注册
2. 区块高度一旦到达你注册区间起始位置的100个区块以内，minerkey就无法修改
3. 受益人`Coinbase`是可以随时修改，并在修改成功的下个区块立即生效，不受第2点的限制
4. 更新minerkey时，新生成的区间的起始位置必须与要修改的minerkey一致，否则无法更新成功
5. 你最多只能提前注册一个区间，即`minerkey.start`必须小于当前区块高度+1000000

如果私钥不慎遗失，并且此时minerkey无法修改，你可以将钱转入新的地址后，重新注册成为矿工即可。

