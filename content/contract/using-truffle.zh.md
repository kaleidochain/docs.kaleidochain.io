---
title: "使用Truffle开发合约"
weight: 3
---

除了通过geth命令行进行部署外也可以通过truffle环境进行部署。

首先安装truffle：

```bash
npm install –g truffle@4.1.14
```

初始化truffle项目：

```bash
mkdir bare-box
cd bare-box
truffle init
```

并配置节点信息：

在 truffle-config.js文件找到development,删除注释,配置节点ip,port为你要连接的节点的信息。

```
development: {
    host: "127.0.0.1",     // Localhost (default: none)
    port: 8545,            // Standard Ethereum port (default: none)
    network_id: "*",       // Any network (default: none)
},
```

在目录`contracts`下撰写合约文件`Game.sol`（注: 文件名称必须与合约名称相同），示例合约内容如下：

```solidity
pragma solidity ^0.4.0;

contract Game
{
	address public creator;
	string greeting;

	function Game(string _greeting) public payable {
		creator = msg.sender;
		greeting = _greeting;
	}

	function greet() constant returns (string){
		return greeting;
	}

	function setGreeting(string _newgreeting){
		greeting = _newgreeting;
	}

	function kill(){
		require(msg.sender == creator,"not creator");
		suicide(creator);
	}
}
```

在目录`migrations`下编写合约部署脚本`2_initial_game.js`，示例内容如下：

```javascript
const Game = artifacts.require("Game");

module.exports = function(deployer) {
    deployer.deploy(Game,"Hello world!");
};
```

最后，调用truffle命令一键编译部署合约:

```bash
truffle migrate --reset;
```

