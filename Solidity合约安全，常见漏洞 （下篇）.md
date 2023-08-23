# Solidity 合约安全，常见漏洞 （下篇）

## 不安全的随机数

目前不可能用区块链上的单一交易安全地产生随机数。区块链需要是完全确定的，否则分布式节点将无法达成关于状态的共识。因为它们是完全确定的，所以任何 "随机"的数字都可以被预测到。下面的掷骰子函数可以被利用。

```solidity
contract UnsafeDice {
    function randomness() internal returns (uint256) {
        return keccak256(abi.encode(msg.sender, tx.origin, block.timestamp, tx.gasprice, blockhash(block.number - 1);
    }

    // our dice can land on one of {0,1,2,3,4,5}function rollDice() public payable {
        require(msg.value == 1 ether);

        if (randomness() % 6) == 5) {
            msg.sender.call{value: 2 ether}("");
        }
    }
}

contract ExploitDice {
    function randomness() internal returns (uint256) {
        return keccak256(abi.encode(msg.sender, tx.origin, block.timestamp, tx.gasprice, blockhash(block.number - 1);
    }

    function betSafely(IUnsafeDice game) public payable {
        if (randomness % 6) == 5)) {
            game.betSafely{value: 1 ether}()
        }

        // else don't do anything
    }
}

```

如何来产生随机数并不重要，因为攻击者可以完全复制它。使用更多的 "熵"的来源，如 msg.sender、时间戳等，不会有任何影响，因为智能合约也可以预测它。

## 错误使用 Chainlink 随机数 Oracle

Chainlink 是一个流行的解决方案，以获得安全的随机数。它分两步进行。首先，智能合约向预言机处发送一个随机数请求，然后在一些区块之后，预言机以一个随机数作为回应。
由于攻击者无法预测未来，所以他们无法预测随机数。
除非智能合约错误地使用预言机：

