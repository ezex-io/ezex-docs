# Key Management System

The **Key Management System (KMS)** in ezeX is a **stateless** service with a single task: generating a key per request.
Being stateless means it has no attached database and no knowledge of other services.

To achieve this, we need two components:
1. A Secure Hierarchical Deterministic (HD) Wallet
2. A Unique Path per User per Chain

## HD Wallet

An **HD (Hierarchical Deterministic) Wallet** is a type of wallet that can generate deterministic keys based on a given path.
These wallets are defined and explained in [BIP-32](https://github.com/bitcoin/bips/blob/master/bip-0032.mediawiki).

We can use any kind of wallet as long as it is secure, supports BIP-32, and ideally complies with the
[SLIP-0010](https://github.com/satoshilabs/slips/blob/master/slip-0010.md) standard.
Currently, ezeX supports **Ed25519** and **Secp256k1** curves.

## Unique Path

The second requirement is a **unique path** to an address for each user per blockchain.
For example, if Alice wants a deposit address in **Bitcoin** and **Pactus**,
we must generate two distinct addresses for her, neither of which should collide with any other user’s addresses.

Here, we can rely on [BIP-44](https://github.com/bitcoin/bips/blob/master/bip-0044.mediawiki).
A **BIP-44 path** is structured as follows:

```
m / purpose' / coin_type' / account' / change / address_index
```

- **m**: The master node (root of the HD wallet).
- **purpose'**: Typically set to `44'` (BIP-44 compliant wallets).
- **coin_type'**: Specifies the cryptocurrency (e.g., `0'` for Bitcoin, `21888'` for Pactus). A full list is available under [SLIP-0044](https://github.com/satoshilabs/slips/blob/master/slip-0044.md).
- **account'**: A unique account number per user per service in ezeX.
- **change**: Indicated change address for Bitcoin.
- **address_index**: Individual addresses under each account.

Since we support multiple blockchains, we relax the path requirements and allow each chain to
define its own structure for deposit, withdrawal, and other addresses.
This is implemented in the [Open-Chain](https://github.com/ezex-io/open-chain) repository. We only need to ensure:
- The coin_type is set correctly.
- Each chain implements required APIs (e.g., public key-to-address conversion, unique path generation).
- Implementations are well-tested and properly documented.

## Deposit Address Generation

In **ezeX**, we generate a unique deposit address per user.
To achieve this, we assign a unique account number per user using the following algorithm:

### Procedure:
1. **salt** = `"DEPOSIT:" + user_id`
2. **r** = `2^31`
3. **account** = `0`
4. **while** `account == 0`:
   1. **salt** = `H(salt)`
   2. **account** = `salt mod r`
   3. **if** `account exists`:
      1. **account** = `0`
5. **return** `account`

Where:
- **H** is a hash function (preferably **SHA-256**) that generates a unique hash for the given salt as a big integer.

Once we have the unique account number, we store it in the deposit database.
The above algorithm only applies to users who do not already have an assigned account number.
