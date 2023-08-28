# Solidity 小白教程：3. 函数类型

## Solidity 中的函数

solidity 官方文档里把函数归到数值类型，但我觉得差别很大，所以单独分一类。我们先看一下 solidity 中函数的形式：

```solidity
function <function name>(<parameter types>) {internal|external|public|private} [pure|view|payable] [returns (<return types>)]
```

**Copy**
看着些复杂，咱们从前往后一个一个看（方括号中的是可写可不写的关键字）：

1. **function**：声明函数时的固定用法，想写函数，就要以 function 关键字开头。
2. **<function name>**：函数名。
3. **(<parameter types>)**：圆括号里写函数的参数，也就是要输入到函数的变量类型和名字。
4. **{internal|external|public|private}**：函数可见性说明符，一共 4 种。没标明函数类型的，默认**public**。合约之外的函数，即"自由函数"，始终具有隐含**internal**可见性。
   - **public**: 内部外部均可见。
   - **private**: 只能从本合约内部访问，继承的合约也不能用。
   - **external**: 只能从合约外部访问（但是可以用**this.f()**来调用，**f**是函数名）。
   - **internal**: 只能从合约内部访问，继承的合约可以用。
5. **Note 1**: 没有标明可见性类型的函数，默认为**public**。**Note 2**: **public|private|internal** 也可用于修饰状态变量。 **public**变量会自动生成同名的**getter**函数，用于查询数值。**Note 3**: 没有标明可见性类型的状态变量，默认为**internal**。
6. **[pure|view|payable]**：决定函数权限/功能的关键字。**payable**（可支付的）很好理解，带着它的函数，运行的时候可以给合约转入**ETH**。**pure**和**view**的介绍见下一节。
7. **[returns ()]**：函数返回的变量类型和名称。

## 到底什么是**Pure**和**View**？

