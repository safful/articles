# Solidity 小白教程：13. 继承

这一讲，我们介绍**solidity**中的继承（**inheritance**），包括简单继承，多重继承，以及修饰器（**modifier**）和构造函数（**constructor**）的继承。

## 继承

继承是面向对象编程很重要的组成部分，可以显著减少重复代码。如果把合约看作是对象的话，**solidity**也是面向对象的编程，也支持继承。

### 规则

- **virtual**: 父合约中的函数，如果希望子合约重写，需要加上**virtual**关键字。
- **override**：子合约重写了父合约中的函数，需要加上**override**关键字。

**注意**：用**override**修饰**public**变量，会重写与变量同名的**getter**函数，例如：

```solidity
mapping(address => uint256) public override balanceOf;
```

### 简单继承

我们先写一个简单的爷爷合约**Yeye**，里面包含 1 个**Log**事件和 3 个**function**: **hip()**, **pop()**, **yeye()**，输出都是”Yeye”。

```solidity
contract Yeye {
    event Log(string msg);

    // 定义3个function: hip(), pop(), man()，Log值为Yeye。
    function hip() public virtual{
        emit Log("Yeye");
    }

    function pop() public virtual{
        emit Log("Yeye");
    }

    function yeye() public virtual {
        emit Log("Yeye");
    }
}
```

我们再定义一个爸爸合约**Baba**，让他继承**Yeye**合约，语法就是**contract Baba is Yeye**，非常直观。在**Baba**合约里，我们重写一下**hip()**和**pop()**这两个函数，加上**override**关键字，并将他们的输出改为**”Baba”**；并且加一个新的函数**baba**，输出也是**”Baba”**。

```solidity
contract Baba is Yeye{
    // 继承两个function: hip()和pop()，输出改为Baba。
    function hip() public virtual override{
        emit Log("Baba");
    }

    function pop() public virtual override{
        emit Log("Baba");
    }

    function baba() public virtual{
        emit Log("Baba");
    }
}
```

我们部署合约，可以看到**Baba**合约里有 4 个函数，其中**hip()**和**pop()**的输出被成功改写成**”Baba”**，而继承来的**yeye()**的输出仍然是**”Yeye”**。

### 多重继承

**solidity**的合约可以继承多个合约。规则：

1. 继承时要按辈分最高到最低的顺序排。比如我们写一个**Erzi**合约，继承**Yeye**合约和**Baba**合约，那么就要写成**contract Erzi is Yeye, Baba**，而不能写成**contract Erzi is Baba, Yeye**，不然就会报错。
2. 如果某一个函数在多个继承的合约里都存在，比如例子中的**hip()**和**pop()**，在子合约里必须重写，不然会报错。
3. 重写在多个父合约中都重名的函数时，**override**关键字后面要加上所有父合约名字，例如**override(Yeye, Baba)**。

例子：

```solidity
contract Erzi is Yeye, Baba{
    // 继承两个function: hip()和pop()，输出值为Erzi。
    function hip() public virtual override(Yeye, Baba){
        emit Log("Erzi");
    }

    function pop() public virtual override(Yeye, Baba) {
        emit Log("Erzi");
    }
```

我们可以看到，**Erzi**合约里面重写了**hip()**和**pop()**两个函数，将输出改为**”Erzi”**，并且还分别从**Yeye**和**Baba**合约继承了**yeye()**和**baba()**两个函数。

### 修饰器的继承

**Solidity**中的修饰器（**Modifier**）同样可以继承，用法与函数继承类似，在相应的地方加**virtual**和**override**关键字即可。

```solidity
contract Base1 {
    modifier exactDividedBy2And3(uint _a) virtual {
        require(_a % 2 == 0 && _a % 3 == 0);
        _;
    }
}

contract Identifier is Base1 {

    //计算一个数分别被2除和被3除的值，但是传入的参数必须是2和3的倍数
    function getExactDividedBy2And3(uint _dividend) public exactDividedBy2And3(_dividend) pure returns(uint, uint) {
        return getExactDividedBy2And3WithoutModifier(_dividend);
    }

    //计算一个数分别被2除和被3除的值
    function getExactDividedBy2And3WithoutModifier(uint _dividend) public pure returns(uint, uint){
        uint div2 = _dividend / 2;
        uint div3 = _dividend / 3;
        return (div2, div3);
    }
}
```

