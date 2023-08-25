# Solidity 合约安全，常见漏洞（第四篇）

## 权力过大的管理员

仅仅因为一个合约有一个所有者或管理员，这并不意味着他们需要无限权力。考虑一个 NFT。按理说，只有所有者才能从 NFT 销售中提取收益，但如果所有者的私钥被泄露，能够暂停合约（阻止转账）就会造成严重的破坏。一般来说，管理员的权限应该尽可能的小，以减少不必要的风险。

## 使用 Ownable2Step 而不是 Ownable

这在技术上不是一个漏洞，但[OpenZeppelin ownable](https://www.rareskills.io/post/openzeppelin-ownable2step)如果所有权被转移到一个不存在的地址，会导致合约所有权的丧失。Ownable2step 要求接收者确认所有权。这可以防止意外地将所有权发送到一个错误的地址。

## 四舍五入的错误

Solidity 没有浮点，所以舍入错误是不可避免的。设计者必须意识到正确的做法是向上舍入还是向下舍入，以及舍入应该对谁有利。
除法应该总是最后进行。下面的代码在小数位数不同的稳定币之间进行了错误的转换。下面的兑换机制允许用户在兑换 dai（精度为 18）时免费获得少量的 USDC（精度为 6）。变量 daiToTake 将四舍五入为零，换取非零的 usdcAmount 时，用户拿不到任何东西。

```solidity
contract Exchange {

  uint256 private constant CONVERSION = 1e12;

  function swapDAIForUSDC(uint256 usdcAmount) external pure returns (uint256 a) {
    uint256 daiToTake = usdcAmount / CONVERSION;
    conductSwap(daiToTake, usdcAmount);
  }
}
```

## 抢跑（Frontrunning）

在 Etheruem（和类似的链）的背景下，Frontrunning 意味着观察一个待定的交易，并通过支付更高的 交易成本在它之前执行另一个交易。也就是说，攻击者已经 "跑到了 "交易的前面。如果该交易是一个有利可图的交易，那么除了支付更高的 交易成本，完全复制该交易是有意义的。
这种现象有时被称为 MEV，意思是矿工可提取的价值，但有时在其他情况下是最大可提取的价值。区块生产者有无限的权力来重新排序交易和插入自己的交易，从历史上看，在以太坊进入股权证明之前，区块生产者就是矿工，因此而得名。

### 抢跑：不受限制的提款

从智能合约中提取以太币可以被认为是一种 "有利可图的交易"。你执行了一个零成本的交易（除了 Gas），最终拥有的加密货币比你开始时更多。

```
contract UnprotectedWithdraw {

    constructor() payable {
        require(msg.value == 1 ether, "must create with 1 eth");
    }

    function unsafeWithdraw() external {
        (bool ok, ) = msg.sender.call{value: address(this).value}("");
        require(ok, "transfer failed").
    }
}
```

如果你部署了这个合约并试图退出，一个先行者机器人会注意到你在 mempool 中对 "unsafeWithdraw "的调用，并复制它来先获得以太币。

### 抢跑：ERC4626 通膨攻击，是抢跑和四舍五入错误的组合

我们已经在[ERC4626 教程](https://www.rareskills.io/post/erc4626)中深入介绍了 ERC-4626 的通膨攻击。但它的要点是，ERC4626 合约根据交易者贡献的 "资产 "的百分比来分配 "份额"代币。大致上，它的运作方式如下：

```
function getShares(...) external {
    // code
    shares_received = assets_contributed / total_assets;
    // more code
}
```

当然，没有人会贡献资产而得不到任何股份，但他们无法预测这种情况会发生，如果有人能在交易中先发制人，获得股份。
例如，当池子里有 20 个资产时，他们贡献了 200 个资产，他们期望得到 100 份额。但是，如果有人在交易中提前存入 200 个资产，那么公式将是 200/220，四舍五入为零，导致受害者失去资产，获得零份额。

### 抢跑：ERC20 授权

我用一个真实的例子来说明这一点，而不是抽象地描述它：

1. 假设 Alice 授权了 Eve 的 100 个代币。Eve 是邪恶的代表，而不是用 Bob ，所以我们将保持惯例。
2. Alice 改变了主意，发送了一个交易，将 Ev e 的授权改为 50。
3. 在将授权额度改为 50 的交易纳入区块之前，它位于 Mempool 中，Eve 可以看到它。
4. Eve 发送了一个交易，要求获得她的 100 个代币，这在将授权改为 50 之前。
5. 对 50 的授权的交易通过了。
6. Eve 又获取了 50 个代币。

现在 Eve 有 150 个代币，而不是 100 或 50。解决这个问题的办法是，在处理不受信任的授权时，在增加或减少授权之前，将授权设置为零。

### 抢跑：三明治攻击

一项资产的价格会随着买卖压力的变化而变化。如果一个大订单在 Mempool 中，交易者有动力去复制这个订单，但要有更高的 gas 成本。这样一来，他们就会购买资产，让用户的大额订单使价格上涨，然后他们马上卖出。卖出订单有时被称为 "尾随"。卖出订单可以通过放置一个较低 gas 成本的卖出订单来完成，这样的序列看起来像这样的

1. 抢跑买入
2. 大额买入（用户）
3. 卖出

对这种攻击的主要防御是提供一个 "滑点"参数。如果 "抢跑买入 "本身将价格推高到某个阈值以上，"大额买入"订单将回退，使抢跑者的交易失败。
这就是所谓的三明治攻击（sandwhich），因为大额买入被抢跑买入和尾随卖出夹在中间。这种攻击也适用于大额卖单，只是方向相反。

### 了解更多关于抢跑的信息

抢跑是一个巨大的话题。[Flashbots](https://www.flashbots.net/)已经对这个话题进行了广泛的研究，并发表了一些工具和研究文章，以帮助最大限度地减少它的负面外部因素。通过适当的区块链架构是否可以 "设计掉 "抢跑是一个争论不休的话题，还没有得到最终的解决。以下两篇文章是关于这个问题的永恒的经典之作：
[以太坊是一个黑暗的森林](https://www.paradigm.xyz/2020/08/ethereum-is-a-dark-forest)
[逃离黑暗森林](https://samczsun.com/escaping-the-dark-forest/)

## 签名相关

数字签名在智能合约的背景下有两种用途：

- 使得地址能够授权区块链上的一些交易，而不进行实际交易
- 根据预定的地址，向智能合约证明发起者有某种权力去做某事

下面是一个安全使用数字签名的例子，让用户拥有铸造 NFT 的特权：

```solidity
import "@openzeppelin/contracts/utils/cryptography/ECDSA.sol";
import "@openzeppelin/contracts/token/ERC721/ERC721.sol";

contract NFT is ERC721("name", "symbol") {
  function mint(bytes calldata signature) external {
    address recovered = keccak256(abi.encode(msg.sender)).toEthSignedMessageHash().recover(signature);
    require(recovered == authorizer, "signature does not match");
  }
}
```

一个典型的例子是 ERC20 中的 Approve 函数。为了授权一个地址从我们的账户中提取一定数量的代币，我们必须进行实际的以太坊交易，这需要花费 Gas。
有时，将数字签名传递给链外的接收者，然后接收者向智能合约提供签名，以证明他们被授权进行交易，这样做更有效率。
ERC20Permit 实现了用数字签名进行审批。该函数描述如下:

```solidity
function permit(address owner,
                address spender,
                uint256 amount,
                uint256 deadline,
                uint8 v,
                bytes32 r,
                bytes32 s
               ) public
```

而不是发送一个实际的授权交易，所有者可以为花费者 "签署"授权（连同一个截止日期）。然后，被授权的花费者可以用提供的签署信息作为参数调用 permint 函数。

### 签名的剖析

你会经常看到 v、r 和 s 这些变量。它们在 solidity 中分别用 uint8、bytes32 和 bytes32 的数据类型表示。有时，签名被表示为一个 65 字节的数组，它是所有这些值连接起来的，如 abi.encodePacked(r, s, v)；
签名的另外两个基本组成部分是消息哈希值（32 字节）和签名地址。这个序列看起来像这样

1. 私钥（privKey）用于生成一个公共地址（ethAddress）。
2. 一个智能合约预先存储地址 ethAddress
3. 一个链外用户对一个消息进行 Hash，并对 Hash 值进行签名。这就产生了一对 msgHash 和签名（r, s, v）。
4. 智能合约收到一条消息，对其进行散列以产生 msgHash，然后将其与(r, s, v)相结合，看得出什么地址。
5. 如果地址与 ethAddress 匹配，则签名有效（在某些假设下，我们很快就会看到！）。

智能合约使用步骤 4 中的[预编译合约](https://www.rareskills.io/post/solidity-precompiles) ecrecover 来完成我们所说的组合，并获得地址。
在这个过程中，有很多步骤都会出现问题。

### 签名：ecrecover 返回地址(0)，当地址无效时，不会被回退

如果一个未初始化的变量与 ecrecover 的输出相比较，这可能导致漏洞。
这段代码有漏洞：

```solidity
contract InsecureContract {

  address signer;
  // defaults to address(0)
  // who lets us give the beneficiary the airdrop without them// spending gas
  function airdrop(address who, uint256 amount, uint8 v, bytes32 r, bytes32 s) external {

    // 如果签名无效，ecrecover 会返回 address(0)
    require(signer == ecrecover(keccak256(abi.encode(who, amount)), v, r, s), "invalid signature");

    mint(msg.sender, AIRDROP_AMOUNT);
  }
}
```

### 签名重放

签名重放发生在合约没有跟踪签名是否先前被使用。在下面的代码中，我们修复了之前的问题，但它仍然不安全。

```solidity
contract InsecureContract {

  address signer;

  function airdrop(address who, uint256 amount, uint8 v, bytes32 r, bytes32 s) external {

    address recovered == ecrecover(keccak256(abi.encode(who, amount)), v, r, s);
    require(recovered != address(0), "invalid signature");
    require(recovered == signer, "recovered signature not equal signer");


    mint(msg.sender, amount);
  }
}
```

人们可以随心所欲地索取空投!
我们可以添加以下几行：

```solidity
bytes memory signature = abi.encodePacked(v, r, s);
require(!used[signature], "signature already used");
// mapping(bytes => bool);
used[signature] = true;
```

唉，这段代码还是不安全啊!

### 签名的可塑性

给定一个有效的签名，攻击者可以做一些快速的算术来推导出一个不同的签名。然后，攻击者可以 "重放"这个修改过的签名。但首先，让我们提供一些代码，证明我们可以从一个有效的签名开始，修改它，并显示新的签名仍然通过。

```solidity
contract Malleable {

  // v = 28
  // r = 0xf8479d94c011613baeffe9239e4ff65e2adbac744c34217ca7d51378e72c5204
  // s = 0x57af17590a914b759c45aaeabaf513d5ef72d7da1bdd19d9f2e1bc371ece5b86
  // m = 0x0000000000000000000000000000000000000000000000000000000000000003
  function foo(bytes calldata msg, uint8 v, bytes32 r, bytes32 s) public pure returns (address, address){
    bytes32 h = keccak256(msg);
    address a = ecrecover(h, v, r, s);


    // 下面是一个数据魔法用来反转签名并创建有效的签名
    // flip s
    bytes32 s2 = bytes32(uint256(0xFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFEBAAEDCE6AF48A03BBFD25E8CD0364141) - uint256(s));

    // invert v
    uint8 v2;
    require(v == 27 || v == 28, "invalid v");
    v2 = v == 27 ? 28 : 27;

    address b = ecrecover(h, v2, r, s2);

    assert(a == b);
    // different signatures, same address!;
    return (a, b);
  }
}
```

因此，我们的运行实例仍然是脆弱的。一旦有人提出一个有效的签名，就可以产生镜像签名并绕过使用的签名检查。

```solidity
contract InsecureContract {

  address signer;

  function airdrop(address who, uint256 amount, uint8 v, bytes32 r, bytes32 s) external {

    address recovered == ecrecover(keccak256(abi.encode(who, amount)), v, r, s);
    require(recovered != address(0), "invalid signature");
    require(recovered == signer, "recovered signature not equal signer");

    bytes memory signature = abi.encodePacked(v, r, s);
    require(!used[signature], "signature already used"); // this can be bypassed
    used[signature] = true;

    mint(msg.sender, amount);
  }
}
```

### 安全签名

在这一点上，你可能想得到一些安全的签名代码，对吗？我们向你推荐我们的[在 solidity 中创建签名](https://www.rareskills.io/post/openzeppelin-verify-signature)和在 foundry 中测试签名的教程。但这里是检查清单：

- 使用 openzeppelin 的库来防止可塑性攻击，并还原到零地址的问题
- 不要使用签名作为密码。信息需要包含攻击者不能轻易重复使用的信息（如 msg.sender）。
- 在链上对你所签署的内容进行 Hash
- 使用 nonce 来防止重放攻击。更好的是，遵循 EIP712，这样用户可以看到他们正在签署的内容，并且可以防止签名在合约和不同链之间被重复使用。

### 签名在没有适当的保障措施的情况下可以被伪造或篡改

如果没有在链上进行 Hash，上面的攻击可以进一步泛化。在上面的例子中，Hash 是在智能合约中完成的，所以上面的例子不容易被下面的攻击所攻击。
让我们来看看还原签名的代码：

```solidity
// 这个代码是不安全的
function recoverSigner(bytes32 hash, uint8 v, bytes32 r, bytes32 s) public returns (address signer) {
  require(signer == ecrecover(hash, v, r, s), "signer does not match");
  // more actions
}
```

用户同时提供哈希值和签名。如果攻击者已经从签名者那里看到了一个有效的签名，他们可以简单地重复使用另一个消息的哈希和签名。
这就是为什么在智能合约中对消息进行哈希非常重要，而不是在链外。
要看这个漏洞的操作，请看我们在 Twitter 上发布的 CTF。
原始挑战：
Part 1: [https://twitter.com/RareSkills_io/status/1650869999266037760](https://twitter.com/RareSkills_io/status/1650869999266037760)
Part 2: [https://twitter.com/RareSkills_io/status/1650897671543197701](https://twitter.com/RareSkills_io/status/1650897671543197701)
解决方案：
[https://twitter.com/RareSkills_io/status/1651527648676573185](https://twitter.com/RareSkills_io/status/1651527648676573185) [https://twitter.com/RareSkills_io/status/1651224817465540611](https://twitter.com/RareSkills_io/status/1651224817465540611)

### 签名作为标识符

签名不应该被用来识别用户。由于可塑性，它们不能被认为是唯一的。Msg.sender 有更强的唯一性保证。

## 某些 Solidity 编译器版本有错误

当审核一个代码库时，对照 Solidity 页面上的 [发布公告](https://blog.soliditylang.org/category/releases/) 检查 Solidity 版本，以查看是否可能存在一个错误。

## 假设智能合约是不可变的

智能合约可以用代理模式（或更少的情况下，metamorphic 模式）进行升级。智能合约不应该依赖任意智能合约的函数保持不变。

## Transfer()和 send()在多签名钱包中会出现问题

solidity 函数 transfer 和 send 不应该被使用。它们有意将随交易转发的 Gas 量限制在 2300，这将导致大多数操作的 Gas 量耗尽。
常用的 gnosisSafe 多签名钱包支持在[回退函数](https://github.com/safe-global/safe-contracts/blob/cb4b2b19b3e336b8defd3b8c9e0e6a2ae130598c/contracts/base/FallbackManager.sol#L61)中转发调用到另一个地址。如果有人使用 transfer 或 send 向多签名钱包发送以太币，那么回退函数可能会耗尽 Gas，转账会失败。下面是 gnosisSafe 回退函数的截图。读者可以清楚地看到有足够多的操作可以用完 2300 个 Gas。
![](https://cdn.nlark.com/yuque/0/2023/png/97322/1692839475530-06794179-b63e-42a1-a496-a41b0af16f27.png#averageHue=%23fefefe&clientId=u7ee225d6-e951-4&from=paste&id=u053a5d91&originHeight=1100&originWidth=1626&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=u09db43b4-a69b-4c42-9c06-15bde5194a6&title=)
如果你需要与使用 transfer 和 send 的合约进行交互，请参阅我们关于[以太坊 access list 交易](https://www.rareskills.io/post/eip-2930-optional-access-list-ethereum)的文章，该文章允许你减少存储和合约访问操作的 Gas 成本。

## 算术溢出检测还有意义吗？

Solidity 0.8.0 已经内置了溢出和下溢保护。因此，除非存在 unchecked 的块，或使用 Yul 中的低级代码，否则就不会有溢出的危险。因此，不应该使用 SafeMath 库，因为它们在额外的检查上浪费 Gas。

## block.timestamp 怎么样？

一些文档指出，block.timestamp 是一个漏洞，因为矿工可以操纵它。这通常用于使用时间戳作为随机数的来源，正如前面记载的那样，无论如何都不应该这样做。合并后的以太坊以精确的 12 秒（或 12 秒的倍数）间隔更新时间戳。然而，以秒为粒度测量时间是一种反模式的做法。在一分钟的范围内，如果验证者错过了他们的区块 slot，并且在区块生产中发生了 24 秒的差距，就会有相当多的错误机会。

## 边缘案例

边缘案例不容易定义，但一旦你看到了足够多的边缘案例，你就会开始对它们形成一种直觉。边缘案例可以是像有人试图索取奖励，但却没有任何抵押品的情况。这是有效的，我们应该给他们零奖励。同样，我们通常希望平均分配奖励，但如果只有一个接收者，技术上不应该有除法呢？

### 边缘案例：实例 1

这个例子取自 Akshay Srivastav 的[twitter thread](https://twitter.com/akshaysrivastv/status/1648310441058115592)，并进行了修改。
考虑这样的情况：如果一组特权地址为其提供了签名，那么某人就可以进行一个特权动作：

```solidity
contract VulnerableMultisigAuthorization {
  struct Authorization {
    bytes signature;
    address authorizer;
    bytes32 hashOfAction;
    // more fields
  }

  // more code
  function takeAction(Authorization[] calldata auths, bytes calldata action) public {
    // logic for avoiding replay attacks
    for (uint256 i; i < auths.length; ++i) {

      require(validateSignature(auths[i].signature, auths[i].authorizer), "invalid signature");
      require(authorizers[auths[i].authorizer], "address is not an authorizer");

    }

    doTheAction(action)
  }
}
```

如果任何一个签名是无效的，或者签名与有效的地址不匹配，就会发生还原。但是如果数组是空的呢？在此案例中，它将直接跳到 doTheAction 而不需要任何签名。

### 边缘案例：例子 2

```solidity
contract ProportionalRewards {

  mapping(address => uint256) originalId;
  address[] stakers;

  function stake(uint256 id) public {
    nft.transferFrom(msg.sender, address(this), id);
    stakers.append(msg.sender);
  }

  function unstake(uint256 id) public {
    require(originalId[id] == msg.sender, "not the owner");

    removeFromArray(msg.sender, stakers);

    sendRewards(msg.sender,
                    totalRewardsSinceLastclaim() / stakers.length());

    nft.transferFrom(address(this), msg.sender, id);
  }
}
```

虽然上面的代码没有显示所有的函数实现，但即使这些函数的行为和它们的名字描述的一样，仍然有一个错误。你能发现它吗？这里有一张图片，给你一些空间，在你向下滚动之前，不要看到答案。
![](https://cdn.nlark.com/yuque/0/2023/png/97322/1692839475598-b821bcf0-b3ac-43bb-9981-3f6d7b6172ac.png#averageHue=%2379887a&clientId=u7ee225d6-e951-4&from=paste&id=u268f475a&originHeight=1456&originWidth=1458&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=u796a264b-9876-4002-b3e4-d8c8a40279a&title=)
removeFromArray 和 sendRewards 函数的顺序是错误的。如果 stakers 数组中只有一个用户，就会出现除以 0 的错误，而用户将无法提取他们的 NFT。此外，奖励可能没有按照作者的意图来划分。如果原来有 4 个 stakers，一个人退出，他将得到三分之一的奖励，因为在退出时数组的长度是 3。

### 边缘案例 3：Compound 奖励计算错误

让我们用一个真实的例子来说明，根据一些估计，这个例子造成了超过 1 亿美元的损失。如果你不完全理解 Compound 协议，不要担心，我们将只关注相关部分。(另外，Compound 协议是 DeFi 历史上最重要和最有影响的协议之一

总之，Compound 的重点是奖励用户将其闲置的加密货币借给其他可能有用途的交易者。贷款人以利息和 COMP 代币的形式获得报酬（借款人可以要求获得 COMP 代币奖励，但我们现在不会关注这个问题）。
Compound Comptroller 是一个代理合约，它将调用委托给可由 Compound 治理机构设置的实现。
在 2021 年 9 月 30 日的治理[提案 62](https://compound.finance/governance/proposals/62)中，实施合约被设置为一个有漏洞的[实现合约](https://etherscan.io/address/0x374abb8ce19a73f2c4efad642bda76c797f19233/advanced#internaltx)。在上线的同一天，人们在[Twitter](https://twitter.com/napgener/status/1443350694635921409?ref_src=twsrc%5Etfw%7Ctwcamp%5Etweetembed%7Ctwterm%5E1443350694635921409%7Ctwgr%5E16d6caea3da2d69f5fad63b233d4b0848bb558c6%7Ctwcon%5Es1_&ref_url=https%3A%2F%2Fwww.coindesk.com%2Ftech%2F2021%2F09%2F30%2Fdefi-money-market-compound-overpays-15m-in-comp-rewards-in-possible-exploit%2F)上看到，一些交易在押注零代币的情况下，却能申领 COMP 奖励。
有漏洞的函数 distributeSupplierComp()
以下是原始代码：

```solidity
/**
 * @notice Calculate COMP accrued by a supplier and possibly transfer it to them
 * @param cToken The market in which the supplier is interacting
 * @param supplier The address of the supplier to distribute COMP to
 */
function distributeSupplierComp(address cToken, address supplier) internal {
  // TODO: Don't distribute supplier COMP if the user is not in the supplier market.
  // This check should be as gas efficient as possible as distributeSupplierComp is called in many places.
  // - We really don't want to call an external contract as that's quite expensive.

  CompMarketState storage supplyState = compSupplyState[cToken];
  uint supplyIndex = supplyState.index;
  uint supplierIndex = compSupplierIndex[cToken][supplier];

  // Update supplier's index to the current index since we are distributing accrued COMP
  compSupplierIndex[cToken][supplier] = supplyIndex;

  if (supplierIndex == 0 && supplyIndex > compInitialIndex) {
    // Covers the case where users supplied tokens before the market's supply state index was set.
    // Rewards the user with COMP accrued from the start of when supplier rewards were first
    // set for the market.
    supplierIndex = compInitialIndex;
  }

  // Calculate change in the cumulative sum of the COMP per cToken accrued
  Double memory deltaIndex = Double({mantissa: sub_(supplyIndex, supplierIndex)});

  uint supplierTokens = CToken(cToken).balanceOf(supplier);

  // Calculate COMP accrued: cTokenAmount * accruedPerCToken
  uint supplierDelta = mul_(supplierTokens, deltaIndex);

  uint supplierAccrued = add_(compAccrued[supplier], supplierDelta);
  compAccrued[supplier] = supplierAccrued;

  emit DistributedSupplierComp(CToken(cToken), supplier, supplierDelta, supplyIndex);
}
```

具有讽刺意味的是，这个漏洞在 TODO 的注释中。"如果用户不在供应市场，就不要分发供应者 COMP"。但代码中没有这方面的检查。而只要用户在他们的钱包中持有份额代币（CToken(cToken).balanceOf(supplier);）
在[提案 64](https://compound.finance/governance/proposals/64)在 2021 年 10 月 9 日修复了这个错误。
虽然这可以说是一个输入验证的错误，但用户并没有提交任何恶意的参数。如果有人试图申领奖励而没有 stake 任何东西，正确的计算结果应该是零。可以说，这更像是一个商业逻辑或边缘案例错误。

## 真实世界的黑客

在现实世界中发生的 DeFi 黑客，很多时候并不属于上述的确定好的类别。

### Pairity 钱包冻结(2017 年 11 月)

parity 钱包并不打算直接使用。它是一个参考实现，智能合约最小克隆实现会指向它。如果需要的话，实现允许克隆者自毁，但这需要所有的钱包所有者都签字同意。

```solidity
// throw unless the contract is not yet initialized.modifier

only_uninitialized { if (m_numOwners > 0) throw; _; }

function initWallet(address[] _owners, uint _required, uint _daylimit) only_uninitialized {
  initDaylimit(_daylimit);
  initMultiowned(_owners, _required);
}
```

钱包所有者被声明：

```solidity
// kills the contract sending everything to `_to`.
function kill(address _to) onlymanyowners(sha3(msg.data)) external {
  suicide(_to);
}
```

一些文献将此描述为 "不受保护的自毁"，即访问控制失败，但这并不十分准确。问题是 initWallet 函数没有在实现合约上被调用，这就允许有人自己调用 initWallet 函数，使自己成为所有者。这使他们有权力调用 kill 函数。根本原因是，实现没有被初始化。因此，这个错误不是由于 solidity 代码有问题而引起的，而是由于部署过程有问题。

### Badger DAO Hack (2021 年 12 月)

在这个黑客攻击中没有利用 Solidity 代码。相反，攻击者获得了 Cloudflare 的 API 密钥，并在网站前端注入了一个脚本，改变了用户的交易，将提款引向攻击者的地址。阅读更多内容，请点击此[文章](https://www.theverge.com/2021/12/2/22814849/badgerdao-defi-120-million-hack-bitcoin-ethereum)。

## 钱包的攻击载体

### 随机数不足的私钥

发现有很多前导零的地址的动机是，它们使用起来更节省 Gas。以太坊交易数据中的一个零字节被收取 4 个 Gas，一个非零字节被收取 16 个 Gas。因此、
Wintermute 被黑了，因为它使用了亵渎的地址（[writeup](https://www.halborn.com/blog/post/explained-the-wintermute-hack-september-2022)）。下面是 1inch 的[写法](https://blog.1inch.io/a-vulnerability-disclosed-in-profanity-an-ethereum-vanity-address-tool/)，说明亵渎地址发生器是如何被破坏的。
信任钱包有一个类似的漏洞，在[这篇文章](https://blog.ledger.com/Funds-of-every-wallet-created-with-the-Trust-Wallet-browser-extension-could-have-been-stolen/)中记录了漏洞情况。
请注意，这并不适用于通过改变 create2 中的盐而发现的带有前导零的智能合约，因为智能合约没有私钥。

### 重复使用 nonce 或 nonce 不够随机

椭圆曲线签名上的 "r "和 "s "点的生成方法如下：

```solidity
r = k * G (mod N)
s = k^-1 * (h + r * privateKey) (mod N)
```

G，r，s，h，和 N 都是公开的。如果 "k"变得公开，那么 "privateKey "就是唯一的未知变量，并且可以被算出。正因为如此，钱包需要完全随机地生成 k，并且永远不会重复使用它。如果随机数不是完全随机的，那么 k 就可以被推断出来。2013 年，Java 库中不安全的随机数生成让很多安卓的比特币钱包受到攻击。(比特币使用与以太坊相同的签名算法。）（[https://arstechnica.com/information-technology/2013/08/all-android-created-bitcoin-wallets-vulnerable-to-theft/）。](https://arstechnica.com/information-technology/2013/08/all-android-created-bitcoin-wallets-vulnerable-to-theft/%EF%BC%89%E3%80%82)

## 大多数漏洞都是特定的应用发生

训练自己快速识别这个列表中的反模式将使你成为一个更有效的智能合约程序员，但大多数智能合约漏洞的后果是由于预期的商业逻辑和代码的实际操作之间的不匹配。
其他可能发生 bug 的领域：

- 糟糕的 Token 激励
- 边界判断的错误
- 拼写错误
- 管理员或用户的私钥被盗

## 许多漏洞本可以通过单元测试被发现

[智能合约单元测试](https://www.rareskills.io/post/foundry-testing-solidity)可以说是智能合约最基本的保障措施，但数量惊人的智能合约要么缺乏单元测试，要么[测试覆盖率](https://www.rareskills.io/post/foundry-forge-coverage)不足。
但单元测试往往只测试合约的 "快乐路径"（预期/设计的行为）。为了测试那些令人惊讶的情况，必须采用额外的测试方法。
在智能合约被送去审计之前，应该先做以下工作：

- 用 Slither 等工具进行静态分析，确保不遗漏基本错误
- 通过单元测试实现 100%的行和分支覆盖
- 突变（Mutation）测试，确保单元测试有健全的断言语句
- 模糊测试，特别是算术测试
- 对有状态的属性进行不变量测试
- 适当时进行形式化验证

对于那些不熟悉这里的一些方法的人，Cyfrin Audits 的 Patrick Collins 在他的[视频](https://www.youtube.com/watch?v=juyY-CTolac)中对有状态和无状态的模糊测试做了幽默的介绍。
完成这些任务的工具正迅速变得更加广泛和容易使用。

## 更多资源

Web3 安全工具

- [https://github.com/safful/Web3-Security-Tools](https://github.com/safful/Web3-Security-Tools)

DeFi 攻击向量列表

- [https://github.com/safful/DeFi-Attack-Vectors](https://github.com/safful/DeFi-Attack-Vectors)

NFT 攻击向量列表

- [https://github.com/safful/NFT-Attack-Vectors](https://github.com/safful/NFT-Attack-Vectors)

Solidity 攻击向量列表

- [https://github.com/safful/Solidity-Smart-Contract-Attack-Vectors](https://github.com/safful/Solidity-Smart-Contract-Attack-Vectors)

## 结语

了解已知的反模式是很重要的。然而，现实世界中的大多数漏洞都是特定的应用。识别这两类漏洞都需要持续不断的刻意练习。
