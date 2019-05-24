---
title: "权限管理合约"
weight: 5
---

kaleido扩展了合约的权限管理功能。通过权限管理合约，可以实现：

1. 用户调用产生的gas消耗由合约支付
2. 合约调用黑名单，禁止某些用户调用合约

合约创建者可以控制调用权限和每次的调用花费上限。

下面的示例说明这种合约的使用方式。


### 环境准备

参考[使用Truffle开发合约](/contract/using-truffle/)中的内容，配置好开发环境。

### 编写合约

游戏合约指调用合约的手续费用由合约自己支付，调用者（用户）不需要支付Kal，这样使得游戏能被所有用户都使用。为了达到这个目的需要将游戏合约注册到系统合约里面，系统合约地址为sysytem(0x1000000000000000000000000000000000000001)，当注册到这个系统合约后，该合约就会在调用的时候从合约地址支付，如果合约地址中Kal为0，调用就会失败。
游戏合约的手续费因为由合约本身支付，所以为了防止无效支付和支付过高手续费用，游戏合约需要和权限合约配合使用，通过权限合约可以控制用户调用的权限，同时通过在权限合约中指定gas_limit和gas_price作为调用合约交易中的上限。权限合约为Authority.sol。
权限合约可以在多个合约之间共享。
在实际使用中需要把游戏合约地址和对应的权限合约地址注册到系统合约中。 
system. setAuthContractAddr(address(authority));


在权限合约中设置对应的gas_limit和gas_price：
authority.setGas(500000000,1000000);

在权限合约中将用户添加到白名单:
authority.grant(addr);

在权限合约中将用户移除白名单,白名单外的用户调用game合约的交易是不会被区块打包:
authority.revoke(addr);

便于白名单合约管理,将拥有者权限移交给用户账号:
authority.changeOwner(msg.sender);

普通逻辑合约Game.sol例子:

```solidity
pragma solidity ^0.4.0;
contract AuthorityInterface{
	function setPayer() public;
	function setGas(uint256 price, uint64 gaslimit) public;
	function grant(address addr)public;
	function revoke(address addr) public;
}
contract AuthIndexInterface{
	function setAuthContractAddr(address add) public;
	function getAuthContractAddr(address add) public view returns(address); 
}
contract Game{
	address public creator;
	string greeting;
	AuthIndexInterface AuthIndex = AuthIndexInterface(0x1000000000000000000000000000000000000001);
	AuthorityInterface Authority;
	function Game(string _greeting) public payable{
		creator = msg.sender;
		greeting = _greeting;
	}

	function () payable {}
	function setauthority(address addr) returns(bool){
		require(msg.sender == creator,"not creator");
		Authority = AuthorityInterface(addr);
		AuthIndex.setAuthContractAddr(addr);
		Authority.setGas(10000000000,1000000);
		Authority.grant(msg.sender);
		return true;
	}
	function grant(address addr) returns(bool){
		require(msg.sender == creator,"not creator");
		Authority.grant(addr);
		return true;
	}
	function revoke(address addr) returns(bool){
		require(msg.sender == creator,"not creator");
		Authority.revoke(addr);
		return true;
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

部署权限管理合约Authority.sol，该合约代码是kaleido内置的，不能随意修改。

```solidity
pragma solidity ^0.4.20;