**Identifier**合约可以直接在代码中使用父合约中的**exactDividedBy2And3**修饰器，也可以利用**override**关键字重写修饰器：

```solidity
modifier exactDividedBy2And3(uint _a) override {
        _;
        require(_a % 2 == 0 && _a % 3 == 0);
    }
```

### 构造函数的继承

子合约有两种方法继承父合约的构造函数。举个简单的例子，父合约**A**里面有一个状态变量**a**，并由构造函数的参数来确定：

```solidity
// 构造函数的继承
abstract contract A {
    uint public a;

    constructor(uint _a) {
        a = _a;
    }
}
```

1. 在继承时声明父构造函数的参数，例如：**contract B is A(1)**
2. 在子合约的构造函数中声明构造函数的参数，例如：

```solidity
contract C is A {
    constructor(uint _c) A(_c * _c) {}
}
```

### 调用父合约的函数

子合约有两种方式调用父合约的函数，直接调用和利用**super**关键字。

1. 直接调用：子合约可以直接用**父合约名.函数名()**的方式来调用父合约函数，例如**Yeye.pop()**。

```solidity
function callParent() public{
        Yeye.pop();
    }
```

1. **super**关键字：子合约可以利用**super.函数名()**来调用最近的父合约函数。**solidity**继承关系按声明时从右到左的顺序是：**contract Erzi is Yeye, Baba**，那么**Baba**是最近的父合约，**super.pop()**将调用**Baba.pop()**而不是**Yeye.pop()**：

```solidity
function callParentSuper() public{
        // 将调用最近的父合约函数，Baba.pop()
        super.pop();
    }
```

### 钻石继承

在面向对象编程中，钻石继承（菱形继承）指一个派生类同时有两个或两个以上的基类。
在多重+菱形继承链条上使用**super**关键字时，需要注意的是使用**super**会调用继承链条上的每一个合约的相关函数，而不是只调用最近的父合约。
我们先写一个合约**God**，再写**Adam**和**Eve**两个合约继承**God**合约，最后让创建合约**people**继承自**Adam**和**Eve**，每个合约都有**foo**和**bar**两个函数。

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.13;

/* 继承树：
  God
 /  \
Adam Eve
 \  /
people
*/

contract God {
    event Log(string message);

    function foo() public virtual {
        emit Log("God.foo called");
    }

    function bar() public virtual {
        emit Log("God.bar called");
    }
}

contract Adam is God {
    function foo() public virtual override {
        emit Log("Adam.foo called");
    }

    function bar() public virtual override {
        emit Log("Adam.bar called");
        super.bar();
    }
}

contract Eve is God {
    function foo() public virtual override {
        emit Log("Eve.foo called");
        Eve.foo();
    }

    function bar() public virtual override {
        emit Log("Eve.bar called");
        super.bar();
    }
}

