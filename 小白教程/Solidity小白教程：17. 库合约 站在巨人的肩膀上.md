# Solidity 小白教程：17. 库合约 站在巨人的肩膀上

这一讲，我们用**ERC721**的引用的库合约**String**为例介绍**solidity**中的库合约（**library**），并总结了常用的库函数。

## 库函数

库函数是一种特殊的合约，为了提升**solidity**代码的复用性和减少**gas**而存在。库合约一般都是一些好用的函数合集（**库函数**），由大神或者项目方创作，咱们站在巨人的肩膀上，会用就行了。
![](https://cdn.nlark.com/yuque/0/2023/jpeg/97322/1694577170812-d444c32d-3651-428d-8d37-3f8495d83c95.jpeg#averageHue=%231585ab&clientId=ubc3453b4-7077-4&from=paste&id=ubaed123e&originHeight=300&originWidth=388&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=ue3c2c5be-3f61-49a7-b929-76f0f66f545&title=)
他和普通合约主要有以下几点不同：

1. 不能存在状态变量
2. 不能够继承或被继承
3. 不能接收以太币
4. 不可以被销毁

## String 库合约

**String 库合约**是将**uint256**类型转换为相应的**string**类型的代码库，样例代码如下：

```solidity
library Strings {
    bytes16 private constant _HEX_SYMBOLS = "0123456789abcdef";

    /**
     * @dev Converts a `uint256` to its ASCII `string` decimal representation.
     */
    function toString(uint256 value) public pure returns (string memory) {
        // Inspired by OraclizeAPI's implementation - MIT licence
        // https://github.com/oraclize/ethereum-api/blob/b42146b063c7d6ee1358846c198246239e9360e8/oraclizeAPI_0.4.25.sol

        if (value == 0) {
            return "0";
        }
        uint256 temp = value;
        uint256 digits;
        while (temp != 0) {
            digits++;
            temp /= 10;
        }
        bytes memory buffer = new bytes(digits);
        while (value != 0) {
            digits -= 1;
            buffer[digits] = bytes1(uint8(48 + uint256(value % 10)));
            value /= 10;
        }
        return string(buffer);
    }

    /**
     * @dev Converts a `uint256` to its ASCII `string` hexadecimal representation.
     */
    function toHexString(uint256 value) public pure returns (string memory) {
        if (value == 0) {
            return "0x00";
        }
        uint256 temp = value;
        uint256 length = 0;
        while (temp != 0) {
            length++;
            temp >>= 8;
        }
        return toHexString(value, length);
    }

    /**
     * @dev Converts a `uint256` to its ASCII `string` hexadecimal representation with fixed length.
     */
    function toHexString(uint256 value, uint256 length) public pure returns (string memory) {
        bytes memory buffer = new bytes(2 * length + 2);
        buffer[0] = "0";
        buffer[1] = "x";
        for (uint256 i = 2 * length + 1; i > 1; --i) {
            buffer[i] = _HEX_SYMBOLS[value & 0xf];
            value >>= 4;
        }
        require(value == 0, "Strings: hex length insufficient");
        return string(buffer);
    }
}
```

他主要包含两个函数，**toString()**将**uint256**转为**string**，**toHexString()**将**uint256**转换为**16 进制**，再转换为**string**。

### 如何使用库合约

我们用 String 库函数的 toHexString()来演示两种使用库合约中函数的办法。
**1. 利用 using for 指令**
指令**using A for B;**可用于附加库函数（从库 A）到任何类型（B）。添加完指令后，库**A**中的函数会自动添加为**B**类型变量的成员，可以直接调用。注意：在调用的时候，这个变量会被当作第一个参数传递给函数：

```solidity
// 利用using for指令
    using Strings for uint256;
    function getString1(uint256 _number) public pure returns(string memory){
        // 库函数会自动添加为uint256型变量的成员
        return _number.toHexString();
    }
```

**2. 通过库合约名称调用库函数**

```solidity
// 直接通过库合约名调用
    function getString2(uint256 _number) public pure returns(string memory){
        return Strings.toHexString(_number);
    }
```

我们部署合约并输入**170**测试一下，两种方法均能返回正确的**16 进制 string** “0xaa”。证明我们调用库函数成功！
![](https://cdn.nlark.com/yuque/0/2023/png/97322/1694577170767-6c06f0dd-5bc2-4372-b4e4-b3bef9061761.png#averageHue=%232e3144&clientId=ubc3453b4-7077-4&from=paste&id=ue6b68f3b&originHeight=750&originWidth=580&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=u3047ae78-a706-48bf-9c14-06667251cd3&title=)

## 总结

这一讲，我们用**ERC721**的引用的库函数**String**为例介绍**solidity**中的库函数（**Library**）。99%的开发者都不需要自己去写库合约，会用大神写的就可以了。我们只需要知道什么情况该用什么库合约。常用的有：

1. [String](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/4a9cc8b4918ef3736229a5cc5a310bdc17bf759f/contracts/utils/Strings.sol)：将**uint256**转换为**String**
2. [Address](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/4a9cc8b4918ef3736229a5cc5a310bdc17bf759f/contracts/utils/Address.sol)：判断某个地址是否为合约地址
3. [Create2](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/4a9cc8b4918ef3736229a5cc5a310bdc17bf759f/contracts/utils/Create2.sol)：更安全的使用**Create2 EVM opcode**
4. [Arrays](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/4a9cc8b4918ef3736229a5cc5a310bdc17bf759f/contracts/utils/Arrays.sol)：跟数组相关的库函数
