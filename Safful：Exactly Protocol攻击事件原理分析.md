# Exactly Protocol 攻击事件原理分析

8 月 18 日，Exactly protocol 遭遇黑客攻击，攻击者已获利约 1204 万美元。
[Safful](https://safful.com/)对此事件第一时间进行了技术分析，并总结了安全防范手段，希望后续项目可以引以为戒，共筑区块链行业的安全防线。

# **一、** **事件分析**

攻击者地址：
0x3747dbbcb5c07786a4c59883e473a2e38f571af9
0x417179df13ba3ed138b0a58eaa0c3813430a20e0
0xe4f34a72d7c18b6f666d6ca53fbc3790bc9da042
攻击合约：
0x6dd61c69415c8ecab3fefd80d079435ead1a5b4d
被攻击合约：
0x675d410dcf6f343219aae8d1dde0bfab46f52106
攻击交易：
0x3d6367de5c191204b44b8a5cf975f257472087a9aadc59b5d744ffdef33a520e
0x1526acfb7062090bd5fed1b3821d1691c87f6c4fb294f56b5b921f0edf0cfad6
0xe8999fb57684856d637504f1f0082b69a3f7b34dd4e7597bea376c9466813585

攻击流程：
（1）攻击者（0x417179df）先通过攻击合约（0x6dd61c69）创建了多个恶意市场代币合约和多个 uniswapPool 合约。

![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1693192270991-d6bf5949-cfb9-482f-9270-6e140462faa5.png#averageHue=%23fefefd&clientId=u13823a1f-66d3-4&from=paste&id=ub17f927b&originHeight=609&originWidth=1280&originalType=url&ratio=2&rotation=0&showTitle=false&size=235087&status=done&style=none&taskId=u6d66e5fc-9777-4acf-b92f-f86a71dbc65&title=)
![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1693192271007-862412ba-1aeb-42f9-a091-ea6acd45f1ef.png#averageHue=%23d2c1af&clientId=u13823a1f-66d3-4&from=paste&id=u448644d6&originHeight=604&originWidth=1280&originalType=url&ratio=2&rotation=0&showTitle=false&size=339711&status=done&style=none&taskId=ue1692a8b-ff91-48fb-902c-ab6a9b6dabd&title=)

（2）随后调用被攻击合约（0x675d410d）的 leverage 函数并传入一个恶意市场代币地址。

![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1693192270614-8e386785-6326-4e83-8753-9b96d6547ce8.png#averageHue=%23fab5b0&clientId=u13823a1f-66d3-4&from=paste&id=u61d6a7db&originHeight=60&originWidth=1280&originalType=url&ratio=2&rotation=0&showTitle=false&size=48510&status=done&style=none&taskId=u68ea8c2d-c517-4109-a91b-d4d2159793b&title=)

（3）在 leverage 函数中通过 deposit 函数到 pool 合约中添加 USDC 和恶意市场代币的流动性并重入到被攻击合约（0x675d410d）的 crossDeleverage 函数

![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1693192271048-2a3e389e-1cc1-4224-aafb-81bd2bccf68e.png#averageHue=%23f8f6f6&clientId=u13823a1f-66d3-4&from=paste&id=udd2ee47a&originHeight=552&originWidth=1280&originalType=url&ratio=2&rotation=0&showTitle=false&size=356752&status=done&style=none&taskId=ub3a41deb-14f1-4c27-b4a4-0e4e5a3a8c0&title=)

（4）在 crossDeleverage 函数中被攻击合约（0x675d410d）会使用 USDC 到 pool 合约中兑换恶意市场代币。

![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1693192270970-b7723e6c-8374-42dd-bc91-17f2d58f1983.png#averageHue=%2322201f&clientId=u13823a1f-66d3-4&from=paste&id=u6f9f8042&originHeight=720&originWidth=850&originalType=url&ratio=2&rotation=0&showTitle=false&size=151582&status=done&style=none&taskId=ud1b8e01d-f085-4055-939e-bff397199c4&title=)

（5）函数调用完成后，攻击合约（0x6dd61c69）移除 pool 中的流动性，随后提出兑换而来的 USDC 代币获利。

![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1693192271378-cbcf7c61-0ac8-427f-bd52-1db32177a8bd.png#averageHue=%23f6eeed&clientId=u13823a1f-66d3-4&from=paste&id=uaea8a05c&originHeight=303&originWidth=1280&originalType=url&ratio=2&rotation=0&showTitle=false&size=203348&status=done&style=none&taskId=u036b61e6-c894-4d91-8440-7575e71e3d7&title=)

（6） 多次循环上面的操作，每次攻击都会更换被攻击用户地址和恶意市场代币地址。

![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1693192271618-73da50a7-f558-42fd-86d4-ae02620cf47b.png#averageHue=%23f9f1f1&clientId=u13823a1f-66d3-4&from=paste&id=u50353c73&originHeight=515&originWidth=1280&originalType=url&ratio=2&rotation=0&showTitle=false&size=321591&status=done&style=none&taskId=uc6feaf58-4fcc-4b7c-a1bd-78f5d8e5605&title=)

（7） 然后将获得的 USDC 发送给攻击者（0xe4f34a72）。

![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1693192271558-296a8879-481b-46fe-a09f-a091cee99d20.png#averageHue=%23f7f5f4&clientId=u13823a1f-66d3-4&from=paste&id=ua6a3df07&originHeight=66&originWidth=1280&originalType=url&ratio=2&rotation=0&showTitle=false&size=50513&status=done&style=none&taskId=ucc8c7ecc-9195-4e16-a800-f5af77a4e2a&title=)

（8）多次执行同样的操作获利

![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1693192271588-71f3e42f-7d22-4467-a129-163205863492.png#averageHue=%23fdfcfb&clientId=u13823a1f-66d3-4&from=paste&id=u43618d59&originHeight=136&originWidth=1280&originalType=url&ratio=2&rotation=0&showTitle=false&size=73585&status=done&style=none&taskId=ua4a2b4ab-bfc7-4d72-bb4c-3ba336e7ce1&title=)

# **二、漏洞分析**

本次攻击利用了 DebtManager（0x675d410d）合约中的漏洞，其中的 leverage 函数未校验传入 market 参数是否为可信的市场合约，导致在 permit 函数修饰器中可将状态变量\_msgSender 修改为攻击者任意的传参地址。

![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1693192271645-fc0a3bde-7720-46b1-b928-21e9182d2249.png#averageHue=%2325201f&clientId=u13823a1f-66d3-4&from=paste&id=ud8e2f8ca&originHeight=404&originWidth=1280&originalType=url&ratio=2&rotation=0&showTitle=false&size=156789&status=done&style=none&taskId=u189d9567-c827-4e1f-9aa9-fed11be6713&title=)
![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1693192271778-85191fb3-ecf2-4ee0-a22b-7c0ed4022af7.png#averageHue=%23251f1f&clientId=u13823a1f-66d3-4&from=paste&id=u20d67a86&originHeight=363&originWidth=1280&originalType=url&ratio=2&rotation=0&showTitle=false&size=88626&status=done&style=none&taskId=u9a11a6a7-ff94-464a-99c1-c36de145c59&title=)

最后在与 pool 进行兑换时会使用用户的 exaUSDC 余额来抵消付给 pool 合约的 USDC 数量。

![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1693192271965-3c80ac3a-6c45-4f05-a214-743fa44ccef5.png#averageHue=%23f6f2f2&clientId=u13823a1f-66d3-4&from=paste&id=u50d4c943&originHeight=458&originWidth=1280&originalType=url&ratio=2&rotation=0&showTitle=false&size=332272&status=done&style=none&taskId=u7ee25311-f639-4725-bac9-bd1a3427ef7&title=)

# **三、** **安全建议**

针对本次攻击事件，我们在开发过程中应遵循以下注意事项：
（1）在涉及到有关外部地址传参和调用的情况下，应严格校验传参地址是否为可信地址。
（2）项目上线前，需要向第三方专业的审计团队进行智能合约审计。
