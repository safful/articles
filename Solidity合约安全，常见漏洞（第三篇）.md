## ERC20 代币问题
如果你只处理受信任的ERC20代币，这些问题大多不适用。然而，当与任意的或部分不受信任的ERC20代币交互时，就有一些需要注意的地方。
### ERC20：转账扣费
当与不信任的代币打交道时，你不应该认为你的余额一定会增加那么多。一个ERC20代币有可能这样实现它的转账函数，如下所示：
```solidity
contract ERC20 {

  // internally called by transfer() and transferFrom()
  // balance and approval checks happen in the caller
  function _transfer(address from, address to, uint256 amount) internal returns (bool) {
    fee = amount * 100 / 99;

    balanceOf[from] -= to;
    balanceOf[to] += (amount - fee);

    balanceOf[TREASURY] += fee;

    emit Transfer(msg.sender, to, (amount - fee));
    return true;
  }
}
```
这种代币对每笔交易都会征收1%的税。因此，如果一个智能合约与该代币进行如下交互，我们将得到意想不到的回退或资产被盗。
```solidity
contract Stake {

  mapping(address => uint256) public balancesInContract;

  function stake(uint256 amount) public {
    token.transferFrom(msg.sender, address(this), amount);
    balancesInContract[msg.sender] += amount; //  这是错误的
  }

  function unstake() public {
    uint256 toSend = balancesInContract[msg.sender];
    delete balancesInContract[msg.sender];

    // this could revert because toSend is 1% greater than
    // the amount in the contract. Otherwise, 1% will be "stolen"// from other depositors.
    token.transfer(msg.sender, toSend);
  }
}
```
### ERC20: rebase 的代币
Rebasing 代币由 [Olympus DAO](https://www.olympusdao.finance/) 的sOhm代币 和 [Ampleforth](https://www.ampleforth.org/) 的AMPL代币所推广。Coingecko维护了一个 Rebasing ERC20代币的[列表](https://www.coingecko.com/en/categories/rebase-tokens)。
当一个代币回溯时，总发行量会发生变化，每个人的余额会根据回溯的方向而增加或减少。
在处理 rebase 代币时，以下代码可能会被破坏：
```solidity
contract WillBreak {
  mapping(address => uint256) public balanceHeld;
  IERC20 private rebasingToken

  function deposit(uint256 amount) external {
    balanceHeld[msg.sender] = amount;
    rebasingToken.transferFrom(msg.sender, address(this), amount);
  }

  function withdraw() external {
    amount = balanceHeld[msg.sender];
    delete balanceHeld[msg.sender];

    // 错误, amount 也许会超出转出范围
    rebasingToken.transfer(msg.sender, amount);
  }
}
```
许多合约的解决方案是简单地不允许rebase代币。然而，我们可以修改上面的代码，在将账户余额转给接受者之前检查 balanceOf(address(this))。那么，即使余额发生变化，它仍然可以工作。
### ERC20: ERC777 在 ERC20 上的包裹
ERC20，如果按照标准实现，ERC20 代币没有转账钩子（hook），因此 transfer 和 transferFrom 不会有重入问题。
带有转账钩子的代币有应用优势，这就是为什么所有的NFT标准都实现了它们，以及为什么ERC777被最终确定。然而，这已经引起了足够的混乱，以至于Openzeppelin [废止](https://github.com/OpenZeppelin/openzeppelin-contracts/pull/4066)了ERC777库。
如果你只想让你的协议与那些行为像 ERC20 代币但有转账hook的代币兼容，那么这只是一个简单的问题，把transfer和transferFrom函数当作它们会向接收者进行一个函数调用即可。
这种 ERC777 的重入发生在Uniswap身上（如果你好奇，Openzeppelin在[这里](https://blog.openzeppelin.com/exploiting-uniswap-from-reentrancy-to-actual-profit/)记录了这个漏洞）。
### ERC20: 不是所有的ERC20代币转账都会返回 true
ERC20规范规定，[ERC20代币](https://www.rareskills.io/post/erc20-snapshot)在转账成功时必须返回true。因为大多数ERC20的实现不可能失败，除非授权不足或转账的金额太多，大多数开发者已经习惯于忽略ERC20代币的返回值，并假设一个失败的trasfer将被回退。
坦率地说，如果你只与一个你知道其行为的受信任的ERC20代币打交道，这并不重要。但在处理任意的ERC20代币时，必须考虑到这种行为上的差异。
在许多合约中都有一个隐含的期望，即失败的转账应该总是回退，而不是返回错误，因为大多数ERC20代币没有返回错误的机制，所以这导致了很多混乱。
使这个问题更加复杂的是，一些ERC20代币并不遵循返回true的协议，特别是Tether。一些代币在转账失败后会回退，这将导致回退的结果冒泡到调用者。因此，一些库包裹了 ERC20 代币的转账调用，以回退恢复并返回一个布尔值。下面是一些实现方法：
参考：[Openzeppelin SafeTransfer](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC20/utils/SafeERC20.sol) 及 [Solady SafeTransfer](https://github.com/Vectorized/solady/blob/main/src/utils/SafeTransferLib.sol) （大大地提高了Gas效率）
### ERC20: 地址投毒
这不是一个智能合约的漏洞，但为了完整起见，我们在这里提到它。
转账零代币是 ERC20规范所允许的。这可能会导致前端应用程序的混乱，并可能欺骗用户，让他们错误的以为他们最近将代币发送给了某地址。[Metamask](https://metamask.io/)在这个[线程](https://twitter.com/MetaMaskSupport/status/1613255316870729728)中有更多关于这个问题的内容。
### ERC20: 查看代码，规避跑路
(在web3术语中，"rugged"意味着''跑路", 直译是"从你脚下拉出地毯" 。)
没有什么能阻止有人在ERC20代币上添加函数，让他们随意创建、转账和销毁代币--或自毁或升级。所以从根本上说，ERC20代币的 "无需信任" 程度是有限制的。
## 借贷协议中的逻辑错误
当考虑到基于DeFi协议的借贷如何被破坏时，考虑在软件层面传播的bug并影响商业逻辑层面是很有帮助的。形成和完成一个债券合约有很多步骤。这里有一些需要考虑的攻击向量。
### 贷款人损失的方式

- 使到期本金减少（可能为零）而不进行任何支付的漏洞。
- 当贷款没有偿还或抵押物降到阈值以下时，买方的抵押物不能被清算。
- 如果协议有一个转移债务所有权的机制，这可能是一个从贷款人那里偷取债券的方式。
- 贷款本金或付款的到期日被不适当地移到以后的日期。
### 借款人损失的方式

- 偿还本金时没有减少本金债务的 bug。
- 一个bug或 gas 攻击使用户无法进行支付。
- 本金或利率被非法提高。
- 预言机的操纵导致抵押物贬值。
- 贷款本金或付款的到期日被不适当地移到一个较早的日期。

如果抵押品从协议中被抽走，那么贷款人和借款人都会损失，因为借款人没有动力去偿还贷款，而借款人则会损失本金。
正如上面所看到的，DeFi协议被 "黑 "的范围比从协议中抽走一堆钱（通常成为新闻的那类事件）要多得多。
## 抵押（staking）协议中的漏洞
成为新闻的那种黑客是抵押协议被黑掉数百万美元，但这并不是唯一要面对的问题，抵押协议可能面临的问题有：

- 奖励能否延迟支付，或过早地被索取？
- 奖励能否被不适当地减少或增加？在更糟糕的情况下，能否阻止用户获得任何奖励？
- 人们能否索取不属于他们的本金或奖励，在最坏的情况下，会耗尽协议所有资金？
- 存放的资产会不会被卡在协议中（部分或全部），或被不适当地延迟提取？
- 相反，如果质押需要时间承诺，用户是否可以在承诺时间之前提取？
- 如果支付的是不同的资产或货币，其价值是否可以在相关的智能合约范围内被操纵？如果协议mint自己的代币来奖励流动性提供者或质押者，这一点是相关的。
- 如果存在预期和披露出的本金损失的风险因素，这种风险是否可以被不适当地操纵？
- 协议的关键参数是否有管理、中心化或治理风险？

需要关注的关键是代码中涉及 "资金退出 "部分的代码。
还有一个 "资金入口 "的漏洞也要寻找。

- 有权参与协议中的资产抵押的用户能否被不适当地阻止？

用户收到的奖励有一个隐含的风险回报和一个预期的资金时间价值。明确这些假设是什么，以及协议会怎样偏离预期是很有帮助的。
## 未检查的返回值
有两种方法来调用外部智能合约：1）用接口定义调用函数；2）使用.call方法。如下图所示：
```solidity
contract A {
  uint256 public x;

  function setx(uint256 _x) external {
    require(_x > 10, "x must be bigger than 10");
    x = _x;
  }
}

interface IA {
  function setx(uint256 _x) external;
}

contract B {
  function setXV1(IA a, uint256 _x) external {
    a.setx(_x);
  }

  function setXV2(address a, uint256 _x) external {
    (bool success, ) =
    a.call(abi.encodeWithSignature("setx(uint256)", _x));
    // success is not checked!
  }
}
```
在合约 B 中，如果 _x 小于 10，setXV2 会默默地失败。当一个函数通过.call方法被调用时，被调用者可以回退，但父函数不会回退。必须检查返回成功的值，并且代码行为必须相应地分支。
## msg.value 在一个循环中
在循环中使用msg.value是很危险的，因为这可能会让发起者 重复使用 msg.value。
这种情况可能会出现在payable的multicalls中。Multicalls使用户能够提交一个交易列表，以避免重复支付21,000的Gas交易费。然而，msg.value在通过函数循环执行时被 "重复使用"，有可能使用户双花。
这就是[Opyn Hack](https://peckshield.medium.com/opyn-hacks-root-cause-analysis-c65f3fe249db)的根本原因。
## 私有变量
私有变量在区块链上仍然是可见的，所以敏感信息不应该被存储在那里。如果它们不能被访问，验证者如何能够处理取决于其值的交易？私有变量不能从外部的Solidity 合约中读取，但它们可以使用以太坊客户端在链外读取。
要读取一个变量，你需要知道它的存储槽。在下面的例子中，myPrivateVar的存储槽是0。
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;
contract PrivateVarExample {
  uint256 private myPrivateVar;

  constructor(uint256 _initialValue) {
    myPrivateVar = _initialValue;
  }
}
```
下面是读取已部署的智能合约的私有变量的javascript代码
```solidity
const Web3 = require("web3");
const PRIVATE_VAR_EXAMPLE_ADDRESS = "0x123..."; // Replace with your contract address

async function readPrivateVar() {
  const web3 = new Web3("http://localhost:8545"); // Replace with your provider's URL

  // Read storage slot 0 (where 'myPrivateVar' is stored)
  const storageSlot = 0;
  const privateVarValue = await web3.eth.getStorageAt(
    PRIVATE_VAR_EXAMPLE_ADDRESS,
    storageSlot
  );

  console.log("Value of private variable 'myPrivateVar':",
              web3.utils.hexToNumberString(privateVarValue));
}

readPrivateVar();
```
## 不安全的代理调用
委托调用（Delegatecall）不应该被用于不受信任的合约，因为它把所有的控制权都交给了委托接受者。在这个例子中，不受信任的合约偷走了合约中所有的以太币。
```solidity
contract UntrustedDelegateCall {
  constructor() payable {
    require(msg.value == 1 ether);
  }

  function doDelegateCall(address _delegate, bytes calldata data) public {
    (bool ok, ) = _delegate.delegatecall(data);
    require(ok, "delegatecall failed");
  }

}

contract StealEther {
  function steal() public {
    // you could also selfdestruct here 
    // if you really wanted to be mean
    (bool ok,) = 
    tx.origin.call{value: address(this).balance}("");
    require(ok);
  }

  function attack(address victim) public {
    UntrustedDelegateCall(victim).doDelegateCall(
      address(this),
      abi.encodeWithSignature("steal()"));
  }
}
```
## 升级与代理有关的bug
我们无法在一个章节中对这个话题进行公正的解释。大多数升级错误通常可以通过使用Openzeppelin的[hardhat插件](https://docs.openzeppelin.com/upgrades-plugins/1.x/)和阅读它所保护的问题来避免出错。
作为一个快速的总结，以下是与智能合约升级有关的问题：

- 自毁（self-destruct）和委托调用（delegatecall）不应该在执行合约中使用。
- 必须注意在升级过程中，存储变量不能相互覆盖
- 在执行合约中应避免调用外部库，因为不可能预测它们会如何影响存储访问。
- 部署者决不能忽视调用初始化函数
- 在基类合约中没有包括间隙（gap）变量，以防止在基类合约中加入新的变量时发生存储碰撞（这由hardhat插件自动处理）。
- 不可变（immutable）变量中的值在升级时不会被保留
- 非常不鼓励在构造函数中做任何事情，因为未来的升级必须执行相同的构造函数逻辑以保持兼容性。
