# Solidity 小白教程：26. 删除合约

## **selfdestruct**

**selfdestruct**命令可以用来删除智能合约，并将该合约剩余**ETH**转到指定地址。**selfdestruct**是为了应对合约出错的极端情况而设计的。它最早被命名为**suicide**（自杀），但是这个词太敏感。为了保护抑郁的程序员，改名为**selfdestruct**。

### 如何使用**selfdestruct**

**selfdestruct**使用起来非常简单：

```solidity
selfdestruct(_addr)；
```

其中**\_addr**是接收合约中剩余**ETH**的地址。

### 例子

```solidity
contract DeleteContract {

    uint public value = 10;

    constructor() payable {}

    receive() external payable {}

    function deleteContract() external {
        // 调用selfdestruct销毁合约，并把剩余的ETH转给msg.sender
        selfdestruct(payable(msg.sender));
    }

    function getBalance() external view returns(uint balance){
        balance = address(this).balance;
    }
}
```

在**DeleteContract**合约中，我们写了一个**public**状态变量**value**，两个函数：**getBalance()**用于获取合约**ETH**余额，**deleteContract()**用于自毁合约，并把**ETH**转入给发起人。
部署好合约后，我们向**DeleteContract**合约转入 1 **ETH**。这时，**getBalance()**会返回 1 **ETH**，**value**变量是 10。
当我们调用**deleteContract()**函数，合约将自毁，所有变量都清空，此时**value**变为默认值**0**，**getBalance()**也返回空值。

### 注意事项

1. 对外提供合约销毁接口时，最好设置为只有合约所有者可以调用，可以使用函数修饰符**onlyOwner**进行函数声明。
2. 当合约被销毁后与智能合约的交互也能成功，并且返回 0。
3. 当合约中有**selfdestruct**功能时常常会带来安全问题和信任问题，合约中的 Selfdestruct 功能会为攻击者打开攻击向量(例如使用**selfdestruct**向一个合约频繁转入 token 进行攻击，这将大大节省了 GAS 的费用，虽然很少人这么做)，此外，此功能还会降低用户对合约的信心。

### 在 remix 上验证

1. 部署合约并且转入 1ETH，查看合约状态

![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1695525268654-78298168-8935-4108-9b65-f1a067dc4db1.png#averageHue=%23242539&clientId=uba4665ba-7cfc-4&from=paste&id=u2ec61eae&originHeight=1289&originWidth=2560&originalType=url&ratio=2&rotation=0&showTitle=false&size=234149&status=done&style=none&taskId=u8e7990c0-5b78-4558-be3f-8c75c2a4041&title=)

1. 销毁合约，查看合约状态

![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1695525268671-9635ca7a-ea88-47a4-9119-bd1c237295b3.png#averageHue=%23242538&clientId=uba4665ba-7cfc-4&from=paste&id=ua37cfca0&originHeight=1289&originWidth=2560&originalType=url&ratio=2&rotation=0&showTitle=false&size=229060&status=done&style=none&taskId=uded4d78e-9136-4e9c-a31f-faf55c06ee5&title=)
从测试中观察合约状态可以发现合约销毁后的 ETH 返回给了指定的地址，并且在合约销毁后依然可以请求交互，所以我们不能根据这个来判断合约是否已经销毁。

## 总结

**selfdestruct**是智能合约的紧急按钮，销毁合约并将剩余**ETH**转移到指定账户。当著名的**The DAO**攻击发生时，以太坊的创始人们一定后悔过没有在合约里加入**selfdestruct**来停止黑客的攻击吧。