contract Authority {
	enum Payer {
		NOTSET,     
		SELF        
	}
	enum Permission {
		NOTSET,     
		PERMIT,     
		FORBID      
	}
	uint    version;   
	address owner;      
	address next;       
	mapping(address => uint256) addr2opauth;    
	mapping(address => uint256) addr2payer;    
	mapping(address => uint256) addr2gasprice;  
	mapping(address => uint64)  addr2gaslimit;  
	mapping(address => uint256) addr2auth;     

	constructor() public {
		owner = msg.sender;
		next = address(0);
		version = 0x01;
	}

	function changeOwner(address newOwner) public {
		assert(msg.sender == owner);
		owner = newOwner;
	}

	function getOwner() public view returns(address) {
		return owner;
	}

	function setNewAuthority(address newNext) public {
		assert(msg.sender == owner);
		next = newNext;
	}

	function getNextAuthority() public view returns(address) {
		return next;
	}

	function grantContractAuth(address addr) public {
		assert(msg.sender == owner);

		addr2opauth[addr] = uint(Permission.PERMIT);
	}

	function revokeContractAuth(address addr) public {
		assert(msg.sender == owner);

		addr2opauth[addr] = uint(Permission.FORBID);
		delete addr2opauth[addr];
	}

	function setPayer() public {
		addr2payer[msg.sender] = uint256(Payer.SELF);
	}

	function getPayer(address contractAddr) public view returns(uint256){
		return addr2payer[contractAddr];
	}

	function setGas(uint256 price, uint64 gaslimit) public {
		addr2payer[msg.sender] = uint(Payer.SELF);
		addr2gasprice[msg.sender] = price;
		addr2gaslimit[msg.sender] = gaslimit;
	}

	function getGas(address contractAddr) public view returns(uint256, uint64){
		return (addr2gasprice[contractAddr], addr2gaslimit[contractAddr]); 
	}

	function getContractInfo(address contractAddr) public view returns(uint256, uint256, uint256, uint64){
		return (addr2opauth[contractAddr], addr2payer[contractAddr], addr2gasprice[contractAddr], addr2gaslimit[contractAddr]);
	}

	function grant(address addr) public {
		addr2auth[addr] = uint(Permission.PERMIT);
	}

	function revoke(address addr) public {
		require(uint(Permission.PERMIT) == addr2opauth[msg.sender]);
		addr2auth[addr] = uint(Permission.FORBID);
		delete addr2auth[addr];
	}

	function getAuth(address addr) public view returns(uint256) {
		return addr2auth[addr];
	}
}
```

### 部署合约

编写合约部署脚本2_initial_game.js，其中`deployer.deploy(Game,"param1",{value:100000000000000000})`中转入合约的Kal用于合约调用费用。

```javascript
const Game = artifacts.require("Game");
const authority = artifacts.require("Authority");
module.exports = function(deployer){
var gameToken,authToken;
deployer.then(function(){
return deployer.deploy(Game,"param1",{value:100000000000000000}); 
}).then(function(result){
  gameToken = result;
return deployer.deploy(authority);
}).then(function(result){
authToken = result;
}).then(function(result){
gameToken.setauthority(authToken.address);
authToken.grantContractAuth(gameToken.address);
});
}
```

编译部署：

```
truffle migrate --reset
```

### 测试

使用命令行连接到测试节点，执行以下代码：

```javascript
var gameabi = [{"constant":true,"inputs":[],"name":"creator","outputs":[{"name":"","type":"address"}],"payable":false,"stateMutability":"view","type":"function"},{"constant":false,"inputs":[{"name":"addr","type":"address"}],"name":"grant","outputs":[{"name":"","type":"bool"}],"payable":false,"stateMutability":"nonpayable","type":"function"},{"constant":false,"inputs":[{"name":"addr","type":"address"}],"name":"revoke","outputs":[{"name":"","type":"bool"}],"payable":false,"stateMutability":"nonpayable","type":"function"},{"constant":true,"inputs":[],"name":"greet","outputs":[{"name":"","type":"string"}],"payable":false,"stateMutability":"view","type":"function"},{"constant":false,"inputs":[{"name":"_newgreeting","type":"string"}],"name":"setGreeting","outputs":[],"payable":false,"stateMutability":"nonpayable","type":"function"}];
var game= web3.eth.contract(gameabi).at("game合约地址");
//只读函数都能成功返回
game.greet({from:eth.accounts[0]});
game.greet({from:eth.accounts[1]});

//success
game.setGreeting("test",{from:game.creator()});

//failure:contract permission denied.
game.setGreeting("test",{from:eth.accounts[1]});

//加入白名单
game.grant(eth.accounts[1],{from:game.creator()});

//success
game.setGreeting("test",{from:eth.accounts[1]});
//移除白名单
game.revoke(eth.accounts[1],{from:game.creator()});

//failure:contract permission denied.
game.setGreeting("test",{from:eth.accounts[1]});
```

