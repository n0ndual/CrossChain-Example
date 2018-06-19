# 跨链Token小例子

InkNetwork上的Chaincode中的token和Eth上的contract中的token可以互相转换，保持token的总量固定，兑换价格是1：1

设计了两种方案，两种方案都包含了三个部分：ETH上一个合约，InkNetwork上一个合约，一个Dapp。

第一种方案：用户user1想把token从InkN转移到Eth，只需要向InkN上的合约发一个交易，Dapp监听到该事件，由token的issuer向Eth上的合约发送一个交易。

第二种方案：用户user1发起两个请求。首先，user1先向InkN上的合约发送一个交易，Issuer收到请求后生成一个加密的凭证给user1；然后user1再把加密凭证发给Eth上的合约。

两种方案的对比：第一种方案用户使用简单，用户只需要发送一次请求，由issuer确保跨链交易的成功；但是issuer需要承担一半的gas，没有很好的补救措施。第二种方案所有的gas都由用户承担，但是用户需要签名进行两次交易。等第一次交易成功后，再进行第二次交易。用户体验较差。

这个小例子采用了第二种方案。

# 方案一：

## Eth上的合约：

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

* 用户user1调用eth上的合约tokenEth的TransferToInkNetwork接口，转移100个token到InkNetwork上

* tokenEth合约把A的token减少100，并发送一个TransferToInkNetwork事件

* Dapp监听到该事件，调用inkNetwork上的合约tokenInk的transferFromEth接口；Dapp中保存有issuer的私钥，Dapp用issuer的私钥对发起的交易加密

* tokenInk合约执行transferFromEth方法，只有issuer签名的交易通过验证，增加用户user1的token数量100个

# 跨链小例子，方案二：

## Dapp

用户user1想要把InkNetwork合约中的100个Token转移到Eth的合约上

* 用户user1在Dapp上填写Eth和InkNetwork的地址和gas价格和交易数量，提交

* Dapp向InkNetwork的合约tokenInk发送交易，需要用户user1签名

* InkNetwork上的合约tokenInk收到交易，成功写入链中后，返回给用户user1一个加密的凭证

* Dapp再向eth上的tokenEth发送合约，传入刚收到的加密凭证，此时需要用户user1再次签名

* 第二个交易成功后，交易即成功，如果第二个交易失败，可能需要回滚第一个交易。
