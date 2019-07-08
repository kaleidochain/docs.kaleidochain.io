---
title: "权限控制"
weight: 50
---

> 权限控制是kaleido为降低用户使用门槛提供的功能模块,通过权限合约功能，合约的创建者(以下称creator)除了支付合约部署费用,还可以为调用合约的交易代扣手续费,合约调用者(tx.from)无需Kal,就可以使用合约。
> creator可以通过系统权限控制合约Authority(0x1000…0003)控制用户代扣费权限,以及代扣收付费交易gasPrice,gasLimit上限。
<pre>
权限控制合约有2种权限模式;
    0-白名单模式(默认),只有加入白名单列表的用户才能发送代扣费交易,其他用户只能发送普通交易调用合约;
    1-黑名单模式,加入了黑名单列表的用户不能发送代扣费交易,但仍可以发送普通交易调用合约;

注:同一creator下的所有合约共享黑/白名单数据列表;

 +  如果需要使用权限控制合约，需要将合约的创建者调用权限控制合约进行设置，设置属性有代扣费交易允许的最大gasLimit和代扣费交易允许的最大gasPrice
 +  针对每个合约可以单独设置gasLimit和gasPrice，对合约还可以设置一个Mode（白名单还是黑名单）
 +  一个合约Creator共用白名单和黑名单数据，合约单独设置具体采用的模式
 +  合约的gasLimit和gasPrice在没有设置的情况下会继承合约创建者设置的gasLimit和gasPrice
 +  当合约创建者没有通过权限合约进行设置操作时，合约就是普通合约，需要由调用者支持gas费用
 +  即使合约创建者和合约通过权限合约进行了设置操作，用户调用合约时通过指定gaslimit>0和gasPrice>0 时，该交易也可以强制用户付费调用，如果用户账户kal足够调用成功，如果余额不足则调用失败
 +  在合约创建者或合约设置gasLimit和gasPrice情况下，白名单用户或者黑名单中不在的用户在调用合约交易时设置gasLimit=0时，该交易由合约创建者付费，此时gasPrice必须小于或等于合约或者合约创建者设置的gasPrice，否则该交易无效，不会被打包
 + 当合约创建者或者合约需要改变付费模式时可以通过设置gasLimit是否为0进行改变，当gasLimit=0 则合约所有调用者都是调用者付费
 + 合约的白名单模式和黑名单模式可以随时改变
 + 当合约创建者账户余额不足以支付交易费用时，设置为合约付费(gasLimit=0)的交易会失败，此时如果用户可以通过设置交易中gasLimit>0 来强制调用合约
</pre>

注:给abigen自动生成的代码，也增加了一个函数 TransactExact,这个函数不会自动设置gasLimit,用于发起合约付费交易.

#### 1 权限控制合约使用

