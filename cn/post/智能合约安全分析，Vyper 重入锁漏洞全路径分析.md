# 智能合约安全分析，Vyper 重入锁漏洞全路径分析

# 事件背景

7 月 30 日 21:10 至 7 月 31 日 06:00 链上发生大规模攻击事件，导致多个 Curve 池的资金损失。漏洞的根源都是由于特定版本的 Vyper 中出现的重入锁故障。

# 攻击分析

通过对链上交易数据初步分析，我们对其攻击的交易进行整理归纳，并对攻击流程进一步的分析，由于攻击涉及多个交易池。
**pETH/ETH 池子被攻击交易：**
[https://etherscan.io/tx/0xa84aa065ce61dbb1eb50ab6ae67fc31a9da50dd2c74eefd561661bfce2f1620c](https://etherscan.io/tx/0xa84aa065ce61dbb1eb50ab6ae67fc31a9da50dd2c74eefd561661bfce2f1620c)
**msETH/ETH 池子被攻击交易：**
[https://etherscan.io/tx/0xc93eb238ff42632525e990119d3edc7775299a70b56e54d83ec4f53736400964](https://etherscan.io/tx/0xc93eb238ff42632525e990119d3edc7775299a70b56e54d83ec4f53736400964)
**alETH/ETH 池子被攻击交易：**
[https://etherscan.io/tx/0xb676d789bb8b66a08105c844a49c2bcffb400e5c1cfabd4bc30cca4bff3c9801](https://etherscan.io/tx/0xb676d789bb8b66a08105c844a49c2bcffb400e5c1cfabd4bc30cca4bff3c9801)
**CRV/ETH 池子被攻击交易：**
[https://etherscan.io/tx/0x2e7dc8b2fb7e25fd00ed9565dcc0ad4546363171d5e00f196d48103983ae477c](https://etherscan.io/tx/0x2e7dc8b2fb7e25fd00ed9565dcc0ad4546363171d5e00f196d48103983ae477c)
[https://etherscan.io/tx/0xcd99fadd7e28a42a063e07d9d86f67c88e10a7afe5921bd28cd1124924ae2052](https://etherscan.io/tx/0xcd99fadd7e28a42a063e07d9d86f67c88e10a7afe5921bd28cd1124924ae2052)
由于其攻击流程基本一致所以我们主要对其中 pETH/ETH 池子 攻击交易进行详细的分析：
0xa84aa065ce61dbb1eb50ab6ae67fc31a9da50dd2c74eefd561661bfce2f1620c
交易由 0x6ec21d1868743a44318c3c259a6d4953f9978538
调用攻击合约 0x9420F8821aB4609Ad9FA514f8D2F5344C3c0A6Ab ，并由该合约创建一次性攻击合约
0x466b85b49ec0c5c1eb402d5ea3c4b88864ea0f04，并通过在一次性攻击合约的构造函数中进行接下来的攻击流程；
![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1693884638405-158f8c35-d0b6-4a8c-aae7-7c861657c833.png#averageHue=%23202531&clientId=u53c0d493-e816-4&from=paste&id=ud51b65d1&originHeight=91&originWidth=1570&originalType=url&ratio=2&rotation=0&showTitle=false&size=57905&status=done&style=none&taskId=u5b383414-1c4b-4df2-a31e-19c665740d0&title=)

攻击者通过闪电贷从 Balancer 处获取 80,000 WETH ，并将其通过合约全部提取为 ETH。
![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1693884638378-07befa66-564b-4268-acc8-c871f485c09a.png#averageHue=%231a1c24&clientId=u53c0d493-e816-4&from=paste&height=459&id=u501b4856&originHeight=459&originWidth=1590&originalType=url&ratio=2&rotation=0&showTitle=false&size=212901&status=done&style=none&taskId=u8d79fd32-e8a7-455e-a03a-a3eaa3f51c9&title=&width=1590)

随后立即向 Curve 的 pETH/ETH 池提供了 40000 ETH 流动性，并收到了约 32,431.41 个 pETH-ETH LP Token
![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1693884639240-be25895c-4d8c-4406-ac5a-056048144bc8.png#averageHue=%231c1f28&clientId=u53c0d493-e816-4&from=paste&height=346&id=u026774d5&originHeight=346&originWidth=1583&originalType=url&ratio=2&rotation=0&showTitle=false&size=202451&status=done&style=none&taskId=ubcbdc31f-7490-475b-9b3a-351858966be&title=&width=1583)

随后在攻击合约中调用移除流动性函数，但在其合约函数调用栈中我们可以看出，该合约在执行移除流动性时又返回调用了合约本身的回退函数，并在回退函数中又调用了交易对的添加流动性操作。
![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1693884638396-adbc6b88-460e-45bf-b4c0-9927fed102d4.png#averageHue=%2312181d&clientId=u53c0d493-e816-4&from=paste&height=396&id=u9f929f67&originHeight=396&originWidth=1625&originalType=url&ratio=2&rotation=0&showTitle=false&size=151190&status=done&style=none&taskId=u2d55aa66-433b-4d33-8136-b8689a71ecc&title=&width=1625)

根据 pETH/ETH 交易对合约源码分析，其中在转账时使用合约的回退函数再次调用该合约，此处发生合约重入，但该 LP 合约添加流动性以及移除流动性都有使用重入锁，如下图所示：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1693884638371-8215ee28-688a-4f1a-b235-74eaa3c7bc6c.png#averageHue=%231c1b1a&clientId=u53c0d493-e816-4&from=paste&height=808&id=u62b5f58d&originHeight=808&originWidth=825&originalType=url&ratio=2&rotation=0&showTitle=false&size=89775&status=done&style=none&taskId=ua9ce1f77-09f1-48fb-961e-e56da43fcdd&title=&width=825)

但实际调用栈中还是发生了重入了，导致后续通过攻击者合约又一次向 LP Token 添加了 40,000 ETH 并获取了约 82,182.76 个 pETH/ETH Lp Token, 并在攻击合约的回调函数结束继续移除最开始添加的 32,431.41 pETH/ETH Lp Token 获得了约 3,740.21 pETH 和 34,316 ETH。
![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1693884638697-a5fb3bcc-c6bd-4a61-8761-2eef444a5974.png#averageHue=%23121920&clientId=u53c0d493-e816-4&from=paste&height=396&id=u3c47212c&originHeight=396&originWidth=1635&originalType=url&ratio=2&rotation=0&showTitle=false&size=155142&status=done&style=none&taskId=ue1e4eb4e-a249-430c-ba13-f38b03b134c&title=&width=1635)

随即又调用移除流动性函数，移除约 10,272.84 pETH/ETH LP Token 获得约 1,184.73 pETH 和 47,506.53 ETH。
![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1693884638695-232a3c3e-4482-41d2-96dd-198730b0886c.png#averageHue=%2311191f&clientId=u53c0d493-e816-4&from=paste&height=371&id=u1f09d728&originHeight=371&originWidth=1639&originalType=url&ratio=2&rotation=0&showTitle=false&size=139392&status=done&style=none&taskId=ua7a0b2b3-2fee-494f-a8f3-a9f2c101ec5&title=&width=1639)

其剩余的 7 万多 pETH/ETH LP Token 依旧留在攻击者合约中，如下图：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1693884638804-187b2f13-64a9-4177-b707-b7757e9a44ca.png#averageHue=%23141719&clientId=u53c0d493-e816-4&from=paste&height=736&id=u75fc2cb8&originHeight=736&originWidth=1419&originalType=url&ratio=2&rotation=0&showTitle=false&size=135286&status=done&style=none&taskId=u3378145d-ffb3-4249-92d7-e1b5d7ff69e&title=&width=1419)
接下来攻击者通过 Curve pETH/ETH 池将约 4,924.94 pETH 交换为的 4,285.10 ETH。
![](https://cdn.nlark.com/yuque/0/2023/png/97322/1693884639234-a0fad913-8546-40ca-a28a-f212d74b6ced.png#averageHue=%23181d23&clientId=u53c0d493-e816-4&from=paste&height=397&id=u88dd2925&originHeight=397&originWidth=1838&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=u9f7ae73f-ea36-4ca5-bb24-abc57e334e9&title=&width=1838)
最后攻击者将约 86,106.65 ETH 兑换为 WETH，并向 Balancer 归还闪电贷资金 80,000 WETH，并将 获利资金约 6,106.65 WETH 转移至 0x9420f8821ab4609ad9fa514f8d2f5344c3c0a6ab 地址。
![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1693884639066-cf1bbb93-7f2e-4559-87a2-a31c611ed3e1.png#averageHue=%2310161a&clientId=u53c0d493-e816-4&from=paste&id=u7127492a&originHeight=219&originWidth=1628&originalType=url&ratio=2&rotation=0&showTitle=false&size=75966&status=done&style=none&taskId=u91a49d7d-fd97-4aab-a7cb-03bd7b5175b&title=)

至此，针对 pETH/ETH 池子的攻击流程分析完毕，是一个很经典的重入获利操作，但是我们通过对被攻击合约的源码进行分析，其合约是存在相应的重入锁机制，正常来说可以防止重入操作，但是并没有拒绝攻击者的重入操作；
我们回顾一下 Vyper 中对重入锁的说明，知道了该重入锁实现是在函数起始部位使用指定的插槽存储是否锁定的操作。
![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1693884639193-33bdccd3-cf7b-4135-992d-0f500edc02de.png#averageHue=%23393736&clientId=u53c0d493-e816-4&from=paste&height=758&id=ubf267f67&originHeight=758&originWidth=779&originalType=url&ratio=2&rotation=0&showTitle=false&size=126367&status=done&style=none&taskId=u4b6e4087-35eb-4c5a-9672-313484d90f2&title=&width=779)

我们将被攻击 LP 合约的字节码进行反编译查看：
[https://library.dedaub.com/ethereum/address/0x466b85b49ec0c5c1eb402d5ea3c4b88864ea0f04/decompiled](https://library.dedaub.com/ethereum/address/0x466b85b49ec0c5c1eb402d5ea3c4b88864ea0f04/decompiled)
![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1693884639142-37efc682-24d2-4e31-a573-f413f2110df2.png#averageHue=%23111820&clientId=u53c0d493-e816-4&from=paste&height=303&id=u221a3618&originHeight=303&originWidth=627&originalType=url&ratio=2&rotation=0&showTitle=false&size=31340&status=done&style=none&taskId=u3062ba64-22d6-4a09-917d-d0dd04b88c0&title=&width=627)

可以看到其两个函数中存储重入锁的插槽并不一致，所以导致其重入锁失效，进而导致被攻击者利用进行获利。
Vyper 项目方官方也发推说明，其某些版本中确实存在重入锁故障。
![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1693884639283-e7e7a932-5785-4687-bc10-e4636c62d2c9.png#averageHue=%230b0b0b&clientId=u53c0d493-e816-4&from=paste&height=203&id=u00ee35c0&originHeight=203&originWidth=600&originalType=url&ratio=2&rotation=0&showTitle=false&size=23319&status=done&style=none&taskId=u9d1ea703-bb47-4b4b-b7cb-05c4f1ac3c2&title=&width=600)

通过对其 0.2.14 版本以及 0.2.15 版本对比，发现其在 Vyper 对应的重入锁相关设置文件 data_positions.py 中存在改动，其改动后的代码对重入锁的存储 key 进行单独设置，并且，每一个重入锁都会占用一个不同存储插槽，从而导致合约的重入锁功能不可用。
![](https://cdn.nlark.com/yuque/0/2023/png/97322/1693884640564-6a662de4-22de-4cad-adf0-8716e2f58d53.png#averageHue=%2313171e&clientId=u53c0d493-e816-4&from=paste&height=303&id=ubccf218f&originHeight=922&originWidth=1788&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=u80a38388-3704-4d45-8f77-c001bb8a527&title=&width=588)

该错误已在 PR 中修复：#2439 和 #2514
其中修复了添加多个重入存储插槽的问题，详见下图：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1693884639519-58d54f96-e4ce-4e7c-b17a-24fd1ff6eeb3.png#averageHue=%23141a23&clientId=u53c0d493-e816-4&from=paste&height=631&id=u9b341034&originHeight=636&originWidth=1307&originalType=url&ratio=2&rotation=0&showTitle=false&size=98525&status=done&style=none&taskId=ud3ca356a-e9c2-4f9a-9d5c-659c6558857&title=&width=1296)

# 事件总结

此次攻击事件涉及的攻击范围较广，其根本原因是因为智能合约的基础设施 Vyper 的 0.2.15、0.2.16、0.3.0 版本存在重入锁设计不合理，从而导致后期使用这些版本的项目中重入锁失效，最终遭受了黑客攻击。

- 建议在项目开发时选择稳定的技术栈以及对应版本，并对项目进行严格的测试，防止类似风险。
