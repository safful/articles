# 智能合约漏洞案例，Palmswap 90 万美元漏洞分析

据[Safful](https://safful.com/)安全团队情报，2023 年 7 月 25 日，BSC 链上的 Palmswap 项目遭到攻击，攻击者获利超 90 万美元。[Safful](https://safful.com/)安全团队介入分析后将结果分享如下：

![](https://cdn.nlark.com/yuque/0/2023/png/97322/1694920985057-bc78f547-0097-4fcb-842b-65762bb638d7.png#averageHue=%23010101&clientId=u35db1b89-c853-4&from=paste&id=u3608b870&originHeight=963&originWidth=1080&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=u8f479b9f-ea98-4d76-a21f-950293fdd86&title=)

# **相关信息**

Palmswap v2 提供了一个高流动性、功能强大且用户友好的去中心化杠杆交易平台。其中 PLP 是 Palmswap 交易平台的流动性提供者代币，它由用于杠杆交易的 USDT 资产指数组成。PLP 可以用 USDT 铸造，然后用 USDT 烧回去。铸造和重新燃烧的价格是用指数中资产的总价值（包括未平仓头寸的损益）除以 PLP 供应量得到。

以下是本次攻击涉及的相关地址：

攻击者 EOA 地址：
0xf84efa8a9f7e68855cf17eaac9c2f97a9d131366

攻击合约地址：
0x55252a6d50bfad0e5f1009541284c783686f7f25

攻击交易：
https://bscscan.com/tx/0x62dba55054fa628845fecded658ff5b1ec1c5823f1a5e0118601aa455a30eac9

被攻击的合约地址：
0xd990094a611c3de34664dd3664ebf979a1230fc1
0xa68f4b2c69c7f991c3237ba9b678d75368ccff8f
0x806f709558cdbba39699fbf323c8fda4e364ac7a

# **攻击核心点**

在 Palmswap 交易平台中，PLP 代币的价格基于金库中的 USDT 数量和 PLP 代币总供应量。

![](https://cdn.nlark.com/yuque/0/2023/png/97322/1694920984948-42f83027-cb16-44b0-a94b-28b18ef24536.png#averageHue=%23202427&clientId=u35db1b89-c853-4&from=paste&id=uc6d4522e&originHeight=386&originWidth=1080&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=u8e33f161-6583-477f-8caf-649a82b1846&title=)

而如果金库合约没有开启管理员模式的话，任何用户都可以通过直接用 USDT 购买 USDP 代币的方式来提高金库中的 USDT 数量，且不改变 PLP 代币的总供应量，这将导致攻击者可以利用闪电贷恶意操控 PLP 代币的价格获利。

# **具体细节分析**

1. 攻击者首先通过闪电贷借出 3,000,000 枚 USDT，并用其中 1,000,000 枚 USDT 去添加流动性，铸造 PLP 代币。

![](https://cdn.nlark.com/yuque/0/2023/png/97322/1694920985034-cc0faf9d-e040-4165-8f3b-198f59a5769b.png#averageHue=%23282e36&clientId=u35db1b89-c853-4&from=paste&id=u1b7fa445&originHeight=173&originWidth=1080&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=u0d94ad82-638d-43a9-9514-6aca2be8e10&title=)

2. 在添加流动性的函数中，攻击者先转入 USDT，并调用金库合约的 buyUSDP 函数购买 USDP 代币，之后用通过购买的 USDP 代币数量计算所需铸造的 PLP 代币数量。

![](https://cdn.nlark.com/yuque/0/2023/png/97322/1694920984896-a5b35758-4295-424d-b8e2-dc8a6eb3fb4d.png#averageHue=%23222222&clientId=u35db1b89-c853-4&from=paste&id=u174f90ee&originHeight=720&originWidth=860&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=ucf4827b1-8dc3-40f1-8eea-f6bac83e235&title=)

此处攻击者用 1,000,000 枚 USDT 购买了 996,769 枚 USDP，并铸造出了约 996,324 枚 PLP 代币。

![](https://cdn.nlark.com/yuque/0/2023/png/97322/1694920984962-cb4370e3-2801-4ff6-9288-35d3c656d942.png#averageHue=%23262d35&clientId=u35db1b89-c853-4&from=paste&id=ua59cf818&originHeight=149&originWidth=1080&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=u21ee8ace-6481-478e-aa6b-dae77f13ab3&title=)

3. 接着攻击者调用金库合约中的 buyUSDP 函数，该函数在通过管理员验证后可以直接用 USDT 购买 USDP 代币，同时会增加金库合约中的 USDT 代币储备。

![](https://cdn.nlark.com/yuque/0/2023/png/97322/1694920985241-9f7500a5-46ea-43a1-9cf0-e002a1f695f6.png#averageHue=%23232323&clientId=u35db1b89-c853-4&from=paste&id=u1bc40093&originHeight=810&originWidth=720&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=u1341a27e-fc51-4d43-b254-7c61a32845e&title=)

![](https://cdn.nlark.com/yuque/0/2023/png/97322/1694920985385-a02981f4-f989-4ce8-8381-c0c0c3933030.png#averageHue=%23222120&clientId=u35db1b89-c853-4&from=paste&id=u52c90a75&originHeight=197&originWidth=1080&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=u51e94092-c808-499b-b6f0-ff8a3925c2e&title=)

但是由于金库合约并没有开启管理员模式，导致任何用户都可以直接调用金库合约的 buyUSDP 函数来购买 USDP 代币，所以攻击者直接将闪电贷剩余的 2,000,000 枚 USDT 转入金库合约并购买 USDP 代币。

![](https://cdn.nlark.com/yuque/0/2023/png/97322/1694920985320-e2fcdf08-c6f6-4fb4-833a-f1c1b58fdf03.png#averageHue=%23f7f7f8&clientId=u35db1b89-c853-4&from=paste&id=ud771a339&originHeight=523&originWidth=1080&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=u43bffc0a-f6f8-44cc-b934-dffbf8a1828&title=)

![](https://cdn.nlark.com/yuque/0/2023/png/97322/1694920985764-97ca24ad-1b29-49b3-8ee6-87ee92fd6e05.png#averageHue=%23282d34&clientId=u35db1b89-c853-4&from=paste&id=uf699fe1c&originHeight=241&originWidth=1080&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=u8222991a-5428-4557-8b34-e5b7d428262&title=)

4. 紧跟着，攻击者立马移除流动性，此时会燃烧掉所有的 PLP 代币并卖出 USDP 代币。但是可以发现，攻击者燃烧掉 996,311 枚 PLP 代币却获取了 1,962,472 枚 USDP 代币。

![](https://cdn.nlark.com/yuque/0/2023/png/97322/1694920985578-d592fbf9-6c7c-45b6-ab90-1d061a5a227d.png#averageHue=%23262b32&clientId=u35db1b89-c853-4&from=paste&id=ube7be260&originHeight=369&originWidth=1080&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=ub3a5c726-9733-4583-a19e-4f54a4be2a7&title=)

这是由于在上一步购买 USDP 后，金库中的 USDT 代币数量突然暴增，而 PLP 代币的价格基于金库中 USDT 的余额进行计算的，因此 PLP 代币的价格也被拉高，使得攻击者获取了超出预期的 USDP 代币。

![](https://cdn.nlark.com/yuque/0/2023/png/97322/1694920985618-55cfcc73-ff03-4a85-a404-ff8b0cab4e0a.png#averageHue=%23252323&clientId=u35db1b89-c853-4&from=paste&id=u20ccd5e1&originHeight=720&originWidth=937&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=u7dea2ddd-cb1d-4720-a976-ea812b57036&title=)

5. 最后，攻击者调用金库合约的 sellUSDP 函数，卖出第三步中获取的剩余 USDP 代币，换成 USDT 后归还闪电贷，获利离场。

![](https://cdn.nlark.com/yuque/0/2023/png/97322/1694920985804-ceca1366-7f3f-49f4-9d24-fd07140a34fe.png#averageHue=%23272c33&clientId=u35db1b89-c853-4&from=paste&id=uf1e1ed4d&originHeight=309&originWidth=1080&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=u577fbbc2-e8c6-4c06-9bda-6f0d1d530f9&title=)

# **总结**

本次攻击事件是由于核心函数的权限管控功能未开启，并且流动性代币的价格计算模型设计得过于简单，仅取决于金库中的 USDT 代币数量和总供应量，导致攻击者可以利用闪电贷来恶意操控价格以获取非预期的利润。[Safful](https://safful.com/)安全团队建议项目方设计流动性代币的价格模型时，加入多方面因素进行限制，例如添加流动性的时间因子等，并且严格限制核心函数的权限。
