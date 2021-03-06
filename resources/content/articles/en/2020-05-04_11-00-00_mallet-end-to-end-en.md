---
title: Mallet end to end tutorial
description: KEVM getting started
parent: 2020-05-04_11-00-00_getting-started
order: 5
last_updated: "2020-12-17T09:00:00+01:00"
redirects:
  - from: /en/kevm/get-started/deploying-smart-contracts/
    type: "301"
---


## Installation prerequisites

- Linux and MacOS: Node.js 10.16.3 (recommended), and the Git tools.
- Windows: Node.js 10.16.3 (recommended), Git tools, and the Windows
Subsystem for Linux (WSL).

Consult the official [nodejs](https://github.com/nodesource/distributions/blob/master/README.md) documentation for reference.


### Installing Node.js for Linux and MacOS

Follow these steps to install Node.js in Linux and MacOS operating systems.

**1. Open a terminal and execute:**

    curl -sL https://deb.nodesource.com/setup_15.x | sudo -E bash -
    sudo apt-get -q install -y nodejs


**2. Verify Node.js is installed with:**

    node --version


**3. Install `nvm` (a version manager for node.js):**

    curl -s -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.37.2/install.sh | bash


**4. Make the `nvm` command available in the current session:**

    export NVM_DIR="$HOME/.nvm"
    [ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"  # This loads nvm


**5. Verify that `nvm` is installed:**

    nvm --version

    0.37.2

Note: Install Mallet only *after* verifying that `nvm` is installed *and* working.


### Installing Mallet 2.0

**1. Clone this [repo](<https://github.com/input-output-hk/mallet>) to get the latest version of Mallet, and then
install it with the `npm` command.**


**2. Clone the Mallet git repository**

Open a terminal window and type:

    git clone https://github.com/input-output-hk/mallet


**3. Install specific node version for Mallet**

After cloning the repository, execute:

    cd mallet

    cat .nvmrc

    $ $ 10.16.3

Then install the required node version:

    nvm install 10.16.3
    nvm use --silent


**4. Download and install Mallet and its dependencies:**

    npm install --silent


**5. Verify that Mallet was installed correctly**

Check the integrity of the Mallet installation by using the `version` command:

    ./mallet --version

    2.1.0

If the version number displays correctly, the installation was successful.


### Installing the `solcjs` compiler

The `solcjs` compiler takes your source code (written in Solidity) and creates a binary file
that you can later deploy to the devnet using Mallet.


**1. Install `solcjs` with `npm`:**

    sudo npm install -g solc


    /usr/lib/node_modules/solc/solcjs
    + solc@0.8.0
    updated 1 package in 1.296s


**2. Verify that `solcjs` was installed:**

    solcjs --version

    0.8.0+commit.c7dfd78e.Emscripten.clang

Smart contracts can only be deployed after the correct version of `solcjs` is installed.

> Note: If you have problems installing any of the prerequisites (node, Mallet or solcjs),
contact the community in Slack:
[Join IOHK | Devnets on Slack](https://join.slack.com/t/iohkdevnets/shared_invite/zt-jvy74l5h-Bhp5SQajefwjig72BIl73A)


<a id="orgfd048b1"></a>

## Create a HelloWorld smart contract

To deploy your smart contracts on the KEVM devnet and test Mallet,
you will need to compile the Solidity code to KEVM (K - Ethereum virtual machine) bytecode.
You can compile the bytecode directly with using [solcjs](https://github.com/ethereum/solc-js#usage-on-the-command-line).


**1. Create a Solidity  file**

Create a `myContract.sol` file:

    cat << EOF >myContract.sol
    // SPDX-License-Identifier: MIT
    pragma solidity >=0.6.0 <0.9.0;


    contract HelloWorld {
      function helloWorld() external pure returns (string memory) {
        return "Hello, World!";
      }
    }
    EOF


**2. Compile with `solcjs`:**

    solcjs --bin --abi --base-path . ./myContract.sol


**3. Verify that the compiled file exists:**

If the file was correctly compiled, there should be a `.bin` file in your directory.

    ls *.bin

    _myContract_sol_HelloWorld.bin


## Mallet 2.0

Mallet, the minimal wallet, is the command line interface (CLI) used to send
transactions, deploy smart contracts, and interact with the IELE and
KEVM devnets.


**1. Connect to the KEVM devnet:**

    ./mallet kevm -d ./data

    Mallet 2.1.0 - IELE/KEVM devnet utility
    Type 'help()' to view the online documentation or 'listCommands()' to view available commands

This will open a session in the read-eval-print-loop (Repl) environment
for Node.js. Mallet commands are imported automatically.

Everything typed in the Repl has to be valid JavaScript. Technically
speaking, the commands are simply functions and properties of Mallet
object. However, we tend to refer to them as *commands* because that
reflects how they are used.


## Using the faucet


**1. Create an account**

Create an account to use this faucet by using this code:

    //execute inthe Mallet Repl
    //mallet kevm -d ./my_data/
    //mallet> .load ../test_smartcontract_deploy.js

    myAccount = newAccount()

The `newAccount` command asks your password, and returns your new account
address.

Note that we are assigning the return value of `newAccount` to a variable
named `myAccount` so we can refer to it later.


**2. Select an account**

Activate the account we have just created by using this code:

    selectAccount(myAccount)

    '0x45402404f51909b640d03f361c742c38d34bb3e7'


**3. Verify the balance of your new account**

Since the account has just been created, its balance should be 0.

    getBalance()

If you don't give any argument, this will return the balance of the
selected account.


**3. Request tokens from the faucet with `requestFunds`:**

    requestFunds()

Fund transfer might take a few minutes.

You can now compile and deploy smart contracts as your account is created *and* funded.


**4. Check the new balance in the account:**

    getBalance()


**5. Bring the compiled smart contract into Mallet**

Using the `_myContract_HelloWorld.bin` created earlier,
[1.4](#orgfd048b1),
you can now import the smart contract into Mallet.


**6. Import the `fileSystem` module:**

    fs = require("fs");


**7. Read the contents of the binary file:**

    myContract = fs.readFileSync('_myContract_sol_HelloWorld.bin', 'utf8');


## Deploying smart contracts

Now that you have the bytecode from `solcjs`, the next step is simply to deploy it.


**1. Prepare the transaction to deploy the contract:**

    tx = { gas: 470000, data: myContract}


**2. Send a transaction with the smart contract:**

    deploymentHash = sendTransaction(tx)

This will return the tx hash on which the contract was deployed to.


**3. View receipt**

You can view transaction details with the following command:

    getReceipt(deploymentHash)


**4. Save your contract address**

To save your contract address, create a variable that takes the return value of getReceipt():

    myContractAddress = getReceipt(deploymentHash).contractAddress


### Test your smart contract

    sendTransaction({to: myContractAddress,gas:10000})


**Getting help**

The `help` command can be useful When running Mallet in the CLI. This command opens the **Readme file** in your default web browser:

    help()

Alternatively, [Join IOHK | Devnets on Slack](https://join.slack.com/t/iohkdevnets/shared_invite/zt-jvy74l5h-Bhp5SQajefwjig72BIl73A) to obtain help from the community.
