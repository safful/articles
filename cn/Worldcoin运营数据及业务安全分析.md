# WorldCoin 运营数据，业务安全分析

Worldcoin 的白皮书中声明，Worldcoin 旨在构建一个连接全球人类的新型数字经济系统，由 OpenAI 创始人 Sam Altman 于 2020 年发起。通过区块链技术在 Web3 世界中实现更加公平、开放和包容的经济体系，并将所有权赋予每个人。并且希望让全世界每一个人都能有最低的生活保障，提高全民基本收入，迎接共同富裕的未来。于 2023 年 7 月 24 日 Sam 在推特社交平台宣布，Worldcoin 正式推出。

# **1.** **Worldcoin 基本面**

## **1.1 业务模型**

Worldcoin 包含三个组成部分：全球数字身份（World ID）、全球数字加密货币（WLD）和一个可以承载和运用 World ID 和 WLD 加密钱包（World App）。此外，通过区块链技术，Worldcoin 将支持智能合约和去中心化应用（DApps）的发展。

2021 年 6 月，Worldcoin 首次向大众公开，并完成了 2500 万美元融资，投资方包括 a16z、Coinbase Ventures、LinkedIn 创始人 Reid Hoffman、Day One Ventures。2023 年 5 月 Worldcoin 完成 1.15 亿美元 C 轮融资，Blockchain Capital 领投，a16z、Bain Capital Crypto 和 Distributed Global 参投，估值达到 30 亿美金。

Worldcoin 围绕基于生物识别技术构建的 World ID 身份识别系统展开叙事，用户通过生物识别设备 Orb 扫描虹膜，获得 World ID，来证明自己的身份，Worldcoin 声明采用零知识证明保护用户的隐私。近年来，人工智能的迅速发展让人产生被 AI 替代的忧虑，因此真实、独特的人类身份证明成为极为重要的课题，Worldcoin 提出了一种名为"人格证明"（Proof of Personhood，PoP）的概念，采用生物识别设备 Orb，通过扫描用户虹膜来验证人的生物信息，通过零知识证明保证参与者的隐私，为人类未来提供一个可以区分人类和 AI 的解决方案。

在白皮书中，World ID 被定义为加密经济的数字身份通行证，当前，全球范围内的数字经济互动很难实现，Worldcoin 希望推动全世界所有人，随时随地都能获得 World ID，参与全球数字经济、数字治理。在未来，Worldcoin 如果成功，参与者可以拥有和直接管理数字货币，实现即时、无边界的资金转账，并且不需要第三方机构。在情况紧急的时候，比如之前乌克兰难民危机期间，可以通过数字货币 USDC 直接援助，这对跨境金融交易产生较深远的影响，极大地提高的交易效率。

## **1.2** **技术实现**

### 1.2.1 如何注册 World ID

