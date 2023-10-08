# 智能合约漏洞，Euler Finance 价值 1.95 亿美元漏洞事件分析

# 事件背景

零时科技区块链安全情报平台监控到消息，北京时间 2023 年 3 月 14 日，ETH 链上 Euler Finance 受到黑客攻击，攻击者获利约 1.97 亿美元，攻击者地址为 0xb66cd966670d962c227b3eaba30a872dbfb995db，被盗资金 100ETH 转移至混币平台 Tornado.Cash,其余资金仍在攻击者地址暂未移动。零时科技安全团队及时对此安全事件进行分析。

# 漏洞及核心

合约中有关代币余额修改的函数中存在检查清算的功能，在执行代币转移后检查抵押资产与借出资金，要求抵押资产大于借出资金。
![](https://cdn.nlark.com/yuque/0/2023/jpeg/97322/1696736497627-89bdc797-ee24-4904-aacf-0a138278b547.jpeg#averageHue=%23222120&clientId=u5c214e72-aeb7-4&from=paste&id=u5461bee2&originHeight=336&originWidth=690&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=u650497bd-f7e8-420a-9876-0b8e7cc37c7&title=)
![](https://cdn.nlark.com/yuque/0/2023/jpeg/97322/1696736497377-e8983a10-5a1b-48b2-85cc-4d0f80304ecf.jpeg#averageHue=%23282120&clientId=u5c214e72-aeb7-4&from=paste&id=u1655df3a&originHeight=124&originWidth=690&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=ufe66e540-3301-40e5-97c7-6caa1d3e68d&title=)
由于函数 donateToReserves 中缺少检查清算逻辑，攻击者可以通过此函数将借款调整为清算状态。
![](https://cdn.nlark.com/yuque/0/2023/jpeg/97322/1696736497746-b2ff0290-9956-489a-ab6d-6e3298238646.jpeg#averageHue=%2321201f&clientId=u5c214e72-aeb7-4&from=paste&id=u23ca8676&originHeight=347&originWidth=690&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=u21b5e8bb-7c09-4799-bf67-a4f4ffe09c8&title=)
攻击者构造出两个攻击合约，其中一个合约执行借款操作，另一个合约执行清算操作。
借款合约通过闪电贷借出 30,000,000 DAI，之后将 20,000,000 DAI 存入 Euler 中获得 19,568,124 eDAI，之后调用 mint 函数借出 200,000,000 dDAI 和 195,681,243eDAI，将资产放大 10 倍。
![](https://cdn.nlark.com/yuque/0/2023/jpeg/97322/1696736497384-9ff0f20f-3ec6-4c4d-a473-2267281b2318.jpeg#averageHue=%23f1f0e5&clientId=u5c214e72-aeb7-4&from=paste&id=ubf3c0423&originHeight=95&originWidth=690&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=u8f2857d9-b4fe-4f1c-810b-b0658bc9512&title=)
之后攻击者调用 repay 函数继续进行质押，将剩余 10,000,000 DAI 进行质押并销毁 10,000,000 eDAI，之后继续调用 mint 函数借出 200,000,000 dDAI 和 195,681,243eDAI，此时攻击者共有 400,000,000 dDAI 和 400,930,610 eDAI。
攻击者调用合约中 donateToReserves 函数进行转账，将 100,000,000 eDAI 转移至 0 地址。
![](https://cdn.nlark.com/yuque/0/2023/jpeg/97322/1696736497670-8bcb9635-2152-45c0-ba9c-499dd8238829.jpeg#averageHue=%23f5f1f1&clientId=u5c214e72-aeb7-4&from=paste&id=u38c42ac2&originHeight=109&originWidth=690&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=u457696d5-2b0f-415e-844e-caf3efae90b&title=)
此时攻击者地址共有 400,000,000 dDAI 和 300,930,610 eDAI，已经达到清算条件，由于此函数中缺少清算判断，未能执行清算。
清算合约调用清算函数执行清算操作，共获得 310,930,612 eDAI 和 254,234,370dDAI
![](https://cdn.nlark.com/yuque/0/2023/jpeg/97322/1696736497744-ae62f9b0-d91b-464d-ba23-243e98e1423c.jpeg#averageHue=%23f6f5f5&clientId=u5c214e72-aeb7-4&from=paste&id=ub5574451&originHeight=34&originWidth=690&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=u1ffa0b9e-e5e2-40c4-9087-fe2135512ed&title=)
![](https://cdn.nlark.com/yuque/0/2023/jpeg/97322/1696736497841-83f3e818-f9bd-403e-b285-6322b24c10d0.jpeg#averageHue=%23f7f6f6&clientId=u5c214e72-aeb7-4&from=paste&id=uf0697fdf&originHeight=28&originWidth=690&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=u47e82442-622b-4475-8d62-7b82f0c1ab7&title=)
之后攻击者调用 withdraw 函数将池子中 DAI 全部取出。
![](https://cdn.nlark.com/yuque/0/2023/jpeg/97322/1696736497988-80ed5f50-aa5a-4ff9-acd6-76dd38c88a74.jpeg#averageHue=%23f3e4e4&clientId=u5c214e72-aeb7-4&from=paste&id=u888092c1&originHeight=79&originWidth=690&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=ufb71878a-cfe5-436e-b027-5545e0f86b8&title=)
此笔交易中攻击者共获利 8,877,507 DAI

# 总结及建议

此次攻击是由于 EToken 合约中 donateToReserves 函数缺失清算检查逻辑，攻击者能够恶意将借贷的资金处于清算状态下而不触发清算，使得攻击者能够无需向合约转移清算资金而触发清算从而获利。

## 安全建议

建议对合约中相关函数添加清算检查操作
建议项目方上线前进行多次审计，避免出现审计步骤缺失
