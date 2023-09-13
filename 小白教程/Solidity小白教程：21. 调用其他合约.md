# Solidity 小白教程：21. 调用其他合约

## 调用已部署合约

开发者写智能合约来调用其他合约，这让以太坊网络上的程序可以复用，从而建立繁荣的生态。很多**web3**项目依赖于调用其他合约，比如收益农场（**yield farming**）。这一讲，我们介绍如何在已知合约代码（或接口）和地址情况下调用目标合约的函数。

## 目标合约

我们先写一个简单的合约**OtherContract**来调用。

```solidity
contract OtherContract {
    uint256 private _x = 0; // 状态变量_x
    // 收到eth的事件，记录amount和gas
    event Log(uint amount, uint gas);

    // 返回合约ETH余额
    function getBalance() view public returns(uint) {
        return address(this).balance;
    }

    // 可以调整状态变量_x的函数，并且可以往合约转ETH (payable)
    function setX(uint256 x) external payable{
        _x = x;
        // 如果转入ETH，则释放Log事件
        if(msg.value > 0){
            emit Log(msg.value, gasleft());
        }
    }

    // 读取_x
    function getX() external view returns(uint x){
        x = _x;
    }
}
```

这个合约包含一个状态变量**\_x**，一个事件**Log**在收到**ETH**时触发，三个函数：

- **getBalance()**: 返回合约**ETH**余额。
- **setX()**: **external payable**函数，可以设置**\_x**的值，并向合约发送**ETH**。
- **getX()**: 读取**\_x**的值。

## 调用**OtherContract**合约

