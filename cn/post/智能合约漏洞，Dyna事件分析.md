# 智能合约漏洞，Dyna 事件分析

# 1. 漏洞简介

[https://twitter.com/BlockSecTeam/status/1628319536117153794](https://twitter.com/BlockSecTeam/status/1628319536117153794)
[https://twitter.com/BeosinAlert/status/1628301635834486784](https://twitter.com/BeosinAlert/status/1628301635834486784)
![](https://cdn.nlark.com/yuque/0/2023/png/97322/1696045614438-c0ca4e12-3e55-462e-8c40-472b24c04fc3.png#averageHue=%23f3f0f0&clientId=u3457cc79-d645-4&from=paste&id=u35adf0f8&originHeight=638&originWidth=699&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=u4c5f6b2c-7311-4123-b4a8-b75358df5c3&title=)

# 2. 相关地址或交易

攻击交易 1：
[https://bscscan.com/tx/0x7fa89d869fd1b89ee07c206c3c89d6169317b7de8b020edd42402d9895f0819e](https://bscscan.com/tx/0x7fa89d869fd1b89ee07c206c3c89d6169317b7de8b020edd42402d9895f0819e)
攻击交易 2：
[https://bscscan.com/tx/0xc09678fec49c643a30fc8e4dec36d0507dae7e9123c270e1f073d335deab6cf0](https://bscscan.com/tx/0xc09678fec49c643a30fc8e4dec36d0507dae7e9123c270e1f073d335deab6cf0)
攻击合约：0xd360b416ce273ab2358419b1015acf476a3b30d9
攻击账号：0x0c925a25fdaac4460cab0cc7abc90ff71f410094
被攻击合约：StakingDYNA 0xa7b5eabc3ee82c585f5f4ccc26b81c3bd62ff3a9

# 3. 获利分析

![](https://cdn.nlark.com/yuque/0/2023/png/97322/1696045614552-f441a921-b65a-4ff3-bf70-179a4811e51f.png#averageHue=%23fefefd&clientId=u3457cc79-d645-4&from=paste&id=uc64c7d0e&originHeight=617&originWidth=1704&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=u1e0d9880-70da-43a3-b3e3-aa7cb6a8891&title=)

# 4. 攻击过程&漏洞原因

整个攻击过程分为两部分：

1. 准备阶段：
   0x7fa89d869fd1b89ee07c206c3c89d6169317b7de8b020edd42402d9895f0819e
   攻击者准备大量账号，调用 StakingDYNA. deposit 存入少量 DYNA 代币。

![](https://cdn.nlark.com/yuque/0/2023/png/97322/1696045614580-244d03a2-3daa-4242-8758-0ad20bc60ad4.png#averageHue=%23f8f5f4&clientId=u3457cc79-d645-4&from=paste&id=u353e7d9c&originHeight=771&originWidth=1765&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=ue5ffbec1-2862-41be-9244-e7a188495dc&title=)
在 deposit 函数中，初次 deposit 的账号将会记录下当前 block.timestamp，存储在 stakeDetail.lastProcessAt 中：
![](https://cdn.nlark.com/yuque/0/2023/png/97322/1696045614424-1d56638d-8ac6-4487-ae3c-dac43d6c0699.png#averageHue=%23242220&clientId=u3457cc79-d645-4&from=paste&id=u2d88db4f&originHeight=420&originWidth=752&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=u0e81068f-1089-46eb-9e58-16448dfbfcd&title=) 2) 攻击阶段：
0xc09678fec49c643a30fc8e4dec36d0507dae7e9123c270e1f073d335deab6cf0
攻击者通过闪电贷获取大量 dyna 代币，先通过上一步的合约调用 StakingDYNA. deposit 将代币存储在 StakingDYNA 合约中，再直接调用 StakingDYNA. redeem 取回利息。
在攻击者第二次 deposit 时，StakingDYNA 合约并未更新时间戳，计算利息的时间差错误计算为 redeem – deposit1，而实际上应该为 redeem – deposit2。因为攻击时 deposit2 与 redeem 在同一 tx 中，interest 应该为 0：
![](https://cdn.nlark.com/yuque/0/2023/png/97322/1696045614440-c43cf70a-b4cb-4d79-9490-39fde46d1891.png#averageHue=%23242120&clientId=u3457cc79-d645-4&from=paste&id=uac44e754&originHeight=413&originWidth=842&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=u6fc5d54e-53fd-4c42-b833-007fd62ef39&title=)
![](https://cdn.nlark.com/yuque/0/2023/png/97322/1696045615013-ca81bc7e-ea5a-4d8e-a302-13d031037f20.png#averageHue=%2323201f&clientId=u3457cc79-d645-4&from=paste&id=uf6e11920&originHeight=244&originWidth=825&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=ua259feda-2f78-4e44-8715-d4be29a9e3a&title=)
攻击准备 tx 时间：
![](https://cdn.nlark.com/yuque/0/2023/png/97322/1696045614998-bd39062b-f24f-4b26-acfc-aa736873b0ad.png#averageHue=%23f9e092&clientId=u3457cc79-d645-4&from=paste&id=ua7f94704&originHeight=253&originWidth=777&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=u247cf218-790d-49b9-85db-52cf9c65491&title=)
实际攻击 tx 时间：
![](https://cdn.nlark.com/yuque/0/2023/png/97322/1696045615081-31db9412-3ff4-470d-9f8b-ea213eeecd7a.png#averageHue=%23f9e092&clientId=u3457cc79-d645-4&from=paste&id=u14d06eda&originHeight=254&originWidth=769&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=ud08aa1fe-bded-4f84-a55a-b9eb9fbcc83&title=)
