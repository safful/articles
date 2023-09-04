# Solidity 小白教程：8. 变量初始值

## 变量初始值

在**solidity**中，声明但没赋值的变量都有它的初始值或默认值。这一讲，我们将介绍常用变量的初始值。

### 值类型初始值

- **boolean**: **false**
- **string**: **""**
- **int**: **0**
- **uint**: **0**
- **enum**: 枚举中的第一个元素
- **address**: **0x0000000000000000000000000000000000000000** (或 **address(0)**)
- **function**
  - **internal**: 空白方程
  - **external**: 空白方程

可以用**public**变量的**getter**函数验证上面写的初始值是否正确：

```solidity
bool public _bool; // false
    string public _string; // ""
    int public _int; // 0
    uint public _uint; // 0
    address public _address; // 0x0000000000000000000000000000000000000000

    enum ActionSet { Buy, Hold, Sell}
    ActionSet public _enum; // 第1个内容Buy的索引0

    function fi() internal{} // internal空白方程
    function fe() external{} // external空白方程
```

### 引用类型初始值

- 映射**mapping**: 所有元素都为其默认值的**mapping**
- 结构体**struct**: 所有成员设为其默认值的结构体
- 数组**array**
  - 动态数组: **[]**
  - 静态数组（定长）: 所有成员设为其默认值的静态数组

可以用**public**变量的**getter**函数验证上面写的初始值是否正确：

```solidity
// Reference Types
    uint[8] public _staticArray; // 所有成员设为其默认值的静态数组[0,0,0,0,0,0,0,0]
    uint[] public _dynamicArray; // `[]`
    mapping(uint => address) public _mapping; // 所有元素都为其默认值的mapping
    // 所有成员设为其默认值的结构体 0, 0
    struct Student{
        uint256 id;
        uint256 score;
    }
    Student public student;
```

### **delete**操作符

**delete a**会让变量**a**的值变为初始值。

```solidity
// delete操作符
    bool public _bool2 = true;
    function d() external {
        delete _bool2; // delete 会让_bool2变为默认值，false
    }
```

## 在 remix 上验证

- 部署合约查看值类型、引用类型的初始值![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1693798634132-9f695ae7-bf99-40eb-8b99-f148cf2a397f.png#averageHue=%23272a3e&clientId=u14a4b245-65c9-4&from=paste&id=ua701ece1&originHeight=1239&originWidth=1726&originalType=url&ratio=2&rotation=0&showTitle=false&size=305020&status=done&style=none&taskId=ub747ef31-7d0f-4002-95e1-205f60ccd73&title=)
- 值类型、引用类型 delete 操作后的默认值![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1693798634152-279bd618-d014-4977-841a-d356d6761408.png#averageHue=%2326283c&clientId=u14a4b245-65c9-4&from=paste&id=ua14523a8&originHeight=1170&originWidth=1802&originalType=url&ratio=2&rotation=0&showTitle=false&size=296966&status=done&style=none&taskId=u4922fb79-f39d-45d9-b3ab-5447227ad88&title=)

## 总结

这一讲，我们介绍了**solidity**中变量的初始值。变量被声明但没有赋值的时候，它的值默认为初始值。不同类型的变量初始值不同，**delete**操作符可以删除一个变量的值并代替为初始值。