> solidity合约中实例化权限控制合约
```js
contract AuthorityInterface {
	//设置用户代扣手续费交易gasPrice上限,合约没设置继承creator
    function setMaxGasPrice(address creatorOrcontract, uint _price) public returns(bool);
	//设置用户代扣手续费交易gasLimit上限,合约没设置继承creator
    function setGasLimit(address creatorOrcontract, uint64 _gas) public returns(bool);
	//设置权限模式,0-表面的模式(默认),1-黑名单模式，只对合约有效
    function setModel(address contract, uint _model) public returns(bool);
	//查询合约 MaxGasPrice,GasPrice,Mode三项的设置
    function getAll(address contractAddress) public view returns(uint price,uint64 gas,uint mod);
	//添加用户memberAddress到自己黑名单列表
    function addBlack(address memberAddress ) public returns(bool);
	//添加用户memberAddress到自己白名单列表
    function addWhite(address memberAddress ) public returns(bool);
	//将用户memberAddress从自己黑名单列表移除
    function removeBlack(address memberAddress ) public returns(bool);
	//将用户memberAddress从自己白名单列表移除
    function removeWhite(address memberAddress ) public returns(bool);
	//用户memberAddress是否在合约contractAddress的黑名单
    function isBlack(address contractAddress,address memberAddress ) public view returns(bool);
	//用户memberAddress是否在合约contractAddress的白名单
    function isWhite(address contractAddress,address memberAddress ) public view returns(bool);
}
AuthorityInterface Authority = AuthorityInterface(0x1000000000000000000000000000000000000003);
```
kalgo控制台实例化权限控制合约
```js
gkal attach http://127.0.0.1:8545
```
```js
var authorityabi=[{"constant":false,"inputs":[{"name":"addr","type":"address"},{"name":"_price","type":"uint256"}],"name":"setMaxGasPrice","outputs":[{"name":"","type":"bool"}],"payable":false,"stateMutability":"nonpayable","type":"function"},{"constant":false,"inputs":[{"name":"addr","type":"address"},{"name":"_gas","type":"uint64"}],"name":"setGasLimit","outputs":[{"name":"","type":"bool"}],"payable":false,"stateMutability":"nonpayable","type":"function"},{"constant":false,"inputs":[{"name":"addr","type":"address"},{"name":"_model","type":"uint256"}],"name":"setModel","outputs":[{"name":"","type":"bool"}],"payable":false,"stateMutability":"nonpayable","type":"function"},{"constant":true,"inputs":[{"name":"Addr","type":"address"}],"name":"getAll","outputs":[{"name":"price","type":"uint256"},{"name":"gas","type":"uint64"},{"name":"mod","type":"uint256"}],"payable":false,"stateMutability":"view","type":"function"},{"constant":false,"inputs":[{"name":"memberAddress","type":"address"}],"name":"addBlack","outputs":[{"name":"","type":"bool"}],"payable":false,"stateMutability":"nonpayable","type":"function"},{"constant":false,"inputs":[{"name":"memberAddress","type":"address"}],"name":"addWhite","outputs":[{"name":"","type":"bool"}],"payable":false,"stateMutability":"nonpayable","type":"function"},{"constant":false,"inputs":[{"name":"memberAddress","type":"address"}],"name":"removeBlack","outputs":[{"name":"","type":"bool"}],"payable":false,"stateMutability":"nonpayable","type":"function"},{"constant":false,"inputs":[{"name":"memberAddress","type":"address"}],"name":"removeWhite","outputs":[{"name":"","type":"bool"}],"payable":false,"stateMutability":"nonpayable","type":"function"},{"constant":true,"inputs":[{"name":"contractAddress","type":"address"},{"name":"memberAddress","type":"address"}],"name":"isBlack","outputs":[{"name":"","type":"bool"}],"payable":false,"stateMutability":"view","type":"function"},{"constant":true,"inputs":[{"name":"contractAddress","type":"address"},{"name":"memberAddress","type":"address"}],"name":"isWhite","outputs":[{"name":"","type":"bool"}],"payable":false,"stateMutability":"view","type":"function"}]
var Authority = web3.eth.contract(authorityabi).at("0x1000000000000000000000000000000000000003");
```

#### 2 通过creator开启合约代扣费功能

> creator设置了MaxGasPrice,GasLimit即开启了代扣费功能,将GasLimit设为小于21000的值即关闭合约扣费功能
```js
//为creator下所有合约开启代扣手续费
Authority.setMaxGasPrice(creatorAddress,1e6);
Authority.setGasLimit(creatorAddress,5e10);

//关闭代扣费功能
Authority.setGasLimit(creatorAddress,0);

//查询creatorAddress设置
Authority.getAll(creatorAddress)
```

#### 3 通过合约开启代扣费,并使用白名单模式控制用户权限

