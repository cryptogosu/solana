---
title: Web3 JavaScript API
---

## What is Solana-Web3.js?

The Solana-Web3.js library aims to provide complete coverage of Solana. The library was built on top of the [Solana JSON RPC API](https://docs.solana.com/developing/clients/jsonrpc-api).

## Common Terminology

| Term | Definition |
|-------------|------------------------|
| Program     | Code written to interpret instructions. |
| Instruction | The smallest unit of a program that a client can include in a transaction. Within its processing code, an instruction may contain one or more cross-program invocations. |
| Transaction | One or more instructions signed by the client using one or more keypairs and executed atomically with only two possible outcomes: success or failure. |

For the full list of terms, see [Solana terminology](https://docs.solana.com/terminology#cross-program-invocation)

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

<!-- Production (minified) -->
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

### Connecting to a Wallet

To allow users to use your dApp or application on Solana, they will need to connect their wallet. A [Keypair](javascript-api.md#Keypair) is a private key with a matching public key, used to sign transactions.

There are two ways to obtain a Keypair:
1. Generate a new Keypair
2. Obtain a Keypair using the secret key

You can obtain a new Keypair with the following:

```javascript
const {Keypair} = require("@solana/web3.js");

let keypair = Keypair.generate();
```

This will generate a brand new keypair for a user to fund and use within your application. 

To allow a user to bring their keypair to your application by accepting a secretKey to create the keypair. You can allow entry of the secretKey using a textbox, and obtain the keypair with `Keypair.fromSecretKey(secretKey)`.

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

let keypair = Keypair.fromSecretKey(secretKey);
```

Many wallets today allow users to bring their keypairs using a variety of extensions or web wallets. You can find ways to connect to external wallets with the [wallet-adapter](https://github.com/solana-labs/wallet-adapter) library.

### Creating and Sending Transactions

To interact with programs on Solana, you create, sign, and send transactions to the network. Transactions are collections of instructions with signatures. The order that instructions exist in a transaction determines the order they are executed.

A transaction in Solana-Web3.js is created using the [`Transaction`](javascript-api.md#Transaction) object and adding desired messages, addresses, or instructions.

Take the example of a transfer transaction:

```javascript
const {Keypair, Transaction, SystemProgram, LAMPORTS_PER_SOL} = require("@solana/web3.js");

let keypair = Keypair.generate();
let transaction = new Transaction();

transaction.add(
  SystemProgram.transfer({
    fromPubkey: keypair.publicKey,
    toPubkey: keypair.publicKey,
    lamports: LAMPORTS_PER_SOL
  })
);
```

The above code achieves creating a transaction ready to be signed and broadcasted to the network. The `SystemProgram.transfer` instruction was added to the transaction, containing the amount of lamports to send, and the to and from addresses.

All that is left is to send the transaction over the network. You can accomplish sending a transaction by using `sendAndConfirmTransaction` if you wish to alert the user or do something after a transaction is finished, or use `sendTransaction` if you don't need to wait for the transaction to be confirmed.

```javascript
const {sendAndConfirmTransaction, clusterApiUrl} = require("@solana/web3.js");

let connection = new Connection(clusterApiUrl('testnet'));

sendAndConfirmTransaction(
  connection,
  transaction,
  [keypair]
);
```

The above code takes in a `TransactionInstruction` using `SystemProgram`, creates a `Transaction`, and sends it over the network.

### Interacting with Programs

The previous section visits sending basic transactions. At the time of writing programs on Solana are either written in Rust or C. Interacting with programs is a more complex transaction, and can be done similarly.

Take the `SystemProgram` for example. The method signature for allocating space in your account on Solana looks like this:

```rust
pub fn allocate(
    pubkey: &Pubkey, 
    space: u64
) -> Instruction
```

In Solana when you want to interact with a program you must first know all the accounts you will be interacting with.

You must always provide every account that the program will be interacting within the instruction. Not only that, but you must provide whether or not the account is `isSigner` or `isWritable`.

In the `allocate` method above, a single account `pubKey` is required, as well as an amount of `space` for allocation. We know that the `allocate` method writes to the account by allocating space within it, making the `pubKey` required to be `isWriteable`. `isSigner` is required when you are designating the account that is running the instruction. In this case, the signer is the account calling to allocate space within itself.

Let's look at how to call this instruction using solana-web3.js:

```javascript
let keypair = web3.Keypair.generate();
let connection = new web3.Connection(web3.clusterApiUrl('testnet'));
```

First, we set up the account Keypair and connection so that we have an account to make transactions on the testnet.

```javascript
let allocateTransaction = new web3.Transaction();
let keys = [{pubkey: keypair.publicKey, isSigner: true, isWritable: true}];
let params = { space: 100 };
```

We create the transaction `allocateTransaction`, keys, and params objects. `keys` represents all accounts that our `allocate` function will interact with. Since the `allocate` function also required space, we created `params` to be used later when invoked the `allocate` function.

```javascript
let allocateStruct = {
  index: 8,
  layout: struct([
    u32('instruction'),
    ns64('space'),
  ])
};
```

The above is created using `@solana/buffer-layout` to facilitate the payload creation. `allocate` takes in the parameter `space` and to interact with the function, we must provide the data as a Buffer format. The `buffer-layout` helps with allocating the buffer and encoding it correctly.

Let's break down this struct.

```javascript
index: 8
```

`index` is set to 8 because the function `allocate` is in the 8th position in the instruction enum for `SystemProgram`.

```rust
/* https://github.com/solana-labs/solana/blob/21bc43ed58c63c827ba4db30426965ef3e807180/sdk/program/src/system_instruction.rs#L142-L305 */
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

```javascript
u32('instruction')
```

The `layout` in the allocate struct must always have `u32('instruction')` first when you are using it to call an instruction.

```javascript
ns64('space')
```

`ns64('space')` is the argument for the `allocate` function. You can see in the original `allocate` function in Rust that space was of the type `u64`. `u64` is an unsigned 64bit integer. Javascript/Typescript by default only provides up to 53bit integers. `ns64` comes from `@solana/buffer-layout` to help with type conversions between Rust and Javascript. You can find more type conversions between Rust/C and Javascript at [solana-labs/buffer-layout](https://github.com/solana-labs/buffer-layout).

```javascript
let data = Buffer.alloc(allocateStruct.layout.span);
let layoutFields = Object.assign({instruction: allocateStruct.index}, params);
allocateStruct.layout.encode(layoutFields, data);
```

Using the previously created bufferLayout, we can allocate a data buffer. We then assign our params `{ space: 100 }` so that it maps correctly to the layout, and encodes it to the data buffer. Now the data is ready to be sent to be program.

```javascript
allocateTransaction.add(new web3.TransactionInstruction({
  keys,
  programId: web3.SystemProgram.programId,
  data,
}));

web3.sendAndConfirmTransaction(connection, allocateTransaction, [keypair]);
```

Finally, we add the transaction instruction with all the account keys, data, and programId and broadcast the transaction to the network.

The full code can be found below. **Note**: You may need to fund the `Keypair` to get it to run on your local network.

```javascript
const {struct, u32, ns64} = require("@solana/buffer-layout");
const {Buffer} = require('buffer');
const web3 = require("@solana/web3.js");

let keypair = web3.Keypair.generate();
let connection = new web3.Connection(web3.clusterApiUrl('testnet'));

let allocateTransaction = new web3.Transaction();
let keys = [{pubkey: keypair.publicKey, isSigner: true, isWritable: true}];
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

web3.sendAndConfirmTransaction(connection, allocateTransaction, [keypair]);
```

## Best Practices

### Principle of Least Privileges

When providing the accounts to interact with a program, determining which accounts need to be `isWritable` can be difficult. The best practice is to narrow down the accounts to only what requires `isWritable` in any transaction. If you do not, you run into issues where you could potentially give a program more power than the program needs and cause some security issues.

### Secret Key Management

The general recommendation is to not have the user input the secret key, but rather have the user either use a separate wallet and handle the account custodial services by the wallet. You can find more examples in the [solana wallet adapter](https://github.com/solana-labs/wallet-adapter).

## API by Example

### Authorized

[Source Documentation](https://solana-labs.github.io/solana-web3.js/classes/Authorized.html)

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

[Source Documentation](https://solana-labs.github.io/solana-web3.js/classes/EpochSchedule.html)

EpochSchedule gives you information on how many slots a leader is valid for, as well as what are the current rules for staking. The following are the parameters of an EpochSchedule:

* `slotsPerEpoch`: The maximum slots per Epoch
* `leaderScheduleSlotOffset`: The number of slots at the beginning of an epoch before the leader schedule is calculated
* `warmup`: Whether or not the epochs will start short and grow
* `firstNormalEpoch`: The first epoch length
* `firstNormalSlot`: The number of slots during the first epoch 

#### Example Usage

```javascript
const {EpochSchedule} = require("@solana/web3.js")

const firstNormalEpoch = 14;
const firstNormalSlot = 524256;
const leaderScheduleSlotOffset = 432000;
const slotsPerEpoch = 432000;
const warmup = true;

const epochSchedule = new EpochSchedule(
  slotsPerEpoch,
  leaderScheduleSlotOffset,
  warmup,
  firstNormalEpoch,
  firstNormalSlot,
);

console.log(epochSchedule);
// EpochSchedule {
//   slotsPerEpoch: 432000,
//   leaderScheduleSlotOffset: 432000,
//   warmup: true,
//   firstNormalEpoch: 14,
//   firstNormalSlot: 524256
// }
```

You can find more usage in [Connection](javascript-api.md#Connection)

### Keypair

[Source Documentation](https://solana-labs.github.io/solana-web3.js/classes/Keypair.html)

The keypair is used to create an account with a public key and secret key within Solana. You can either generate, generate from a seed, or create from a secret key.

#### Example Usage

```javascript
const {Keypair} = require("@solana/web3.js")

let account = Keypair.generate();

console.log(account.publicKey.toBase58());
console.log(account.secretKey);

// 2DVaHtcdTf7cm18Zm9VV8rKK4oSnjmTkKE6MiXe18Qsb
// Uint8Array(64) [
//   152,  43, 116, 211, 207,  41, 220,  33, 193, 168, 118,
//    24, 176,  83, 206, 132,  47, 194,   2, 203, 186, 131,
//   197, 228, 156, 170, 154,  41,  56,  76, 159, 124,  18,
//    14, 247,  32, 210,  51, 102,  41,  43,  21,  12, 170,
//   166, 210, 195, 188,  60, 220, 210,  96, 136, 158,   6,
//   205, 189, 165, 112,  32, 200, 116, 164, 234
// ]


let seed = Uint8Array.from([70,60,102,100,70,60,102,100,70,60,102,100,70,60,102,100,70,60,102,100,70,60,102,100,70,60,102,100,70,60,102,100]);
let accountFromSeed = Keypair.fromSeed(seed);

console.log(accountFromSeed.publicKey.toBase58());
console.log(accountFromSeed.secretKey);

// 3LDverZtSC9Duw2wyGC1C38atMG49toPNW9jtGJiw9Ar
// Uint8Array(64) [
//    70,  60, 102, 100,  70,  60, 102, 100,  70,  60, 102,
//   100,  70,  60, 102, 100,  70,  60, 102, 100,  70,  60,
//   102, 100,  70,  60, 102, 100,  70,  60, 102, 100,  34,
//   164,   6,  12,   9, 193, 196,  30, 148, 122, 175,  11,
//    28, 243, 209,  82, 240, 184,  30,  31,  56, 223, 236,
//   227,  60,  72, 215,  47, 208, 209, 162,  59
// ]


let accountFromSecret = Keypair.fromSecretKey(account.secretKey);

console.log(accountFromSecret.publicKey.toBase58());
console.log(accountFromSecret.secretKey);

// 2DVaHtcdTf7cm18Zm9VV8rKK4oSnjmTkKE6MiXe18Qsb
// Uint8Array(64) [
//   152,  43, 116, 211, 207,  41, 220,  33, 193, 168, 118,
//    24, 176,  83, 206, 132,  47, 194,   2, 203, 186, 131,
//   197, 228, 156, 170, 154,  41,  56,  76, 159, 124,  18,
//    14, 247,  32, 210,  51, 102,  41,  43,  21,  12, 170,
//   166, 210, 195, 188,  60, 220, 210,  96, 136, 158,   6,
//   205, 189, 165, 112,  32, 200, 116, 164, 234
// ]
```

Using `generate` generates a random Keypair for use as an account on Solana. Using `fromSeed`, you can generate a Keypair using a deterministic constructor. `fromSecret` creates a Keypair from a secret Uint8array. You can see that the publicKey for the `generate` Keypair and `fromSecret` Keypair are the same because the secret from the `generate` Keypair is used in `fromSecret`.

**Warning**: Do not use `fromSeed` unless you are creating a seed with high entropy. Do not share your seed. Treat the seed like you would a private key.

#### Example Usage

### Loader

### Lockup

[Source Documentation](https://solana-labs.github.io/solana-web3.js/classes/Lockup.html)

Lockup is used in conjunction with the [StakeProgram](javascript-api.md#StakeProgram) to create an account. The Lockup is used to determine how long the stake will be locked, or unable to be retrieved. If the Lockup is set to 0 for both epoch and the unix timestamp, the lockup will be disabled for the stake account.

#### Example Usage

```javascript
const {Authorized, Keypair, Lockup, StakeProgram} = require("@solana/web3.js");

let account = Keypair.generate();
let stakeAccount = Keypair.generate();
let authorized = new Authorized(account.publicKey, account.publicKey);
let lockup = new Lockup(0, 0, account.publicKey);

let createStakeAccountInstruction = StakeProgram.createAccount({
    fromPubkey: account.publicKey,
    authorized: authorized,
    lamports: 1000,
    lockup: lockup,
    stakePubkey: stakeAccount.publicKey
});
```
The above code creates a `createStakeAccountInstruction` to be used when creating an account with the `StakeProgram`. The Lockup is set to 0 for both the epoch and unix timestamp, disabling lockup for the account. 

See [StakeProgram](javascript-api.md#StakeProgram) for more.

### Message

### NonceAccount

[Source Documentation](https://solana-labs.github.io/solana-web3.js/classes/NonceAccount.html)

Normally a transaction is rejected if a transaction's `recentBlockhash` field is too old. To provide for certain custodial services, Nonce Accounts are used.

You can create a nonce account by first creating a normal account, then using `SystemProgram` to make the account a Nonce Account.

#### Example Usage

```javascript
const web3 = require('@solana/web3.js');

// Create connection
const connection = new web3.Connection(
web3.clusterApiUrl('testnet'),
'confirmed',
);

// Generate accounts
const account = web3.Keypair.generate();
const nonceAccount = web3.Keypair.generate();

// Fund account
const airdropSignature = await connection.requestAirdrop(
account.publicKey,
web3.LAMPORTS_PER_SOL,
);

await connection.confirmTransaction(airdropSignature);

// Get Minimum amount for rent exemption
const minimumAmount = await connection.getMinimumBalanceForRentExemption(
web3.NONCE_ACCOUNT_LENGTH,
);

// Form CreateNonceAccount transaction
const transaction = new web3.Transaction().add(
web3.SystemProgram.createNonceAccount({
    fromPubkey: account.publicKey,
    noncePubkey: nonceAccount.publicKey,
    authorizedPubkey: account.publicKey,
    lamports: minimumAmount,
}),
);
// Create Nonce Account
await web3.sendAndConfirmTransaction(
    connection,
    transaction,
    [account, nonceAccount]
);

const nonceAccountData = await connection.getNonce(
nonceAccount.publicKey,
'confirmed',
);

console.log(nonceAccountData);
// NonceAccount {
//   authorizedPubkey: PublicKey {
//     _bn: <BN: 919981a5497e8f85c805547439ae59f607ea625b86b1138ea6e41a68ab8ee038>
//   },
//   nonce: '93zGZbhMmReyz4YHXjt2gHsvu5tjARsyukxD4xnaWaBq',
//   feeCalculator: { lamportsPerSignature: 5000 }
// }

const nonceAccountInfo = await connection.getAccountInfo(
    nonceAccount.publicKey,
    'confirmed'
);

const nonceAccountFromInfo = web3.NonceAccount.fromAccountData(nonceAccountInfo.data);

console.log(nonceAccountFromInfo);
// NonceAccount {
//   authorizedPubkey: PublicKey {
//     _bn: <BN: 919981a5497e8f85c805547439ae59f607ea625b86b1138ea6e41a68ab8ee038>
//   },
//   nonce: '93zGZbhMmReyz4YHXjt2gHsvu5tjARsyukxD4xnaWaBq',
//   feeCalculator: { lamportsPerSignature: 5000 }
// }
```

The above example shows both how to create a `NonceAccount` using `SystemProgram.createNonceAccount`, as well as how to retrieve the `NonceAccount` from accountInfo. Using the nonce, you can create transactions offline with the nonce in place of the `recentBlockhash`.

### PublicKey

[Source Documentation](https://solana-labs.github.io/solana-web3.js/classes/PublicKey.html)

PublicKey is used throughout `@solana/web3.js` in transactions, keypairs, and programs. You require publickey when listing each account in a transaction and as a general identifier on Solana.

A PublicKey can be created with a base58 encoded string, buffer, Uint8Array, number, and an array of numbers.

#### Example Usage

```javascript

```

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
