# 智能合约漏洞，价值 5200 万美元的 Vyper 漏洞攻击原理分析

7 月 30 日，因为 Vyper 部分版本中的漏洞，导致 Curve、JPEG'd 等项目陆续受到攻击，损失总计超过 5200 万美元。

[Safful](https://safful.com/) 对此事件第一时间进行了技术分析，并总结了安全防范手段，希望后续项目可以引以为戒，共筑区块链行业的安全防线。

# **一、** **事件分析**

以 JPEG'd 被攻击为例：
攻击者地址：0x6ec21d1868743a44318c3c259a6d4953f9978538
攻击者合约：0x9420F8821aB4609Ad9FA514f8D2F5344C3c0A6Ab
攻击交易：0xa84aa065ce61dbb1eb50ab6ae67fc31a9da50dd2c74eefd561661bfce2f1620c
（1）攻击者（0x6ec21d18）创建 0x466B85B4 的合约，通过闪电贷向 [Balancer: Vault]借了 80,000 枚 WETH。
![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1696906049447-93f1300b-a2e2-4f07-a1c9-8eb8651a79f6.png#averageHue=%23fafafa&clientId=ue4c03dee-d978-4&from=paste&id=u3fb1594b&originHeight=28&originWidth=556&originalType=url&ratio=2&rotation=0&showTitle=false&size=12505&status=done&style=none&taskId=u7ba3598f-b9a0-4a9a-a4c6-493cbdad432&title=)
（2）攻击者（0x6ec21d18）向 pETH-ETH-f（0x9848482d）流动性池中添加了 40,000 枚 WETH，获得 32,431 枚 pETH。
![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1696906049491-4e6bf42e-97b7-4de1-af75-e36997a16eff.png#averageHue=%23cfc8bc&clientId=ue4c03dee-d978-4&from=paste&id=u22cf65b3&originHeight=47&originWidth=556&originalType=url&ratio=2&rotation=0&showTitle=false&size=41209&status=done&style=none&taskId=u447e15ce-26c7-4c45-9ff1-029654c74d9&title=)
（3）随后攻击者（0x6ec21d18）从 pETH-ETH-f（0x9848482d）流动性池中重复地移除流动性。
![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1696906049465-4a61a00b-bae4-455c-a776-5c6afb135f6a.png#averageHue=%23d4d1ba&clientId=ue4c03dee-d978-4&from=paste&id=uff33f993&originHeight=79&originWidth=556&originalType=url&ratio=2&rotation=0&showTitle=false&size=63755&status=done&style=none&taskId=ua0f0d058-af4c-4ab1-a79a-55b0e459fb4&title=)
（4）最终，攻击者（0x6ec21d18）获得 86,106 枚 WETH，归还闪电贷后，获利 6,106 枚 WETH 离场。

# **二、漏洞分析**

（1）该攻击是典型的重入攻击。对遭受攻击的项目合约进行字节码反编译，我们从下图可以发现：add_liquidity 和 remove_liquidity 两个函数在进行校验存储槽值时，所要验证的存储槽是不一样的。使用不同的存储槽，重入锁可能会失效。此时，怀疑是 Vyper 底层设计漏洞。
![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1696906049532-1b2f8898-3802-4a35-a791-8ce15d7484a9.png#averageHue=%23181f26&clientId=ue4c03dee-d978-4&from=paste&id=ua4dc7d46&originHeight=233&originWidth=554&originalType=url&ratio=2&rotation=0&showTitle=false&size=56803&status=done&style=none&taskId=ud094af57-a309-49b7-91d9-d13976d1811&title=)
（2）结合 Curve 官方的推文所说。最终，定位是 Vyper 版本漏洞。该漏洞存在于 0.2.15、0.2.16、0.3.0 版本中，在重入锁设计方面存在缺陷。我们对比 0.2.15 之前的 0.2.14 以及 0.3.0 之后的 0.3.1 版本，发现这部分代码在不断更新中，老的 0.2.14 和交心的 0.3.1 版本没有这个问题。
![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1696906049575-f04ab97b-f7f2-40ee-9029-33ab6c71bc77.png#averageHue=%23e6e1dc&clientId=ue4c03dee-d978-4&from=paste&id=udb1fa037&originHeight=237&originWidth=556&originalType=url&ratio=2&rotation=0&showTitle=false&size=96890&status=done&style=none&taskId=ub5fa5834-3a59-4bd3-8c4b-9795dec6725&title=)
![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1696906049774-a3c82871-cbc0-4ae2-a0c5-2fb687aa0649.png#averageHue=%23e4dcd6&clientId=ue4c03dee-d978-4&from=paste&id=u67a2c4e5&originHeight=372&originWidth=556&originalType=url&ratio=2&rotation=0&showTitle=false&size=116422&status=done&style=none&taskId=ub094b5da-c954-49b3-b576-d2be4d71d1d&title=)
（3）在 Vyper 对应的重入锁相关设置文件 data_positions.py 中，storage_slot 的值会被覆盖。在 ret 中，第一次获取锁的 slot 为 0，然后再次调用函数时会将锁的 slot 加 1，此时的重入锁会失效。
![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1696906049769-52bb86db-5745-4463-98e6-facefece006f.png#averageHue=%23e2fce8&clientId=ue4c03dee-d978-4&from=paste&id=u2a7e0414&originHeight=211&originWidth=556&originalType=url&ratio=2&rotation=0&showTitle=false&size=56520&status=done&style=none&taskId=ud276cfc7-1cb1-4f42-ada8-f89af97bd56&title=)
漏洞总结：本次攻击事件根本原因是 Vyper 的 0.2.15、0.2.16、0.3.0 版本的重入锁相关设计不合理，并且没有进行足够全面的功能测试。导致后期使用这些版本的项目中重入锁失效，最终遭受了黑客攻击。

# **三、安全建议**

对于本次攻击事件，开发人员在日常开发中应当采取有以下的安全措施：
（1） 项目方需保障功能设计合理并对代码进行全面测试，防止遗漏某些功能的测试。
（2） 项目发版前，需要向第三方专业的审计团队寻求技术帮助。
