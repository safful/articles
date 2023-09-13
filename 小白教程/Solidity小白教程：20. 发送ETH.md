# Solidity 小白教程：20. 发送 ETH

**Solidity**有三种方法向其他合约发送**ETH**，他们是：**transfer()**，**send()**和**call()**，其中**call()**是被鼓励的用法。

## 接收 ETH 合约

我们先部署一个接收**ETH**合约**ReceiveETH**。**ReceiveETH**合约里有一个事件**Log**，记录收到的**ETH**数量和**gas**剩余。还有两个函数，一个是**receive()**函数，收到**ETH**被触发，并发送**Log**事件；另一个是查询合约**ETH**余额的**getBalance()**函数。

```solidity
contract ReceiveETH {
    // 收到eth事件，记录amount和gas
    event Log(uint amount, uint gas);

    // receive方法，接收eth时被触发
    receive() external payable{
        emit Log(msg.value, gasleft());
    }

    // 返回合约ETH余额
    function getBalance() view public returns(uint) {
        return address(this).balance;
    }
}
```

部署**ReceiveETH**合约后，运行**getBalance()**函数，可以看到当前合约的**ETH**余额为**0**。
![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1694577267043-b8b5eccd-65be-43b4-834f-f4eaf783cbdf.png#averageHue=%2326283b&clientId=u443feafa-b85f-4&from=paste&id=ud40e70e1&originHeight=747&originWidth=1163&originalType=url&ratio=2&rotation=0&showTitle=false&size=102217&status=done&style=none&taskId=ubae1d90c-218e-4d0e-8660-e79c60b3fab&title=)

## 发送 ETH 合约

我们将实现三种方法向**ReceiveETH**合约发送**ETH**。首先，先在发送 ETH 合约**SendETH**中实现**payable**的**构造函数**和**receive()**，让我们能够在部署时和部署后向合约转账。

```solidity
contract SendETH {
    // 构造函数，payable使得部署的时候可以转eth进去
    constructor() payable{}
    // receive方法，接收eth时被触发
    receive() external payable{}
}
```

### transfer

- 用法是**接收方地址.transfer(发送 ETH 数额)**。
- **transfer()**的**gas**限制是**2300**，足够用于转账，但对方合约的**fallback()**或**receive()**函数不能实现太复杂的逻辑。
- **transfer()**如果转账失败，会自动**revert**（回滚交易）。

代码样例，注意里面的**\_to**填**ReceiveETH**合约的地址，**amount**是**ETH**转账金额：

```solidity
// 用transfer()发送ETH
function transferETH(address payable _to, uint256 amount) external payable{
    _to.transfer(amount);
}
```

部署**SendETH**合约后，对**ReceiveETH**合约发送 ETH，此时**amount**为 10，**value**为 0，**amount**>**value**，转账失败，发生**revert**。
![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1694577267080-7f26784a-3d4e-47a3-87f8-a8d4c5966a3d.png#averageHue=%2325273b&clientId=u443feafa-b85f-4&from=paste&id=uc90ddae4&originHeight=785&originWidth=1675&originalType=url&ratio=2&rotation=0&showTitle=false&size=144247&status=done&style=none&taskId=ua15cfead-750b-43f6-8501-f83fee52c02&title=)
此时**amount**为 10，**value**为 10，**amount**<=**value**，转账成功。
![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1694577267025-1b895709-8e42-424b-b966-16deae412033.png#averageHue=%2326273b&clientId=u443feafa-b85f-4&from=paste&id=u677ef28b&originHeight=796&originWidth=1519&originalType=url&ratio=2&rotation=0&showTitle=false&size=121249&status=done&style=none&taskId=u405ca78c-df65-45d9-b7d1-9d2d792ad80&title=)
在**ReceiveETH**合约中，运行**getBalance()**函数，可以看到当前合约的**ETH**余额为**10**。
![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1694577266763-c8d2c122-8ef9-4b9d-ae57-01fc3b0fb4f1.png#averageHue=%232b3044&clientId=u443feafa-b85f-4&from=paste&id=u19bb10ee&originHeight=156&originWidth=540&originalType=url&ratio=2&rotation=0&showTitle=false&size=9423&status=done&style=none&taskId=u4819d2e7-c11a-4f4f-9230-8cd54cde50a&title=)

### send

- 用法是**接收方地址.send(发送 ETH 数额)**。
- **send()**的**gas**限制是**2300**，足够用于转账，但对方合约的**fallback()**或**receive()**函数不能实现太复杂的逻辑。
- **send()**如果转账失败，不会**revert**。
- **send()**的返回值是**bool**，代表着转账成功或失败，需要额外代码处理一下。

代码样例：

```solidity
// send()发送ETH
function sendETH(address payable _to, uint256 amount) external payable{
    // 处理下send的返回值，如果失败，revert交易并发送error
    bool success = _to.send(amount);
    if(!success){
        revert SendFailed();
    }
}
```

对**ReceiveETH**合约发送 ETH，此时**amount**为 10，**value**为 0，**amount**>**value**，转账失败，因为经过处理，所以发生**revert**。
![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1694577267027-ce81d763-b9aa-402b-812b-0f52e0c7e6cf.png#averageHue=%2326283c&clientId=u443feafa-b85f-4&from=paste&id=ud1842cdd&originHeight=810&originWidth=1399&originalType=url&ratio=2&rotation=0&showTitle=false&size=128500&status=done&style=none&taskId=u55ce446b-6086-40fc-8863-72a869246af&title=)
此时**amount**为 10，**value**为 11，**amount**<=**value**，转账成功。
![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1694577267136-60fdc0df-745d-4141-a7f4-e8ad3022b17f.png#averageHue=%2326283b&clientId=u443feafa-b85f-4&from=paste&id=u0f45b054&originHeight=790&originWidth=1646&originalType=url&ratio=2&rotation=0&showTitle=false&size=132130&status=done&style=none&taskId=uf45e5f33-e298-4c23-9d8f-9aa02292fb5&title=)

### call

- 用法是**接收方地址.call{value: 发送 ETH 数额}("")**。
- **call()**没有**gas**限制，可以支持对方合约**fallback()**或**receive()**函数实现复杂逻辑。
- **call()**如果转账失败，不会**revert**。
- **call()**的返回值是**(bool, data)**，其中**bool**代表着转账成功或失败，需要额外代码处理一下。

代码样例：

```solidity
// call()发送ETH
function callETH(address payable _to, uint256 amount) external payable{
    // 处理下call的返回值，如果失败，revert交易并发送error
    (bool success,) = _to.call{value: amount}("");
    if(!success){
        revert CallFailed();
    }
}
```

对**ReceiveETH**合约发送 ETH，此时**amount**为 10，**value**为 0，**amount**>**value**，转账失败，因为经过处理，所以发生**revert**。
![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1694577267407-b094697f-a385-43b5-90d1-66e0320578ca.png#averageHue=%2326283b&clientId=u443feafa-b85f-4&from=paste&id=ueaa247ea&originHeight=823&originWidth=1447&originalType=url&ratio=2&rotation=0&showTitle=false&size=130243&status=done&style=none&taskId=ud4cb3b62-2955-46af-88db-a28975286df&title=)
此时**amount**为 10，**value**为 11，**amount**<=**value**，转账成功。
![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1694577267445-c445368f-0b5f-4f97-8e9a-72db3a8e4423.png#averageHue=%2326283b&clientId=u443feafa-b85f-4&from=paste&id=u13bdbffb&originHeight=810&originWidth=1655&originalType=url&ratio=2&rotation=0&showTitle=false&size=134685&status=done&style=none&taskId=u33340743-6405-4963-a7bf-c34c306b491&title=)
运行三种方法，可以看到，他们都可以成功地向**ReceiveETH**合约发送**ETH**。

## 总结

这一讲，我们介绍**solidity**三种发送**ETH**的方法：**transfer**，**send**和**call**。

- **call**没有**gas**限制，最为灵活，是最提倡的方法；
- **transfer**有**2300 gas**限制，但是发送失败会自动**revert**交易，是次优选择；
- **send**有**2300 gas**限制，而且发送失败不会自动**revert**交易，几乎没有人用它。
