---
title: Web3 JavaScript API
---

## What is Solana-Web3.js?

The Solana-Web3.js library aims to provide complete coverage of Solana. The library was built on top of the [Solana JSON RPC API](https://docs.solana.com/developing/clients/jsonrpc-api).

## Common Terminology

| Term | Definition |
|-------------|------------------------|
| Instruction | The smallest unit of a program that a client can include in a transaction. Within its processing code, an instruction may contain one or more cross-program invocations. |
| Program     | The code that interprets instructions.                                                                                                                                   |
| Transaction | One or more instructions signed by the client using one or more keypairs and executed atomically with only two possible outcomes: success or failure.                    |

For the full list of terms, see [Solana terminology](https://docs.solana.com/terminology#cross-program-invocation)

## Features

## Getting Started

### Installation

#### yarn

```bash
$ yarn add @solana/web3.js
```

#### npm

```bash
$ npm install --save @solana/web3.js
```

#### Bundle

```html
<!-- Development (un-minified) -->
<script src="https://unpkg.com/@solana/web3.js@latest/lib/index.iife.js"></script>

<!-- Production (un-minified) -->
<script src="https://unpkg.com/@solana/web3.js@latest/lib/index.iife.min.js"></script>
```

### Usage

#### Javascript

```javascript
const solanaWeb3 = require('@solana/web3.js');
console.log(solanaWeb3);
```


#### ES6

```javascript
import * as solanaWeb3 from '@solana/web3.js';
console.log(solanaWeb3);
```


#### Browser Bundle

```javascript
// solanaWeb3 is provided in the global namespace by the bundle script
console.log(solanaWeb3);
```

## Quickstart

### Connecting to a wallet

To allow users to use your dApp or application on Solana, they will need to connect their wallet. Wallets are represented by [keyPairs](javascript-api.md#Keypair) in solana-web3.js, and can be used in sending transactions, interacting with programs, and signing information within the Solana ecosystem.

There are two ways to obtain a keyPair:
1. Generate a new keyPair
2. Obtain a keyPair using the secret key

You can obtain a new keyPair with the following:

```javascript
const {Keypair} = require("@solana/web3.js");

let keyPair = Keypair.generate();
```

This will generate a brand new wallet for a user to fund and use within your application. 

To allow a user to bring their own wallet to your application by accepting a secretKey to create the keyPair. You can allow entry of the secretKey using a textbox, and obtain the wallet with `Keypair.fromSecretKey(secretKey)`.

```javascript
const {Keypair} = require("@solana/web3.js");

let secretKey = Uint8Array.from([
  202, 171, 192, 129, 150, 189, 204, 241, 142,  71, 205,
  2,  81,  97,   2, 176,  48,  81,  45,   1,  96, 138,
  220, 132, 231, 131, 120,  77,  66,  40,  97, 172,  91,
  245,  84, 221, 157, 190,   9, 145, 176, 130,  25,  43,
  72, 107, 190, 229,  75,  88, 191, 136,   7, 167, 109,
  91, 170, 164, 186,  15, 142,  36,  12,  23
]);

let keyPair = Keypair.fromSecretKey(secretKey);
```

Many wallets today allow users to bring their wallet using a variety of extensions or web wallets. You can find ways to connect to external wallets with the [wallet-adapter](https://github.com/solana-labs/wallet-adapter) library.

### Creating and Sending Transactions

In order to achieve anything in Solana, you need to create transactions. Transactions are a collection of signatures which can comprise of messages, accounts, or instructions. The order that instructions exist in a transaction determine the order they are executed.

A transaction in Solana-Web3.js is created using the [`Transaction`](javascript-api.md#Transaction) object and adding desired messages, addresses, or instructions.

Take the example of a transfer transaction:

```javascript
const {Keypair, Transaction, SystemProgram, LAMPORTS_PER_SOL} = require("@solana/web3.js");

let keyPair = Keypair.generate();
let transaction = new Transaction();

transaction.add(
  SystemProgram.transfer({
    fromPubkey: keyPair.publicKey,
    toPubkey: keyPair.publicKey,
    lamports: LAMPORTS_PER_SOL
  })
);
```

The above code achieves creating a transaction ready to be signed and broadcasted to the network. The `SystemProgram.transfer` instruction was added to the transaction, containing the amount of lamports to send, and the to and from addressees.

All that is left is to send the transaction over the network. You can accomplish sending a transaction by using `sendAndConfirmTransaction` if you wish to alert the user or do something after a transaction is finished, or use `sendTransaction` if you don't need to wait for the transaction to be confirmed.

```javascript
const {sendAndConfirmTransaction, clusterApiUrl} = require("@solana/web3.js");

let connection = new Connection(clusterApiUrl('testnet'));

sendAndConfirmTransaction(
  connection,
  transaction,
  [keyPair]
);
```

The above codes takes in a `TranasctionInstruction` using `SystemProgram`, creates a `Trasaction`, and sends it over the network.

### Interacting with Programs

The previous section visits sending basic transactions. Interacting with programs is a more complex transaction, and can be done similarly.

## Best Practices

## API by Example

### Account

***DEPRECATED*** since v1.10.0, please use [Keypair](javascript-api.md#Keypair) instead.

[Source Documentation](https://solana-labs.github.io/solana-web3.js/classes/Account.html)

Accounts on Solana hold arbitrary amounts of data such as whether or not the account is executable or how many lamports are stored on the account. Account are represented by a 256 bit public key.

#### Example Usage

```javascript
const {Account} = require("@solana/web3.js");

let account = new Account();

console.log(account);

// Account {
//   _keypair: {
//     publicKey: Uint8Array(32) [
//       161,  11, 126, 129,  80, 209,  62, 74,
//       101,  13, 106, 187,  79,  99, 163, 45,
//       143, 201,  17, 227,  63, 239,  86, 10,
//       184, 137,   7, 226, 131, 126, 120, 90
//     ],
//     secretKey: Uint8Array(64) [
//       111,  52,  11, 68,  37,  98, 130, 244,  94,  33, 243,
//        34,  46,  42, 98, 222, 121, 107,  50,  79,  53, 192,
//       124, 197,  45, 25, 232, 147,  36,  20, 222, 160, 161,
//        11, 126, 129, 80, 209,  62,  74, 101,  13, 106, 187,
//        79,  99, 163, 45, 143, 201,  17, 227,  63, 239,  86,
//        10, 184, 137,  7, 226, 131, 126, 120,  90
//     ]
//   }
// }

console.log(account.publicKey.toBase58());

//BqeogTgggg2GyWcLWydUzk2opmZECzKLHYJNsTjmUWGh

console.log(account.secretKey.toJSON());

// {
//   type: 'Buffer',
//   data: [
//     111,  52,  11, 68,  37,  98, 130, 244,  94,  33, 243,
//      34,  46,  42, 98, 222, 121, 107,  50,  79,  53, 192,
//     124, 197,  45, 25, 232, 147,  36,  20, 222, 160, 161,
//      11, 126, 129, 80, 209,  62,  74, 101,  13, 106, 187,
//      79,  99, 163, 45, 143, 201,  17, 227,  63, 239,  86,
//      10, 184, 137,  7, 226, 131, 126, 120,  90
//   ]
// }
```

### Authorized

### BpfLoader

### Connection

### Enum

### EpochSchedule

### Keypair

[Source Documentation](https://solana-labs.github.io/solana-web3.js/classes/Keypair.html)

#### Example Usage

### Loader

### Lockup

### Message

### NonceAccount

### PublicKey

### Secp256k1Program

### StakeInstruction

### StakeProgram

### Struct

### SystemInstruction

### SystemProgram

### Transaction

### TransactionInstruction

### ValidatorInfo

### VoteAccount

## Examples

## Contributing


See [solana-web3](https://solana-labs.github.io/solana-web3.js/).
