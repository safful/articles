# Solidity 小白教程：1. HelloWeb3 (三行代码)

## Solidity 简述

**Solidity**是以太坊虚拟机（**EVM**）智能合约的语言。同时，我认为**solidity**是玩链上项目必备的技能：区块链项目大部分是开源的，如果你能读懂代码，就可以规避很多亏钱项目。
**Solidity**具有两个特点：

1. 基于对象：学会之后，能帮你挣钱找对象。
2. 高级：不会**solidity**，在币圈显得很 low。

## 开发工具：remix

本教程中，我会用**remix**来跑**solidity**合约。**remix**是以太坊官方推荐的智能合约开发 IDE（集成开发环境），适合新手，可以在浏览器中快速部署测试智能合约，你不需要在本地安装任何程序。
网址：[remix.ethereum.org](https://remix.ethereum.org/)
进入**remix**，我们可以看到最左边的菜单有三个按钮，分别对应文件（写代码的地方），编译（跑代码），部署（部署到链上）。我们点新建（**Create New File**）按钮，就可以创建一个空白的**solidity**合约。

![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1693193272652-8b2c2a2f-379c-4549-be7f-8c19aae4bf51.png#averageHue=%23ebeceb&clientId=u5d3bcfb8-88b1-4&from=paste&id=ue4d6b5d1&originHeight=1216&originWidth=1844&originalType=url&ratio=2&rotation=0&showTitle=false&size=352594&status=done&style=none&taskId=u09afef05-b2fc-4404-b241-6b56678b656&title=)

## 第一个 Solidity 程序

很简单，只有 1 行注释+3 行代码：

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.4;
contract HelloWeb3{
    string public _string = "Hello Web3!";
}
```

**Copy**
我们拆开分析，学习 solidity 代码源文件的结构：

1. 第 1 行是注释，会写一下这个代码所用的软件许可（license），这里用的是 MIT license。如果不写许可，编译时会警告（warning），但程序可以运行。solidity 的注释由“//”开头，后面跟注释的内容（不会被程序运行）。

```solidity
// SPDX-License-Identifier: MIT
```

**Copy**

1. 第 2 行声明源文件所用的 solidity 版本，因为不同版本语法有差别。这行代码意思是源文件将不允许小于 0.8.4 版本或大于等于 0.9.0 的编译器编译（第二个条件由**^**提供）。Solidity 语句以分号（;）结尾。

```solidity
pragma solidity ^0.8.4;
```

**Copy**

1. 第 3-4 行是合约部分，第 3 行创建合约（contract），并声明合约的名字 HelloWeb3。第 4 行是合约的内容，我们声明了一个 string（字符串）变量\_string，并给他赋值 “Hello Web3!”。

```solidity
contract HelloWeb3{
    string public _string = "Hello Web3!";
}
```

**Copy**
以后我们会更细的介绍 solidity 中的变量。

## 编译并部署代码

在编辑代码的页面，按 ctrl+S 就可以编译代码，非常方便。
编译好之后，点击左侧菜单的“部署”按钮，进入部署页面。

![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1693193272494-9487ae6b-8db5-411a-b9bf-48ddab388fed.png#averageHue=%232a2c3f&clientId=u5d3bcfb8-88b1-4&from=paste&id=ua7383265&originHeight=1004&originWidth=616&originalType=url&ratio=2&rotation=0&showTitle=false&size=81715&status=done&style=none&taskId=u592ef13d-9278-4d6c-8061-61b51170c25&title=)

在默认情况下，remix 会用 JS 虚拟机来模拟以太坊链，运行智能合约，类似在浏览器里跑一条测试链。并且 remix 会分配几个测试账户给你，每个里面有 100 ETH（测试代币），可劲儿用。你点 Deploy（黄色按钮），就可以部署咱们写好的合约了。

![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1693193272597-72cc7c4c-192e-4790-a3f4-e512e19921c6.png#averageHue=%232a2d41&clientId=u5d3bcfb8-88b1-4&from=paste&id=ud09972cb&originHeight=864&originWidth=616&originalType=url&ratio=2&rotation=0&showTitle=false&size=67120&status=done&style=none&taskId=uf78f9071-0817-40c5-9e24-5825e64ff5f&title=)

部署成功后，你会在下面看到名为**HelloWeb3**的合约，点击**\_string**，就能看到我们代码中写的 “Hello Web3!” 了。

## 总结

这一讲，我们简单介绍了**solidity**，**remix**工具，并完成了第一个**solidity**程序--**HelloWeb3**。下面我们将继续**solidity**旅程！

### 中文 solidity 资料推荐：

1. [Solidity 中文文档](https://solidity-cn.readthedocs.io/zh/develop/introduction-to-smart-contracts.html)（官方文档的中文翻译）
