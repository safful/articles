# Solidity 合约漏洞，Dyna 价值 78BNB 漏洞分析

# 1. 漏洞简介

[https://twitter.com/BlockSecTeam/status/1628319536117153794](https://twitter.com/BlockSecTeam/status/1628319536117153794)
[https://twitter.com/BeosinAlert/status/1628301635834486784](https://twitter.com/BeosinAlert/status/1628301635834486784)
![](https://cdn.nlark.com/yuque/0/2023/png/97322/1696649870592-d8634448-51f8-42fd-bea5-874f4e71fef5.png#averageHue=%23f3f0f0&clientId=u7581863d-7fe2-4&from=paste&id=uf5580d59&originHeight=638&originWidth=699&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=ucb180bb5-e0f6-4242-8a1c-cde3b3f7d5b&title=)

# 2. 相关地址或交易

攻击交易 1：
[https://bscscan.com/tx/0x7fa89d869fd1b89ee07c206c3c89d6169317b7de8b020edd42402d9895f0819e](https://bscscan.com/tx/0x7fa89d869fd1b89ee07c206c3c89d6169317b7de8b020edd42402d9895f0819e)
攻击交易 2：
[https://bscscan.com/tx/0xc09678fec49c643a30fc8e4dec36d0507dae7e9123c270e1f073d335deab6cf0](https://bscscan.com/tx/0xc09678fec49c643a30fc8e4dec36d0507dae7e9123c270e1f073d335deab6cf0)
攻击合约：0xd360b416ce273ab2358419b1015acf476a3b30d9
攻击账号：0x0c925a25fdaac4460cab0cc7abc90ff71f410094
被攻击合约：StakingDYNA 0xa7b5eabc3ee82c585f5f4ccc26b81c3bd62ff3a9

# 3. 获利分析

![](https://cdn.nlark.com/yuque/0/2023/png/97322/1696649870695-c916128f-2a45-4dd4-a3d9-25725d081ac8.png#averageHue=%23fefefd&clientId=u7581863d-7fe2-4&from=paste&id=uf45f7b93&originHeight=617&originWidth=1704&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=ub25b0db3-4a9d-40a2-8246-0809c262718&title=)

# 4. 攻击过程&漏洞原因

整个攻击过程分为两部分：

1. 准备阶段：
   0x7fa89d869fd1b89ee07c206c3c89d6169317b7de8b020edd42402d9895f0819e
   攻击者准备大量账号，调用 StakingDYNA. deposit 存入少量 DYNA 代币。

![](https://cdn.nlark.com/yuque/0/2023/png/97322/1696649870638-cad42b43-db37-4373-aaa7-8f729993001e.png#averageHue=%23f8f5f4&clientId=u7581863d-7fe2-4&from=paste&id=u70485beb&originHeight=771&originWidth=1765&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=u65693578-e966-40bb-9347-c29bbc69244&title=)
在 deposit 函数中，初次 deposit 的账号将会记录下当前 block.timestamp，存储在 stakeDetail.lastProcessAt 中：
![](https://cdn.nlark.com/yuque/0/2023/png/97322/1696649870571-69636ecb-a9bb-4566-8959-a7869d0d555e.png#averageHue=%23242220&clientId=u7581863d-7fe2-4&from=paste&id=udfe54e63&originHeight=420&originWidth=752&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=u7c3632a5-fb4e-4634-9962-fde66e5d9ad&title=) 2) 攻击阶段：
0xc09678fec49c643a30fc8e4dec36d0507dae7e9123c270e1f073d335deab6cf0
攻击者通过闪电贷获取大量 dyna 代币，先通过上一步的合约调用 StakingDYNA. deposit 将代币存储在 StakingDYNA 合约中，再直接调用 StakingDYNA. redeem 取回利息。
在攻击者第二次 deposit 时，StakingDYNA 合约并未更新时间戳，计算利息的时间差错误计算为 redeem – deposit1，而实际上应该为 redeem – deposit2。因为攻击时 deposit2 与 redeem 在同一 tx 中，interest 应该为 0：
![](https://cdn.nlark.com/yuque/0/2023/png/97322/1696649870595-ec5e3332-31c0-436e-9b66-836a1457fcab.png#averageHue=%23242120&clientId=u7581863d-7fe2-4&from=paste&id=u23d11403&originHeight=413&originWidth=842&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=ude7b073a-5bf2-4a0f-ba57-ec46e2698e8&title=)
![](https://cdn.nlark.com/yuque/0/2023/png/97322/1696649871058-1a59cfbc-a34e-4971-bf90-79de2e323282.png#averageHue=%2323201f&clientId=u7581863d-7fe2-4&from=paste&id=u9bdbbbe3&originHeight=244&originWidth=825&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=u5e320669-8402-4cef-8e43-64e5d003600&title=)
攻击准备 tx 时间：
![](https://cdn.nlark.com/yuque/0/2023/png/97322/1696649871113-dfc506de-5e2a-4f54-9c65-d90957f0b856.png#averageHue=%23f9e092&clientId=u7581863d-7fe2-4&from=paste&id=u9c7bde3b&originHeight=253&originWidth=777&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=u529f46da-a5af-454f-b47b-fa6ca992a25&title=)
实际攻击 tx 时间：
![](https://cdn.nlark.com/yuque/0/2023/png/97322/1696649871171-df366999-5965-42d4-a098-36765f4e77f7.png#averageHue=%23f9e092&clientId=u7581863d-7fe2-4&from=paste&id=uf9a745d4&originHeight=254&originWidth=769&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=ud22f815c-5124-4177-ac2d-cb94db83bc5&title=)
