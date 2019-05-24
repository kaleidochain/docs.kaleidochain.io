---
title: "委托池"
date: 2019-05-23T18:27:26+08:00
---

除了持有token的外部账户可以参与挖矿外，持有token的合约也可以参与挖矿。这为多种形式的挖矿提供了技术机制。

### 委托池的用途

在你初期没有token的时候，你可以部署一个官方提供的委托合约代码，向官方申请委托授权。官方经过审核后，就会为你的合约进行抵押和委托。

通过委托池挖矿，你也可以实现集中多人的小量token到一起来参与挖矿的功能。你只需要部署自己的委托池合约，并确保合约可以执行（参考合约抵押机制）。发布自己的合约地址，

最后，你使用自己的委托合约注册矿工，就可以开始挖矿。

### 委托池的实现

外部账户注册矿工是通过发起交易的方式，调用系统矿工合约来注册的。合约的注册方式有点不同。合约必须要实现一个调用系统矿工合约进行注册的接口，外部账户或其它合约通过调用该接口来注册矿工。

在有多层合约嵌套调用时，需要注意以下细节。在注册系统矿工合约时，谁是矿工取决于msg.sender是谁。正常情况下，应该使用call调用系统矿工合约时，来实现将合约自己注册成为矿工。如果是要将调用合约的外部账号注册成为矿工，那么就需要使用delegatecall来调用，以使得msg.sender继承自调用者。具体可参考关于call和delegatecall的[区别说明](https://ethereum.stackexchange.com/questions/3667/difference-between-call-callcode-and-delegatecall)。

注册成为矿工时可以指定受益人，因此合约提供的注册矿工的接口通常都要做权限管理，限制某些地址才能调用、或者某些地址才能成为受益人，以防止收益人账号被他人修改。最简单的实现是，只要合约的创建人才可调用该接口。

下面是一个简单但功能完整的委托池的实现：

```solidity
import "./lib/kaleido/SysContract.sol";
import "./lib/math/SafeMath.sol";

contract DelegatePoolDemo {
    using SafeMath for uint256;

    mapping(address=>uint256) private _balances;

    // register self into minerdb
    function registerMiner(bytes minerkey) public {
        require(msg.sender == SysContract.creatorOf(address(this)));
        
        SysContract.minerRegister(minerkey);
    }
    
    /**
     * @dev Gets the balance of the specified address.
     * @param owner The address to query the balance of.
     * @return A uint256 representing the amount owned by the passed address.
     */
    function balanceOf(address owner) public view returns (uint256) {
        return _balances[owner];
    }
    
    /**
     * @dev Delegate token to this contract.
     */
    function delegate() public payable returns (bool) {
        require(msg.value > 0);
        _balances[msg.sender] = _balances[msg.sender].add(msg.value);
        return true;
    }
    
    /**
     * @dev Transfer token to a specified address.
     * @param to The address to transfer to.
     * @param value The amount to be transferred.
     */
    function transfer(address to, uint256 value) public {
        require(to != address(0), "transfer to the zero address");

        _balances[from] = _balances[from].sub(value);
        to.transfer(value);
    }
}

```