>合约设置自己独立MaxGasPrice,GasLimit,只开启本合约代扣费功能;如果没有设置,继承合约creator的MaxGasPrice,GasLimit;
```js
//为合约开启代扣手续费
//如果没设置会继承creator设置的MaxGasPrice
Authority.setMaxGasPrice(contractAddress,1e6);
//如果没设置会继承creator设置的GasLimit
Authority.setGasLimit(contractAddress,5e10);
//设置白名单模式;
Authority.setModel(contractAddress,0);

//用户加入白名单列表(该用户将不能发送代扣费交易)
Authority.addWhite(contractAddress,senderAddress);

//关闭代扣费,这里将GasLimit设置为1关闭,如果设置为0,会继承creator的GasLimit
Authority.setGasLimit(contractAddress,1);
```

#### 4 通过合约开启代扣费,并使用黑名单模式控制用户权限
```js
//为合约开启代扣手续费
//如果没设置会继承creator设置的MaxGasPrice
Authority.setMaxGasPrice(contractAddress,1e6);
//如果没设置会继承creator设置的GasLimit
Authority.setGasLimit(contractAddress,5e10);
//设置黑名单模式;
Authority.setModel(contractAddress,1);

//用户加入黑名单列表(该用户将不能发送代扣费交易)
Authority.addBlack(contractAddress,senderAddress);

//关闭代扣费,这里将GasLimit设置为1关闭,如果设置为0,会继承creator的GasLimit
Authority.setGasLimit(contractAddress,1);
```
#### 5 用户发起一比代扣费交易
```js
//假设已有一个game合约,确认合约创建者有足够抵押(创建者账户Kal+抵押合约中创建者抵押额度)
var game = web3.eth.contract(gameabi).at(gameAddress);
//第1节中控制台实例化权限控制合约
var Authority = web3.eth.contract(authorityabi).at("0x1000000000000000000000000000000000000003");
//查询game合约,权限设置
var conf = Authority.getAll(game.address);
//game合约允许代扣费交易的最大gasPrice
var GasPrice = conf[0];
//game合约允许代扣费交易的最大gasLimit
var MaxGasLimit = conf[1];
//game合约权限模式 0-白名单模式 1-黑名单模式
var mode = conf[2];

//判断game合约是否开启合约代扣费
if(GasPrice == 0 || MaxGasLimit <21000){
	return false;
}
//如果是白名单模式,查询自己(userAddress)是否在白名单列表
if(mode == 0 && !Authority.isWhite(game.address,userAddress)){
	return false;
}
//如果是黑名单模式,查询自己(userAddress)是否不在黑名单列表
if(mode == 1 && Authority.isBlack(game.address,userAddress)){
	return false;
}
//确认无误,用户(userAddress)发送一比代扣费交易
//交易gasPrice填game合约允许的最大GasPrice,也可以填一个小于GasPrice的值
//交易gasLimit置为0,指定这是一比代扣费交易
game.setgreet("test",{from:userAddress,gasPrice:GasPrice,gas:0})

```

#### 6 一个使用权限控制白名单模式合约实例:
```js
pragma solidity ^0.4.0;
contract AuthorityInterface {
    function setMaxGasPrice(address addr, uint _price) public returns(bool);
    function setGasLimit(address addr, uint64 _gas) public returns(bool);
    function setModel(address addr, uint _model) public returns(bool);
    function getAll(address Addr) public view returns(uint price,uint64 gas,uint mod);
    function addBlack(address memberAddress ) public returns(bool);
    function addWhite(address memberAddress ) public returns(bool);
    function removeBlack(address memberAddress ) public returns(bool);
    function removeWhite(address memberAddress ) public returns(bool);
    function isBlack(address contractAddress,address memberAddress ) public view returns(bool);
    function isWhite(address contractAddress,address memberAddress ) public view returns(bool);
}
contract Game{
    address public creator;     
    string greeting;
    AuthorityInterface Authority = AuthorityInterface (0x1000000000000000000000000000000000000003);
	//构造函数
    function Game(string _greeting) public payable{
        creator = msg.sender;
        greeting = _greeting;
		//开启合约代扣费,设置允许代扣费交易最大gasPrice,gasLimit
        Authority.setMaxGasPrice(this,1e10);
        Authority.setGasLimit(this,1e6);
        //设置白名单权限控制模式
		Authority.setModel(this,0);
		//将创建者加入白名单列表
        Authority.addWhite(msg.sender); 
    }

    function () payable {}
	//添加用户到白名单
    function addWhite(address addr) returns(bool){
        require(msg.sender == creator,"not creator");
        Authority.addWhite(addr);
        return true;
    }
	//用户从白名单移除
    function removeWhite(address addr) returns(bool){
        require(msg.sender == creator,"not creator");
        Authority.removeWhite(addr);
        return true;
    }
	//查询变量
    function greet() constant returns (string){
        return greeting;
    }
	//设置变量
    function setGreeting(string _newgreeting){
        greeting = _newgreeting;
    }
    //注销合约
    function kill(){ 
        require(msg.sender == creator,"not creator");
        suicide(creator);  
    }
	//查询合约权限配置
	function getAll() public veiw returns(uint,uint,uint){
		return Authority.getAll(address(this));
	}
	//查询用户是否为白名单
	function isWhite(address addr) public view returns(bool){
		return Authority.isWhite(address(this),addr);
	}
}

```

