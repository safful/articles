# Solidity 合约安全，常见漏洞 （上篇）

这个智能合约安全系列提供了一个广泛的列表，列出了在 Solidity 智能合约中容易反复出现的问题和漏洞。
Solidity 中的安全问题可以归结为智能合约的行为方式不符合它们的意图。这可以分为四大类：

- 资金被盗
- 资金被锁住或冻结在合约内
- 人们收到的奖励比预期的少（奖励被延迟或减少）。
- 人们收到的奖励比预期的多（导致通货膨胀和贬值）。

我们不可能对所有可能出错的事情做一个全面的列表。然而，正如传统的软件工程有常见的漏洞主题，如 SQL 注入、缓冲区超限和跨网站脚本，智能合约中也有反复出现的**反模式(anti-pattern)**。

## 智能合约的黑客和漏洞

把这个系列看作是一本书，不反复推敲，就不可能详细讨论每一个概念。
警告：这个安全系列有 1 万多字，所以可以把它收藏起来，分块阅读。
然而，这个系列可以作为一个清单，说明应该注意什么和研究什么。如果一个主题感觉不熟悉，这应该作为一个指标，说明值得花时间去练习识别该类漏洞。

## 重入攻击

每当一个智能合约调用另一个智能合约的函数，向其发送以太币，或向其代币转账，那么就有可能出现重入。

- 当以太币被转账时，接收合约的回退或接收函数被调用。这就把控制权交给了接收方。
- 一些代币协议通过调用一个预先确定的函数来提醒接收智能合约收到了代币。这就把控制流交给了接收函数。
- 当攻击合约收到控制权时，它不一定要调用交出控制权的同一个函数。它可以调用受害者智能合约中的不同函数（跨函数重入），甚至是不同的合约（跨合约重入）。
- 只读重入发生在合约处于中间状态时访问一个视图函数。

