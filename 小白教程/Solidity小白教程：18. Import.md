# Solidity 小白教程：18. Import

**solidity**支持利用**import**关键字导入其他源代码中的合约，让开发更加模块化。

## **import**用法

- 通过源文件相对位置导入，例子：

```
文件结构
├── Import.sol
└── Yeye.sol

// 通过文件相对位置import
import './Yeye.sol';
```

- 通过源文件网址导入网上的合约，例子：

```
// 通过网址引用
import 'https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/utils/Address.sol';
```

- 通过**npm**的目录导入，例子：

```solidity
import '@openzeppelin/contracts/access/Ownable.sol';
```

- 通过**全局符号**导入特定的合约，例子：

```solidity
import {Yeye} from './Yeye.sol';
```

- 引用(**import**)在代码中的位置为：在声明版本号之后，在其余代码之前。

## 测试导入结果

我们可以用下面这段代码测试是否成功导入了外部源代码：

```solidity
contract Import {
    // 成功导入Address库
    using Address for address;
    // 声明yeye变量
    Yeye yeye = new Yeye();

    // 测试是否能调用yeye的函数
    function test() external{
        yeye.hip();
    }
}
```

![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1694577197519-ed03aa29-9df3-42b9-8edc-cf555e68ed99.png#averageHue=%2326283b&clientId=u15e6f7b1-eaa0-4&from=paste&id=ufa706899&originHeight=1414&originWidth=2532&originalType=url&ratio=2&rotation=0&showTitle=false&size=1056898&status=done&style=none&taskId=u9e9807e9-a79e-45de-ab4e-6c60418521b&title=)

## 总结

这一讲，我们介绍了利用**import**关键字导入外部源代码的方法。通过**import**关键字，可以引用我们写的其他文件中的合约或者函数，也可以直接导入别人写好的代码，非常方便。