#### 7 kalgo控制台测试,
>测试合约前确认合约创建者有足够抵押(创建者账户Kal+抵押合约中创建者额度);
>用户发送合约代扣费交易需将交易gasLimit置为0。

> 1.控制台实例化合约
kalgo attach http://127.0.0.1:8545
```js
	var gameabi = [{"constant":true,"inputs":[],"name":"creator","outputs":[{"name":"","type":"address"}],"payable":false,"stateMutability":"view","type":"function"},{"inputs":[{"name":"_greeting","type":"string"}],"payable":true,"stateMutability":"payable","type":"constructor"},{"payable":true,"stateMutability":"payable","type":"fallback"},{"constant":false,"inputs":[{"name":"addr","type":"address"}],"name":"grant","outputs":[{"name":"","type":"bool"}],"payable":false,"stateMutability":"nonpayable","type":"function"},{"constant":false,"inputs":[{"name":"addr","type":"address"}],"name":"revoke","outputs":[{"name":"","type":"bool"}],"payable":false,"stateMutability":"nonpayable","type":"function"},{"constant":true,"inputs":[],"name":"greet","outputs":[{"name":"","type":"string"}],"payable":false,"stateMutability":"view","type":"function"},{"constant":false,"inputs":[{"name":"_newgreeting","type":"string"}],"name":"setGreeting","outputs":[],"payable":false,"stateMutability":"nonpayable","type":"function"},{"constant":false,"inputs":[],"name":"kill","outputs":[],"payable":false,"stateMutability":"nonpayable","type":"function"}]
	//gameAddress 从部署合约控制台获取
	var game = web3.eth.contract(gameabi).at(gameAddress);
```
> 2.将用户添加到合白名单
```js
	//使用creator账户将用户地址加入白名单
	game.addWhite(eth.accounts[0],{from:creatorAddress});
	//查询用户是否是白名单
	game.isWhite(eth.accounts[0]);
> 3. 用户发起一比代扣费交易
	//查询game合约gasPrice,gasLimit上限
	var conf = game.getAll()
	//game合约允许代扣费交易的最大gasPrice
	var GasPrice = conf[0];
	//game合约允许代扣费交易的最大gasLimit
	var MaxGasLimit = conf[1];
	//game合约权限模式 0-白名单模式 1-黑名单模式
	var mode = conf[2];
	//用户查询自己是否是白名单
	if (!game.isWhite(eth.accounts[0])){
		return false;
	}
	//用户发送一笔代扣费交易,交易gasPrice使用game合约允许的最大GasPrice,
	//将交易gas设置为0,表示这是一比代扣费交易
    game.setGreeting("test",{from:eth.accounts[0],gasPrice:GasPrice,gas:0});
	//查询是否生效 显示:`test`
	game.greet();
```