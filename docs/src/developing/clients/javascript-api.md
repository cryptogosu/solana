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

Take the `SystemProgram` for example. The method signature for allocating space in your account on Solana looks like this:

```rust
pub fn allocate(
    pubkey: &Pubkey, 
    space: u64
) -> Instruction
```

In Solana when you want to interact with a program you must first know all the accounts you will be interacting with.

You must always provide every account that the program will be interacting with in the instruction. Not only that, but you must provide whether or not the account is `isSigner` or `isWritable`.

In the `allocate` method above, a single account `pubKey` is required, as well as an amount of `space` for allocation. We know that the `allocate` method writes to the account by allocating space within it, making the `pubKey` required to be `isWriteable`. `isSigner` is required when you are designating the account that is running the instruction. In this case, the signer is the account calling to allocate space within itself.

Let's look at how to call this instruction using solana-web3.js:

```javascript
let keyPair = web3.Keypair.generate();
let connection = new web3.Connection(web3.clusterApiUrl('testnet'));
```

First we set up the account keyPair and connection so that we have an account to make transactions on the testnet.

```javascript
let allocateTransaction = new web3.Transaction();
let keys = [{pubkey: keyPair.publicKey, isSigner: true, isWritable: true}];
let params = { space: 100 };
```

We create the transaction `allocateTransaction`, keys, and params objects. `keys`
represented all accounts that our `allocate` function will interact with. Since the `allocate` function also required space, we created `params` to be used later when invoked the `allocate` function.

```javascript
let allocateStruct = {
  index: 8,
  layout: struct([
    u32('instruction'),
    ns64('space'),
  ])
};
```

The above is created using `@solana/buffer-layout` in order to facilitate the payload creation. `allocate` takes in the parameter `space` and in order to interact with the function, we must provide the data as a Buffer format. The `buffer-layout` helps with allocating the buffer and encoding it correclty.

Let's look at what is within this struct.

`index` is set to 8, because the function `allocate` is in the 8th position in the instruction enum for `SystemProgram`.

```rust
pub enum SystemInstruction {
    /** 0 **/CreateAccount {/**/},
    /** 1 **/Assign {/**/},
    /** 2 **/Transfer {/**/},
    /** 3 **/CreateAccountWithSeed {/**/},
    /** 4 **/AdvanceNonceAccount,
    /** 5 **/WithdrawNonceAccount(u64),
    /** 6 **/InitializeNonceAccount(Pubkey),
    /** 7 **/AuthorizeNonceAccount(Pubkey),
    /** 8 **/Allocate {/**/},
    /** 9 **/AllocateWithSeed {/**/},
    /** 10 **/AssignWithSeed {/**/},
    /** 11 **/TransferWithSeed {/**/},
}
```

The `layout` in allocate struct must always have `u32('instruction')` first when you are using it to call an instruction. `ns64('space')` is the argument for the `allocate` function. `n64` is the javascript equivalent to `u64` in rust.

```javascript
let data = Buffer.alloc(allocateStruct.layout.span);
let layoutFields = Object.assign({instruction: allocateStruct.index}, params);
allocateStruct.layout.encode(layoutFields, data);
```

Using the previously created bufferLayout, we can allocate a data buffer. We then assign our params `{ space: 100 }` so that it maps correctly to the layout, and encode it to the data buffer. Now the data is ready to be send to be program.

```javascript
allocateTransaction.add(new web3.TransactionInstruction({
  keys,
  programId: web3.SystemProgram.programId,
  data,
}));

web3.sendAndConfirmTransaction(connection, allocateTransaction, [keyPair]);
```

Finally, we add the transaction instruction with all the account keys, data, and programId and broadcast the transaction to the network.

The full code can be found below. **Note**: You may need to fund the `keyPair` in order to get it to run on your local network.

```javascript
const {struct, u32, ns64} = require("@solana/buffer-layout");
const {Buffer} = require('buffer');
const web3 = require("@solana/web3.js");

let keyPair = web3.Keypair.generate();
let connection = new web3.Connection(web3.clusterApiUrl('testnet'));

let allocateTransaction = new web3.Transaction();
let keys = [{pubkey: keyPair.publicKey, isSigner: true, isWritable: true}];
let params = { space: 100 };

let allocateStruct = {
  index: 8,
  layout: struct([
    u32('instruction'),
    ns64('space'),
  ])
};

let data = Buffer.alloc(allocateStruct.layout.span);
let layoutFields = Object.assign({instruction: allocateStruct.index}, params);
allocateStruct.layout.encode(layoutFields, data);

allocateTransaction.add(new web3.TransactionInstruction({
  keys,
  programId: web3.SystemProgram.programId,
  data,
}));

web3.sendAndConfirmTransaction(connection, allocateTransaction, [keyPair]);
```

## Best Practices

### Simulate Before Sending

Before you send a transaction, you should [simulate the transaction](javascript-api.md#simulateTransaction). Simulating the transaction before sending a real transaction can reduce fees to the user of your dApp, as well as provide a better experience

### Principle of Least Privileges

When providing the accounts to interact with a program, determining which accounts need to be `isWritable` can be difficult. Best practice is to narrow down the accounts to only what requires `isWritable` in any transaction. If you do not, you run into issues where you could potentially give a program more power than the program needs and cause some security issues.

## API by Example

### Authorized

Authorized is an object used when creating an authorized account for staking within Solana. You can designate a `staker` and `withdrawer` separately, allowing for a different account to withdraw other than the staker.

#### Example Usage

```javascript
const {Authorized, Keypair} = require("@solana/web3.js")

let authorizedAccount = Keypair.generate();

let authorized = new Authorized(authorizedAccount, authorizedAccount);
```

You can find more usage of the `Authorized` object under [`StakeProgram`](javascript-api.md#StakeProgram)

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