![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1693192395716-ed3ac059-b94e-4d41-80f0-b34b9e047475.png#averageHue=%23f2f2f2&clientId=uea72d92a-067a-4&from=paste&id=ufb8a8345&originHeight=720&originWidth=921&originalType=url&ratio=2&rotation=0&showTitle=false&size=82248&status=done&style=none&taskId=u8fcd4831-0dca-463d-873b-ca296e05940&title=)
图 1 Worldcoin 生态的核心组件及交互流程

用户想要获得 World ID, 图 1 涉及以下几个简单的步骤：

1. 下载 World App
2. 定位最近的运营商，获得生物识别硬件 Orb
3. 使用 Orb 采集虹膜图像，Orb 设备确定某人是人类且身份唯一，使用神经网格将虹膜图像转化成哈希值（虹膜代码），并发送给注册服务器
4. 唯一性检查（Uniqueness service）验证这个代码与之前的代码足够不同，将身份承诺发送给注册排序器（Signup Sequencer）
5. Sequencer 服务器将用户身份证明的公匙插入以太坊主网上的默克尔树上
6. 用户得到有且仅有一个 World ID

总的来说，用户通过 Orb 扫描虹膜，Orb 和 World App 程序认证用户是真人，且虹膜代码与之前所有系统其他用户不一样，就把一个且仅有一个密钥放在列表上。用户得到一个 World ID 作为在 Worldcoin 生态系统的身份认证。但如果用户是机器人，不能把任何密钥放在列表上，也就得不到 World ID。

### 1.2.2 Orb

为了能有效区分真实人类和虚拟个体或者 AI 用户，Worldcoin 虹膜生物识别技术需要一套硬件设备 Orb，由两个半球组成，具备两个核心技术：虹膜图像捕获和处理系统。采集用户虹膜，实现身份验证，这种生物特征识别的手段。

Worldcoin 声明，出于对用户隐私安全方面的考虑，Worldcoin 主要从以下两个方面做数据处理：被扫描的虹膜图像将会在 Orb 本地运行计算虹膜代码的算法，然后被删除，接着输出参与者的虹膜代码，这个代码不会与用户的任何信息关联，也不会与用户的加密钱包-以太坊钱包关联。另外，Worldcoin 允许参与者永久删除原始生物识别数据（虹膜图像），用于保护用户的隐私和数据安全。

值得注意的是，Orb 虹膜扫描身份认证，在实际操作和推广落地方面已暴露了一些隐患。比如在东南亚、非洲等地，已经出现 Orb 数据买卖的问题，比如账户黑市，只需 30 美金左右的价格就可以买到一个虹膜验证；村民扫描虹膜帮助完成 World App 注册之后，就可领取 20 美金的奖励。

在 Worldcoin 正式推出之后，就虹膜数据是否涉及隐私安全问题，相继受到部分国家相关监管部门的调查。现阶段 Worldcoin 采用的虹膜扫描生物识别硬件 Orb 是由 Worldcoin 基金会负责运营，硬件缺乏完全的去中心化，因此，即使软件层完美且完全去中心化，Worldcoin 基金会仍然有能力在系统中插入后门，创建任意多个虚假的人类身份。Orb 在数据安全和隐私保护方面还需要进一步验证。

### 1.2.3 World App 安全性（Android 版本）

对如下版本 World App 的安全测试，我们发现如下安全风险。
APP 名称：World App
版本号：V2.2.0.6
包名：com.worldcoin
SHA256: fe8c50821cf4b8dc434221532c1847ba4af63f4a99926a9487c6d0378dbf386d
（1）恶意代码注入漏洞
将 APK 注入恶意代码，重新编译成盗版 APP 发布到网上，用于钓鱼攻击。测试中注入了获取通讯录的代码，可以将通讯录上传到私服。同时也可以上传手机相册、文件、账号密码等等各种信息到私服，导致隐私泄露，账号资产被盗等。

![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1693192395745-f5db4a64-e4b8-48f1-b61f-eeaaff41dcec.png#averageHue=%231f1f1e&clientId=uea72d92a-067a-4&from=paste&id=ufbcab1c7&originHeight=577&originWidth=1280&originalType=url&ratio=2&rotation=0&showTitle=false&size=175719&status=done&style=none&taskId=ub5fe2ecb-cfd2-42ca-88d9-c0305c63934&title=)
![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1693192395387-d82c60c3-8948-449a-9e16-19d8a99459d7.png#averageHue=%232e2e2d&clientId=uea72d92a-067a-4&from=paste&id=ua80832b4&originHeight=309&originWidth=1280&originalType=url&ratio=2&rotation=0&showTitle=false&size=119392&status=done&style=none&taskId=u0c9a2051-f4fb-4213-9fed-bf1485ef082&title=)

（2）源代码泄露风险
虽然对一些包名做了混淆处理，但是 class 中代码清晰可见，有破解和漏洞利用风险。

![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1693192395520-4b9c2ffe-b200-4a2e-9f24-b6978056f9bf.png#averageHue=%23fcfcfb&clientId=uea72d92a-067a-4&from=paste&id=u776f73c0&originHeight=426&originWidth=1280&originalType=url&ratio=2&rotation=0&showTitle=false&size=174269&status=done&style=none&taskId=u7c408e23-7f74-4df2-9248-19ab73f586b&title=)

（3）资源文件泄露风险
APP 中的 assets 目录和 res 目录下的主要资源没有做保护，可以随意修改和利用。

（4）本地数据泄露风险
APP 开发中将账号密码、key 等重要数据存储在本地，没有加密存储，容易被一些恶意程序拿到文件以及关键信息，例如开发中用到的 shared-preference、数据库等。

![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1693192395954-b0169f22-2420-4873-beed-a25a82c05ee7.png#averageHue=%23e0e0df&clientId=uea72d92a-067a-4&from=paste&id=ufe744faa&originHeight=720&originWidth=891&originalType=url&ratio=2&rotation=0&showTitle=false&size=502097&status=done&style=none&taskId=u5a2553de-37c5-4057-b4a8-a7c1730fb01&title=)

（5）Hook 调试风险
使用 Hook 框架，攻击者可以绕过系统限制、修改别人发布的代码、模拟调用隐藏 API、获取进程中的数据信息、插入恶意代码等。

![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1693192395863-33801914-a888-42ea-9d19-e0014db39441.png#averageHue=%237f7f7f&clientId=uea72d92a-067a-4&from=paste&id=uf945e386&originHeight=1233&originWidth=720&originalType=url&ratio=2&rotation=0&showTitle=false&size=188942&status=done&style=none&taskId=ubee7ff77-15d8-4378-9d87-333650c42ed&title=)

总的来说，World App 客户端做为具有金融属性的应用，没有做有效的安全策略，风险系数比较高。

## **1.3 WLD 代币经济学**

### 1.3.1 WLD 的效用

Worldcoin（WLD）是 ERC-20 代币，用户可以通过 Orb 确认自己是人类且具备唯一性，在 World App 上获得 World ID，并且可以在 Optimism 主网上获得 WLD 资助。

Worldcoin 表示，用户未来将可以在 World APP 或者其他钱包应用中使用 WLD 来进行付款，汇款，转账，支付服务费，买卖商品等服务。除此之外，WLD 还具有社区治理属性，World ID 的一人一票，结合一代币一票，这两种机制组合成新的治理方式。Worldcoin 基金会将在项目启动后，向社区征求意见，并且一起讨论如何将 World ID 和 WLD 在新的治理模式下交互运行。

虽然 WLD 同时具备支付和治理场景，但也存在安全隐患，前期 WLD 的分配非常集中，前 6 个地址持币量占比超过 90%。

### 1.3.2 WLD 总量&分配

Worldcoin 代币在启动后的 15 年内，供应总量为 100 亿枚，15 年后，如果用户通过治理开启通胀模式，每年最大通胀率为 1.5%。初始启动时的最大流通供应量为 1.43 亿枚 WLD，其中 1 亿枚借给在美国境外运营的做市商（贷款周期为 3 个月），4300 万枚分配给在项目 Pre-Launch 阶段使用 Orb 验证的用户。未来，75%的代币将分配给 Worldcoin 社区，剩下 25%分配给 TFH（Tools for Humanity，World App 开发团队）和初始团队。其中，开发团队占比 9.8%，投资者占比 13.5%，另外预留了 1.7% WLD 代币为未来 TFH 发展做储备（具体分配情况如下图 2）。

![](https://cdn.nlark.com/yuque/0/2023/jpeg/97322/1693192396139-b24b9ad1-6939-4908-b7e9-9470266c309d.jpeg#averageHue=%23f9e6d5&clientId=uea72d92a-067a-4&from=paste&id=u54d0ca97&originHeight=363&originWidth=548&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=u08db7409-b448-4d26-8b26-41e8939083e&title=)
图 2：Worldcoin 代币初始分配占比

在 Worldcoin 实现去中心化和自给自足之前，Worldcoin 基金会将支持 Worldcoin 生态系统，将 75%的代币通过社区分配给三个群体：首先是用户赠款，目标占比在 60%及以上，包括 1 枚欢迎补助金和阶段常规性补助金（也被称为创世纪赠款），第一阶段（一般是启动一周内）是 25 枚 WLD，而后的每一阶段的创世纪赠款随时间的推移预计会减少。

Worldcoin 设计了一个理想的代币分配模型（如图 3），但目前还存在很多不确定因素，如每周新用户认证数量、Worldcoin 用户使用量、商家数量等。

![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1693192395984-bef7d171-1b5e-49be-af31-f06fd49c00bf.png#averageHue=%23f9eadb&clientId=uea72d92a-067a-4&from=paste&id=u9ca2ab72&originHeight=720&originWidth=1078&originalType=url&ratio=2&rotation=0&showTitle=false&size=117695&status=done&style=none&taskId=u1698c278-b855-4d82-aea2-38dace1724d&title=)
图 3：代币分配模型

### 1.3.3 WLD 解锁&释放

关于 WLD 释放模型，有两个特点：用户赠款不被锁定，而团队和投资者代币会被锁定。图 5 显示了未来 15 年的 WLD 解锁计划。这 15 年内，Worldcoin 社区的 WLD 代币的释放速度将由治理分配，主要影响因素是 Worldcoin 用户数量的增长速度。图 4 呈现了代币解锁的最早时间，并且 100 亿代币中，75 亿分配给社区，25 亿分配给 TFH 和初始团队。这其中社区 WLD 的释放模型会根据不同的时间段做区分，具体如表 2 所示。

![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1693192396223-799c0dee-b863-4612-b8bb-a7721beccaf9.png#averageHue=%23f6d6b9&clientId=uea72d92a-067a-4&from=paste&id=ufea28041&originHeight=720&originWidth=1078&originalType=url&ratio=2&rotation=0&showTitle=false&size=80926&status=done&style=none&taskId=ud2678aeb-29eb-41fd-bbaa-1b6d346b72f&title=)
图 4：WLD 代币释放模型

|                         |                           |                               |
| ----------------------- | ------------------------- | ----------------------------- |
| 时间段                  | 期间 WLD 释放数量（十亿） | 截至时间 WLD 释放总数（十亿） |
| 正式启动                | 0.500                     | 0.500                         |
| 启动 - 第 3 年末        | 3.500                     | 4.000                         |
| 第 4 年初 - 第 6 年末   | 1.750                     | 5.750                         |
| 第 7 年初 - 第 9 年末   | 0.875                     | 6.625                         |
| 第 10 年初 - 第 15 年末 | 0.875                     | 7.500                         |

表 2：Worldcoin 社区在未来 15 年内每个时间段 WLD 释放模型
每个时间段内，释放速度相同，不同时间段释放速度不同，前几年释放速度较快，第 10 年之后趋于平缓。根据 ChainAegis 数据分析，截至 8 月 2 日，WLD 总释放量达到 5.29 亿。

### 1.3.4 Worldcoin 智能合约安全

Worldcoin 智能合约虽然经过了审计，但合约中在权限和中心化方面仍然存在一些风险项。
在 World ID 合约中，WorldIDIdentityManagerImplV1 合约继承了 openzeppelin 中的 Ownable2StepUpgradeable 合约。其中定义了中心化账户\_owner 账户以及限制只能\_owner 账户调用函数的 onlyOwner 修改器。

![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1693192396351-4f5aba54-0a22-4313-881f-8086a0758a62.png#averageHue=%23f5f0f3&clientId=uea72d92a-067a-4&from=paste&id=u14c263dc&originHeight=576&originWidth=1280&originalType=url&ratio=2&rotation=0&showTitle=false&size=290238&status=done&style=none&taskId=u9ee4ee13-41e4-49c7-876e-c0295df98dc&title=)

在 WorldIDIdentityManagerImplV1 合约中，多处使用到了 onlyOwner 修改器，尤其是涉及到一些状态变量的设置和修改函数。比如跟 stateBridge 相关的函数，在满足其他条件时，只能由\_owner 来调用函数才能有效。

![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1693192396437-9bfa2082-67b6-45b6-8c09-2a8a9e5158b8.png#averageHue=%23fefefe&clientId=uea72d92a-067a-4&from=paste&id=uabbc8559&originHeight=720&originWidth=1091&originalType=url&ratio=2&rotation=0&showTitle=false&size=215741&status=done&style=none&taskId=u9e99f068-ffda-40f9-8ad8-b48320cdf7a&title=)

还有其他的一些跟合约状态相关的设置、修改等函数。这些函数必须由\_owner 账户来调用，而且这些状态跟合约的运行状态、资产安全等有着密切的联系。一旦\_owner 账户的私钥被恶意的攻击者窃取，这将会对整个合约甚至项目带来巨大的经济损失，甚至是毁灭性打击。因此，妥善保存类似\_owner 的高权限中心化账户的私钥是项目方必须考虑的问题。

另外，也可以采用其他技术手段来缓解审计彻底解决这种中心化风险。包括但不限于：
（1）使用 m-n 多重签名。在 n 个账户中，只有其中的 m 个账户批准交易，交易才可以执行。
（2）在执行交易前，加入时间锁，只有通过一定的时间后，交易才能执行。这方便其他账户审核交易是否存在风险。只有安全的交易才能够顺利执行。
（3）使用 DAO 机制，再加上时间锁定等机制，可以彻底解决合约中的中心化风险。但也要预防 DAO 中存在的一些漏洞，如针对 DAO 的闪电贷攻击、治理攻击等。

# **2.** **运营数据**

## **2.1 注册情况**

从 2021 年 5 月到 2023 年 7 月，超过 200 万人在 30 多个国家通过 Orb 设备验证了他们的 World ID。具体分布如下方图 5 所示。很显然，认证的用户大多来自发展中国家，亚洲和非洲占比最高，均超过 30%。7 月 25 日消息，Worldcoin 发推称，今年夏天和秋天将在全球超过 35 个城市扩展至 1500 个可用 Orbs。

![](https://cdn.nlark.com/yuque/0/2023/png/97322/1693192396593-d25c4179-1eb6-4a66-9a82-2f5977af6f74.png#averageHue=%23f6f5f5&clientId=uea72d92a-067a-4&from=paste&id=u9f5218ea&originHeight=720&originWidth=1085&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=ucd4fd320-4605-475a-b15f-1228177470d&title=)

图 5：全球 Orb 认证用户分布（由 TFH 提供）

据 ChainAegis 数据显示，截至 8 月 5 日 Optimism 链上持有 WLD 代币的 Optimism 钱包数量为 408,721 个，环比 7 月 25 号提升 148%。最大的地址占比超一半，占据 57%的供应量，并且前 100 持有者总共拥有 95.08%的 WLD。

## **2.2 交易情况**

### 2.2.1 币价、流通量、市值

7 月 24 日 Worldcoin 开盘之后币价随即大幅度拉升，最高涨至 3.58 美元，但在发布后的 24 小时之内持续性下跌，最低跌到 1.66 美元，在经历两天较大波动之后，币价稳定在 2.3 美元左右，且呈现较轻微地上升趋势（图 6）。WLD 的交易量在发布后 24 小时之内达到峰值 6.47 亿，随着 Worldcoin 话题热度降温，以及各国政府的监管政策实施，以及其相关负面新闻的播报影响，用户对 WLD 保持较理性的态度，交易量逐日下滑，于 8 月 2 日跌至 1.14 亿。

![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1693192396577-03b7e8ed-4e79-4677-8b45-d695a96654d8.png#averageHue=%23fbfafa&clientId=uea72d92a-067a-4&from=paste&id=u5c1419f2&originHeight=713&originWidth=1280&originalType=url&ratio=2&rotation=0&showTitle=false&size=131782&status=done&style=none&taskId=u26a05680-48fa-4cff-9731-3c5ffeca44a&title=)
![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1693192396599-c6a1f783-96e2-4886-a615-7b9e130dbf73.png#averageHue=%23fcfcfc&clientId=uea72d92a-067a-4&from=paste&id=u66ed6ec6&originHeight=720&originWidth=1206&originalType=url&ratio=2&rotation=0&showTitle=false&size=89023&status=done&style=none&taskId=ue7d26bbd-73b4-4640-be86-febd0abffdc&title=)
![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1693192396748-36c7b0c8-811c-43e0-99fd-da3319a401eb.png#averageHue=%23fdfcfc&clientId=uea72d92a-067a-4&from=paste&id=u16562418&originHeight=720&originWidth=1206&originalType=url&ratio=2&rotation=0&showTitle=false&size=77640&status=done&style=none&taskId=u098f5f28-b554-4d2c-a18c-ef170985c4f&title=)
![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1693192396885-f7c47cc4-addb-4fd7-bbca-a15bdcaa53c0.png#averageHue=%23fbfbfb&clientId=uea72d92a-067a-4&from=paste&id=ucb93fad2&originHeight=720&originWidth=1206&originalType=url&ratio=2&rotation=0&showTitle=false&size=71519&status=done&style=none&taskId=udbdeaa83-2232-44fc-93f3-9ddac68bae5&title=)
图 6：WLD 日交易流通数据
![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1693192396903-b8007158-6185-437f-a198-aac27c0c0c31.png#averageHue=%23fcfcfc&clientId=uea72d92a-067a-4&from=paste&id=ub4c16fe7&originHeight=720&originWidth=1206&originalType=url&ratio=2&rotation=0&showTitle=false&size=90241&status=done&style=none&taskId=u27357a37-3f51-460e-8c5c-845bd528e60&title=)
图 7：WLD 日流动性指标（该指标 = 交易量/市值）

为了分析交易量与市值的关系，这里引入流动性指标的概念。显然，WLD 的流通性显著下滑，在 7 月底跌到 0.5 以下后在 8 月 2 日有小幅提升。综合来看，WLD 因为上线时间短，流通性较其他加主流密货币低，且价格也较不稳定。

### 2.2.2 WLD 交易情况

|                    |            |               |               |                  |
| ------------------ | ---------- | ------------- | ------------- | ---------------- |
| **Exchange**       | **Price**  | **+2% Depth** | **-2% Depth** | **Volume (24h)** |
| **Binance**        | $2.26      | $606,381      | $943,085      | $19,967,686      |
| **Topcredit Int**  | \* $2.2648 | $612,931      | $983,221      | \*\* $19,962,230 |
| **BIKA**           | \* $2.2658 | $320,639      | $482,037      | \*\* $8,769,475  |
| **OKX**            | $2.26      | $711,296      | $972,697      | $6,284,371       |
| **Huobi**          | $2.26      | $519,439      | $571,740      | $5,239,872       |
| **Bitget**         | $2.27      | $373,904      | $806,713      | $4,954,172       |
| **Bibox**          | $2.27      | $179,053      | $183,768      | $4,606,109       |
| **Pionex**         | $2.27      | $402,034      | $779,562      | $3,174,091       |
| **Bybit**          | $2.27      | $533,278      | $558,871      | $2,882,775       |
| **Hotcoin Global** | \* $2.2657 | $373,925      | $525,709      | \*\* $2,631,019  |

表 3：WLD TOP10 交易对均为 WLD/USDT，具体 8 月 3 日当天近 24 小时的交易价格，交易量以及深度数据
表 3 为 WLD 于 8 月 3 日当天近 24 小时交易量排名 TOP 榜单，均为 WLD/USDT 交易对。最高的是 WLD/USDT 在币安交易所，交易量达到 1,997 万美元。

为了对 WLD 进行长周期观测，这里选取中心化交易所币安和去中心化交易所 Uniswap 分别来看每天的交易情况。币安交易所交易体量大，特别是 WLD/USDT 交易对，日交易量排名第一，并且第一天飙升至 4,400 万，随后逐天下滑，跌至 500 万以下，在八月初有小幅提升，但增势不强（见图 8）；而 WLD/BTC 交易量远小于 WLD/USDT，交易走势相似。Uniswap V3 在发布日爆发虽不及中心化交易所，但是在发布稳定后三天开始有较明显的提升，并且 OP 链交易量呈上升趋势（见图 10、11）。

![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1693192397078-272f58f4-cd96-4cbd-b49d-ebabb3edb9dc.png#averageHue=%23efeeed&clientId=uea72d92a-067a-4&from=paste&id=ua2472bf6&originHeight=720&originWidth=1184&originalType=url&ratio=2&rotation=0&showTitle=false&size=138529&status=done&style=none&taskId=u3dad53fc-191f-4e44-8109-a223b9db150&title=)
图 8：WLD/USDT 在币安交易所交易情况

![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1693192397207-67357da4-9e48-4470-a842-9eab23148e92.png#averageHue=%23f1f1f1&clientId=uea72d92a-067a-4&from=paste&id=u908eca0a&originHeight=720&originWidth=1191&originalType=url&ratio=2&rotation=0&showTitle=false&size=140514&status=done&style=none&taskId=ue068489c-24df-4f91-a46f-4c412fd119e&title=)
图 9：WLD/BTC 在币安交易所交易情况

![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1693192397239-6d4b6bf3-4cd5-4212-b263-eec0f78b0055.png#averageHue=%23f7f7f7&clientId=uea72d92a-067a-4&from=paste&id=ud45db78e&originHeight=720&originWidth=1191&originalType=url&ratio=2&rotation=0&showTitle=false&size=150772&status=done&style=none&taskId=u4d07d0e9-2180-4bf7-98ed-14918e6aa85&title=)
图 10：WLD/USD 在 Uniswap V3 交易情况

![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1693192397528-8d4c26a8-82a2-4705-bbb6-bdda0068592a.png#averageHue=%23f7f7f4&clientId=uea72d92a-067a-4&from=paste&id=udf2265cf&originHeight=720&originWidth=1191&originalType=url&ratio=2&rotation=0&showTitle=false&size=150982&status=done&style=none&taskId=ucb055b00-cabd-40cb-9a73-7a998e9b544&title=)
图 11：WLD/USD 在 Uniswap V3 交易情况

# **3.全球监管**

Worldcoin 正式推出以来，各国监管机构对这个项目进行了调查，具体的事件轴和调查机构分布如图 12。
（1）英国在 Worldcoin 发布的第二天宣布对 Worldcoin 进行审查。
（2）法国隐私机构 CNIL 对 Worldcoin 生物识别数据收集和存储合法性保持怀疑态度，宣布展开调查。
（3）考虑到数据处理的数据是生物虹膜，数据较为敏感，且未来会大规模覆盖，德国数据监管机构在 7 月底宣布对其进行调查。
（4）肯尼亚是 Worldcoin 项目开启的核心地区，在发行后，当地财产、安全和数据保护服务机构在 8 月 2 号展开对其调查，并且肯尼亚内政部在 Facebook 页面发布暂停 Worldcoin 的运营。
（5）德国的右翼政党成员表示对 Worldcoin 采用的生物识别设备跟医疗无关，而是用于对全球监控，并且这种监控是永久地，涉及到扫描者生活方方面面，例如日常行为轨迹和购物习惯等。

![](https://cdn.nlark.com/yuque/0/2023/jpeg/97322/1693192397652-c2580042-3d29-4134-ab93-68350a191767.jpeg#averageHue=%23f3f3f2&clientId=uea72d92a-067a-4&from=paste&id=u5a60ca75&originHeight=395&originWidth=978&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=ud59c5a93-51d5-4a44-b9e4-f7e4af4fe64&title=)
图 12：各国监管机构对 Worldcoin 项目调查时间轴

# 4.总结

Worldcoin 作为去中心化的项目，有着广阔的愿景和野心，希望通过人格认证和公平空投席卷 Web 3 世界和数字经济时代。其技术背景 Orb 和 POP 机制相对历史的身份认证机制在抗欺诈和可扩展性方面具有一定优势，但是仍然存在隐私安全和合规风险，也受到各国监管部门的审查，需不断优化和改善。目前项目还在初期阶段，未来将会受诸多因素的影响，例如大规模硬件设备的制作，用户对新生物技术的接受度以及各地政府监管政策可行性。

在未来，加密技术、大数据和 AI 技术需要相互结合，提供更高效地技术支持和安全隐私保障，才能真正承载更多的用户和更丰富的使用场景，才能实现将全球数十亿人带入 Web 3 领域和数字经济时代的愿景。
