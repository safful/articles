# 针对 ERC777 任意调用合约 Hook 攻击

[Safful](https://safful.com/)发现了一个有趣的错误，有可能成为一些 DeFi 项目的攻击媒介。这个错误尤其与著名的 ERC777 代币标准有关。此外，它不仅仅是众所周知的黑客中常见的简单的重入问题。

![](https://cdn.nlark.com/yuque/0/2023/jpeg/97322/1693366103259-6d40709d-88dc-49f8-af4e-a6cf22e7ab6f.jpeg#averageHue=%23eaebf0&clientId=uffbaa021-2564-4&from=paste&id=uc9ee60ce&originHeight=269&originWidth=800&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=u33c368b2-251a-4f46-8ea5-9a01f04ac52&title=)

这篇文章对 ERC777 进行了全面的解释，涵盖了所有必要的细节。深入研究 ERC777 代币的具体细节的资源很少，这篇文章对于有兴趣深入了解 ERC777 代币的人来说是一个有价值的详细指南。
在文章的最后部分，将解释我们最近的发现。

## 简短描述攻击载体

这个漏洞利用了 ERC777 的特性，能够设置一个 Hook 接收函数。通过利用在目标合约中进行任意调用的能力，恶意调用者可以调用 ERC777 注册表合约，并为目标合约分配一个特定的 Hook 地址。因此，只要目标合约在未来收到 ERC777 代币，攻击者的 Hook 合约就会被触发。这个 Hook 可以以各种方式加以利用：要么用于重入攻击以窃取代币，要么只是回退交易，从而阻止目标合约发送或接收 ERC777 代币。

## ERC777 和它的 Hook

### 什么是 ERC777

ERC777 是带有转账 Hook 的代币标准之一。
这里是 EIP 描述：[https://eips.ethereum.org/EIPS/eip-777](https://eips.ethereum.org/EIPS/eip-777) ， 这里是一篇 [ERC777 实践](https://learnblockchain.cn/2019/09/27/erc777)。
实现 ERC777 代币的主要动机在于希望能够模仿原生代币转账的行为。通过在代币接收时触发智能合约，开发人员可以执行特定的逻辑，以增强功能并创建更多动态的代币交互。
然而，这些在转账过程中的额外调用使 ERC777 与 ERC20 代币不同。这些 Hook 引入了一个新的攻击载体，可能会影响到那些没有设计时考虑到在代币转账过程中处理额外调用的智能合约。这种出乎意料的行为会给这些合约带来安全风险。
以下是以太坊主网上一些具有一定流动性的 ERC777 代币的列表：
VRA：[https://etherscan.io/address/0xf411903cbc70a74d22900a5de66a2dda66507255](https://etherscan.io/address/0xf411903cbc70a74d22900a5de66a2dda66507255)
AMP：[https://etherscan.io/address/0xff20817765cb7f73d4bde2e66e067e58d11095c2](https://etherscan.io/address/0xff20817765cb7f73d4bde2e66e067e58d11095c2)
LUKSO：[https://etherscan.io/address/0xa8b919680258d369114910511cc87595aec0be6d](https://etherscan.io/address/0xa8b919680258d369114910511cc87595aec0be6d)
SKL：[https://etherscan.io/address/0x00c83aecc790e8a4453e5dd3b0b4b3680501a7a7](https://etherscan.io/address/0x00c83aecc790e8a4453e5dd3b0b4b3680501a7a7)
imBTC：[https://etherscan.io/address/0x3212b29e33587a00fb1c83346f5dbfa69a458923](https://etherscan.io/address/0x3212b29e33587a00fb1c83346f5dbfa69a458923)
CWEB：[https://etherscan.io/address/0x505b5eda5e25a67e1c24a2bf1a527ed9eb88bf04](https://etherscan.io/address/0x505b5eda5e25a67e1c24a2bf1a527ed9eb88bf04)
FLUX：[https://etherscan.io/address/0x469eda64aed3a3ad6f868c44564291aa415cb1d9](https://etherscan.io/address/0x469eda64aed3a3ad6f868c44564291aa415cb1d9)

### 当 Hook 发生时

ERC20 代币只是在转账过程中更新余额。但 ERC777 代币是这样做的：

1. 对代币发起者的地址进行 Hook 调用
2. 更新余额
3. 对代币接收方地址进行 Hook 调用

这在 VRA 代币中得到了很好的说明：
![](https://cdn.nlark.com/yuque/0/2023/png/97322/1693366103323-ad132151-dace-4ebe-95f5-22c73789525b.png#averageHue=%2316191b&clientId=uffbaa021-2564-4&from=paste&id=u45841c2e&originHeight=486&originWidth=1244&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=ua23c3809-cc9d-46aa-9505-2cc37db0c1f&title=)
源码: [https://etherscan.io/address/0xf411903cbc70a74d22900a5de66a2dda66507255](https://etherscan.io/address/0xf411903cbc70a74d22900a5de66a2dda66507255)
现在，让我们检查一下这些调用的代码：
![](https://cdn.nlark.com/yuque/0/2023/png/97322/1693366103562-d3b12a3d-b11a-45d6-9748-b8def12ea9f0.png#averageHue=%2316191b&clientId=uffbaa021-2564-4&from=paste&id=u286d5d8b&originHeight=538&originWidth=1522&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=uef5bf3f1-de25-49c4-90f9-7aafcfc8282&title=)
正如你所看到的：

1. 这个函数从\_ERC1820_REGISTRY 中读取称为 implementer（实现者）的合约
2. 如果该函数找到了一个实现者，那么这个实现者就会被调用。

让我们研究一下这个注册表，看看什么是实现者。

### 注册表和实现者

所有 ERC777 代币都与注册表（Registry）的合约有关：[https://etherscan.io/address/0x1820a4B7618BdE71Dce8cdc73aAB6C95905faD24。](https://etherscan.io/address/0x1820a4B7618BdE71Dce8cdc73aAB6C95905faD24%E3%80%82)
这个地址被 ERC777 代币用来存储设定的 Hook 接收者。这些 Hook 接收者被称为 "接口实现者"。
这意味着 Alice 可以选择 Bob 作为她的接口实现者。如果 Alice 接收或发送 ERC777 代币，Bob 将收到 Hook。
Alice 可以管理不同的 Hook 类型。因此，当 Alice 发送代币时，她可以选择 Bob 作为接口实现者，而只有当 Alice 收到代币时，她选择 Tom 作为实现者。
在大多数情况下，她也可以为不同的代币选择不同的接口实现者。
这些偏好设置被存储在这个映射的注册表中：
![](https://cdn.nlark.com/yuque/0/2023/png/97322/1693366103404-fd77d648-23db-4e9a-a963-f96a5923dec1.png#averageHue=%23191b1d&clientId=uffbaa021-2564-4&from=paste&id=u9a7f0799&originHeight=170&originWidth=1100&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=u90daf3c4-9ab7-494f-8f7c-dfab2cb296f&title=)
\_interfaceHash 是 Alice 为一个事件选择接口实现者的标识。
而任何人都可以用这个函数读取 Alice 的接口实现者：
![](https://cdn.nlark.com/yuque/0/2023/png/97322/1693366103397-78794d71-27b4-4dcb-a44a-18b37fe4a06e.png#averageHue=%23181b1d&clientId=uffbaa021-2564-4&from=paste&id=u8732731f&originHeight=364&originWidth=1280&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=ub00f77b9-96ee-4353-9a5a-001197e7034&title=)
正如你所看到的，这就是我们之前在 VRA 代码中遇到的函数。
变量\_TOKENS_SENDER_INTERFACE_HASH 被用作\_interfaceHash，它可以是任何字节。但是 VRA 代币使用这些字节来识别这种类型的 Hook：
![](https://cdn.nlark.com/yuque/0/2023/png/97322/1693366103885-9953c6be-90c4-42f3-af68-50d8978e0738.png#averageHue=%2316191b&clientId=uffbaa021-2564-4&from=paste&id=u854a792a&originHeight=190&originWidth=1018&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=uf8851f91-5ac0-484c-8ba3-76dde224c4b&title=)

### 接收 Hook

设置一个 Hook 接收函数，Alice 只需在注册表上调用这个函数并输入 Bob 的地址作为\_implementer 参数。
![](https://cdn.nlark.com/yuque/0/2023/png/97322/1693366104030-d38445f3-996a-4e68-82b1-19cf7f647165.png#averageHue=%23181c1e&clientId=uffbaa021-2564-4&from=paste&id=u62fbfc46&originHeight=534&originWidth=1500&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=u6772aacb-978c-47bf-92a6-7cadc96a53e&title=)
她还必须指定一个\_interfaceHash。她会从 VRA 代币代码中获取这个\_TOKENS_SENDER_INTERFACE_HASH。
还有一个重要的细节。
在为上面的 VRA 设置实现者后，Alice 也将会意识到，即使其他 ERC777 代币被转账，Bob 也会收到调用。
比如 [imBTC](https://etherscan.io/address/0x3212b29e33587a00fb1c83346f5dbfa69a458923)， imBTC 在发送的代币上有相同的\_interfaceHash。
这是由于所有 ERC777 代币共享相同的注册表合约来存储 Hook 的偏好设置。但这取决于 ERC777 代币为他们的 Hook 指定名称，虽然有时它们是相似的，但并不总是如此。

### 如何找到 ERC777 代币

调用注册表是所有 ERC777 都具有的特征。
因此，我们可以尝试[dune.com](http://dune.com/)来调用所有调用注册表的智能合约。
![](https://cdn.nlark.com/yuque/0/2023/png/97322/1693366104007-3d6a72f5-4b7c-48c1-9754-f52c7f22b84f.png#averageHue=%23171a1c&clientId=uffbaa021-2564-4&from=paste&id=ue073a0e6&originHeight=392&originWidth=912&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=u759054e5-cfd8-4134-9b5f-469df2453dd&title=)
我们可以使用这个 SQL 脚本。事实上，我们应该另外过滤出代币地址，但至少我们有一个完美的开始，结果有 78 个地址。
译者备注：dune [traces 表](https://dune.com/docs/data-tables/raw/evm/traces/) 会记录交易内部调用记录。

### 这个注册表是唯一可能的吗？

理论上，没有人能够保证某些代币恰好使用这个 0x1820 合约作为注册表。
但我们可以用[dune.com](http://dune.com/)来检查。
![](https://cdn.nlark.com/yuque/0/2023/png/97322/1693366104022-275425c8-b5a6-428c-862b-49051e457bc0.png#averageHue=%2317191a&clientId=uffbaa021-2564-4&from=paste&id=uf08be1e3&originHeight=332&originWidth=912&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=u91263bc7-7c04-4fe1-b0b9-64056955b0f&title=)
它返回这些地址

```
0x1820a4b7618bde71dce8cdc73aab6c95905fad24
0xc0ce3461c92d95b4e1d3abeb5c9d378b1e418030
0x820c4597fc3e4193282576750ea4fcfe34ddf0a7
```

我们检查过，0x1820 是唯一拥有有价值的 ERC777 代币的注册表。其他注册表的代币并不那么有价值。

### 可 Hook 代币的普遍情况

ERC777 不仅是一个带有 Hook 的标准。还有 ERC223、ERC995 或 ERC667。
它们并不那么稀奇。你一定听说过实现 ERC667 的[LINK 代币](https://etherscan.io/token/0x514910771af9ca656af840dff83e8264ecf986ca)。

## 使用任意调用的攻击载体

这是最近为[Safful](https://safful.com/)的客户发现的攻击载体。
研究人员通常认为 ERC777 代币会对调用发起者和接收者进行调用。但实际上，发起者和接收者可选择任意 "Bob" 作为 Hook 接收者。
因此，想象一下结合那些具有任何数据对任何地址进行任意调用的合约会发生什么？
就有任意调用功能的可以广泛存在于 DEX 聚合器、钱包、multicall 合约中。
译者注：任意调用功能是指在合约中存在类似这样的函数：
function execute(address target, uint value, string memory signature, bytes memory data, uint eta) public payable;
它可以调用任何的其他的方法。
攻击方法：

1. 攻击者找到一个允许任意调用的函数的目标合约（Target）
2. 攻击者在目标（Target）上调用：
3. registy1820.setInterfaceImplementer(Target, hookHash, Attacker)
4. 现在，我们的 Attacker 是 Target 的实现者
5. Attacker 会随着 ERC777 代币中使用的 hookHash 而被调用。
6. 每当目标合约（Target）收到 ERC777 代币时，Attacker 就会收到一个 Hook 调用。
7. 下面的攻击，取决于 Target 代码 而不同：
   - Attacker 可以在一些用户执行目标合约中的函数时进行重入
   - Attacker 可以直接回退，这样用户的交易就直接被还原了

如果 DEX 聚合器计算最佳兑换路径是通过某个有 ERC777 代币的 DEX 交易对时，那么可能会遇到问题。

## 保护

经过与客户数小时的讨论，我们找到了一个不会破坏任意调用的解决方案。
项目方最好限制使用 Registry1820 作为任意调用的地址。因此，没有攻击者能够利用任意调用来设置接口实现者。

## 经验之谈

项目和审计人员必须注意到 ERC777 中描述的 Hook 行为。这些代币不仅对接收者和发起者进行调用，也对其他一些 Hook 接收者进行调用。
在这个意义上，允许任意调用的项目必须特别注意，并考虑 ERC777 的另一个攻击载体。
