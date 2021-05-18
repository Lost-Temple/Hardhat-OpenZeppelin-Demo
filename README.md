# OpenZeppelin 智能合约库源码

```shell
git clone https://github.com/OpenZeppelin/openzeppelin-contracts.git
```

## 开发智能合约 

### 安装hardhat

```shell
cd work/solidity/learn
npm init -y
npm install --save-dev hardhat
```

### 运行hardhat生成脚手架

```
npx hardhat
```

### 安装openzeppelin的智能合约库

```shell
npm install @openzeppelin/contracts
```

## 编写智能合约

```shell
cd constract
vim Box.sol
```

```
// contracts/Box.sol
// SPDX-License-Identifier: MIT
pragma solidity ^0.6.0;

contract Box {
    uint256 private value;

    // Emitted when the stored value changes
    event ValueChanged(uint256 newValue);

    // Stores a new value in the contract
    function store(uint256 newValue) public {
        value = newValue;
        emit ValueChanged(newValue);
    }

    // Reads the last stored value
    function retrieve() public view returns (uint256) {
        return value;
    }
}
```

### 编译

```shell
cd ..
npx hardhat compile
```

注：这样直接编译可能会出错，是由于合约代码中我们指定了版本是^0.6.0， 所以我们

```shelll
vim hardhat.config.js 
```

修改内部的版本相关的代码

```javascript
module.exports = {
  solidity: "0.6.12",
};
```

再重新编译一次应该就能成功

### 添加一个智能合约

```shell
mkdir -p ./constract/access-control
cd constract/access-control
vim Auth.sol
```

````
// contracts/access-control/Auth.sol
// SPDX-License-Identifier: MIT
pragma solidity ^0.6.0;

contract Auth {
    address private administrator;

    constructor() public {
        // Make the deployer of the contract the administrator
        administrator = msg.sender;
    }

    function isAdministrator(address user) public view returns (bool) {
        return user == administrator;
    }
}
````

修改Box.sol，使Box.sol 引用 Auth.sol

```
// contracts/Box.sol
// SPDX-License-Identifier: MIT
pragma solidity ^0.6.0;

// Import Auth from the access-control subdirectory
import "./access-control/Auth.sol";

contract Box {
    uint256 private value;
    Auth private auth;

    event ValueChanged(uint256 newValue);

    constructor(Auth _auth) public {
        auth = _auth;
    }

    function store(uint256 newValue) public {
        // Require that the caller is registered as an administrator in Auth
        require(auth.isAdministrator(msg.sender), "Unauthorized");

        value = newValue;
        emit ValueChanged(newValue);
    }

    function retrieve() public view returns (uint256) {
        return value;
    }
}
```

## 使用OpenZeppelin库中的智能合约
```
// contracts/Box.sol
// SPDX-License-Identifier: MIT
pragma solidity ^0.6.0;

// Import Ownable from the OpenZeppelin Contracts library
import "@openzeppelin/contracts/access/Ownable.sol";

// Make Box inherit from the Ownable contract
contract Box is Ownable {
    uint256 private value;

    event ValueChanged(uint256 newValue);

    // The onlyOwner modifier restricts who can call the store function
    function store(uint256 newValue) public onlyOwner {
        value = newValue;
        emit ValueChanged(newValue);
    }

    function retrieve() public view returns (uint256) {
        return value;
    }
}
```

## 布署合约
### 启动Hardhat网络（相当于一个本地的测试网)

```shell
npx hardhat node
```

布署合约时，我们需要用在script中用到ethers，所以我们需要安装它
```shell
npm install --save-dev @nomiclabs/hardhat-ethers ethers
```

我们需要添加一段配置到 hardhat.config.js中去
```javascript
// hardhat.config.js
require('@nomiclabs/hardhat-ethers');

module.exports = {
...
};
```

布署合约
```shell
npx hardhat run --network localhost scripts/deploy.js
Deploying Box...
Box deployed to: 0x5FbDB2315678afecb367f032d93F642f64180aa3
```

## 通过控制台和合约进行交互
```shell
npx hardhat console --network localhost
```
```shell
> const Box = await ethers.getContractFactory("Box")
undefined
> const box = await Box.attach("0x5FbDB2315678afecb367f032d93F642f64180aa3")
undefined
```

发送交易,因为Box的合约的第一个方法，store，是会修改区块链状态的（有写操作），所以我们需要发送一个交易去执行这个方法
```shell
> await box.store(42)
{
  hash: '0x3d86c5c2c8a9f31bedb5859efa22d2d39a5ea049255628727207bc2856cce0d3',
  type: 0,
  accessList: null,
  blockHash: '0x1c7a773ce5e8aceee7072dcb8d956c17f2c37c1eb0be991b31ffe9e2e0dfb828',
  blockNumber: 2,
  transactionIndex: 0,
  confirmations: 1,
  from: '0xf39Fd6e51aad88F6F4ce6aB8827279cffFb92266',
  gasPrice: BigNumber { _hex: '0x01dcd65000', _isBigNumber: true },
  gasLimit: BigNumber { _hex: '0xb833', _isBigNumber: true },
  to: '0x5FbDB2315678afecb367f032d93F642f64180aa3',
  value: BigNumber { _hex: '0x00', _isBigNumber: true },
  nonce: 1,
  data: '0x6057361d000000000000000000000000000000000000000000000000000000000000002a',
  r: '0x932d4f5a0aa6eee9a24918178b691d5160df289fddd2d10ceea01a7a67d5b565',
  s: '0x29681633432ca58fd2f9d8aaceeff9ebc7300b3127e4e12acec6c4b2479cc0d2',
  v: 62710,
  creates: null,
  chainId: 31337,
  wait: [Function (anonymous)]
}
```
查询(只读操作)
```shell
> await box.retrieve()

BigNumber { _hex: '0x2a', _isBigNumber: true }

```
Box合约返回的 uint256 数据类型时对于js来说数据太大，所以使用big number 对象。我们可以使用 (await box.retrieve()).toString来显示big number
```shell
> (await box.retrieve()).toString()
'42'
```

## 通过程序和智能合约进行交互
我们新建一个 scripts/index.js 文件
```
// scripts/index.js
async function main() {
  // Our code will go here
}

main()
  .then(() => process.exit(0))
  .catch(error => {
    console.error(error);
    process.exit(1);
  });
```

我们可以测试获取帐户列表
```javascript
// Retrieve accounts from the local node
const accounts = await ethers.provider.listAccounts();
console.log(accounts);
```
