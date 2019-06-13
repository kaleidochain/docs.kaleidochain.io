---
title: "安装运行节点"
date: 2019-05-23T12:05:32+08:00
weight: 1
---

我们目前只提供基于docker的二进制版本，仅需要[安装docker](https://docs.docker.com/install/)并拉取镜像即可。

```bash
docker pull kaleidochain/kalgo
```

### 准备运行目录

首次运行，你需要准备一个数据目录，该目录可以在任意位置，但我们建议使用`$HOME/.kaleido`。为了后面使用方便，可以将该目录设置为环境变量`KALEIDO_HOME`。选取的目录将交由kaleido进行使用和管理，因此建议选择的目录下不要存储其它数据和文件。

```bash
mkdir $HOME/.kaleido
export KALEIDO_HOME=$HOME/.kaleido
echo $KALEIDO_HOME
```

后面的命令中，我们将使用环境变量`$KALEIDO_HOME`来表示该目录。

### 作为全节点启动

使用以下命令启动一个名为`kalnode`的全节点，通过参数`--testnet`使其连接到测试网络。

```bash
docker run -d --name kalnode -p 38883:38883 -p 38883:38883/udp \
-v $KALEIDO_HOME:/root/.kaleido kaleidochain/kalgo --testnet
```

启动后，该节点会连接上测试网络，下载区块数据。
我们可以通过IPC连上该节点，查看连接上的peer节点数、当前区块高度等。

```bash
docker exec -it kalnode kalgo --testnet attach
> net.peerCount
15
> eth.blockNumber
17568
> exit
```

如果你想让自己的节点也显示在[网络状态页面](http://stats-testnet.kaleidochain.io)中，增加参数`--ethstats "your-node-name:bpFe9vOevM@stats-testnet.kaleidochain.io:38881"`启动即可，其中`your-node-name`需要使用自己的节点名称来代替。

```bash
docker run -d --name kalnode -p 38883:38883 -p 38883:38883/udp \
-v $KALEIDO_HOME:/root/.kaleido kaleidochain/kalgo --testnet \
--ethstats "your-node-name:bpFe9vOevM@stats-testnet.kaleidochain.io:38881"
```

### 作为服务节点启动

如果你有钱包或其它应用，就需要在全节点的基础上，通过`--rpc`和`--ws`参数来开启RPC和WS服务，同时映射8545/8546端口到主机。
这里需要注意的是，由于节点运行在docker容器中，所以RPC和WS服务监听的IP必须是`0.0.0.0`。在服务器上运行时，应该设置合适的`rpcvhosts/wsorigins`参数，或者设置机器端口的访问权限，以避免被恶意人员访问到RPC和WS服务。

```bash
docker stop kalnode && docker rm kalnode

docker run -d --name kalnode \
-p 38883:38883 -p 38883:38883/udp -p 8545:8545 -p 8546:8546 \
-v $KALEIDO_HOME:/root/.kaleido \
kaleidochain/kalgo --testnet \
--rpc --rpcaddr 0.0.0.0 --rpcvhosts '*' \
--ws --wsaddr 0.0.0.0 --wsorigins '*'
```

我们也可以使用如下命令，直接连接RPC服务来使用。

```bash
docker exec -it kalnode kalgo attach http://127.0.0.1:8545
> eth.blockNumber
17568
> exit
```

接下来，你的钱包应用就可以选择连接上你自己的全节点了。

如果需要查看详细日志，我们可以带上日志参数`--vmodule 'p2p/discv5/*=3,p2p/discover/*=3,*=5'`重新启动全节点，然后就可以使用`docker logs`命令查看到详细日志。

```bash
docker stop kalnode && docker rm kalnode

docker run -d --name kalnode \
-p 38883:38883 -p 38883:38883/udp -p 8545:8545 -p 8546:8546 \
-v $KALEIDO_HOME:/root/.kaleido \
kaleidochain/kalgo --testnet \
--rpc --rpcaddr 0.0.0.0 --rpcvhosts '*' \
--ws --wsaddr 0.0.0.0 --wsorigins '*' \
--vmodule 'p2p/discv5/*=3,p2p/discover/*=3,*=5'

docker logs --tail=100 -f kalnode
```

### 更新版本

如有新版本，只需要更新docker镜像，删除之前的容器，重新创建新的容器即可。

```bash
docker pull kaleidochain/kalgo

docker stop kalnode && docker rm kalnode

docker run -d --name kalnode \
-p 38883:38883 -p 38883:38883/udp \
-v $KALEIDO_HOME:/root/.kaleido \
kaleidochain/kalgo --testnet

docker logs --tail=100 -f kalnode
```

