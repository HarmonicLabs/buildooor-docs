# Simple Transactions

In this section we will look at wallet - wallet transfers

We will send ADA from the newly generated wallet, which you have just funded, to your
wallet.

---

To build and send transactions we will use the TxBuilder module with `buildSync`.

You can use `TxBuilderRunner` it is simpler, but you lose a lot of the control that you have using `buildSync`.

---

When we are building transactions, we need to set up our `TxBuilder`

Then we can use the `buildSync` method to construct the transaction.