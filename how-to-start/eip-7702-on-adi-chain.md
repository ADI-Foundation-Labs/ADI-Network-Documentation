---
description: >-
  Set-code transactions: what they do on ADI, how to send one, and which
  networks support them.
---

# EIP-7702 on ADI Chain

An EIP-7702 set-code transaction points an EOA at a contract. Calls to that EOA run the contract's code in the EOA's context. This enables single-signature wallets, spending limits, session keys, batched approvals, and no separate smart-account deployment.

### Support status

{% hint style="warning" %}
**Mainnet doesn't support EIP-7702 transactions currently**, since the protocol/sequencer upgrade has not yet been executed. Stay tuned!
{% endhint %}

| Network     | Chain ID | RPC                                       | `web3_clientVersion` | EIP-7702 status                                                                     |
| ----------- | -------- | ----------------------------------------- | -------------------- | ----------------------------------------------------------------------------------- |
| ADI Mainnet | `36900`  | `https://rpc.adifoundation.ai`            | `zksync-os/v0.21.1`  | Not enabled. Type `0x4` transactions and `authorizationList` requests are rejected. |
| ADI Testnet | `99999`  | `https://rpc.ab.testnet.adifoundation.ai` | `zksync-os/v0.24.2`  | Enabled. Type `0x4` transactions mine and delegation applies.                       |

### How a delegation looks on chain

A delegation changes the EOA's code to a 23-byte designator. `eth_getCode` returns it.

```
0xef0100917c0e0cb7f2ff0d0e2785c05e04d26cf5157932
       ^^^^^^ delegate address
```

The designator is `0xef0100 || address`. The account keeps its address, balance, and nonce. You do not deploy a contract at the EOA.

### The authorization tuple

| Field     | Meaning                                                       |
| --------- | ------------------------------------------------------------- |
| `chainId` | Chain where the authorization applies.                        |
| `address` | Delegate address. Use the zero address to clear a delegation. |
| `nonce`   | Authority nonce required when the transaction is included.    |
| `yParity` | Signature recovery parity.                                    |
| `r`       | ECDSA signature value.                                        |
| `s`       | ECDSA signature value.                                        |

* `nonce` must equal the authority's nonce when the transaction is included. When the authority sends the transaction, use sender nonce + 1. The transaction consumes the current nonce first.
* Set `address` to `0x0000000000000000000000000000000000000000` to clear an existing delegation.

### Send one with Foundry

Deploy this contract to ADI Testnet before running the commands.

```solidity
contract Answer {
    function answer() external pure returns (uint256) {
        return 42;
    }
}
```

```bash
export ADI_TESTNET=https://rpc.ab.testnet.adifoundation.ai
export PRIVATE_KEY=0x... # Fund this account with the testnet faucet.
export DELEGATE=0x917c0e0cb7f2ff0d0e2785c05e04d26cf5157932
export AUTHORITY=$(cast wallet address --private-key $PRIVATE_KEY)

# Sign for the authority nonce after the transaction you are about to send.
NONCE=$(cast nonce --rpc-url $ADI_TESTNET $AUTHORITY)
AUTH=$(cast wallet sign-auth $DELEGATE --private-key $PRIVATE_KEY \
  --rpc-url $ADI_TESTNET --nonce $((NONCE + 1)))

# Send a type-4 transaction that carries the authorization.
cast send --rpc-url $ADI_TESTNET --private-key $PRIVATE_KEY \
  --nonce $NONCE --auth "$AUTH" $DELEGATE --gas-limit 200000
```

The transaction `to` may be any existing address. The authorization and call are independent.

### Read the result

Read the authority's code. It returns the designator.

```bash
cast code --rpc-url $ADI_TESTNET $AUTHORITY
# 0xef0100917c0e0cb7f2ff0d0e2785c05e04d26cf5157932
```

Call the authority. The delegated `Answer` code returns `42`.

```bash
cast call --rpc-url $ADI_TESTNET $AUTHORITY "answer()(uint256)"
# 42
```

{% hint style="success" %}
`eth_getCode` exposes the designator. Wallets and block explorers can use it to detect a delegated EOA.
{% endhint %}

### Clear a delegation

Sign an authorization for the zero address. Then send it in another type-4 transaction.

```bash
NONCE=$(cast nonce --rpc-url $ADI_TESTNET $AUTHORITY)
AUTH=$(cast wallet sign-auth 0x0000000000000000000000000000000000000000 \
  --private-key $PRIVATE_KEY --rpc-url $ADI_TESTNET --nonce $((NONCE + 1)))
cast send --rpc-url $ADI_TESTNET --private-key $PRIVATE_KEY \
  --nonce $NONCE --auth "$AUTH" $DELEGATE --gas-limit 200000

cast code --rpc-url $ADI_TESTNET $AUTHORITY
# 0x
```

The authority becomes a plain EOA from the next block.

### RPC support

ADI Testnet accepts `authorizationList` on `eth_call`, `eth_estimateGas`, and `eth_fillTransaction`. It accepts serialized type `0x4` transactions on `eth_sendRawTransaction`.

`eth_fillTransaction` returns the serialized type-4 transaction in `raw`. It returns the decoded transaction object in `tx`. This is the fastest way to inspect the tuple that the node built.

A type-4 transaction requires `to`. Contract creation is unavailable. Requests without `to` return `-32602` and `EIP-7702 transactions require a` to `address`.

### Gas

These values were measured on Testnet for plain ADI transfers to an EOA.

| Transaction                                   | Gas used |
| --------------------------------------------- | -------- |
| Transfer without authorization                | `21,000` |
| Transfer with one applied authorization       | `36,800` |
| Transfer with one skipped stale authorization | `46,000` |

Each tuple costs `25,000` intrinsic gas. An applied tuple refunds up to `12,500`. The exact net amount depends on the refund cap.

### Tips

* **Check the code, not only status**: a stale authorization nonce silently skips the authorization. The transaction can return status `1` with no delegation set.
* **Accept inbound transfers**: a delegate that rejects plain value transfers stops the EOA from receiving funds. Sending from a delegated EOA remains ordinary, and a 1 wei transfer used `21,000` gas in testing.
* **Expect delegation to persist**: a reverted transaction does not undo a processed delegation. The sender still pays for gas.
* **Always provide `to`**: a set-code transaction cannot create a contract.
* **Treat signatures as security-critical**: review every authorization before signing. See [signing-malicious-transactions.md](../for-new-users/signing-malicious-transactions.md "mention").

### Related

* [gas-abstraction-and-account-abstraction.md](gas-abstraction-and-account-abstraction.md "mention")
* [json-rpc-api](../references/json-rpc-api/ "mention")
* [changelog.md](../references/changelog.md "mention")
