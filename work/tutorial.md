
## Tutorial

```bash
$ npm init -y
$ npm install -D @nomiclabs/hardhat-ethers @nomiclabs/hardhat-waffle avalanche
$ npm install -D dotenv ethereum-waffle ethers hardhat solc
```

```bash
$ touch contracts/Avaxbox.sol
```

```sol
pragma solidity >=0.8.0;

contract Avaxbox {
    struct Message {
        uint id;
        uint valueInWei;
        uint timestamp;
        string text;
        address sender;
        address payable receiver;
    }

    mapping(address => uint) private numOfMessagesAtAddress;
    mapping(address => mapping(uint => Message)) private addressToMessage;

    event NewMessage(
        uint id,
        uint value,
        uint timestamp,
        address indexed sender,
        address indexed receiver
    );

    function sendMessage(
        address payable _receiver,
        string memory _text,
        uint _timestamp
    ) public payable {
        uint _messageId = numOfMessagesAtAddress[_receiver] + 1;
        Message storage _receiverMessage = addressToMessage[_receiver][_messageId];

        _receiverMessage.id = _messageId;
        _receiverMessage.timestamp = _timestamp;
        _receiverMessage.text = _text;
        _receiverMessage.sender = msg.sender;
        _receiverMessage.receiver = _receiver;

        if (msg.value > 0) {
            _receiverMessage.valueInWei = msg.value;
            // Send the AVAX to the receiver
            sendAvax(_receiver, msg.value);
        }

        numOfMessagesAtAddress[_receiver]++;

        emit NewMessage(
            _messageId,
            msg.value,
            _timestamp,
            msg.sender,
            _receiver
        );
    }

    // Function to transfer AVAX from this contract to address from input
    function sendAvax(address payable _to, uint _amount) private {
        // Note that "to" is declared as payable
        (bool success, ) = _to.call{value: _amount}("");
        require(success, "Failed to send AVAX");
    }

    function getNumOfMessages() public view returns (uint) {
        return numOfMessagesAtAddress[msg.sender];
    }

    function getOwnMessages(
        uint _startIndex,
        uint _count
    ) public view returns (Message[] memory) {
        Message[] memory _userMessages = new Message[](_count);

        for (; _startIndex < _count; _startIndex++) {
            _userMessages[_startIndex] = addressToMessage[msg.sender][_startIndex + 1];
        }

        return _userMessages;
    }
}
```

```bash
$ touch hardhat.config.js
```

```js
require('dotenv').config()
require('@nomiclabs/hardhat-waffle')
const { task } = require('hardhat/config')

/**
 * This is a hardhat task to print the list of accounts on the local
 * avalanche chain
 *
 * Prints out an array of Hex addresses
 */
task('accounts', 'Prints the list of accounts', async (args, hre) => {
  const accounts = await hre.ethers.getSigners()
  accounts.forEach((account) => {
    console.log(account.address)
  })
})

/**
 * This is a hardhat task to print the list of accounts on the local
 * avalanche chain as well as their balances
 *
 * Prints out an array of strings containing the address Hex and balance in Wei
 */
task(
  'balances',
  'Prints the list of AVAX account balances',
  async (args, hre) => {
    const accounts = await hre.ethers.getSigners()
    for (const account of accounts) {
      const balance = await hre.ethers.provider.getBalance(account.address)
      console.log(`${account.address} has balance ${balance.toString()}`)
    }
  }
)

const config = {
  solidity: {
    compilers: [{ version: '0.8.0' }], // Version of Solidity defined in the Avaxbox.sol contract
  },
  networks: {
    // Configure each network to the respective Avalanche instances
    local: {
      url: 'http://localhost:9650/ext/bc/C/rpc', // Local node we started using `yarn start:avalanche`
      gasPrice: 225000000000,
      chainId: 43112, // Every network has a chainId for identification
      accounts: [
        // List of private keys for development accounts - DO NOT TRANSFER ASSETS TO THESE ACCOUNTS ON A MAINNET
        '0x56289e99c94b6912bfc12adc093c9b51124f0dc54ac7a766b2bc5ccf558d8027',
        '0x7b4198529994b0dc604278c99d153cfd069d594753d471171a1d102a10438e07',
        '0x15614556be13730e9e8d6eacc1603143e7b96987429df8726384c2ec4502ef6e',
        '0x31b571bf6894a248831ff937bb49f7754509fe93bbd2517c9c73c4144c0e97dc',
        '0x6934bef917e01692b789da754a0eae31a8536eb465e7bff752ea291dad88c675',
        '0xe700bdbdbc279b808b1ec45f8c2370e4616d3a02c336e68d85d4668e08f53cff',
        '0xbbc2865b76ba28016bc2255c7504d000e046ae01934b04c694592a6276988630',
        '0xcdbfd34f687ced8c6968854f8a99ae47712c4f4183b78dcc4a903d1bfe8cbf60',
        '0x86f78c5416151fe3546dece84fda4b4b1e36089f2dbc48496faf3a950f16157c',
        '0x750839e9dbbd2a0910efe40f50b2f3b2f2f59f5580bb4b83bd8c1201cf9a010a',
      ],
    },
    // fuji: {
    //   url: 'https://api.avax-test.network/ext/bc/C/rpc', // Public Avalanche testnet
    //   gasPrice: 225000000000,
    //   chainId: 43113,
    //   accounts: [process.env.TEST_ACCOUNT_PRIVATE_KEY], // Use your account private key on the Avalanche testnet
    // },
    // mainnet: {
    //   url: 'https://api.avax.network/ext/bc/C/rpc', // Public Avalanche mainnet
    //   gasPrice: 225000000000,
    //   chainId: 43114,
    //   accounts: [process.env.MAIN_ACCOUNT_PRIVATE_KEY], // Use your account private key on the Avalanche mainnet
    // },
  },
}

module.exports = config
```

```bash
$ mkdir scripts
$ touch scripts/deploy.js
```


```js
async function deploy() {
    // Hardhat gets signers from the accounts configured in the config
    const [deployer] = await hre.ethers.getSigners()
  
    console.log('Deploying contract with the account:', deployer.address)
  
    // Create an instance of the contract by providing the name
    const ContractSource = await hre.ethers.getContractFactory('Avaxbox')
    // The deployed instance of the contract
    const deployedContract = await ContractSource.deploy()
  
    console.log('Contract deployed at:', deployedContract.address)
  }
  
  deploy()
    .then(() => process.exit(0))
    .catch(err => {
      console.error(err)
      process.exit(1)
    })
  
```

```bash
$ npm run accounts
$ npm run balances
```

```bash
# $ npm run deploy --network local
$ npm run deploy
```


## Ref

https://blog.logrocket.com/build-dapp-avalanche-complete-guide/