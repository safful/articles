# 智能合约漏洞案例，跨链桥 Allbridge 57 万美元漏洞分析

## 事件背景

[Safful](https://safful.com/)区块链安全情报平台监控到消息，北京时间 2023 年 4 月 1 日，BSC 链上 Allbridge 跨链桥受到黑客攻击，攻击者获利约 57 万美元，攻击者地址为 0xc578d755cd56255d3ff6e92e1b6371ba945e3984，被盗资金转移至 Tornado.cash 混币平台。[Safful](https://safful.com/)安全团队及时对此安全事件进行分析。

## 合约漏洞

合约中执行兑换操作函数 swapToVUsd 中计算兑换结果方式为合约中当前记录 BUSD 余额与计算转入 token 后的数量转换为 BUSD 的差值得到的，因此攻击者通过存取大量资金以及进行大量代币兑换实现对池子中代币价格控制。
![](https://cdn.nlark.com/yuque/0/2023/jpeg/97322/1694318405618-ea8ea08f-e5b5-46fb-9491-856e36f27d56.jpeg#averageHue=%23211f1f&clientId=u95e9e491-8b91-4&from=paste&id=u73b3cd36&originHeight=265&originWidth=690&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=u9b6bc897-fbba-4d88-8454-ea4a1ffcd48&title=)

## 攻击步骤

1. 攻击者通过闪电贷借出 7,500,000 BUSD
   ![](https://cdn.nlark.com/yuque/0/2023/jpeg/97322/1694318405469-6dc7c3d9-1838-4664-9d0e-01372c5e7d4c.jpeg#averageHue=%23f9efdd&clientId=u95e9e491-8b91-4&from=paste&id=u5946e36e&originHeight=37&originWidth=690&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=u7c2e15b7-1c3d-402a-9899-a913c5a07c5&title=)
2. 将 2,003,300 BUSD 兑换为 2,000,296 USDT，此时合约中 BUSD 余额为 11,405,966，USDT 余额为 8,296,249
   ![](https://cdn.nlark.com/yuque/0/2023/jpeg/97322/1694318405467-21ddde6d-9cb0-4cf1-b1ec-7e007b0a010d.jpeg#averageHue=%23f7f3f3&clientId=u95e9e491-8b91-4&from=paste&id=ueaf1e157&originHeight=193&originWidth=690&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=ub262b89e-3c88-45f8-84ad-6ea542bd8e8&title=)
3. 调用合约中 deposit 函数，向合约中存入 5,000,000 BUSD
   ![](https://cdn.nlark.com/yuque/0/2023/jpeg/97322/1694318405508-b6b092e0-1701-4a68-ae4c-0a8acabe756f.jpeg#averageHue=%23f4f3f2&clientId=u95e9e491-8b91-4&from=paste&id=u48736fdc&originHeight=43&originWidth=690&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=u420ba443-44b1-48f3-9da3-3169b96ef7c&title=)
4. 此时攻击者地址剩余 496,700 BUSD，攻击者将剩余 BUSD 全部兑换为 USDT，共 495,488 个
   ![](https://cdn.nlark.com/yuque/0/2023/jpeg/97322/1694318405454-32dadbd5-adfb-4ccd-a255-6ed7e5931d21.jpeg#averageHue=%23f6f4f4&clientId=u95e9e491-8b91-4&from=paste&id=uc1ef9fd8&originHeight=207&originWidth=690&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=udff08d9b-7cce-4e21-a43b-b6f04e7034f&title=)
5. 将之前兑换得到的 2,000,296 USDT 存入合约
   ![](https://cdn.nlark.com/yuque/0/2023/jpeg/97322/1694318405873-8b44fe02-cf37-490a-a68b-ee63e54a43a0.jpeg#averageHue=%23f4f3f2&clientId=u95e9e491-8b91-4&from=paste&id=u5f7cbacb&originHeight=55&originWidth=690&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=u0736ef4b-8dd6-44b2-8865-9743b596dc6&title=)
6. 调用 Allbridge Core: Bridge 合约中 swap 函数，使用 495,784 USDT 兑换 490,849 BUSD
   ![](https://cdn.nlark.com/yuque/0/2023/jpeg/97322/1694318405917-e2b10be4-3352-4c2c-aa5a-a085b1aa6eb9.jpeg#averageHue=%23ecebea&clientId=u95e9e491-8b91-4&from=paste&id=u79f5e0a1&originHeight=44&originWidth=690&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=u204e57af-6963-4a8e-806f-3641a6b2044&title=)
7. 取出之前存入的 4,830,999 BUSD
   ![](https://cdn.nlark.com/yuque/0/2023/jpeg/97322/1694318405851-fe5b0160-6262-4ccd-a449-d78bb9f50c43.jpeg#averageHue=%23f4f3f2&clientId=u95e9e491-8b91-4&from=paste&id=u1d980ec1&originHeight=97&originWidth=690&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=u779bc3c2-dafd-4aef-b78b-ebf6e520373&title=)
8. 调用 Allbridge Core: Bridge 合约中 swap 函数，使用 40,000 BUSD 兑换出 789,632 USDT
   ![](https://cdn.nlark.com/yuque/0/2023/jpeg/97322/1694318405919-98bd1611-f5b3-4e5a-aa07-39f899e5b873.jpeg#averageHue=%23f8efe2&clientId=u95e9e491-8b91-4&from=paste&id=ube1e3f0c&originHeight=46&originWidth=690&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=u4d4444f4-a055-4b13-b374-b30c6a2f591&title=)
9. 将存入的资金提出，并将 USDT 兑换为 BUSD
   ![](https://cdn.nlark.com/yuque/0/2023/jpeg/97322/1694318406263-83f58588-73f2-44e4-b58b-f7125dd88370.jpeg#averageHue=%23f9e7da&clientId=u95e9e491-8b91-4&from=paste&id=uced83470&originHeight=110&originWidth=690&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=u2e784287-0efa-4107-a1ec-9bf74274fdc&title=)
10. 归还闪电贷
    ![](https://cdn.nlark.com/yuque/0/2023/jpeg/97322/1694318406226-df46ce78-c6d4-461e-b750-763e5a39c8eb.jpeg#averageHue=%23f9e9dc&clientId=u95e9e491-8b91-4&from=paste&id=u4d05b5f5&originHeight=48&originWidth=690&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=u4c746ae7-5e02-4db7-bc79-52087b94fa8&title=)
    攻击者此次攻击中共获利 549,874 BUSD

## 总结及建议

此次攻击是由于攻击者可以通过大额存取资金和进行兑换，从而修改交易池中代币的比例，实现用较少的 BUSD 兑换出大额 USDT 从而获利。

### 安全建议

建议对合约中进行代币兑换的函数添加最大兑换比判断，避免当池子中代币数量差值较大时执行兑换产生较大损失。
建议项目方上线前进行多次审计，避免出现审计步骤缺失。