- 请求随机数的智能合约必须在随机数返回之前不做任何事情。否则，攻击者可以监视返回随机数的预言机的 mempool，并在前面运行预言机，知道随机数会是什么。
- 随机数预言机本身可能会试图操纵你的应用程序。如果没有其他节点的共识，他们不能挑选随机数，但如果你的应用程序同时请求几个随机数，他们可以扣留和重新排序。
- 最终性在[以太坊](https://www.rareskills.io/post/solidity-precompiles)或大多数其他 EVM 链上不是即时的。仅仅因为某些区块是最新的，并不意味着它不一定会保持这种状态。这被称为 "链上重组"。事实上，链可以改变的不仅仅是最后一个区块。这就是所谓的 "深度重组"。Etherscan 报告了各种链的 re-orgs，例如以太坊重组和 Polygon 重组。在 Polygon 上，重组的深度可以达到 30 个或更多的区块，所以等待更少的区块会使应用变得脆弱（当 zk-evm 成为 Polygon 上的标准共识时，这种情况可能会改变，因为最终性将与以太坊的一致，但这是未来的预测，而不是目前的事实）。
- 下面是其他 Chainlink 随机数的安全考虑。

## 从价格 Oracle 中获取陈旧的数据

Chainlink 没有 SLA（服务水平协议）来保持它的价格预言机在一定时间范围内的更新。当链上的交易严重拥堵时，价格更新可能会被延迟。
使用价格预言机的智能合约必须明确地检查数据是否陈旧，即最近在某个阈值内被更新。否则，它不能对价格做出可靠的决策。
还有一个更复杂的问题，如果价格没有变化超过[偏差阈值](https://blog.chain.link/chainlink-price-feeds-secure-defi/)，预言机可能不会更新价格以节省 Gas，所以这可能会影响到什么时间阈值被认为是 "陈旧"。
了解智能合约所依赖的 Oracle 的服务水平协议是很重要的。

## 只依赖一个预言机

无论一个预言机看起来多么安全，将来都可能发现攻击。对此的唯一防御措施是使用多个独立的预言机。

## 一般来说，预言机是很难正确的

区块链可以是相当安全的，但首先把数据放到链上就必须进行某种链外操作，这就放弃了区块链提供的所有安全保证。即使预言机者保持诚实，他们的数据来源也可以被操纵。例如，一个信使可以可靠地报告来自中心化交易所的价格，但这些价格可以被大量的买入和卖出订单所操纵。同样，依赖于传感器数据或一些 web2 API 的预言机也会受到传统黑客攻击的影响。
一个好的智能合约架构在可能的情况下会完全避免使用预言机。

## 混合计算

考虑以下合约

```solidity
contract MixedAccounting {
  uint256 myBalance;

  function deposit() public payable {
    myBalance = myBalance + msg.value;
  }

  function myBalanceIntrospect() public view returns (uint256) {
    return address(this).balance;
  }

  function myBalanceVariable() public view returns (uint256) {
    return myBalance;
  }

  function notAlwaysTrue() public view returns (bool) {
    return myBalanceIntrospect() == myBalanceVariable();
  }
}
```

上面的合约没有接收或回退函数，所以直接将以太传送给它就会回退。然而，合约可以用自毁的方式强行向它发送以太。在此案例中，myBalanceIntrospect()会比 myBalanceVariable() 大。两种以太币的计算方法都没有问题，但如果你同时使用这两种方法，那么合约可能会有不一致的行为。
这同样适用于 ERC20 代币。

```solidity
contract MixedAccountingERC20 {

  IERC20 token;
  uint256 myTokenBalance;

  function deposit(uint256 amount) public {
    token.transferFrom(msg.sender, address(this), amount);
    myTokenBalance = myTokenBalance + amount;
  }

  function myBalanceIntrospect() public view returns (uint256) {
    return token.balanceOf(address(this));
  }

  function myBalanceVariable() public view returns (uint256) {
    return myTokenBalance;
  }

  function notAlwaysTrue() public view returns (bool) {
    return myBalanceIntrospect() == myBalanceVariable();
  }
}
```

我们再次不能假设 myBalanceIntrospect()和 myBalanceVariable()总是返回相同的值。可以直接将 ERC20 代币转账到 MixedAccountingERC20，绕过存款函数，不更新 myTokenBalance 变量。
在用反省检查余额时，应避免严格使用相等检查，因为余额可以被外人随意改变。

## 把加密证明当作密码一样对待

这不是 Solidity 的一个怪癖，更多的是开发者对如何使用密码学来赋予地址特殊权限有普遍误解。下面的代码是不安全的：

```solidity
contract InsecureMerkleRoot {
  bytes32 merkleRoot;
  function airdrop(bytes[] calldata proof, bytes32 leaf) external {

    require(MerkleProof.verifyCalldata(proof, merkleRoot, leaf), "not verified");
    require(!alreadyClaimed[leaf], "already claimed airdrop");
    alreadyClaimed[leaf] = true;

    mint(msg.sender, AIRDROP_AMOUNT);
  }
}
```

这段代码是不安全的，原因有三：

1. 任何知道被选中进行空投的地址的人都可以重新创建 Merkle 树并创造一个有效的证明。
2. 叶子没有 Hash。攻击者可以提交一个与 Merkle 根相同的叶子，并绕过 require 语句。
3. 即使上述两个问题被修复，一旦有人提交了有效的证明，他们就可以被抢跑。

加密证明（Merkle 树、签名等）需要与 msg.sender 绑定，攻击者在没有获得私钥的情况下无法操纵。

## Solidity 不会向上转型 uint 大小

```solidity
function limitedMultiply(uint8 a, uint8 b) public pure returns (uint256 product) {
  product = a * b;
}
```

尽管 product 是一个[uint256](https://www.rareskills.io/post/uint-max-value-solidity)变量，但乘法结果不会大于 255，否则代码将被回退。
这个问题可以通过向上转型每个变量来解决：

```solidity
function unlimitedMultiply(uint8 a, uint8 b) public pure returns (uint256 product) {
  product = uint256(a) * uint256(b);
}
```

在结构中的整数相乘，也会出现这样的情况。当乘以在结构中的小数值时，你应该注意到这一点：

```solidity
struct Packed {
  uint8 time;
  uint16 rewardRate
}

//...

Packed p;
p.time * p.rewardRate; // this might revert!
```

## Solidity 截断不会回退

Solidity 并不检查将一个整数转换为一个较小的整数是否安全。除非某些业务逻辑能确保向下转型是安全的，否则应该使用 [SafeCast](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/utils/math/SafeCast.sol) 这样的库。

```solidity
function test(int256 value) public pure returns (int8) {
  return int8(value + 1); // overflows and does not revert
}
```

## 对存储指针的写入不会保存新数据

这段代码看起来像是把 myArray[1]中的数据复制到了 myArray[0]中，但其实不是。如果你把函数的最后一行注释掉，编译器会说这个函数应该变成一个视图函数。对 foo 的写入并没有写到底层存储。

```solidity
contract DoesNotWrite {
  struct Foo {
    uint256 bar;
  }
  Foo[] public myArray;

  function moveToSlot0() external {
    Foo storage foo = myArray[0];
    foo = myArray[1]; // myArray[0] 不会改变
    // we do this to make the function a state
    // changing operation
    // and silence the compiler warning
    myArray[1] = Foo({bar: 100});
  }
}
```

所以不要写到存储指针。

## 删除包含动态数据类型的结构体并不会删除动态数据

如果一个映射（或动态数组）在一个结构体内，并且该结构被删除，那么映射或数组将不会被删除。
除了删除数组之外，删除关键字只能删除一个存储槽。如果该存储槽包含对其他存储槽的引用，这些存储槽不会被删除。

```solidity
contract NestedDelete {

  mapping(uint256 => Foo) buzz;

  struct Foo {
    mapping(uint256 => uint256) bar;
  }

  Foo foo;

  function addToFoo(uint256 i) external {
    buzz[i].bar[5] = 6;
  }

  function getFromFoo(uint256 i) external view returns (uint256) {
    return buzz[i].bar[5];
  }

  function deleteFoo(uint256 i) external {
    // internal map still holds the data in the
    // mapping and array
    delete buzz[i];
  }
}
```

现在让我们做以下交易序列

1. addToFoo(1)
2. getFromFoo(1) 返回 6
3. deleteFoo(1)
4. getFromFoo(1) 仍然返回 6!

记住，在 Solidity 中，map 永远不会是 "空"的。因此，如果有人访问一个已经被删除的项目，交易将不会回退，而是返回该数据类型的零值。
