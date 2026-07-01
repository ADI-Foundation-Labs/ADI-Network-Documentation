---
description: >-
  The JSON-RPC methods served by the ADI Sequencer - the Ethereum-compatible
  namespaces plus the extensions.
---

# JSON-RPC API

The node serves everything over one JSON-RPC endpoint: `https://rpc.ab.testnet.adifoundation.ai/` for Testnet and `https://rpc.adifoundation.ai/` for Mainnet.

Most of it is plain Ethereum JSON-RPC, so ethers, viem, web3.js and Foundry all work without any special handling. On top of that there are two extra namespaces: `zks` for the rollup-specific bits (bridge addresses, L2->L1 proofs, finality) and `ots` for the Otterscan explorer.

Every call is a normal JSON-RPC 2.0 POST:

```bash
curl -s -X POST -H 'Content-Type: application/json' \
  --data '{"jsonrpc":"2.0","method":"eth_blockNumber","params":[],"id":1}' \
  https://rpc.ab.testnet.adifoundation.ai
```

## What's available

| Namespace | What it covers |
| --------- | -------------- |
| `eth`     | The usual chain / state / transaction methods, log filters, and WebSocket subscriptions. |
| `zks`     | Rollup extras - contract addresses, L2->L1 log proofs, genesis, block metadata, and the [finality API](finality.md). |
| `debug`   | Raw block/header/receipt bytes and transaction/block tracing. |
| `ots`     | Otterscan support: block details, transaction search, contract creator, internal ops. |
| `net`     | Network id. |
| `web3`    | Client version and `keccak256`. |

## eth

Standard Ethereum JSON-RPC, grouped by what you'd use it for.

| Group | Methods |
| ----- | ------- |
| Chain & state | `eth_chainId`, `eth_blockNumber`, `eth_syncing`, `eth_gasPrice`, `eth_maxPriorityFeePerGas`, `eth_feeHistory`, `eth_getBalance`, `eth_getCode`, `eth_getStorageAt`, `eth_getTransactionCount` |
| Blocks & headers | `eth_getBlockByHash`, `eth_getBlockByNumber`, `eth_getBlockReceipts`, `eth_getBlockTransactionCountByHash` / `ByNumber`, `eth_getHeaderByHash` / `ByNumber` |
| Transactions | `eth_getTransactionByHash`, `eth_getTransactionReceipt`, `eth_getTransactionByBlockHashAndIndex`, `eth_getTransactionByBlockNumberAndIndex`, `eth_getTransactionBySenderAndNonce`, `eth_getRawTransactionByHash` |
| Execution | `eth_call`, `eth_estimateGas` |
| Submitting | `eth_sendRawTransaction`, `eth_sendRawTransactionSync` |
| Filters | `eth_newFilter`, `eth_newBlockFilter`, `eth_newPendingTransactionFilter`, `eth_getFilterChanges`, `eth_getFilterLogs`, `eth_getLogs`, `eth_uninstallFilter` |
| Subscriptions (WebSocket) | `eth_subscribe` / `eth_unsubscribe` for `newHeads`, `logs`, `newPendingTransactions`, `syncing` |

The `latest`, `safe`, and `finalized` block tags map onto the rollup's path to L1 - see [Finality](finality.md).

## zks

The rollup-specific methods.

| Method | What it returns |
| ------ | --------------- |
| `zks_getBridgehubContract` | Address of the L1 Bridgehub contract. |
| `zks_getBytecodeSupplierContract` | Address of the bytecode supplier contract. |
| `zks_getL2ToL1LogProof` | Merkle proof for an L2->L1 log, by tx hash and log index. `null` until the batch is executed on L1. |
| `zks_getGenesis` | The chain's genesis input. |
| `zks_getBlockMetadataByNumber` | Extended metadata for an L2 block. |
| `zks_getBlockFinality` | Finality of a block. |
| `zks_getTransactionFinality` | Finality of a transaction. |
| `zks_getBatchFinality` | Finality of an L1 batch. |
| `zks_getFinalityStatus` | Every finality frontier the node tracks, in one call. |

The four finality methods have their own page, with request/response examples - see [Finality](finality.md).

## debug

Transaction and block tracing: `debug_traceBlockByHash`, `debug_traceBlockByNumber`, `debug_traceTransaction`, `debug_traceCall`.

## ots

Otterscan explorer support: `ots_getApiLevel`, `ots_hasCode`, `ots_getHeaderByNumber`, `ots_getBlockDetails`, `ots_getBlockDetailsByHash`, `ots_getBlockTransactions`, `ots_searchTransactionsBefore`, `ots_searchTransactionsAfter`, `ots_getTransactionBySenderAndNonce`.

## net and web3

| Method | Returns |
| ------ | ------- |
| `net_version` | Network (chain) id, as a decimal string. |
| `web3_clientVersion` | Client version string. |
| `web3_sha3` | Keccak-256 of the input data. |
