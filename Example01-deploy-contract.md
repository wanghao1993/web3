# infura

注册infura拿到一个key

# 领水
准备一个钱包领水，用metamask，okx都可以
[领水网站](https://dev.chain.link/faucet)

# 初始化一个项目

```bash
pnpm init
```

# 安装依赖

```bash
pnpm add dotenv web3 solc
```
# 编写合约

这里使用`solidity`. 随便写点内容

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Incrementer {
    uint256 public number;

    constructor(uint256 _initialNumber) {
        number = _initialNumber;
    }

    function increment(uint256 _value) public {
        number = number + _value;
    }

    function reset() public {
        number = 0;
    }

    function getNumber() public view returns (uint256) {
        return number;
    }
}

```

# 部署

这里部署使用`web3.js`.

主要分以下几步：

- 通过`dotenv`加载配置
- 编译`sol`内容
- 
```js
let { Web3 } = require("web3");
let solc = require("solc");
let fs = require("fs");

// Get privatekey from environment
require("dotenv").config();
let privatekey = process.env.SIGNER_PRIVATE_KEY;
if (privatekey.slice(0, 2) !== "0x") privatekey = "0x" + privatekey;

// Load contract
const source = fs.readFileSync("Incrementer.sol", "utf8");

// compile solidity
const input = {
  language: "Solidity",
  sources: {
    "Incrementer.sol": {
      content: source,
    },
  },
  settings: {
    outputSelection: {
      "*": {
        "*": ["*"],
      },
    },
  },
};

const compiledCode = JSON.parse(solc.compile(JSON.stringify(input)));
const contractFile = compiledCode.contracts["Incrementer.sol"]["Incrementer"];

// Get bin & abi
const bytecode = contractFile.evm.bytecode.object;
const abi = contractFile.abi;

// Create web3 with sepolia provider，you can change sepolia to other testnet
const web3 = new Web3(
  "https://sepolia.infura.io/v3/" + process.env.INFURA_API_KEY
);

// Create account from privatekey
const accounts = web3.eth.accounts.wallet.add(privatekey);

/*
   -- Deploy Contract --
*/
const Deploy = async () => {
  // Create contract instance
  const deployContract = new web3.eth.Contract(abi);

  // Create Tx
  const deployTx = deployContract.deploy({
    data: "0x" + bytecode,
    arguments: [0], // Pass arguments to the contract constructor on deployment(_initialNumber in Incremental.sol)
  });

  // optionally, estimate the gas that will be used for development and log it
  const gas = await deployTx.estimateGas({
    from: accounts,
  });
  console.log("estimated gas:", gas);

  try {
    // Deploy the contract to the Ganache network
    // Your deployed contrac can be viewed at: https://sepolia.etherscan.io/address/${tx.options.address}
    // You can change sepolia in above url to your selected testnet.
    const tx = await deployTx.send({
      from: accounts[0].address,
      gas,
      // gasPrice: 10000000000,
    });
    console.log("Contract deployed at address: " + tx.options.address);
  } catch (error) {
    console.error(error);
  }
};

// We recommend this pattern to be able to use async/await everywhere
// and properly handle errors.
Deploy()
  .then(() => process.exit(0))
  .catch((error) => {
    console.error(error);
    process.exit(1);
  });

```