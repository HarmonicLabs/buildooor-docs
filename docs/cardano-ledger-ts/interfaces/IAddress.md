[@harmoniclabs/cardano-ledger-ts](../globals.md) / IAddress

# Interface: IAddress

Interface representing a Cardano address with its components.

## Properties

### network

> **network**: [`NetworkT`](../type-aliases/NetworkT.md)

The network identifier for the address (mainnet or testnet).

#### Defined in

[src/ledger/Address.ts:12](https://github.com/HarmonicLabs/cardano-ledger-ts/blob/94dd590ffe94133126b0d8d49920fc7b002e1975/src/ledger/Address.ts#L12)

***

### paymentCreds

> **paymentCreds**: [`Credential`](../classes/Credential.md)

The payment credentials associated with the address.

#### Defined in

[src/ledger/Address.ts:13](https://github.com/HarmonicLabs/cardano-ledger-ts/blob/94dd590ffe94133126b0d8d49920fc7b002e1975/src/ledger/Address.ts#L13)

***

### stakeCreds?

> `optional` **stakeCreds**: [`StakeCredentials`](../classes/StakeCredentials.md)

Optional stake credentials associated with the address.

#### Defined in

[src/ledger/Address.ts:14](https://github.com/HarmonicLabs/cardano-ledger-ts/blob/94dd590ffe94133126b0d8d49920fc7b002e1975/src/ledger/Address.ts#L14)

***

### type?

> `optional` **type**: [`AddressType`](../type-aliases/AddressType.md)

Optional type of the address ("base" | "pointer" | "enterprise" | "bootstrap" | "unknown").

#### Defined in

[src/ledger/Address.ts:15](https://github.com/HarmonicLabs/cardano-ledger-ts/blob/94dd590ffe94133126b0d8d49920fc7b002e1975/src/ledger/Address.ts#L15) 