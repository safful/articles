# 智能合约漏洞，价值 50 万美元 BNO 攻击事件原理分析

北京时间 2023 年 7 月 18 日，Ocean BNO 遭受闪电贷攻击，攻击者已获利约 50 万美元。

[Safful](https://safful.com/) 对此事件第一时间进行了技术分析，并总结了安全防范手段，希望后续项目可以引以为戒，共筑区块链行业的安全防线。

# **一、 事件分析**

攻击者地址：
0xa6566574edc60d7b2adbacedb71d5142cf2677fb
攻击合约：
0xd138b9a58d3e5f4be1cd5ec90b66310e241c13cd
被攻击合约：
0xdCA503449899d5649D32175a255A8835A03E4006
攻击交易：
0x33fed54de490797b99b2fc7a159e43af57e9e6bdefc2c2d052dc814cfe0096b9
攻击流程：
（1）攻击者（0xa6566574）通过 pancakeSwap 闪电贷借取 286449 枚 BNO。
![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1696906148684-a0939b4f-42b6-4fe7-b5a2-d6897e2d3010.png#averageHue=%23f5f3f2&clientId=u69b456c8-1fef-4&from=paste&id=u71b94102&originHeight=59&originWidth=1280&originalType=url&ratio=2&rotation=0&showTitle=false&size=47601&status=done&style=none&taskId=u587d18be-f20b-433a-9ba6-2508c53fe3f&title=)
（2）随后调用被攻击合约（0xdCA50344）的 stakeNft 函数质押两个 nft。
![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1696906148879-baeaceab-3016-4482-942b-b121f2f01d83.png#averageHue=%23f9f7f7&clientId=u69b456c8-1fef-4&from=paste&id=u8161adb2&originHeight=326&originWidth=1280&originalType=url&ratio=2&rotation=0&showTitle=false&size=184446&status=done&style=none&taskId=u5b9763c5-7707-4b6a-94cd-11d3b03cd21&title=)
（3）接着调用被攻击合约（0xdCA50344）的 pledge 函数质押 277856 枚 BNO 币。
![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1696906148799-919aa5a6-4ef1-47e4-a55e-3c3da3935445.png#averageHue=%23f7f6f5&clientId=u69b456c8-1fef-4&from=paste&id=u8a439400&originHeight=121&originWidth=1280&originalType=url&ratio=2&rotation=0&showTitle=false&size=84581&status=done&style=none&taskId=ud35c64d4-65ce-4b69-bdc4-f7afe0a8498&title=)
（4）调用被攻击合约（0xdCA50344）的 emergencyWithdraw 函数提取回全部的 BNO
![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1696906149163-768880e7-fc00-41d1-9c67-68014479d29a.png#averageHue=%23f8f6f5&clientId=u69b456c8-1fef-4&from=paste&id=uda3a5b64&originHeight=61&originWidth=1280&originalType=url&ratio=2&rotation=0&showTitle=false&size=34485&status=done&style=none&taskId=u83911d33-a6a5-472e-837e-e10de348508&title=)
（5）然后调用被攻击合约（0xdCA50344）的 unstakeNft 函数，取回两个质押的 nft 并收到额外的 BNO 代币。
![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1696906148844-33870d0f-2d79-4fc7-b8fc-d8aff1ed9191.png#averageHue=%23f9f4f4&clientId=u69b456c8-1fef-4&from=paste&id=u368e38a7&originHeight=338&originWidth=1280&originalType=url&ratio=2&rotation=0&showTitle=false&size=198702&status=done&style=none&taskId=u86bed0c9-c63e-4655-a5c0-6bb68f19714&title=)
（6）循环上述过程，持续获得额外的 BNO 代币
![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1696906148985-b6adaab2-98e5-4dd5-b163-4d8ebd917199.png#averageHue=%23faf5f5&clientId=u69b456c8-1fef-4&from=paste&id=uf09bad19&originHeight=508&originWidth=1280&originalType=url&ratio=2&rotation=0&showTitle=false&size=252839&status=done&style=none&taskId=u7cf8388e-132e-4bbb-be41-daea209bb87&title=)
（7）最后归还闪电贷后将所有的 BNO 代币换成 50.5W 个 BUSD 后获利离场。
![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1696906149103-7c3a8701-a600-4e4b-ba47-b3c56826fe17.png#averageHue=%23f8f6f6&clientId=u69b456c8-1fef-4&from=paste&id=u0cf5a2e7&originHeight=341&originWidth=1280&originalType=url&ratio=2&rotation=0&showTitle=false&size=237592&status=done&style=none&taskId=u0332c614-abfe-46e9-be85-1c8e1d1e7bc&title=)

# **二、漏洞分析**

本次攻击的根本原因是：被攻击合约（0xdCA50344）中的奖励计算机制和紧急提取函数的交互逻辑出现问题，导致用户在提取本金后可以得到一笔额外的奖励代币。
![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1696906149286-862fc7af-b7b2-4297-86e3-d5f1ac989077.png#averageHue=%23242020&clientId=u69b456c8-1fef-4&from=paste&id=u3841ad7d&originHeight=510&originWidth=1280&originalType=url&ratio=2&rotation=0&showTitle=false&size=191649&status=done&style=none&taskId=u207cabc1-4701-4ce1-b0bc-885927ed4aa&title=)
![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1696906149270-8dd3fe58-8cd9-4603-be3b-74f044e74e42.png#averageHue=%2321201f&clientId=u69b456c8-1fef-4&from=paste&id=ub504d4cb&originHeight=248&originWidth=1280&originalType=url&ratio=2&rotation=0&showTitle=false&size=95074&status=done&style=none&taskId=uf63b1151-e154-4ade-8169-ea1230572bc&title=)
合约提供 emergencyWithdraw 函数用于紧急提取代币，并清除了攻击者的 allstake 总抵押量和 rewardDebt 总债务量，但并没有清除攻击者的 nftAddtion 变量，而 nftAddition 变量也是通过 allstake 变量计算得到。
![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1696906149475-4a9e407f-e66e-40fe-90cf-eecd19cfdbdd.png#averageHue=%23232020&clientId=u69b456c8-1fef-4&from=paste&id=u40ddcc5a&originHeight=575&originWidth=1280&originalType=url&ratio=2&rotation=0&showTitle=false&size=207278&status=done&style=none&taskId=u7eac9287-7553-4a49-abc2-bf1ab58672b&title=)
而在 unstakeNft 函数中仍然会计算出用户当前奖励，而在 nftAddition 变量没有被归零的情况下，pendingFit 函数仍然会返回一个额外的 BNO 奖励值，导致攻击者获得额外的 BNO 代币。
![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1696906149494-132f2e19-49d6-45a9-970c-b6fb108fe268.png#averageHue=%2321201f&clientId=u69b456c8-1fef-4&from=paste&id=udf493fbb&originHeight=720&originWidth=1151&originalType=url&ratio=2&rotation=0&showTitle=false&size=238704&status=done&style=none&taskId=udab19596-f281-46f0-a0a2-fcde260ab37&title=)

# **三、安全建议**

针对本次攻击事件，我们在开发过程中应遵循以下注意事项：
（1）在进行奖励计算时，校验用户是否提取本金。
（2）项目上线前，需要向第三方专业的审计团队寻求技术帮助。
