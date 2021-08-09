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

### Creating a Transaction

### Interacting with Programs

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