我刚开始学**solidity**的时候，一直不理解**pure**跟**view**关键字，因为别的语言没有类似的关键字。**solidity**加入这两个关键字，我认为是因为**gas fee**。合约的状态变量存储在链上，**gas fee**很贵，如果不改变链上状态，就不用付**gas**。包含**pure**跟**view**关键字的函数是不改写链上状态的，因此用户直接调用他们是不需要付 gas 的（合约中非**pure**/**view**函数调用它们则会改写链上状态，需要付 gas）。
在以太坊中，以下语句被视为修改链上状态：

1. 写入状态变量。
2. 释放事件。
3. 创建其他合约。
4. 使用**selfdestruct**.
5. 通过调用发送以太币。
6. 调用任何未标记**view**或**pure**的函数。
7. 使用低级调用（low-level calls）。
8. 使用包含某些操作码的内联汇编。

我画了一个马里奥插画，帮助大家理解。在插画里，我把合约中的状态变量（存储在链上）比作碧池公主，三种不同的角色代表不同的关键字。
![](https://cdn.nlark.com/yuque/0/2023/png/97322/1693193361994-3895c791-1f86-4ec6-ac79-dd000c6ec41b.png#averageHue=%23f4eee8&clientId=uc606aab3-8d53-4&from=paste&id=ucfb2ebc9&originHeight=1028&originWidth=1758&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=u8bf96969-141a-4fd4-ac87-da2f69e641e&title=)

- **pure**，中文意思是“纯”，在**solidity**里理解为“纯纯牛马”。包含**pure**关键字的函数，不能读取也不能写入存储在链上的状态变量。就像小怪一样，看不到也摸不到碧池公主。
- **view**，“看”，在**solidity**里理解为“看客”。包含**view**关键字的函数，能读取但也不能写入状态变量。类似马里奥，能看到碧池，但终究是看客，不能入洞房。
- 不写**pure**也不写**view**，函数既可以读取也可以写入状态变量。类似马里奥里的**boss**，可以对碧池公主为所欲为 🐶。

## 代码

### 1. pure v.s. view

我们在合约里定义一个状态变量 **number = 5**。

```solidity
// SPDX-License-Identifier: MIT
    pragma solidity ^0.8.4;
    contract FunctionTypes{
        uint256 public number = 5;
```

**Copy**
定义一个**add()**函数，每次调用，每次给**number + 1**。

```solidity
// 默认
    function add() external{
        number = number + 1;
    }
```

**Copy**
如果**add()**包含了**pure**关键字，例如 **function add() pure external**，就会报错。因为**pure**（纯纯牛马）是不配读取合约里的状态变量的，更不配改写。那**pure**函数能做些什么？举个例子，你可以给函数传递一个参数 **\_number**，然后让他返回 **\_number+1**。

```solidity
// pure: 纯纯牛马
    function addPure(uint256 _number) external pure returns(uint256 new_number){
        new_number = _number + 1;
    }
```

**Copy**
**Example:**
![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1693193361573-2c631f56-6a0b-41f9-8837-807ffe33c610.png#averageHue=%2327293c&clientId=uc606aab3-8d53-4&from=paste&id=ue382ae46&originHeight=1328&originWidth=2622&originalType=url&ratio=2&rotation=0&showTitle=false&size=410816&status=done&style=none&taskId=u298d6b28-40f7-46a7-9922-f85fe509b62&title=)
如果**add()**包含**view**，比如**function add() view external**，也会报错。因为**view**能读取，但不能够改写状态变量。可以稍微改写下方程，让他不改写**number**，而是返回一个新的变量。

```solidity
// view: 看客
    function addView() external view returns(uint256 new_number) {
        new_number = number + 1;
    }
```

**Copy**
**Example:**
![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1693193361592-76394601-1ea8-4c39-91ba-d69d318c2550.png#averageHue=%2326283c&clientId=uc606aab3-8d53-4&from=paste&id=ubecdebcb&originHeight=1448&originWidth=2570&originalType=url&ratio=2&rotation=0&showTitle=false&size=423786&status=done&style=none&taskId=u828d1bf6-4360-46cd-af58-426eeca741f&title=)

### 2. internal v.s. external

```solidity
// internal: 内部
    function minus() internal {
        number = number - 1;
    }

    // 合约内的函数可以调用内部函数
    function minusCall() external {
        minus();
    }
```

**Copy**
我们定义一个**internal**的**minus()**函数，每次调用使得**number**变量减 1。由于是**internal**，只能由合约内部调用，而外部不能。因此，我们必须再定义一个**external**的**minusCall()**函数，来间接调用内部的**minus()**。 **Example:**
![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1693193361525-16b100a5-bd15-49f0-b9e0-69ed104dbc55.png#averageHue=%2327293c&clientId=uc606aab3-8d53-4&from=paste&id=uf9b6bc97&originHeight=1206&originWidth=2796&originalType=url&ratio=2&rotation=0&showTitle=false&size=425059&status=done&style=none&taskId=ue69cda3e-1f5b-4d91-90ce-81aa5a8e927&title=)

### 3. payable

```solidity
// payable: 递钱，能给合约支付eth的函数
    function minusPayable() external payable returns(uint256 balance) {
        minus();
        balance = address(this).balance;
    }
```

**Copy**
我们定义一个**external payable**的**minusPayable()**函数，间接的调用**minus()**，并且返回合约里的**ETH**余额（**this**关键字可以让我们引用合约地址)。 我们可以在调用**minusPayable()**时，往合约里转入 1 个**ETH**。
![](https://cdn.nlark.com/yuque/0/2023/png/97322/1693193361997-f6bb6bfa-b0fe-4531-80c9-738512ad8ffb.png#averageHue=%232c2e40&clientId=uc606aab3-8d53-4&from=paste&id=uedf2914b&originHeight=148&originWidth=588&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=u721a7970-fa07-4468-bf30-fa3c242b963&title=)
我们可以在返回的信息中看到，合约的余额是 1 ETH。
![](https://cdn.nlark.com/yuque/0/2023/png/97322/1693193362500-8a995c20-e87a-418b-bcee-7ed3102b7a56.png#averageHue=%23252639&clientId=uc606aab3-8d53-4&from=paste&id=ue90584d7&originHeight=128&originWidth=1130&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=u07403b91-1e89-4bf8-a232-5c6754d92d0&title=)
**Example:**
![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1693193362113-4bef0768-b50e-4c07-8987-6908ab2c80fe.png#averageHue=%2327293c&clientId=uc606aab3-8d53-4&from=paste&id=u73c856b8&originHeight=1300&originWidth=2366&originalType=url&ratio=2&rotation=0&showTitle=false&size=387942&status=done&style=none&taskId=u032f7834-9ccf-439b-8954-0d22b013881&title=)

## 总结

在这一讲，我们介绍了**solidity**中的函数类型，比较难理解的是**pure**和**view**，在其他语言中没出现过。**solidity**引入**pure**和**view**关键字主要是为了节省**gas**和控制函数权限：如果用户直接调用**pure**/**view**方程是不消耗**gas**的（合约中非**pure**/**view**函数调用它们则会改写链上状态，需要付 gas）。