我们可以利用合约的地址和合约代码（或接口）来创建合约的引用：**\_Name(\_Address)**，其中**\_Name**是合约名，**\_Address**是合约地址。然后用合约的引用来调用它的函数：**\_Name(\_Address).f()**，其中**f()**是要调用的函数。
下面我们介绍 4 个调用合约的例子，在 remix 中编译合约后，分别部署**OtherContract**和**CallContract**：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1694577303160-445194c6-af1f-4474-94d3-e6b30576aba9.png#averageHue=%2326293d&clientId=u795f53b4-3177-4&from=paste&id=udb01cc83&originHeight=951&originWidth=1920&originalType=url&ratio=2&rotation=0&showTitle=false&size=189922&status=done&style=none&taskId=u15ecc64d-3051-486a-be60-f9cf6cfb6ce&title=)
![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1694577303119-34d8ff89-ac34-4594-a247-d9762f0dfda3.png#averageHue=%2325273a&clientId=u795f53b4-3177-4&from=paste&id=u441ad91c&originHeight=951&originWidth=1920&originalType=url&ratio=2&rotation=0&showTitle=false&size=189580&status=done&style=none&taskId=ubbb40d28-df3c-4bb0-a572-ce414bc13ee&title=)
![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1694577303131-2cbd9aa5-20e0-4518-8caa-75f0656ea3c4.png#averageHue=%2325273a&clientId=u795f53b4-3177-4&from=paste&id=u09323fee&originHeight=951&originWidth=1920&originalType=url&ratio=2&rotation=0&showTitle=false&size=189047&status=done&style=none&taskId=u9274ddd7-b356-46ef-bb47-83555ac9709&title=)

### 1. 传入合约地址

我们可以在函数里传入目标合约地址，生成目标合约的引用，然后调用目标函数。以调用**OtherContract**合约的**setX**函数为例，我们在新合约中写一个**callSetX**函数，传入已部署好的**OtherContract**合约地址**\_Address**和**setX**的参数**x**：

```solidity
function callSetX(address _Address, uint256 x) external{
        OtherContract(_Address).setX(x);
    }
```

复制**OtherContract**合约的地址，填入**callSetX**函数的参数中，成功调用后，调用**OtherContract**合约中的**getX**验证**x**变为 123
![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1694577303092-e4a34e65-9c58-49ea-8a6b-e4a09c7f064a.png#averageHue=%2326283b&clientId=u795f53b4-3177-4&from=paste&id=uf994da9d&originHeight=951&originWidth=1920&originalType=url&ratio=2&rotation=0&showTitle=false&size=194476&status=done&style=none&taskId=u9e934d8b-cdc8-4174-803e-8c35a1e833d&title=)
![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1694577303148-62e7368f-8766-461f-92b9-e2ff8b72f1e7.png#averageHue=%2326283c&clientId=u795f53b4-3177-4&from=paste&id=ucde74e34&originHeight=951&originWidth=1920&originalType=url&ratio=2&rotation=0&showTitle=false&size=183465&status=done&style=none&taskId=ue483e516-2335-44d6-b55e-f4f8f6a65ee&title=)

### 2. 传入合约变量

我们可以直接在函数里传入合约的引用，只需要把上面参数的**address**类型改为目标合约名，比如**OtherContract**。下面例子实现了调用目标合约的**getX()**函数。
**注意**该函数参数**OtherContract \_Address**底层类型仍然是**address**，生成的**ABI**中、调用**callGetX**时传入的参数都是**address**类型

```solidity
function callGetX(OtherContract _Address) external view returns(uint x){
        x = _Address.getX();
    }
```

复制**OtherContract**合约的地址，填入**callGetX**函数的参数中，调用后成功获取**x**的值
![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1694577303565-1782fb00-4d59-46db-98fb-53207cfeadf2.png#averageHue=%2326283c&clientId=u795f53b4-3177-4&from=paste&id=u8edf8d23&originHeight=951&originWidth=1920&originalType=url&ratio=2&rotation=0&showTitle=false&size=190259&status=done&style=none&taskId=u3c4ceef4-0387-4b49-90e6-b9a2ab6c840&title=)

### 3. 创建合约变量

我们可以创建合约变量，然后通过它来调用目标函数。下面例子，我们给变量**oc**存储了**OtherContract**合约的引用：

```solidity
function callGetX2(address _Address) external view returns(uint x){
        OtherContract oc = OtherContract(_Address);
        x = oc.getX();
    }
```

复制**OtherContract**合约的地址，填入**callGetX2**函数的参数中，调用后成功获取**x**的值
![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1694577303617-3958a801-8adb-40fe-955a-2b9039afe902.png#averageHue=%2326283c&clientId=u795f53b4-3177-4&from=paste&id=u9eb1ec89&originHeight=951&originWidth=1920&originalType=url&ratio=2&rotation=0&showTitle=false&size=190304&status=done&style=none&taskId=ud87ad5ab-8189-41f4-b12e-1d312e9c8ef&title=)

### 4. 调用合约并发送**ETH**

如果目标合约的函数是**payable**的，那么我们可以通过调用它来给合约转账：**\_Name(\_Address).f{value: \_Value}()**，其中**\_Name**是合约名，**\_Address**是合约地址，**f**是目标函数名，**\_Value**是要转的**ETH**数额（以**wei**为单位）。
**OtherContract**合约的**setX**函数是**payable**的，在下面这个例子中我们通过调用**setX**来往目标合约转账。

```solidity
function setXTransferETH(address otherContract, uint256 x) payable external{
        OtherContract(otherContract).setX{value: msg.value}(x);
    }
```

复制**OtherContract**合约的地址，填入**setXTransferETH**函数的参数中，并转入 10ETH
![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1694577303653-324b98ca-06eb-469f-beb1-f43ff0338ce2.png#averageHue=%2325273b&clientId=u795f53b4-3177-4&from=paste&id=u273f011b&originHeight=951&originWidth=1920&originalType=url&ratio=2&rotation=0&showTitle=false&size=193283&status=done&style=none&taskId=uc97441d6-dbe5-4f63-8cfd-2195feba53b&title=)
转账后，我们可以通过**Log**事件和**getBalance()**函数观察目标合约**ETH**余额的变化。
![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1694577303661-53242f83-1f17-4385-bc56-2a421c88cf8b.png#averageHue=%2325273a&clientId=u795f53b4-3177-4&from=paste&id=u0ced83c3&originHeight=951&originWidth=1920&originalType=url&ratio=2&rotation=0&showTitle=false&size=157771&status=done&style=none&taskId=ucbce3bcc-bcc2-42a8-a79a-7f1714bf68c&title=)

## 总结

这一讲，我们介绍了如何通过目标合约代码（或接口）和地址来创建合约的引用，从而调用目标合约的函数。
