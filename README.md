## OpenZeppelin 智能合约库源码

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

```shel
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

