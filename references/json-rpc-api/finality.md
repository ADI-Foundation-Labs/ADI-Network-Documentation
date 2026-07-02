---
description: >-
  The zks finality methods - how far a block, transaction, or batch has
  progressed through the L1 finality pipeline.
---

# Finality

Blocks don't become final the moment they're produced - they're sealed by the sequencer, later committed to L1, and finally executed on L1. The `zks` finality methods let you ask exactly where something sits in that journey, whether you're pointing at a block, a transaction, or a whole batch.

The three stages, least to most final:

| Stage | JSON | Meaning | Matches tag |
| ----- | ---- | ------- | ----------- |
| Pending   | `"pending"`   | sealed by the sequencer, not yet on L1 | `latest` |
| Committed | `"committed"` | committed to L1 | `safe` |
| Executed  | `"executed"`  | executed on L1 - fully final | `finalized` |

These stages are exactly the `latest`, `safe`, and `finalized` block tags you pass to `eth` methods like `eth_getBlockByNumber`.

{% hint style="info" %}
A batch and its blocks can only be linked once the batch is **executed** on L1 (the same rule `zks_getL2ToL1LogProof` follows). So `batchNumber` on a block/transaction lookup, and `blockNumber` on a batch lookup, stay `null` until execution - then they fill in.
{% endhint %}

## The response shape

`zks_getBlockFinality`, `zks_getTransactionFinality` and `zks_getBatchFinality` all return the same object:

| Field | Type | Notes |
| ----- | ---- | ----- |
| `stage` | string | `"pending"`, `"committed"` or `"executed"`. |
| `blockNumber` | number \| null | The L2 block - the one you asked for, the transaction's block, or (for a batch) the batch's last block. Only filled in for a batch once it's executed. |
| `batchNumber` | number \| null | The L1 batch the block/transaction landed in, or the batch you asked about. Only filled in for a block/transaction once it's executed. |

## zks\_getBlockFinality

Finality of a block, by number or tag. `null` if the node doesn't have that block.

```bash
curl -s -X POST -H 'Content-Type: application/json' \
  --data '{"jsonrpc":"2.0","method":"zks_getBlockFinality","params":["0x10"],"id":1}' \
  https://rpc.ab.testnet.adifoundation.ai
```

```json
{"jsonrpc":"2.0","id":1,"result":{"stage":"executed","blockNumber":16,"batchNumber":1}}
```

## zks\_getTransactionFinality

Finality of a transaction, by hash. `null` if the node hasn't seen it.

```bash
curl -s -X POST -H 'Content-Type: application/json' \
  --data '{"jsonrpc":"2.0","method":"zks_getTransactionFinality","params":["0xabc...def"],"id":1}' \
  https://rpc.ab.testnet.adifoundation.ai
```

```json
{"jsonrpc":"2.0","id":1,"result":{"stage":"committed","blockNumber":42,"batchNumber":null}}
```

The block is committed but not executed yet, which is why `batchNumber` is still `null`.

## zks\_getBatchFinality

Finality of an L1 batch, by number. Anything past the committed frontier comes back as `pending`. This one always returns an object - never `null`.

```bash
curl -s -X POST -H 'Content-Type: application/json' \
  --data '{"jsonrpc":"2.0","method":"zks_getBatchFinality","params":[3],"id":1}' \
  https://rpc.ab.testnet.adifoundation.ai
```

```json
{"jsonrpc":"2.0","id":1,"result":{"stage":"executed","blockNumber":55,"batchNumber":3}}
```

Once the batch is executed, `blockNumber` is its last L2 block.

## zks\_getFinalityStatus

Every frontier the node is tracking, in a single call. Handy for a dashboard or a sync check - no params needed.

```bash
curl -s -X POST -H 'Content-Type: application/json' \
  --data '{"jsonrpc":"2.0","method":"zks_getFinalityStatus","params":[],"id":1}' \
  https://rpc.ab.testnet.adifoundation.ai
```

```json
{"jsonrpc":"2.0","id":1,"result":{"lastSealedBlock":368727,"lastCommittedBlock":368691,"lastCommittedBatch":11881,"lastExecutedBlock":368691,"lastExecutedBatch":11881}}
```

| Field | Meaning | Tag |
| ----- | ------- | --- |
| `lastSealedBlock` | newest block the sequencer sealed | `latest` |
| `lastCommittedBlock` | newest block committed to L1 | `safe` |
| `lastCommittedBatch` | newest batch committed to L1 | - |
| `lastExecutedBlock` | newest block whose batch executed on L1 | `finalized` |
| `lastExecutedBatch` | newest batch executed on L1 | - |
