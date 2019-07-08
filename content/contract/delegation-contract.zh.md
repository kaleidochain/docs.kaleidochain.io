---
title: "抵押机制"
weight: 45
---
kaleido使用抵押机制分配合约交易次数,防止用户恶意调用代扣费合约,堵塞网络;24小时内同一创建者的所有合约交易量不能超过创建者(以下称creator)抵押额度值,抵押额度由2个部分组成:创建者账户上Kal额度,以及系统抵押合约上其他用户为creator抵押的Kal额度;用户可以将账户上Kal通过系统抵押合约(Delegation("0x1000...0004"))抵押给其他creator,以增加其合约24小时调用交易次数;

每个交易需要抵押的Kal与gasPrice成正比，目前抵押需要量计算公式为:
+ 抵押量 = 单次交易抵押量 * 交易次数
+ 单次交易抵押量 =  gasPrice * C = 1e9 * 2e6 = 2e15 = 0.002 Kal
   gasPrice 默认值为1e9 Wei

### 1 solidity合约实例化系统抵押合约

```solidity
// 系统抵押合约接口
contract DelegationInterface {
	//用户为creatorAddress抵押Kal
	function delegate(address creatorAddress) public payable;
	//取回creatorAddress的抵押
	function withdraw(address creatorAddress, uint amount) public;
	//查询creatorAddress接受到的抵押
	function totalReceivedToken(address creatorAddress) public view returns(uint);
	//查询userAddress为其他creator抵押的额度
	function totalDelegatedToken(address userAddress) public view returns(uint)
	//查询userAddress给 creatorAddress抵押的额度
	function getAmount(address userAddress, address creatorAddress) public view returns(uint)
}

DelegationInterface Delegation = DelegationInterface(0x1000000000000000000000000000000000000004);
```
	
### 2 kalgo控制台中实例化系统合约

    kalgo attach http://127.0.0.1:8545;
 

    var delegationabi = [{"constant":false,"inputs":[{"name":"to","type":"address"}],"name":"delegate","outputs":[],"payable":true,"stateMutability":"payable","type":"function"},{"constant":false,"inputs":[{"name":"addr","type":"address"},{"name":"amount","type":"uint256"}],"name":"withdraw","outputs":[],"payable":false,"stateMutability":"nonpayable","type":"function"},{"constant":true,"inputs":[{"name":"addr","type":"address"}],"name":"totalReceivedToken","outputs":[{"name":"","type":"uint256"}],"payable":false,"stateMutability":"view","type":"function"},{"constant":true,"inputs":[{"name":"addr","type":"address"}],"name":"totalDelegatedToken","outputs":[{"name":"","type":"uint256"}],"payable":false,"stateMutability":"view","type":"function"},{"constant":true,"inputs":[{"name":"from","type":"address"},{"name":"to","type":"address"}],"name":"getAmount","outputs":[{"name":"","type":"uint256"}],"payable":false,"stateMutability":"view","type":"function"}];
    var Delegation = web3.eth.contract(delegationabi).at("0x1000000000000000000000000000000000000004");

### 3 用户没有自己的合约,可以将Kal抵押给其他creator,抵押的Kal可以随时取出,并且只能由自己取出。

```js
//抵押1e18个kal给其他creator
//creatorAddress-其他合约创建者账户地址
//eth.acccounts[0]-用户账户地址
Delegation.delegate(creatorAddress,{from:eth.acccounts[0],value:1e18});

//查询自己已抵押给creator的额度
Delegation.getAmount(eth.accounts[0],creatorAddress);

//查询自己所有抵押出去的额度
Delegation.totalDelegatedToken(eth.acccounts[0]);

//用户随时取出自己抵押给creator的额度
Delegation.withdraw(creatorAddress,value,{from:eth.accounts[0]});
```

### 4 其他用户为自己(creator)的抵押

```js
//查询某个用户为自己(creator)抵押的额度
//userAddress-抵押给自己额度的用户地址
//creatorAddress-合约创建者账户地址
Delegation.getAmount(userAddress,creatorAddress);
//查询自己(creator)收到的所有抵押额度
Delegation.totalReceivedToken(creatorAddress);
```

