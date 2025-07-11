---
sidebar_position: 2
slug: '/getting-started'
---

# Getting Started

This getting started guide will walk you through all of the basics of using Buildooor,
from set up to submitting complex transactions with different validators.

The first step to following this guide is to start an empty node project, everything else
will be included in the docs, from imports to scripts, so you can grab snippets from
this documentation and they will work together seamlessly.

With Buildooor installed, the first thing you need to do is set up your `Provider` we will
use blockfrost for these examples, but you are able to use any others that you wish.

[Provider Example](/docs/examples/provider)

With Blockfrost set up we can start looking at some examples that will help get you up and
running with Buildooor.

---

Simple Transactions
Compiling Validators
Validator Transactions

---

## Simple Transactions

For this guide we will need to generate a wallet and fund it, so we can run the 
transactions.

[Generating Wallets](/docs/examples/generating-wallets)

Once you have your wallet, fund it from the Testnet Faucet -Link-, or send some test ADA from your wallet.

> NOTE:
> If you ar sending from your wallet, make sure you switch your wallet network to Pre-Prod.
> DO NOT send ADA on the Mainnet network.

Then we can start building and testing transactions.

[Simple Transaction](/docs/examples/simple-transaction)

---

## Compiling Validators

Now we have had a look at transferring ADA values between wallets, we can start to look at some more complex transaction examples.

In order to do that, we will need to look at compiling validators first

[Compiling Validators](/docs/examples/compiling-validators)

---

## Simple Minting

Now we can compile validators easily, we can take a lok at minting assets on Cardano.

For simple minting we will mint some tokens with a minting validator:

[Simple Minting](/docs/examples/simple-minting)

---

## Sending Assets

Our previous minting validator didnt have any parameters.

To look at sending assets, we will also take a first-look at applying parameters with our `makeValidator()` function.

[Sending Assets](/docs/examples/sending-assets)

---

## OneShot Minting

Our final validator example will take a look at other parameters, in this case it will be using the common `oneshot` design pattern.

[One Shot](/docs/examples/one-shot-minting)

---

## Transaction Chaining

For the last example we will look at transaction chaining, we will reuse the oneshot validator and build a few sequential transactions that use the token.

[Transaction Chaining](/docs/examples/transaction-chaining)