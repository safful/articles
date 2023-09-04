# Solidity 小白教程：9. 常数 constant 和 immutable

这一讲，我们介绍**solidity**中两个关键字，**constant**（常量）和**immutable**（不变量）。状态变量声明这个两个关键字之后，不能在合约后更改数值；并且还可以节省**gas**。另外，只有数值变量可以声明**constant**和**immutable**；**string**和**bytes**可以声明为**constant**，但不能为**immutable**。

## constant 和 immutable

### constant

**constant**变量必须在声明的时候初始化，之后再也不能改变。尝试改变的话，编译不通过。

```solidity
// constant变量必须在声明的时候初始化，之后不能改变
    uint256 constant CONSTANT_NUM = 10;
    string constant CONSTANT_STRING = "0xAA";
    bytes constant CONSTANT_BYTES = "WTF";
    address constant CONSTANT_ADDRESS = 0x0000000000000000000000000000000000000000;
```

### immutable

**immutable**变量可以在声明时或构造函数中初始化，因此更加灵活。

```solidity
// immutable变量可以在constructor里初始化，之后不能改变
    uint256 public immutable IMMUTABLE_NUM = 9999999999;
    address public immutable IMMUTABLE_ADDRESS;
    uint256 public immutable IMMUTABLE_BLOCK;
    uint256 public immutable IMMUTABLE_TEST;
```

你可以使用全局变量例如**address(this)**，**block.number** ，或者自定义的函数给**immutable**变量初始化。在下面这个例子，我们利用了**test()**函数给**IMMUTABLE_TEST**初始化为**9**：

```solidity
// 利用constructor初始化immutable变量，因此可以利用
    constructor(){
        IMMUTABLE_ADDRESS = address(this);
        IMMUTABLE_BLOCK = block.number;
        IMMUTABLE_TEST = test();
    }

    function test() public pure returns(uint256){
        uint256 what = 9;
        return(what);
    }
```

## 在 remix 上验证

1. 部署好合约之后，通过 remix 上的**getter**函数，能获取到**constant**和**immutable**变量初始化好的值。![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1693798656740-1fe6aee2-a748-4bf3-a35f-189aadb7cb6f.png#averageHue=%232c3347&clientId=uc99542cf-b174-4&from=paste&id=u072e336e&originHeight=636&originWidth=404&originalType=url&ratio=2&rotation=0&showTitle=false&size=36377&status=done&style=none&taskId=udd005ee8-5864-4f57-b04d-cfdd8bf5d65&title=)
2. **constant**变量初始化之后，尝试改变它的值，会编译不通过并抛出**TypeError: Cannot assign to a constant variable.**的错误。

![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1693798656739-9146e8d8-18bb-4293-a968-ef8463db72f1.png#averageHue=%23272a3e&clientId=uc99542cf-b174-4&from=paste&id=u4bcba180&originHeight=79&originWidth=670&originalType=url&ratio=2&rotation=0&showTitle=false&size=11277&status=done&style=none&taskId=ue2a252cd-a92a-4499-8c88-1d343aba49d&title=)

1. **immutable**变量初始化之后，尝试改变它的值，会编译不通过并抛出**TypeError: Immutable state variable already initialized.**的错误。![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1693798656716-31a1d92f-eb5a-4eb6-8dce-747a044c922a.png#averageHue=%23272a3f&clientId=uc99542cf-b174-4&from=paste&id=u7dba6d8b&originHeight=120&originWidth=449&originalType=url&ratio=2&rotation=0&showTitle=false&size=9796&status=done&style=none&taskId=u4d40e8ca-2dc5-4761-b6d2-deed27bb321&title=)

## 总结

这一讲，我们介绍**solidity**中两个关键字，**constant**（常量）和**immutable**（不变量），让不应该变的变量保持不变。这样的做法能在节省**gas**的同时提升合约的安全性。
