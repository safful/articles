# Solidity 小白教程：12. 事件

这一讲，我们用转账 ERC20 代币为例来介绍**solidity**中的事件（**event**）。

## 事件

**Solidity**中的事件（**event**）是**EVM**上日志的抽象，它具有两个特点：

- 响应：应用程序（[ethers.js](https://learnblockchain.cn/docs/ethers.js/api-contract.html#id18)）可以通过**RPC**接口订阅和监听这些事件，并在前端做响应。
- 经济：事件是**EVM**上比较经济的存储数据的方式，每个大概消耗 2,000 **gas**；相比之下，链上存储一个新变量至少需要 20,000 **gas**。

### 声明事件

事件的声明由**event**关键字开头，接着是事件名称，括号里面写好事件需要记录的变量类型和变量名。以**ERC20**代币合约的**Transfer**事件为例：

```solidity
event Transfer(address indexed from, address indexed to, uint256 value);
```

我们可以看到，**Transfer**事件共记录了 3 个变量**from**，**to**和**value**，分别对应代币的转账地址，接收地址和转账数量，其中**from**和**to**前面带有**indexed**关键字，他们会保存在以太坊虚拟机日志的**topics**中，方便之后检索。

### 释放事件

我们可以在函数里释放事件。在下面的例子中，每次用**\_transfer()**函数进行转账操作的时候，都会释放**Transfer**事件，并记录相应的变量。

```solidity
// 定义_transfer函数，执行转账逻辑
    function _transfer(
        address from,
        address to,
        uint256 amount
    ) external {

        _balances[from] = 10000000; // 给转账地址一些初始代币

        _balances[from] -=  amount; // from地址减去转账数量
        _balances[to] += amount; // to地址加上转账数量

        // 释放事件
        emit Transfer(from, to, amount);
    }
```

## EVM 日志 **Log**

以太坊虚拟机（EVM）用日志**Log**来存储**Solidity**事件，每条日志记录都包含主题**topics**和数据**data**两部分。
![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1694260249914-6e6524dc-b11b-4e73-81d6-d585b4101e16.png#averageHue=%23e4695d&clientId=u1f827033-8e8f-4&from=paste&id=u73ea5b91&originHeight=608&originWidth=1994&originalType=url&ratio=2&rotation=0&showTitle=false&size=134797&status=done&style=none&taskId=u3e70a9e4-9f3a-4cc1-baf0-a1ca701026c&title=)

### 主题 **Topics**

日志的第一部分是主题数组，用于描述事件，长度不能超过**4**。它的第一个元素是事件的签名（哈希）。对于上面的**Transfer**事件，它的签名就是：

```solidity
keccak256("Transfer(addrses,address,uint256)")

//0xddf252ad1be2c89b69c2b068fc378daa952ba7f163c4a11628f55a4df523b3ef
```

除了事件签名，主题还可以包含至多**3**个**indexed**参数，也就是**Transfer**事件中的**from**和**to**。
**indexed**标记的参数可以理解为检索事件的索引“键”，方便之后搜索。每个 **indexed** 参数的大小为固定的 256 比特，如果参数太大了（比如字符串），就会自动计算哈希存储在主题中。

### 数据 **Data**

事件中不带 **indexed**的参数会被存储在 **data** 部分中，可以理解为事件的“值”。**data** 部分的变量不能被直接检索，但可以存储任意大小的数据。因此一般 **data** 部分可以用来存储复杂的数据结构，例如数组和字符串等等，因为这些数据超过了 256 比特，即使存储在事件的 **topic** 部分中，也是以哈希的方式存储。另外，**data** 部分的变量在存储上消耗的 gas 相比于 **topic** 更少。

## **Remix**演示

以 **Event.sol** 合约为例，编译部署。
然后调用 **\_transfer** 函数。
![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1694260250495-cb27fa09-710c-4e84-9211-8be7e96d5b14.png#averageHue=%2327283c&clientId=u1f827033-8e8f-4&from=paste&id=u501d260f&originHeight=1700&originWidth=2738&originalType=url&ratio=2&rotation=0&showTitle=false&size=1462634&status=done&style=none&taskId=u0aaf1696-1bb0-424f-bdc3-a12f41816df&title=)
点击右侧的交易查看详情，可以看到日志的具体内容。
![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1694260250098-1842c394-5c45-46d9-b347-dadbe86bd1eb.png#averageHue=%23252639&clientId=u1f827033-8e8f-4&from=paste&id=u37377885&originHeight=1368&originWidth=2360&originalType=url&ratio=2&rotation=0&showTitle=false&size=761354&status=done&style=none&taskId=u5b5a8340-098a-4528-9199-8585dd361aa&title=)

### 在 etherscan 上查询事件

我们尝试用**\_transfer()**函数在**Rinkeby**测试网络上转账 100 代币，可以在**etherscan**上查询到相应的**tx**：[网址](https://rinkeby.etherscan.io/tx/0x8cf87215b23055896d93004112bbd8ab754f081b4491cb48c37592ca8f8a36c7)。
点击**Logs**按钮，就能看到事件明细：
![](https://cdn.nlark.com/yuque/0/2023/png/97322/1694260250420-006b3b24-c73b-449b-84b8-9e4c250983c0.png#averageHue=%239a9998&clientId=u1f827033-8e8f-4&from=paste&id=uec43a3d0&originHeight=980&originWidth=1772&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=ue1535ea1-59fd-4097-94f9-079fc45217a&title=)
**Topics**里面有三个元素，**[0]**是这个事件的哈希，**[1]**和**[2]**是我们定义的两个**indexed**变量的信息，即转账的转出地址和接收地址。**Data**里面是剩下的不带**indexed**的变量，也就是转账数量。

## 总结

这一讲，我们介绍了如何使用和查询**solidity**中的事件。很多链上分析工具包括**Nansen**和**Dune Analysis**都是基于事件工作的。
