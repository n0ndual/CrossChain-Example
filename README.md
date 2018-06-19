# 跨链Token小例子

部署在InkNetwork上的Chaincode中的token和Eth上的token可以互相转换，保持token的总量固定，兑换价格是1：1

实现包括三部分：ETH上一个合约，InkNetwork上一个合约，一个Dapp

## ETH上的合约：

```
contract tokenEth{

    address public issuer
    ...

    function transferFromInkNetwork(address _ethAccount, uint256 number) public {
        require(msg.sender == issuer)
    }
    function transferToInkNetwork(address _inkAccount, uint256 number) public{

    }
    event TransferToInkNetwork(address _from, unit _value)
    event TransferFromInkNetwork(address _from, unit _value)
    ...
}
```

## InkNetwork上的合约

```
contract tokenInk{

    address public issuer
    ...


    function transferFromEth(address _inkAccount, uint256 number) public {
        require(msg.sender == ethOwner)
    }
    function transferToEth(address _ethAccount, uint256 number) public{

    }
    ...
}
```

## Dapp

部署一个Dapp，监听某一个链的合约的token转移事件，然后发起向另一个链的交易请求。

* 用户A调用eth上的合约tokenEth的TransferToInkNetwork接口，转移100个token到InkNetwork上

* tokenEth合约把A的token减少100，并发送一个TransferToInkNetwork事件

* Dapp监听到该事件，调用inkNetwork上的合约tokenInk的transferFromEth接口；Dapp中保存有issuer的私钥，Dapp用issuer的私钥对发起的交易加密

* tokenInk合约执行transferFromEth方法，只有issuer签名的交易通过验证，增加用户A的token数量100个
