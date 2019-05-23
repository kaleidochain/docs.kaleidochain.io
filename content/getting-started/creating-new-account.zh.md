---
title: "创建新的账户"
date: 2019-05-23T12:08:28+08:00
weight: 2
---

如果你还没有账户，可以按照本指引创建一个属于你自己的账户。这里介绍了通过成熟可信的钱包工具生成和命令行生成的方法。

### 使用钱包客户端生成账户（推荐）

你可以使用现有的以太坊钱包，连接到kaleido区块链网络后，创建新的账户。

下面以MetaMask为例，说明连接kaleido测试网络和创建账户的流程。

在MetaMask中，点击右上角的下拉菜单。在打开的列表中，点击最底部的`Custom RPC`。在`New Network`文本框中，点击`Show Advanced Options`后，显示出4个文本框，分别填写如下信息：

- New RPC URL: `http://testnet.kaleidochain.io:8545`
- ChainID: `889`
- Symbol: `KAL`
- Nickname: `Kaleido Test Network`

填写后如下图。

最后点击旁边的`Save`按钮。

当最上方的网络名称栏显示`Kaleido Test Network`即表示连接成功。

然后在MetaMask中创建账户即可。一旦你有了token，就可以方便的使用钱包进行转账了。

### 使用命令行生成账户

在运行你自己的节点之后，你还可以通过命令行生成一个新账户，具体如下。将其中的`your-password`替换为你自己的密码即可。

```bash
docker exec -it kalnode geth --testnet attach
> personal.newAccount('your-password')
"0xe40046ef6f0d4a05d90ca62d8ead47e21c886fc1"
> eth.accounts
["0xe40046ef6f0d4a05d90ca62d8ead47e21c886fc1"]
```
