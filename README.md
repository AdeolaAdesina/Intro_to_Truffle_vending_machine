# Intro_to_Truffle_vending_machine

We use truffle to test and deploy production ready solidity code.

We will install truffle and ganache from https://trufflesuite.com/

Now go ahead and open up folder space for this..

Run 

```
npm i truffle -g

npm install truffle -g
```


Before doing this, if you're on Mac, you need to be a superuser. With ```sudo -s```, then enter password

Now we will create another folder to house the project with ```mkdir vending_machine```

To create a scaffolding of our project, we will leverage truffle with ```truffle init```

Now we can look at the scaffold in VScode..


Contracts
Migrations to keep track of migrations
Test
truffle-config.js

So we will create a new contract called vendingmachine.sol

or use ```touch VendingMachine.sol```

```
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.11;

contract VendingMachine {

    // state variables
    address public owner;
    mapping (address => uint) public donutBalances;

    // set the owner as th address that deployed the contract
    // set the initial vending machine balance to 100
    constructor() {
        owner = msg.sender;
        donutBalances[address(this)] = 100;
    }

    function getVendingMachineBalance() public view returns (uint) {
        return donutBalances[address(this)];
    }

    // Let the owner restock the vending machine
    function restock(uint amount) public {
        require(msg.sender == owner, "Only the owner can restock.");
        donutBalances[address(this)] += amount;
    }

    // Purchase donuts from the vending machine
    function purchase(uint amount) public payable {
        require(msg.value >= amount * 2 ether, "You must pay at least 2 ETH per donut");
        require(donutBalances[address(this)] >= amount, "Not enough donuts in stock to complete this purchase");
        donutBalances[address(this)] -= amount;
        donutBalances[msg.sender] += amount;
    }
}
```


Now create a new file under migrations, name it 2_vending_machine_migrations.js

or use ```touch 2_vending_machine_migrations.js```


 ```
 const VendingMachine = artifacts.require("VendingMachine");

module.exports = function (deployer) {
    deployer.deploy(VendingMachine);
};
```

NB: Migration just means deployment.


Now under test folder, create a new file called VendingMachine.test.js

NB: For Mac users, if you are unable to create files in VScode, add command to path with
Command+Shift+P

then type shell command and choose "install code command in PATH"



Now in VendingMachine.test.js

```
const VendingMachine = artifacts.require("VendingMachine");

contract("VendingMachine", (accounts) => {
    before(async() => {
        instance = await VendingMachine.deployed() //deploys the contract to im-memmory env
    })

    //test to check the initial balance
    it('ensures the starting balance of the vending machine is 100', async () => {
        let balance = await instance.getVendingMachineBalance()
        assert.equal(balance, 100, 'The initial balance should be 100 donuts')
    })

    it('ensures the balance of the vending machine can be updated', async () => {
        await instance.restock(100)
        let balance = await instance.getVendingMachineBalance()
        assert.equal(balance, 200, 'The balance should be 200 donuts after restocking')
    })

    it('allows donut to be purchased', async () => {
        await instance.purchase(1, {from: accounts[0], value: web3.utils.toWei('3', 'ether')})

        let balance = await instance.getVendingMachineBalance()
        assert.equal(balance, 199, 'The balance should be 199 donuts after sale')
    })
})
```

We will run our test with ```truffle test```


Now we will set up our truffle-config.js

Now let's install ganache cli version with ```npm i -g ganache-cli```

We will start ganache by running ```ganache-cli``` - gives you an entire virtual blockchain.

Copy all the available accounts and private keys


Now go into your project, open up your project truffle-config files.

Go to networks, uncomment the development network

Open up a new terminal and do ```truffle migrate```


# Truffle Console

Now we can do ```truffle console``` to enter the truffle console to do some testing

- VendingMachine.deployed().then((x) => { contract = x })

- contract.getVendingMachineBalance().then((b) => { bal = b })

- bal

- bal.toString()




# Deploy to Ethereum Testnet

First, set up your metamask

Next setup an infura account

Create a dotenv ( install with ```npm i dotenv``` and a gitignore file


Go to the first line in truffle-config and write:

```
require('dotenv').config()
const HDWalletProvider = require("@truffle/hdwallet.provider")
```

Then we'll use HDWalletProvider which comes with truffle. It's a connect between our wallet and truffle for paying gas fees to mainnet.

Now let's setup env file:

```
PRIVATE_KEY_0= 
PRIVATE_KEY_1= 
SECRET_KEY=
```

We'll export our secret keys from metamask. Then check our secret phase and paste it in secret key.

Now let's go back to truffle-config

```
const private_keys = [
  process.env.PRIVATE_KEY_0,
  process.env.PRIVATE_KEY_1,
]
```

Now let's go to network config now and uncomment the goerli network:

```
 goerli: {
       provider: () => new HDWalletProvider({
        private_keys: private_keys,
        providerOrUrl: 'https://goerli.infura.io/v3/925c2a8a2d5944b4a337c6b2ab53ba43',
        numberOfAddress: 2
      }
        ),
       network_id: 5,       // Goerli's id
       confirmations: 2,    // # of confirmations to wait between deployments. (default: 0)
       timeoutBlocks: 200,  // # of blocks before a deployment times out  (minimum/default: 50)
       skipDryRun: true     // Skip dry run before migrations? (default: false for public nets )
     },
     
```

Now we'll get some test ether in our wallet from - https://goerlifaucet.com/

Now let's do our deployment ```truffle migrate --network goerli```

Install ```npm i @truffle/hdwallet-provider```

