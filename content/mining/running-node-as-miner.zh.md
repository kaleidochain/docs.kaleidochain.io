---
title: "运行挖矿节点"
date: 2019-05-23T17:49:25+08:00
weight: 3
---

一旦注册好矿工合约，将投票权成功委托给了挖矿凭证，那么就可以使用该挖矿凭证挖矿了。加上参数`--mine --etherbase <miner addr>`重新启动前面的全节点即可。

注意这里的`--etherbase`的值，应该写你刚才发起矿工注册的交易的账号地址（而不是收益地址）。挖矿程序会使用该地址，在挖矿合约中查找你注册的挖矿信息。

```bash
docker stop kalnode && docker rm kalnode

docker run -d --name kalnode -v $KALEIDO_HOME:/root/.ethereum \
-p 38883:38883 -p 38883:38883/udp \
kaleidochain/kalgo --testnet \
--mine --etherbase 'your-address'
```

可以使用以下命令查询验证你的节点是否正在挖矿：

```bash
docker exec -it kalnode geth --testnet attach
> eth.mining
true
```

如果仍然没有开始挖矿，可能有几个原因：

1. minerkey没有注册成功
2. 节点启动时的参数没有设置，或者没有与minerkey匹配
3. 没有token，或只有少于1个单位的token
4. 区块高度还没有到你注册的区间范围内
5. minerkey对应的私钥数据目录不存在或不匹配

如有以上问题，可以查阅之前的文档，重新操作一次。或者将相关信息和问题发到社区中寻求帮助。

### 持续挖矿

由于挖矿凭证是有区块范围的，区块高度超过相应范围后，之前的挖矿凭证就失效了。你需要提前为下一个区间范围注册好另一个挖矿凭证，以保持持续不间断的挖矿。

为避免矿工合约数据过大，矿工合约限制了最多能提前注册未来1个区间。
你可以提前为未来1个区间生成minerkey并提前注册，并预估好这几个区间跑完的时间，设置好提醒方式，以提醒你再次注册。
在测试网络中，1个区间大约可以跑24天。你可以设置每2~3周来检查或注册一次。

