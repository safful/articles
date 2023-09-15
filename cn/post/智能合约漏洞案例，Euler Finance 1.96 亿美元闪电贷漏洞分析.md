# 智能合约漏洞案例，Euler Finance 1.96 亿美元闪电贷漏洞分析

2023 年 3 月 13 日上午 08:56:35 +UTC，DeFi 借贷协议 Euler Finance 遭遇闪电贷攻击。
Euler Finance 是一种作为无许可借贷协议运行的协议。其主要目标是为用户提供各种加密货币的借贷便利。这家总部位于英国的科技初创公司利用数学原理在以太坊和其他区块链网络上开发非托管协议，重点是实现高性能。
**根据链上数据分析，攻击者已成功执行多笔交易，造成约 1.96 亿美元**的盗窃，成为 2023 年迄今为止最大的黑客攻击。被盗资产包括价值数百万的 DAI、USDC、Staked Ether (StETH) 和 Wrapped Bitcoin (WBTC)。
被盗资产明细如下：
![](https://cdn.nlark.com/yuque/0/2023/png/97322/1694318282054-42830260-719b-4a30-bded-fe45489abbc3.png#averageHue=%23e2e0da&clientId=u0cd855e3-4520-4&from=paste&id=ued3159d8&originHeight=274&originWidth=610&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=u476fae3d-eac2-4c4f-93e7-7fc7bd9763f&title=)

## 详细分析

由于 Etoken 的 donateToReserves 函数中缺乏流动性检查，所以可能会发生攻击。攻击者使用不同的货币执行多次调用以产生利润，导致六种不同代币的巨额损失 1.96 亿美元。目前，资金仍留在攻击者的账户中。
攻击者地址为：[https://etherscan.io/address/0xb66cd966670d962c227b3eaba30a872dbfb995db](https://etherscan.io/address/0xb66cd966670d962c227b3eaba30a872dbfb995db)
攻击者的合约地址为：[https://etherscan.io/address/0x036cec1a199234fc02f72d29e596a09440825f1c](https://etherscan.io/address/0x036cec1a199234fc02f72d29e596a09440825f1c)
其中一项攻击交易可以在这里找到：[https://etherscan.io/tx/0xc310a0affe2169d1f6feec1c63dbc](https://etherscan.io/tx/0xc310a0affe2169d1f6feec1c63dbc7f7c62a887fa48795d327d4d2da2d6b111d)[7](https://etherscan.io/tx/0xc310a0affe2169d1f6feec1c63dbc7f7c62a887fa48795d327d4d2da2d6b111d)[f7c62a887fa48795d327d4d2da2d6b111d](https://etherscan.io/tx/0xc310a0affe2169d1f6feec1c63dbc7f7c62a887fa48795d327d4d2da2d6b111d)
![](https://cdn.nlark.com/yuque/0/2023/png/97322/1694318282114-a6c90bdf-a8ad-4dba-a169-3b6cfd878ec5.png#averageHue=%23f4f4ed&clientId=u0cd855e3-4520-4&from=paste&id=u75f11023&originHeight=330&originWidth=569&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=u8fa346b4-bb17-46a4-93db-132e5ab5a98&title=)

1. 攻击者首先通过闪电贷向 Aave 借了 3000 万个 DAI，然后部署了两份合约：一份用于借贷，一份用于清算。
   ![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1694318281711-abe5fb53-6056-4cc9-86ab-ea1aa6a8141e.png#averageHue=%23232c35&clientId=u0cd855e3-4520-4&from=paste&id=u97985070&originHeight=190&originWidth=700&originalType=url&ratio=2&rotation=0&showTitle=false&size=155313&status=done&style=none&taskId=u4cb3bfad-22ab-444d-85b2-325dc9cc9ea&title=)
2. 攻击者随后调用 deposit 函数并向 Euler 协议合约质押 2000 万个 DAI，获得 1950 万个 eDAI 作为回报。
   ![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1694318281703-e0244c4a-8cb2-4272-97d0-7bf11ce13a7d.png#averageHue=%23252b33&clientId=u0cd855e3-4520-4&from=paste&id=ud8f6c50b&originHeight=178&originWidth=700&originalType=url&ratio=2&rotation=0&showTitle=false&size=135918&status=done&style=none&taskId=u74a6da41-2caf-49be-9fee-f6df887ef87&title=)
3. Euler Protocol 允许用户通过调用 mint 函数借入最多 10 倍于存款的资金。攻击者利用此功能借入了 1.956 亿个 eDAI 和 2 亿个 dDAI。
   ![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1694318281743-48b06f8e-af9c-4662-962f-9d90f6e3d900.png#averageHue=%23242c34&clientId=u0cd855e3-4520-4&from=paste&id=uc356a8fe&originHeight=237&originWidth=700&originalType=url&ratio=2&rotation=0&showTitle=false&size=187339&status=done&style=none&taskId=u3d8ddc70-7f17-47dc-adb5-5651aa226db&title=)
4. 攻击者调用 repay 函数，用闪电贷借来剩下的 1000wDai 来偿还债务，并销毁 1000 万 dDAI。然后他们再次调用 mint 功能借入 1.956 亿个 eDAI 和 2 亿个 dDAI。
   ![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1694318282182-04a3fd7f-e0be-4ee5-89e8-615bb77f4ef4.png#averageHue=%23262c34&clientId=u0cd855e3-4520-4&from=paste&id=uf304aede&originHeight=366&originWidth=700&originalType=url&ratio=2&rotation=0&showTitle=false&size=287721&status=done&style=none&taskId=ueef7fd7d-c66e-4b2f-a155-4539dd65649&title=)
5. 攻击者随后调用 donateToReserves 函数并捐赠了偿还债务所需金额的 10 倍，发送了 1 亿个 eDAI。然后他们调用 liquidate 函数启动清算过程，获得了 3.1 亿个 dDAI 和 2.5 亿个 eDAI。
   ![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1694318282196-44014235-8d2d-47fb-b512-267fe3ea3e45.png#averageHue=%23262d36&clientId=u0cd855e3-4520-4&from=paste&id=u15c8a51a&originHeight=312&originWidth=700&originalType=url&ratio=2&rotation=0&showTitle=false&size=266704&status=done&style=none&taskId=u81bcb2b3-ac08-4e2a-8856-952a4c72a12&title=)
6. 攻击者调用了 withdraw 函数，获得了 3890 万个 DAI，用来偿还闪电贷借来的 3000 万个 DAI。他们从这次攻击中获利 887 万 DAI。
   ![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1694318282241-166d8d7a-7006-40df-89d3-f801bb45a9e0.png#averageHue=%23272e37&clientId=u0cd855e3-4520-4&from=paste&id=u33045d05&originHeight=437&originWidth=700&originalType=url&ratio=2&rotation=0&showTitle=false&size=375702&status=done&style=none&taskId=u6b9196b9-8000-4ce9-80c5-3ab8b0b5154&title=)

## 核心漏洞

首先，让我们看一下 donateToReserves 函数，这是用户容易被清算的地方。
![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1694318282393-c78b0511-0234-44cf-860e-471e7f3075ac.png#averageHue=%23fcfbfb&clientId=u0cd855e3-4520-4&from=paste&id=u0e3d2e70&originHeight=342&originWidth=700&originalType=url&ratio=2&rotation=0&showTitle=false&size=144256&status=done&style=none&taskId=u77121153-58b8-4989-8a8e-a4c63ec23b3&title=)
对比下图中的 donateToReserves 函数和 mint 函数，我们可以看到 donateToReserves 函数中少了一个关键步骤 checkLiquidity。
![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1694318282431-ca114fcd-05af-4d7e-9898-793af4119db6.png#averageHue=%23fcfcfc&clientId=u0cd855e3-4520-4&from=paste&id=uab21be3b&originHeight=321&originWidth=700&originalType=url&ratio=2&rotation=0&showTitle=false&size=143620&status=done&style=none&taskId=u3e5ad7d6-8148-4e6a-b4d8-caacdde6a6d&title=)
接下来，我们跟进检查了 checkLiquidity 的实现。我们发现了 Call InternalModule 函数，它调用 RiskManager 来检查并确保用户的 Etoken > Dtoken。
![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1694318282550-bde09263-762a-40cf-9a65-97ab23b31a77.png#averageHue=%23f0f0ef&clientId=u0cd855e3-4520-4&from=paste&id=u80fd7886&originHeight=197&originWidth=700&originalType=url&ratio=2&rotation=0&showTitle=false&size=101701&status=done&style=none&taskId=ua26edae3-8d88-42d0-b6bd-12fb3bf3c22&title=)
每次操作都需要检查用户的流动性，调用 checkLiquidity。
但是 donateToReserves 函数不执行这个操作，让用户先通过协议的某些函数让自己处于清算状态，然后完成清算。

## 关于攻击的官方更新

[Euler Finance 已在其官方推特](https://twitter.com/eulerfinance)(@eulerfinance)上确认了此次攻击，并表示他们目前正在与安全专业人员和执法部门合作解决该问题。
![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1694318282647-1415bd1f-7176-4e3f-b122-0fff222aea0b.png#averageHue=%23040404&clientId=u0cd855e3-4520-4&from=paste&id=u43ed32e2&originHeight=441&originWidth=700&originalType=url&ratio=2&rotation=0&showTitle=false&size=109664&status=done&style=none&taskId=u3bd083ff-db13-4b9e-b836-c3a5c2a899f&title=)
Euler Finance 最近更新了他们为协议用户收回资金的努力。他们概述了自攻击发生以来他们采取的几项行动，包括通过禁用 EToken 模块来尽快停止直接攻击，该模块阻止了存款和易受攻击的捐赠功能。
此外，他们还与各种安全组织合作，例如 TRM Labs、Chainalysis 和更广泛的以太坊安全社区，以协助调查和追回资金。Euler Finance 还与美国和英国的执法部门共享信息。
最后，该公司试图联系攻击者以了解更多有关潜在恢复选项的信息。
![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1694318282659-d8675b46-9c4b-41c6-939e-1c4649641e58.png#averageHue=%23070707&clientId=u0cd855e3-4520-4&from=paste&id=u34837d7c&originHeight=556&originWidth=700&originalType=url&ratio=2&rotation=0&showTitle=false&size=186531&status=done&style=none&taskId=ua29c794a-d4d3-4627-a251-86836226a2d&title=)

## 结论

最近针对 Euler Finance 协议的攻击凸显了实施严格安全措施的重要性，例如进行彻底[审计和定期检查](https://safful.com/)漏洞。
随着去中心化金融生态系统的不断发展，项目必须优先考虑用户资金的安全并采用最佳实践来降低未来类似攻击的风险。
