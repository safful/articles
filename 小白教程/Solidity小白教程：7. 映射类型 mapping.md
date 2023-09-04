# Solidity 小白教程：7. 映射类型 mapping

这一讲，我们将介绍 solidity 中的哈希表：映射（**Mapping**）类型。

## 映射 Mapping

在映射中，人们可以通过键（**Key**）来查询对应的值（**Value**），比如：通过一个人的**id**来查询他的钱包地址。
声明映射的格式为**mapping(\_KeyType => \_ValueType)**，其中**\_KeyType**和**\_ValueType**分别是**Key**和**Value**的变量类型。例子：

```solidity
mapping(uint => address) public idToAddress; // id映射到地址
    mapping(address => address) public swapPair; // 币对的映射，地址到地址
```

## 映射的规则

- **规则 1**：映射的**\_KeyType**只能选择**solidity**默认的类型，比如**uint**，**address**等，不能用自定义的结构体。而**\_ValueType**可以使用自定义的类型。下面这个例子会报错，因为**\_KeyType**使用了我们自定义的结构体：

```solidity
// 我们定义一个结构体 Struct
    struct Student{
        uint256 id;
        uint256 score;
    }
     mapping(Student => uint) public testVar;
```

- **规则 2**：映射的存储位置必须是**storage**，因此可以用于合约的状态变量，函数中的**storage**变量，和 library 函数的参数（见[例子](https://github.com/ethereum/solidity/issues/4635)）。不能用于**public**函数的参数或返回结果中，因为**mapping**记录的是一种关系 (key - value pair)。
- **规则 3**：如果映射声明为**public**，那么**solidity**会自动给你创建一个**getter**函数，可以通过**Key**来查询对应的**Value**。
- **规则 4**：给映射新增的键值对的语法为**\_Var[_Key] = \_Value**，其中**\_Var**是映射变量名，**\_Key**和**\_Value**对应新增的键值对。例子：

```solidity
function writeMap (uint _Key, address _Value) public{
        idToAddress[_Key] = _Value;
    }
```

## 映射的原理

- **原理 1**: 映射不储存任何键（**Key**）的资讯，也没有 length 的资讯。
- **原理 2**: 映射使用**keccak256(key)**当成 offset 存取 value。
- **原理 3**: 因为 Ethereum 会定义所有未使用的空间为 0，所以未赋值（**Value**）的键（**Key**）初始值都是各个 type 的默认值，如 uint 的默认值是 0。

## 在 Remix 上验证 (以 **Mapping.sol**为例)

- 映射示例 1 部署![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1693798612516-5fdc22f1-1f82-4b93-8944-e459bb3c1aee.png#averageHue=%23282a3e&clientId=u75ac8242-c489-4&from=paste&id=uf4021973&originHeight=915&originWidth=1198&originalType=url&ratio=2&rotation=0&showTitle=false&size=483751&status=done&style=none&taskId=u94c2934f-a5e1-4ee7-ae9a-ee63d226685&title=)
- 映射示例 2 初始值![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1693798612523-487e399f-3699-4043-8142-532524d90ca8.png#averageHue=%2327293d&clientId=u75ac8242-c489-4&from=paste&id=uaee64ffe&originHeight=568&originWidth=1238&originalType=url&ratio=2&rotation=0&showTitle=false&size=327619&status=done&style=none&taskId=ufbe7c071-9c21-434d-8abe-f0d14269be9&title=)
- 映射示例 3 key-value pair![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1693798612495-6b53afcc-e705-469d-a817-7f5b5ee2d889.png#averageHue=%2327283c&clientId=u75ac8242-c489-4&from=paste&id=ud89217a7&originHeight=554&originWidth=1405&originalType=url&ratio=2&rotation=0&showTitle=false&size=315977&status=done&style=none&taskId=u20199092-e893-40d4-9430-f33909a3322&title=)

## 总结

这一讲，我们介绍了**solidity**中哈希表——映射（**Mapping**）的用法。至此，我们已经学习了所有常用变量种类，之后我们会学习控制流**if-else**,** while**等。