contract people is Adam, Eve {
    function foo() public override(Adam, Eve) {
        super.foo();
    }

    function bar() public override(Adam, Eve) {
        super.bar();
    }
}
```

在这个例子中，调用合约**people**中的**super.bar()**会依次调用**Eve**、**Adam**，最后是**God**合约。
虽然**Eve**、**Adam**都是**God**的子合约，但整个过程中**God**合约只会被调用一次。原因是 Solidity 借鉴了 Python 的方式，强制一个由基类构成的 DAG（有向无环图）使其保证一个特定的顺序。更多细节你可以查阅[Solidity 的官方文档](https://solidity-cn.readthedocs.io/zh/develop/contracts.html?highlight=%E7%BB%A7%E6%89%BF#index-16)。

## 在 Remix 上验证

- 合约简单继承示例, 可以观察到 Baba 合约多了 Yeye 的函数![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1694260272685-c586118f-74fb-45e9-80f8-a444367cee78.png#averageHue=%2325263a&clientId=udd864f96-6845-4&from=paste&id=u4bb8dfd2&originHeight=1526&originWidth=2508&originalType=url&ratio=2&rotation=0&showTitle=false&size=406412&status=done&style=none&taskId=uedc14c5f-3a0c-4dcb-b716-f9ac137eba3&title=)![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1694260272737-e5befcc2-659d-4ded-a092-f68cec15153c.png#averageHue=%2326283b&clientId=udd864f96-6845-4&from=paste&id=ud1e12f76&originHeight=1528&originWidth=2512&originalType=url&ratio=2&rotation=0&showTitle=false&size=463277&status=done&style=none&taskId=u4831518a-c1e4-4611-a16c-974989045c6&title=)
- 合约多重继承可以参考简单继承的操作步骤来增加部署 Erzi 合约，然后观察暴露的函数以及尝试调用来查看日志
- 修饰器继承示例![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1694260272820-9e942705-1541-41f3-b066-ad8cd03b8b0f.png#averageHue=%2327293c&clientId=udd864f96-6845-4&from=paste&id=ua3a0976d&originHeight=1512&originWidth=2482&originalType=url&ratio=2&rotation=0&showTitle=false&size=539889&status=done&style=none&taskId=u3f7a057c-a0c0-481c-8b0f-2fb0ba090a8&title=)![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1694260272706-4eb6fdf0-1f08-4fb1-a24f-9ce09773480e.png#averageHue=%2326273b&clientId=udd864f96-6845-4&from=paste&id=u80562960&originHeight=1530&originWidth=2720&originalType=url&ratio=2&rotation=0&showTitle=false&size=402637&status=done&style=none&taskId=u4baa1788-fb5b-4ca4-8b8c-d3f43dac74f&title=)![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1694260272838-8ed496f8-0dea-4f92-b2f7-14daa5510e71.png#averageHue=%2326283b&clientId=udd864f96-6845-4&from=paste&id=u4aac37fb&originHeight=1546&originWidth=2726&originalType=url&ratio=2&rotation=0&showTitle=false&size=487259&status=done&style=none&taskId=ua70fd2ae-a140-4c0d-94b2-e7ac3e4b62b&title=)
- 构造函数继承示例![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1694260273195-b788cb40-97f0-46f9-b0f9-c7ee086aaa9c.png#averageHue=%2325273a&clientId=udd864f96-6845-4&from=paste&id=u823a89ed&originHeight=1388&originWidth=2492&originalType=url&ratio=2&rotation=0&showTitle=false&size=326837&status=done&style=none&taskId=u50dd93e3-9db9-4a8a-8ba7-ce6f32802bd&title=)![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1694260273342-0e6f9d1c-df97-48bd-9cd7-583d7e059f2a.png#averageHue=%2326273b&clientId=udd864f96-6845-4&from=paste&id=u3bfd44c8&originHeight=1420&originWidth=2504&originalType=url&ratio=2&rotation=0&showTitle=false&size=345222&status=done&style=none&taskId=uf0dbfe5e-e2aa-44cc-82f0-109834f812c&title=)
- 调用父合约示例![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1694260273345-c7bfa5a5-b889-4712-89f2-23b61e503ddc.png#averageHue=%2326273a&clientId=udd864f96-6845-4&from=paste&id=u06be5e24&originHeight=1376&originWidth=2482&originalType=url&ratio=2&rotation=0&showTitle=false&size=397269&status=done&style=none&taskId=u7aac4c0e-05ec-475a-8ac1-6fde56ba6b3&title=)![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1694260273428-c3455b40-d6b6-4881-bb58-c1a33c527c86.png#averageHue=%2325273a&clientId=udd864f96-6845-4&from=paste&id=u219ac9a8&originHeight=1356&originWidth=2492&originalType=url&ratio=2&rotation=0&showTitle=false&size=365801&status=done&style=none&taskId=uf55e4ae5-4183-44c1-b5e4-846fc146f48&title=)
- 菱形继承示例![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1694260273320-749e69cb-5c7c-45ff-b85e-eb6658cc12b3.png#averageHue=%231c1e28&clientId=udd864f96-6845-4&from=paste&id=ubdc75f6d&originHeight=1221&originWidth=1199&originalType=url&ratio=2&rotation=0&showTitle=false&size=174837&status=done&style=none&taskId=ucda856b1-5d6f-4594-a152-110040a074c&title=)

## 总结

这一讲，我们介绍了**solidity**继承的基本用法，包括简单继承，多重继承，修饰器和构造函数的继承、调用父合约中的函数，以及多重继承中的菱形继承问题。
