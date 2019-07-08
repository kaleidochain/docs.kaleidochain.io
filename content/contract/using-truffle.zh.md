---
title: "使用Truffle开发合约"
weight: 30
---

####  合约开发与部署
 kaleido合约开发部署完全兼容以太坊,并有更丰富的功能模块,合约开发部署推荐使用truffle框架。

#### 1 环境安装
```js
//nodejs 8.12.0+版本安装
略
//truffle框架4.1.14版本安装
npm install -g truffle@4.1.14
```
#### 2 truffle项目初始化
```
git clone https://github.com/truffle-box/bare-box.git
```
#### 3 项目配置节点
在 truffle-config.js文件找到development,删除注释,配置节点ip,port:
```
// development: {
    //  host: "127.0.0.1",     // kaleido节点ip,可以使用:106.75.184.214公共节点,也可以是自己搭建的节点)
    //  port: 8545,            // 节点rcp服务端口(默认为:8545)
    //  network_id: "*",       // 网络id,默认即可
    // },
```
#### 4 写入合约代码
cd contracts/;vim Game.sol;
```js
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
#### 5 新建合约部署脚本
cd migrations;vim 2_initial_game.js;
```js
const Game = artifacts.require("Game");
module.exports = function(deployer) {
    deployer.deploy(Game,"构造函数参数1");
};
    
```
#### 6 合约部署

```
truffle migrate --reset --network development
```
#### 7 kalgo控制台测试
 调用合约前需要确认合约创建者是否有足够抵押(创建者账户上Kal+抵押合约创建者Kal),否则合约不能被调用,抵押步骤见第2节。
在build/contracts/Game.json拷贝abi,使用kalgo客户端连上节点:

kalgo attach http://127.0.0.1:8545;
```js
var gameabi = [{"constant":true,"inputs":[],"name":"creator","outputs":[{"name":"","type":"address"}],"payable":false,"stateMutability":"view","type":"function"},{"constant":false,"inputs":[],"name":"kill","outputs":[],"payable":false,"stateMutability":"nonpayable","type":"function"},{"constant":false,"inputs":[{"name":"_newgreeting","type":"string"}],"name":"setGreeting","outputs":[],"payable":false,"stateMutability":"nonpayable","type":"function"},{"constant":true,"inputs":[],"name":"greet","outputs":[{"name":"","type":"string"}],"payable":false,"stateMutability":"view","type":"function"},{"inputs":[{"name":"_greeting","type":"string"}],"payable":false,"stateMutability":"nonpayable","type":"constructor"}];
var gameContract= web3.eth.contract(gameabi).at(gameAddress);

//显示:构造函数参数1
game.greet();
```

#### 8 合约创建者

 与以太坊不一样处在kaleido链上,每个合约的创建者都会被记录在系统合约ConCreator("0x1000....0001");有两种方式可以获取合约的创建者

>8.1 在kalgo控制台,通过系统合约可以查询合约创建者:
```js
var concreatorabi = [{"constant":true,"inputs":[{"name":"conAddr","type":"address"}],"name":"get","outputs":[{"name":"","type":"address"}],"payable":false,"stateMutability":"view","type":"function"}];
var concreator = web3.eth.contract(concreatorabi).at("0x1000000000000000000000000000000000000001");

//查询合约(contractAddress)创建者
concreator.get(contractAddress);
```
>8.2 在solidity合约中查询合约创建者
```js
contract ConCreatorInterface {
	function get(address conAddr) public view returns(address);
}
ConCreatorInterface concreator = ConCreatorInterface(0x1000000000000000000000000000000000000001);

//查询合约(contractAddress)创建者
concreator.get(contractAddress);
```