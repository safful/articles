# Solidity 小白教程：6. 引用类型, array, struct

这一讲，我们将介绍**solidity**中的两个重要变量类型：数组（**array**）和结构体（**struct**）。

## 数组 array

数组（**Array**）是**solidity**常用的一种变量类型，用来存储一组数据（整数，字节，地址等等）。数组分为固定长度数组和可变长度数组两种：

- 固定长度数组：在声明时指定数组的长度。用**T[k]**的格式声明，其中**T**是元素的类型，**k**是长度，例如：

```solidity
// 固定长度 Array
    uint[8] array1;
    bytes1[5] array2;
    address[100] array3;
```

- 可变长度数组（动态数组）：在声明时不指定数组的长度。用**T[]**的格式声明，其中**T**是元素的类型，例如：

```solidity
// 可变长度 Array
    uint[] array4;
    bytes1[] array5;
    address[] array6;
    bytes array7;
```

**注意**：**bytes**比较特殊，是数组，但是不用加**[]**。另外，不能用**byte[]**声明单字节数组，可以使用**bytes**或**bytes1[]**。在 gas 上，**bytes**比**bytes1[]**便宜。因为**bytes1[]**在**memory**中要增加 31 个字节进行填充，会产生额外的 gas。但是在**storage**中，由于内存紧密打包，不存在字节填充。

### 创建数组的规则

在 solidity 里，创建数组有一些规则：

- 对于**memory**修饰的**动态数组**，可以用**new**操作符来创建，但是必须声明长度，并且声明后长度不能改变。例子：

```solidity
// memory动态数组
    uint[] memory array8 = new uint[](5);
    bytes memory array9 = new bytes(9);
```

- 数组字面常数(Array Literals)是写作表达式形式的数组，用方括号包着来初始化 array 的一种方式，并且里面每一个元素的 type 是以第一个元素为准的，例如**[1,2,3]**里面所有的元素都是 uint8 类型，因为在 solidity 中如果一个值没有指定 type 的话，默认就是最小单位的该 type，这里 int 的默认最小单位类型就是 uint8。而**[uint(1),2,3]**里面的元素都是 uint 类型，因为第一个元素指定了是 uint 类型了，我们都以第一个元素为准。 下面的合约中，对于 f 函数里面的调用，如果我们没有显式对第一个元素进行 uint 强转的话，是会报错的，因为如上所述我们其实是传入了 uint8 类型的 array，可是 g 函数需要的却是 uint 类型的 array，就会报错了。

```solidity
// SPDX-License-Identifier: GPL-3.0
pragma solidity >=0.4.16 <0.9.0;

contract C {
    function f() public pure {
        g([uint(1), 2, 3]);
    }
    function g(uint[3] memory) public pure {
        // ...
    }
}
```

- 如果创建的是动态数组，你需要一个一个元素的赋值。

```solidity
uint[] memory x = new uint[](3);
    x[0] = 1;
    x[1] = 3;
    x[2] = 4;
```

### 数组成员

- **length**: 数组有一个包含元素数量的**length**成员，**memory**数组的长度在创建后是固定的。
- **push()**: **动态数组**和**bytes**拥有**push()**成员，可以在数组最后添加一个**0**元素。
- **push(x)**: **动态数组**和**bytes**拥有**push(x)**成员，可以在数组最后添加一个**x**元素。
- **pop()**: **动态数组**和**bytes**拥有**pop()**成员，可以移除数组最后一个元素。

**Example:**
![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1693366532542-0f436886-f3fb-4c71-9d46-7fdf3217d333.png#averageHue=%2326283b&clientId=u8f9e4392-4fa6-4&from=paste&id=u3a20238c&originHeight=1086&originWidth=2278&originalType=url&ratio=2&rotation=0&showTitle=false&size=274524&status=done&style=none&taskId=u77b35d38-4bd1-4177-a31d-172cb7b662e&title=)

## 结构体 struct

**Solidity**支持通过构造结构体的形式定义新的类型。创建结构体的方法：

```solidity
// 结构体
    struct Student{
        uint256 id;
        uint256 score;
    }
```

```solidity
Student student; // 初始一个student结构体
```

给结构体赋值的两种方法：

```solidity
//  给结构体赋值
    // 方法1:在函数中创建一个storage的struct引用
    function initStudent1() external{
        Student storage _student = student; // assign a copy of student
        _student.id = 11;
        _student.score = 100;
    }
```

**Example:**
![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1693366532583-170d0191-189e-4ed5-b409-12c901f8b8e0.png#averageHue=%23272a3e&clientId=u8f9e4392-4fa6-4&from=paste&id=u1232f6a9&originHeight=1128&originWidth=2548&originalType=url&ratio=2&rotation=0&showTitle=false&size=356846&status=done&style=none&taskId=u3c660f64-99f2-450c-923d-10d6e22308a&title=)

```solidity
// 方法2:直接引用状态变量的struct
    function initStudent2() external{
        student.id = 1;
        student.score = 80;
    }
```

**Example:**
![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1693366532581-cdd7433e-2355-42a4-baa7-4d5be11af67e.png#averageHue=%23272a3d&clientId=u8f9e4392-4fa6-4&from=paste&id=uf78378a9&originHeight=808&originWidth=2580&originalType=url&ratio=2&rotation=0&showTitle=false&size=254136&status=done&style=none&taskId=u7195d259-e73c-4c96-a1da-ba5e29d9bb8&title=)

## 总结

这一讲，我们介绍了 solidity 中数组（**array**）和结构体（**struct**）的基本用法。下一讲我们将介绍 solidity 中的哈希表——映射（**mapping**）。