尽管重入可能是最知名的智能合约漏洞，但它只占发生的黑客的一小部分。安全研究员 Pascal Caversaccio(pcaveraccio)在 github 上保持着一个最新的[重入攻击列表](https://github.com/pcaversaccio/reentrancy-attacks)。截至 2023 年 4 月，该库中已经记录了 46 个重入攻击。

## 访问控制

这似乎是一个简单的错误，但忘记对谁可以调用一个敏感函数（如提取以太币或改变所有权）进行限制，这种情况经常发生，令人惊讶。
即使[修改器](https://learnblockchain.cn/docs/solidity/contracts.html#modifier)已经写了，也会修改器没有正确实现的情况，比如下面的例子中缺少 require 语句：

```solidity
// DO NOT USE!
modifier onlyMinter {
  minters[msg.sender] == true_;
}
```

上面这个代码是这次审计中的一个真实例子：[https://code4rena.com/reports/2023-01-rabbithole/#h-01-bad-implementation-in-minter-access-control-for-rabbitholereceipt-and-rabbitholetickets-contracts](https://code4rena.com/reports/2023-01-rabbithole/#h-01-bad-implementation-in-minter-access-control-for-rabbitholereceipt-and-rabbitholetickets-contracts)
这里是访问控制出错的另一种方式：

```solidity
function claimAirdrop(bytes32 calldata proof[]) {

  bool verified = MerkleProof.verifyCalldata(proof, merkleRoot, keccak256(abi.encode(msg.sender)));
  require(verified, "not verified");
  require(alreadyClaimed[msg.sender], "already claimed");

  _transfer(msg.sender, AIRDROP_AMOUNT);
}
```

在此案例中，"alreadyClaimed "永远不会被设置为真，所以申领者可以发出多次调用该函数。

### 现实案例：交易机器人被利用

最近的一个访问控制不足的例子是，一个交易机器人（它的名字叫 0xbad，因为地址是以其开头）接收闪电贷的函数没有受到保护。它积累了超过一百万美元的利润，直到有一天，攻击者注意到任何地址都可以调用 flashloan 接收函数，而不仅仅是 flashloan 提供者。
正如交易机器人通常的情况一样，执行交易的智能合约代码没有经过验证，但攻击者还是发现了这个弱点。更多信息见[rekt news](https://rekt.news/ripmevbot/)报道。

## 不正确的输入验证

如果访问控制是关于控制谁调用一个函数，那么输入验证就是控制他们用什么来调用合约。
这通常归结为忘记在适当的地方设置 require 语句。
下面是一个基本的例子：

```solidity
contract UnsafeBank {
  mapping(address => uint256) public balances;

  // allow depositing on other's behalf
  function deposit(address for) public payable {
    balances += msg.value;
  }

  function withdraw(address from, uint256 amount) public {
    require(balances[from] <= amount, "insufficient balance");

    balances[from] -= amount;
    msg.sender.call{value: amout}("");
  }
}
```

上面的合约确实检查了你提取的金额没有超过你账户中的金额，但它并没有阻止你从一个任意的账户中提取。

### 现实案例：Sushiswap

[Sushiswap](https://www.sushi.com/)由于一个外部函数的一个参数没有被检查，经历了一次这种类型的黑客攻击。
![](https://cdn.nlark.com/yuque/0/2023/png/97322/1692749045212-1781f653-c608-435f-9de6-78e1f1b75dc5.png#averageHue=%23626361&clientId=u8329a058-22b3-4&from=paste&id=u5d21cefc&originHeight=968&originWidth=1458&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=u16525a2b-a353-45ee-87d2-5ab7e6931e4&title=)
[https://twitter.com/peckshield/status/1644907207530774530](https://twitter.com/peckshield/status/1644907207530774530)

### 不适当的访问控制和不适当的输入验证之间有什么区别？

不适当的访问控制意味着 msg.sender 没有足够的限制。不当的输入验证意味着函数的参数没有得到充分的处理。这种反模式还有一个反面：在函数调用中放置过多的限制。

## 过多的函数限制

过多的验证可能意味着资金不会被盗，但它可能意味着资金被锁定在合约中。拥有多重的保障措施也不是一件好事。

### 现实生活中的例子：Akutars NFT

最引人注目的事件之一是 Akutars NFT，它最终导致价值 3400 万美元的 Eth 卡在智能合约内，无法提取。
该合约有一个用心良苦的机制，以防止合约的所有者退出，直到所有付款超过荷兰拍卖价格的退款被给予。但由于下面链接的 Twitter 线程中记录的一个错误，所有者无法提取资金。
![](https://cdn.nlark.com/yuque/0/2023/png/97322/1692749045181-604764f2-7067-4cb1-9357-8b544eeb305b.png#averageHue=%23fdfdfd&clientId=u8329a058-22b3-4&from=paste&id=u10f5fdfb&originHeight=1002&originWidth=1462&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=ua401e346-8576-400a-8632-c70bda019ee&title=)
[https://twitter.com/0xInuarashi/status/1517674505975394304](https://twitter.com/0xInuarashi/status/1517674505975394304)

### 掌握好平衡

Sushiswap 给了不被信任的用户太多的权力，而 Akutars NFT 给管理员的权力太少。在设计智能合约时，必须对每一类用户的自由度做出主观判断，这个决定不能留给自动化测试和工具。必须考虑与去中心化、安全和用户体验的重大权衡。
对于智能合约程序员来说，明确写出用户应该和不应该对某些函数做什么是开发过程中的一个重要部分。
我们将在后面重新审视权力过大的管理员这个话题。

## 安全问题可以归结为管理资金退出合约的方式

正如介绍中所说，智能合约被黑的主要方式有四种：

- 金钱被盗
- 钱被冻结
- 奖励不足
- 奖励过高

这里的 "钱 "是指任何有价值的东西，如代币，而不仅仅是加密货币。在对智能合约进行编码或审计时，开发人员必须有意识地了解价值流入和流出合约的预期方式。上面列出的问题是智能合约被黑的主要途径，但还有很多其他的根本原因，可以串联成重大问题，下面记录了这些问题。

## 双重投票或 msg.sender 欺骗

使用 vanilla ERC20 代币或 NFT 作为门票来计算投票权重是不安全的，因为攻击者可以用一个地址投票，将代币转账到另一个地址，并从该地址再次投票。
下面是一个最小案例：

```solidity
// A malicious voter can simply transfer their tokens to
// another address and vote again.
contract UnsafeBallot {

  uint256 public proposal1VoteCount;
  uint256 public proposal2VoteCount;

  IERC20 immutable private governanceToken;

  constructor(IERC20 _governanceToken) {
    governanceToken = _governanceToken;
  }

  function voteFor1() external notAlreadyVoted {
    proposal1VoteCount += governanceToken.balanceOf(msg.sender);
  }

  function voteFor2() external notAlreadyVoted {
    proposal2VoteCount += governanceToken.balanceOf(msg.sender);
  }

  // prevent the same address from voting twice,
  // however the attacker can simply
  // transfer to a new address
  modifier notAlreadyVoted {
    require(!alreadyVoted[msg.sender], "already voted");
    _;
    alreadyVoted[msg.sender] = true;
  }
}
```

为了防止这种攻击，应该使用[ERC20 Snapshot](https://www.rareskills.io/post/erc20-snapshot)或[ERC20 Votes](https://www.rareskills.io/post/erc20-votes-erc5805-and-erc6372)。通过对过去的一个时间点进行快照，当前的代币余额不能被操纵以获得非法投票权。

## 闪电贷治理攻击

然而，使用具有快照或投票函数的 ERC20 代币并不能完全解决这个问题，如果有人可以通过闪电贷来暂时增加他们的余额，然后在同一交易中对他们的余额进行快照。如果该快照被用于投票，他们将有一个不合理的大量投票权可供支配。
闪存贷款将大量的以太币或代币借给一个地址，但如果这笔钱没有在同一次交易中偿还，就会回退。

```solidity
contract SimpleFlashloan {

  function borrowERC20Tokens() public {
    uint256 before = token.balanceOf(address(this));

    // send tokens to the borrower
    token.transfer(msg.sender, amount);

    // hand control back to the borrower to
    // let them do something
    IBorrower(msg.sender).onFlashLoan();

    // require that the tokens got returned
    require(token.balanceOf(address(this) >= before);
                }
                }
```

攻击者可以利用 flashloan 突然获得大量的选票，使提案对他们有利和/或做一些恶意的事情。

## 闪电贷价格攻击

这可以说是对 DeFi 最常见的（或至少是最引人注目的）攻击，占了数亿美元的损失。这里有一个[列表](https://www.immunebytes.com/blog/top-10-flash-loan-attacks/)。
区块链上的资产价格通常被计算为资产之间的当前汇率。例如，如果一个合约目前是 1 美元兑 100 个 k9 币，那么你可以说 k9 币的价格是 0.01 美元。然而，价格通常会随着买卖压力的变化而变化，而闪电贷会产生巨大的买卖压力。
当查询另一个智能合约的资产价格时，开发者需要非常小心，因为他们假设他们所调用的智能合约对闪电贷的操纵是免疫的。

## 绕过合约检查

你可以通过查看一个地址的字节码大小来 "检查 "它是否是一个智能合约。外部拥有的账户（普通钱包）没有任何字节码。这里有几种方法可以做到这一点

```solidity
import "@openzeppelin/contracts/utils/Address.sol"
contract CheckIfContract {

  using Address for address;

  function addressIsContractV1(address _a) {
    return _a.code.length == 0;
  }

  function addressIsContractV2(address _a) {

    // use the openzeppelin libraryreturn _a.isContract();
  }
}
```

然而，这有一些限制

- 如果一个合约从构造函数中进行外部调用，那么它的表面字节码大小将为零，因为智能合约部署代码还没有返回运行时代码
- 该代码现在可能是空的，但攻击者可能知道他们可以在未来使用 create2 在那里部署一个智能合约。

一般来说，检查一个地址是否是合约通常是（但不总是）一种反模式。多签名钱包本身就是智能合约，做任何可能破坏多签名钱包的事情都会破坏可组合性。
这方面的例外是在调用转移钩子之前检查目标是否是智能合约。稍后会有更多关于这方面的内容。

## tx.origin

使用 tx.origin 很少有好的理由。如果 tx.origin 被用来识别交易发起人，那么中间人攻击是可能的。如果用户被骗去调用一个恶意的智能合约，那么该智能合约就可以利用 tx.origin 所拥有的所有权限来进行破坏。
请考虑下面这个练习，以及代码上方的注释。

```solidity
contract Phish {
  function phishingFunction() public {

    // this fails, because this contract does not have approval/allowance
    token.transferFrom(msg.sender, address(this), token.balanceOf(msg.sender));

    // this also fails, because this creates approval for the contract,// not the wallet calling this phishing function
    token.approve(address(this), type(uint256).max);
  }
}
```

这并不意味着你可以安全地调用任意的智能合约。但大多数协议中都有一层安全机制，如果使用 tx.origin 进行认证，就会被绕过。
有时，你可能会看到像这样的代码：

```solidity
require(msg.sender == tx.origin, "no contracts");
```

当一个智能合约调用另一个智能合约时，msg.sender 将是智能合约，tx.origin 将是用户的钱包，从而给出一个可靠的指示，即传入的调用是来自智能合约。即使调用发生在构造函数中也是如此。
大多数情况下，这种设计模式不是一个好主意。多签名钱包和来自 EIP 4337 的钱包将无法与具有这种代码的函数交互。这种模式通常可以在 NFT mint 中看到，在那里，我们有理由相信大多数用户都在使用传统的钱包。但随着账户抽象化的普及，这种模式的阻碍作用将大于其帮助。

## Gas 导致拒绝服务

悲痛攻击（griefing attack）意味着黑客试图为其他人 "制造悲痛"，即使他们没有从这样做中获得经济利益。
一个智能合约可以通过进入一个无限循环，恶意地用完转发给它的所有 Gas。考虑一下下面的例子：

```solidity
contract Mal {

  fallback() external payable {

    // infinite loop uses up all the gas
    while (true) {

    }
  }
}
```

如果另一个合约将以太坊分配到一个地址列表中，如以下：

```solidity
contract Distribute {
  funtion distribute(uint256 total) public nonReentrant {
    for (uint i; i < addresses.length; ) {

      (bool ok, ) addresses.call{value: total / addresses.length}("");
      // ignore ok, if it reverts we move on
      // traditional gas saving trick for for loops
      unchecked {
        ++i;
      }
    }
  }
}
```

那么该函数将在向 Mal 合约发送以太币时进行还原。上面代码中的调用转发了 63 / 64 的可用 Gas，所以很可能没有足够的 Gas 来完成只剩下 1/64 的 Gas 的操作。
一个智能合约可以返回一个消耗大量 Gas 的大型内存数组
考虑下面的例子

```solidity
function largeReturn() public {

  // result might be extremely long!
  (book ok, bytes memory result) =
  otherContract.call(abi.encodeWithSignature("foo()"));

  require(ok, "call failed");
}
```

内存数组在 724 字节之后使用了四次方的 Gas，所以仔细选择的返回数据大小可以让调用者感到悲伤。
即使变量的结果没有被使用，它仍然被复制到内存中。如果你想把返回的大小限制在一定范围内，你可以使用汇编

```solidity
function largeReturn() public {
  assembly {
    let ok := call(gas(), destinationAddress, value, dataOffset, dataSize, 0x00, 0x00);
    // nothing is copied to memory until you
    // use returndatacopy()
  }
}
```

### 删除别人可以添加的数组也是一个拒绝服务的向量

尽管擦除存储是一个节省 Gas 的操作，但它仍然有一个成本。如果一个数组变得太长，它就不可能被删除。下面是一个最小案例

```solidity
contract VulnerableArray {

  address[] public stuff;

  function addSomething(address something) public {
    stuff.push(something);
  }

  // 如果 stuff 太长,由于 gas 问题将无法删除
  function deleteEverything() public onlyOwner {
    delete stuff;
  }
}
```

### ERC777, ERC721, 和 ERC1155 也可以是悲痛的向量

如果一个智能合约转账到有转账 hook 的代币，攻击者可以设置一个不接受代币的合约（它要么没有 onReceive 函数，要么将该函数编程为回退）。这将使代币无法转账，并导致整个交易被回退。
在使用 safeTransfer 或 transfer 之前，要考虑到接收方可能会强迫交易回退的可能性。

```solidity
contract Mal is IERC721Receiver, IERC1155Receiver, IERC777Receiver {

  // this will intercept any transfer hook
  fallback() external payable {

    // infinite loop uses up all the gaswhile (true) {

  }
}

// we could also selectively deny transactions
function onERC721Received(address operator,
                              address from,
                              uint256 tokenId,
                              bytes calldata data
                             ) external returns (bytes4) {

  if (wakeUpChooseViolence()) {
    revert();
  }
  else {
    return IERC721Receiver.onERC721Received.selector;
  }
}
}
```
