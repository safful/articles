# 智能合约漏洞，BEVO 代币损失 4.5 万美元攻击事件分析

# 一、事件背景

北京时间 2023 年 1 月 31 日，在 twitter 上看到这样一条消息：
![](https://cdn.nlark.com/yuque/0/2023/png/97322/1696391222879-87f85c15-f87a-4100-b2c4-97caefa6239e.png#averageHue=%23d0decf&clientId=u3b8f95e0-bd71-4&from=paste&id=ufad067a0&originHeight=978&originWidth=1220&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=uedfd48f4-bb46-4d07-acba-2c085102915&title=)
BEVO 代币被攻击，总共损失 45000 美元，导致 BEVO 代币的价格下跌了 99%。
有趣的是，这个事件中还出现了抢跑。
twitter：[https://twitter.com/peckshield/status/1619996999054667784](https://twitter.com/peckshield/status/1619996999054667784)
成功交易链接：[https://bscscan.com/tx/0xb97502d3976322714c828a890857e776f25c79f187a32e2d548dda1c315d2a7d](https://bscscan.com/tx/0xb97502d3976322714c828a890857e776f25c79f187a32e2d548dda1c315d2a7d)
被抢跑交易：[https://bscscan.com/tx/0x581c7674a6adfaa4351422781f6b674e2b7ac0fab0db9d46bfcb559ddd96cff8](https://bscscan.com/tx/0x581c7674a6adfaa4351422781f6b674e2b7ac0fab0db9d46bfcb559ddd96cff8)
分析工具：
[https://phalcon.blocksec.com/tx/bsc/0xb97502d3976322714c828a890857e776f25c79f187a32e2d548dda1c315d2a7d](https://phalcon.blocksec.com/tx/bsc/0xb97502d3976322714c828a890857e776f25c79f187a32e2d548dda1c315d2a7d)
[https://dashboard.tenderly.co/tx/bsc/0xb97502d3976322714c828a890857e776f25c79f187a32e2d548dda1c315d2a7d](https://dashboard.tenderly.co/tx/bsc/0xb97502d3976322714c828a890857e776f25c79f187a32e2d548dda1c315d2a7d)

# 二、事件分析

## 原因

被攻击的原因在于 deliver 方法，当调用该合约的 deliver 方法时，会减少代币的总价值，导致计算余额出现了误差，造成储备量与余额的不平衡，攻击者通过 skim 获利。

## 攻击步骤

![](https://cdn.nlark.com/yuque/0/2023/png/97322/1696391222873-fc2e0233-2964-4e4d-bc51-28c830635575.png#averageHue=%23f2f3f0&clientId=u3b8f95e0-bd71-4&from=paste&id=u9b21a31c&originHeight=680&originWidth=1291&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=uc5b23119-2ceb-4f32-9e9b-ff2a0ba77d1&title=)

1. 攻击者通过闪电贷获得了 192.5 WBNB，并通过 Pancake 交换出来 3,028,774,323,006,137,313 BEVO 代币。
2. 调用 deliver 方法，减少总额\_rTotal 数量。

```
function deliver(uint256 tAmount) public {
        address sender = _msgSender();
        require(!_isExcluded[sender], "Excluded addresses cannot call this function");
        (uint256 rAmount,,,,,,) = _getValues(tAmount);
        _rOwned[sender] = _rOwned[sender].sub(rAmount);
        _rTotal = _rTotal.sub(rAmount);
        _tFeeTotal = _tFeeTotal.add(tAmount);
    }
```

3.调用 skim 方法，在 skim 中要查询池子中 BEVO 的余额，过程中会根据\_getRate()方法计算 BEVO 的价值比率，然后计算出当前余额。
先看\_getRate(),计算公式是 rSupply.div(tSupply)

```
function _getRate() private view returns(uint256) {
        (uint256 rSupply, uint256 tSupply) = _getCurrentSupply();
        return rSupply.div(tSupply);
    }
```

![](https://cdn.nlark.com/yuque/0/2023/png/97322/1696391222917-4a123d59-920f-4557-ad69-d431b122fee0.png#averageHue=%2322242e&clientId=u3b8f95e0-bd71-4&from=paste&id=u1eed49c3&originHeight=80&originWidth=531&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=u2f93a6a6-3ea3-4dea-a20a-d25e5f2b230&title=)
而从\_getCurrentSupply()方法中得出的 rSupply 就和\_rTotal 有关，由于 deliver 导致\_rTotal 减少，所以 rSupply.div(tSupply)的比值 currentRate 也随之减小。
![](https://cdn.nlark.com/yuque/0/2023/png/97322/1696391222986-6725601b-efe3-4530-b0bd-aaf7827ec3b8.png#averageHue=%231c1d20&clientId=u3b8f95e0-bd71-4&from=paste&id=u99abf98c&originHeight=205&originWidth=785&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=u10a61135-9aff-46a8-a1db-96566372f51&title=)
![](https://cdn.nlark.com/yuque/0/2023/png/97322/1696391223441-4320f92a-73f6-4db7-8f78-14b8be439a2f.png#averageHue=%2321222a&clientId=u3b8f95e0-bd71-4&from=paste&id=uef8fe5a4&originHeight=90&originWidth=648&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=ue39c9dec-1ac7-438b-97e2-745e686a925&title=)
![](https://cdn.nlark.com/yuque/0/2023/png/97322/1696391223579-3485f268-4668-4b64-985d-12d8dbcb9942.png#averageHue=%2321212a&clientId=u3b8f95e0-bd71-4&from=paste&id=u4c439fae&originHeight=89&originWidth=699&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=u07362c0b-c7ad-432b-9015-62a0ac434dd&title=)
最终的余额是 rAmount.div(currentRate)计算出来的，rAmount 在执行过程中没有被改变过，所以分子不变分母减小，自然导致了余额变大了。
计算出来余额：6844218532359160336,储备量：2298813336114922094,多出的 4545405196244238242 就被发给了攻击者。 4.攻击者又调用了一次 deliver，将\_rototal 再减少 4545405196244238242，使余额与储备量不一致。然后调用了 swap 方法获利了 337 个 BNB。除去闪电贷 192.5 个 BNB，攻击者最终获得 144.5 个 BNB，总计 45000 美元。

# 三、总结

这起事件后风波没有停止，后来还出现了其他类似的攻击手法，有的攻击者在 bsc 链上批量扫描符合要求的代币并实施攻击。
从这次攻击事件来看，金额计算模型设计需要谨慎，避免出现参数可控导致的问题，这是保证项目安全及其重要的部分。
