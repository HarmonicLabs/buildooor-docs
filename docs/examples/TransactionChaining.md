---
sidebar_position: 8
slug: '/examples/transaction-chaining'
---

# Transaction Chaining

This last example is going to cover Transaction Chaining.

This is an offchain technique to submit sequences of transactions that depend on each other.

Using this technique, we dont need to wait for transactions to settle on chain.

Because of the 'First In First Out' way that cardano processes UTxOs, we can spend a transaction that hasnt finalised yet, because we know that it will be processed before the following one.

If you have been following this guide, you will realise that each of the transactions that have been submitted return a transaction hash before the transaction shows on the explorer. 

This is because transactions are submitted to the mempool and propogate the network before they are finalised in a block. 

But the transaction hash has already been calcutlated and you already know the sequence of outputs, because you built the transaction.

We can tap into this time between blocks, and transaction finalisation, to help submit these multiple sequential transactions faster.