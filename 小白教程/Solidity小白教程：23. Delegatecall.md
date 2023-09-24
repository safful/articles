# Solidity 小白教程：23. Delegatecall

## **delegatecall**

**delegatecall**与**call**类似，是**solidity**中地址类型的低级成员函数。**delegate**中是委托/代表的意思，那么**delegatecall**委托了什么？
当用户**A**通过合约**B**来**call**合约**C**的时候，执行的是合约**C**的函数，**语境**(**Context**，可以理解为包含变量和状态的环境)也是合约**C**的：**msg.sender**是**B**的地址，并且如果函数改变一些状态变量，产生的效果会作用于合约**C**的变量上。
![](https://cdn.nlark.com/yuque/0/2023/png/97322/1695525174451-c5415a37-8b0d-48a1-bcf3-51d89b442213.png#averageHue=%23f5f5f5&clientId=u8d6eb8e7-199a-4&from=paste&id=u9247d260&originHeight=698&originWidth=1860&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=u81786b54-9a33-474a-8e0a-198c1095053&title=)
而当用户**A**通过合约**B**来**delegatecall**合约**C**的时候，执行的是合约**C**的函数，但是**语境**仍是合约**B**的：**msg.sender**是**A**的地址，并且如果函数改变一些状态变量，产生的效果会作用于合约**B**的变量上。
![](https://cdn.nlark.com/yuque/0/2023/png/97322/1695525173056-e4cbf812-5855-4665-88b1-76a68b0fccf6.png#averageHue=%23f5f5f5&clientId=u8d6eb8e7-199a-4&from=paste&id=u2308cc39&originHeight=702&originWidth=1862&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=u57aa8994-0ef2-4d75-9730-905441113c8&title=)
大家可以这样理解：一个**富商**把它的资产（**状态变量**）都交给一个**VC**代理（**目标合约**的函数）来打理。执行的是**VC**的函数，但是改变的是**富商**的状态。
**delegatecall**语法和**call**类似，也是：

```solidity
目标合约地址.delegatecall(二进制编码);
```

其中**二进制编码**利用结构化编码函数**abi.encodeWithSignature**获得：

```solidity
abi.encodeWithSignature("函数签名", 逗号分隔的具体参数)
```

**函数签名**为**"函数名（逗号分隔的参数类型)"**。例如**abi.encodeWithSignature("f(uint256,address)", \_x, \_addr)**。
和**call**不一样，**delegatecall**在调用合约时可以指定交易发送的**gas**，但不能指定发送的**ETH**数额
_**注意\*\***：\***\*delegatecall\*\***有安全隐患，使用时要保证当前合约和目标合约的状态变量存储结构相同，并且目标合约安全，不然会造成资产损失。\*\*_

## 什么情况下会用到**delegatecall**?

目前**delegatecall**主要有两个应用场景：

1. 代理合约（**Proxy Contract**）：将智能合约的存储合约和逻辑合约分开：代理合约（**Proxy Contract**）存储所有相关的变量，并且保存逻辑合约的地址；所有函数存在逻辑合约（**Logic Contract**）里，通过**delegatecall**执行。当升级时，只需要将代理合约指向新的逻辑合约即可。
2. EIP-2535 Diamonds（钻石）：钻石是一个支持构建可在生产中扩展的模块化智能合约系统的标准。钻石是具有多个实施合同的代理合同。 更多信息请查看：[钻石标准简介](https://eip2535diamonds.substack.com/p/introduction-to-the-diamond-standard)。

## **delegatecall**例子

调用结构：你（**A**）通过合约**B**调用目标合约**C**。

### 被调用的合约 C

我们先写一个简单的目标合约**C**：有两个**public**变量：**num**和**sender**，分别是**uint256**和**address**类型；有一个函数，可以将**num**设定为传入的**\_num**，并且将**sender**设为**msg.sender**。

```solidity
// 被调用的合约C
contract C {
    uint public num;
    address public sender;

    function setVars(uint _num) public payable {
        num = _num;
        sender = msg.sender;
    }
}
```

### 发起调用的合约 B

首先，合约**B**必须和目标合约**C**的变量存储布局必须相同，两个变量，并且顺序为**num**和**sender**

```solidity
contract B {
    uint public num;
    address public sender;
```

接下来，我们分别用**call**和**delegatecall**来调用合约**C**的**setVars**函数，更好的理解它们的区别。
**callSetVars**函数通过**call**来调用**setVars**。它有两个参数**\_addr**和**\_num**，分别对应合约**C**的地址和**setVars**的参数。

```solidity
// 通过call来调用C的setVars()函数，将改变合约C里的状态变量
    function callSetVars(address _addr, uint _num) external payable{
        // call setVars()
        (bool success, bytes memory data) = _addr.call(
            abi.encodeWithSignature("setVars(uint256)", _num)
        );
    }
```

而**delegatecallSetVars**函数通过**delegatecall**来调用**setVars**。与上面的**callSetVars**函数相同，有两个参数**\_addr**和**\_num**，分别对应合约**C**的地址和**setVars**的参数。

```solidity
// 通过delegatecall来调用C的setVars()函数，将改变合约B里的状态变量
    function delegatecallSetVars(address _addr, uint _num) external payable{
        // delegatecall setVars()
        (bool success, bytes memory data) = _addr.delegatecall(
            abi.encodeWithSignature("setVars(uint256)", _num)
        );
    }
}
```

### 在 remix 上验证

1. 首先，我们把合约**B**和**C**都部署好

![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1695525172069-7dd3ef23-e73c-4290-978f-e6265a99747a.png#averageHue=%2326293b&clientId=u8d6eb8e7-199a-4&from=paste&id=u54d4dd24&originHeight=866&originWidth=1920&originalType=url&ratio=2&rotation=0&showTitle=false&size=207946&status=done&style=none&taskId=u93d192db-9d27-44f2-bb67-ba272a88812&title=)

1. 部署之后，查看**C**合约状态变量的初始值，**B**合约的状态变量也是一样。

![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1695525172130-65c8bb11-5104-47f7-a503-3df0b20cf832.png#averageHue=%23262a3c&clientId=u8d6eb8e7-199a-4&from=paste&id=u790a84f1&originHeight=866&originWidth=1920&originalType=url&ratio=2&rotation=0&showTitle=false&size=208182&status=done&style=none&taskId=u6868cc21-bfb5-4143-ab4d-3f90a9e6835&title=)

1. 此时，调用合约**B**中的**callSetVars**，传入参数为合约**C**地址和**10**

![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1695525172083-70d121dc-f904-4b30-b96b-865e5b5cd405.png#averageHue=%23262a3c&clientId=u8d6eb8e7-199a-4&from=paste&id=u20f78557&originHeight=866&originWidth=1920&originalType=url&ratio=2&rotation=0&showTitle=false&size=203194&status=done&style=none&taskId=u235697e5-48aa-4906-871b-65c9124b10e&title=)

1. 运行后，合约**C**中的状态变量将被修改：**num**被改为**10**，**sender**变为合约**B**的地址

![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1695525172470-e4d90f07-cd0c-4b85-813d-af152f2a20a4.png#averageHue=%23262a3c&clientId=u8d6eb8e7-199a-4&from=paste&id=ub4c88b3b&originHeight=866&originWidth=1920&originalType=url&ratio=2&rotation=0&showTitle=false&size=207856&status=done&style=none&taskId=u16aab871-89a3-42da-927d-e6857965ae0&title=)

1. 接下来，我们调用合约**B**中的**delegatecallSetVars**，传入参数为合约**C**地址和**100**

![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1695525172521-cc8be813-6a6b-4df3-8d02-5763a3e50867.png#averageHue=%23262a3c&clientId=u8d6eb8e7-199a-4&from=paste&id=ua3eb9096&originHeight=866&originWidth=1920&originalType=url&ratio=2&rotation=0&showTitle=false&size=189993&status=done&style=none&taskId=u414e0acf-5a4c-44a6-9328-f664a14d5c7&title=)

1. 由于是**delegatecall**，语境为合约**B**。在运行后，合约**B**中的状态变量将被修改：**num**被改为**100**，**sender**变为你的钱包地址。合约**C**中的状态变量不会被修改。

![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1695525172544-1d184a9e-b3d8-45cb-993b-f3f67ff3c31f.png#averageHue=%2326293c&clientId=u8d6eb8e7-199a-4&from=paste&id=u0a53792f&originHeight=866&originWidth=1920&originalType=url&ratio=2&rotation=0&showTitle=false&size=195390&status=done&style=none&taskId=uec744f99-ade0-46a7-a82d-177a91ede25&title=)

## 总结

这一讲我们介绍了**solidity**中的另一个低级函数**delegatecall**。与**call**类似，它可以用来调用其他合约；不同点在于运行的语境，**B call C**，语境为**C**；而**B delegatecall C**，语境为**B**。目前**delegatecall**最大的应用是代理合约和**EIP-2535 Diamonds**（钻石）。
