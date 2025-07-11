---
sidebar_position: 7
slug: '/examples/simple-transaction'
---

# Simple Transactions

In this section we will look at wallet - wallet transfers

We will send ADA from the newly generated wallet, which you have just funded, to your
wallet.

---

To build and send transactions we will use the TxBuilder module with `buildSync`.

You can use `TxBuilderRunner` it is simpler, but you lose a lot of the control that you have using `buildSync`.

---

When we are building transactions, we need to set up our `TxBuilder` first.

create a file called `getTxBuilder.ts` and insert the following code to initialise `TxBuilder` with our `Provider`:

```ts
import {
    defaultMainnetGenesisInfos,
    IProvider,
    TxBuilder,
} from "@harmoniclabs/buildooor";

let _txBuilder: TxBuilder | undefined = undefined;
export async function getTxBuilder(
  provider: Partial<IProvider>
): Promise<TxBuilder> {
  if (_txBuilder instanceof TxBuilder) return _txBuilder;

  _txBuilder = new TxBuilder(
    await provider.getProtocolParameters!(),
    (await provider.getGenesisInfos!()) ?? defaultMainnetGenesisInfos
  );

  return _txBuilder;
}
```

then we can create our `simpleTransaction.ts` file to create our transaction, sign and submit it to the blockchain:

```ts
import { Address, ITxBuildInput, ITxBuildOutput, Value } from '@harmoniclabs/buildooor'
import { blockfrost } from './blockfrost.js'
import { getTxBuilder } from './getTxBuilder.js'
import { user1Details } from './user1.js'

export async function simpleTransaction() {
  // First we get our blockfrost provider
  const b = await blockfrost()

  // Then we get our TxBuilder
  const txBuilder = await getTxBuilder(b)

  // Then we get our wallet details
  const { address, pkh, XPrv } = await user1Details()

  ...
}
```

we need to query the blockchain for the utxos at the user1 wallet:

```ts
export async function simpleTransaction() {
  ...

  // We get our UTxOs from our wallet address using our `BlockfrostProvider` with the `getUtxos()` method passing the address as a parameter
  const utxos = await b.getUtxos(address)

  // we will use the first utxo in the returned list
  const spendingUtxo = utxos[0]

  // and we will send 10ADA to our testnet address from user1
  const adaToSend = 10000000 // 10 ADA in lovelace

  ...
}
```

In order to send it to a different address, we need to know that address. In your wallet, you can switch network to `preprod` and get your address there, that way you will send it to your main wallet from `user1` - but make sure you are sending on the `preprod` network.

```ts
export async function simpleTransaction() {
  ...

  // address string from your wallet
  const recipientAddress = <YourWalletAddressString>

  ...
```

We are going to send 10ADA as specified above in our code, so we need to turn this integer into a `lovelace value`.

> NOTE:
> Lovelace is the smallest denomination of ADA
> 1 ADA == 1,000,000 lovelace

```ts
export async function simpleTransaction() {
  ...
  
  // Value.lovelaces creates a Lovelace value from an integer amount of lovelaces
  const valueToSend = Value.lovelaces(adaToSend)

  ...
```

Next we need to set up our Inputs and Outputs in the expected format for `buildSync`.

`buildSync` expects an `ITxBuildInput` as an input value.

[See the ITxBuildInput docs](/TxBuilder/buildSync)

There are fields for `Scripts` & `RefScripts` but we dont need these as we are spending from a wallet address, not a script address

```ts
export async function simpleTransaction() {
  ...
  
  // An ITxBuildInput requires a UTxO object
  // there is also optional Script or RefScript fields for spending from Validators
  const input: ITxBuildInput = {
    // we add our spendingUtxo to the utxo field
    utxo: spendingUtxo
  }

  ...
```

Outputs are similar, in that there is a defined type they need to conform to `ITxBuildOutput` that `buildSync` expects

[See the ITxBuildOutput docs](/TxBuilder/buildSync)

```ts
export async function simpleTransaction() {
  ...

  // An ITxBuildOutput requires an Address and a Value
  // there is also optional Datum & RefScript fields if the need to be included
  const output: ITxBuildOutput = {
    address: Address.fromBech32(recipientAddress),
    value: valueToSend,
  }

  ...
```

Then we can use the `buildSync()` method to construct the transaction.

```ts
export async function simpleTransaction() {
  ...

  // buildSync requires a change address ( usually your own ) used for returning any
  // excess assets back to your wallet
  const tx = txBuilder.buildSync({
    inputs: [input],
    outputs: [output],
    changeAddress: address,
  })

  ...
}
```

Finally, now that we have built our transaction, we can sign and submit it with our `user1` private key:

```ts
export async function simpleTransaction() {
  ...

  // we sign the transaction with our `user1` private key (as that is the wallet we are spending from)
  tx.signWith(XPrv)

  const txHash = await b.submitTx(tx)

  return console.log(`Transaction submitted successfully with hash: ${txHash}
  View the Transaction on CardanoScan:
    https://preprod.cardanoscan.io/transactions/${txHash}
  
  * It can take a minute or two to show up in the explorer * 
  `)
}
```

With this done, you can add the function call at the bottom of the page and run the transaction.

Return to [Getting Started](/docs/getting-started) to continue ...