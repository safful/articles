# Solidity 合约漏洞，价值 38BNB 漏洞分析

# 1. 漏洞简介

[https://twitter.com/NumenAlert/status/1626447469361102850](https://twitter.com/NumenAlert/status/1626447469361102850)
[https://twitter.com/bbbb/status/1626392605264351235](https://twitter.com/bbbb/status/1626392605264351235)
![](https://cdn.nlark.com/yuque/0/2023/png/97322/1696474489373-c9cfaf73-f531-48eb-9472-be538c28fa87.png#averageHue=%23fefefc&clientId=uebe236ee-cebb-4&from=paste&id=uf68b3317&originHeight=532&originWidth=739&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=u6f58902d-3cce-4fba-b6c9-e57adf7a1e8&title=)

# 2. 相关地址或交易

攻击交易：
[https://bscscan.com/tx/0x146586f05a4513136deab3557ad15df8f77ffbcdbd0dd0724bc66dbeab98a962](https://bscscan.com/tx/0x146586f05a4513136deab3557ad15df8f77ffbcdbd0dd0724bc66dbeab98a962)
攻击账号：0x187473cf30e2186f8fb0feda1fd21bad9aa177ca
攻击合约：0xd1b5473ffbadd80ff274f672b295ba8811b32538
被攻击合约：Starlink 0x518281f34dbf5b76e6cdd3908a6972e8ec49e345

# 3. 获利分析

![](https://cdn.nlark.com/yuque/0/2023/png/97322/1696474489493-a038dbc3-884e-463f-98dc-e5cfe21fc744.png#averageHue=%23fab546&clientId=uebe236ee-cebb-4&from=paste&id=ua09b580b&originHeight=297&originWidth=1701&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=u9fe654ec-fafb-4a84-be14-e504e86c9fd&title=)

# 4. 攻击过程&漏洞原因

查看攻击交易过程，发现攻击合约反复调用 Starlink. transfer 、Cake-LP. Skim、Cake-LP. Sync 这三个步骤，且调用完后查看了 Cake-LP、0xd1b5473ffbadd80ff274f672b295ba8811b32538 合约的 Starlink 余额。
![](https://cdn.nlark.com/yuque/0/2023/png/97322/1696474489491-3962dd12-f72d-42e3-a270-0cb27726623f.png#averageHue=%23f8f4f3&clientId=uebe236ee-cebb-4&from=paste&id=uf20964a3&originHeight=697&originWidth=1495&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=u41545bdd-e317-4ab0-a17e-9989d850e91&title=)
![](https://cdn.nlark.com/yuque/0/2023/png/97322/1696474489364-39dda171-b4a6-4b99-934e-c713f23a6edc.png#averageHue=%23242120&clientId=uebe236ee-cebb-4&from=paste&id=ua383d563&originHeight=648&originWidth=989&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=u8a89394a-8b15-438f-a91d-8f7b3a9f1f0&title=)
从中可发现 Cake-LP 的 Starlink 余额正急剧减少，势必导致池子的价格失衡。查看原因发现，当调用池子 Cake-LP 的 skim 函数时，会将池子中的代币转给地址 0x9cbb175a448b7f06c179e167e1afeffb81e51dfa。
![](https://cdn.nlark.com/yuque/0/2023/png/97322/1696474489398-67c27f7d-2912-4ca0-89cb-0a1711c038d7.png#averageHue=%23f7f3f2&clientId=uebe236ee-cebb-4&from=paste&id=u75960b4f&originHeight=323&originWidth=1473&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=u333820cc-f49a-420d-a25e-c55b545cd93&title=)
在多次重复操作后可查看到池子的 Starlink 代币数量仅为 558 单位，攻击者可以使用少量 Starlink 代币换取全部 WBNB 代币：
![](https://cdn.nlark.com/yuque/0/2023/png/97322/1696474490009-47d04ddd-b92f-4de8-b3be-7d82905bd8be.png#averageHue=%23f8f4f3&clientId=uebe236ee-cebb-4&from=paste&id=u7b89639a&originHeight=732&originWidth=1830&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=u95647b58-5def-420c-ad7d-69b8b18deef&title=)
攻击者归还闪电贷，离场。
