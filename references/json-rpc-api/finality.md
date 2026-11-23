---
description: >-
  The zks finality methods - how far a block, transaction, or batch has
  progressed through the L1 finality pipeline.
---

# Finality

Blocks move from sealing to commitment and then execution. Use the `zks` finality methods to query a block, transaction, or batch.

| Stage     | JSON          | Meaning                                   | Block tag      |
| --------- | ------------- | ----------------------------------------- | -------------- |
| Pending   | `"pending"`   | sealed by the sequencer, not yet on L1    | `latest`       |
| Committed | `"committed"` | batch data committed to L1, proof pending | `safe`         |
| Executed  | `"executed"`  | proof verified on L1, immutable           | see note below |

{% hint style="warning" %}
`safe` equals `lastCommittedBlock` exactly. `finalized` does **not** equal `lastExecutedBlock`. The difference is not a fixed offset. In a two-minute testnet sample, it widened from 268 to 359 blocks while `finalized` stayed frozen on one height. It appears to track the newest batch whose block linkage has backfilled, then jumps. For the exact stage of a block, transaction, or batch, call the `zks` finality methods. Never infer executed status from the `finalized` tag alone. \[Measured 2026-11-23]&#x20;
{% endhint %}

{% hint style="info" %}
Batch-to-block linkage backfills after execution. It is normal to see `zks_getBatchFinality` return `stage: "executed"` with `blockNumber: null`, and `zks_getBlockFinality` return `stage: "executed"` with `batchNumber: null`, for several minutes. Gate logic on `stage`, never on `blockNumber` or `batchNumber`.
{% endhint %}

`zks_getBlockFinality` returns `null` for a block the node does not have. `zks_getBatchFinality` never returns `null`. Any batch past the committed frontier returns `{stage: "pending", blockNumber: null}`.

## What one block confirms

**Sealing is inclusion, not settlement.** ADI has a single sequencer that seals L2 blocks. Over 200-block samples on each network: median gap 2 s, p95 5 s on mainnet and 13 s on testnet, longest observed gap 108 s. A transaction in a sealed block is included. Nothing about it is final. \[Measured 2026-11-23]&#x20;

**Until a batch executes, treat the block as revisable.** The chain has no cryptographic finality before proof verification on Ethereum. Only `stage: "executed"` is irreversible. Build indexers and caches with a rewind path. Key confirmation logic on the `zks` stage, not block depth.

**How the frontiers behave.** `lastExecutedBlock`, `lastCommittedBlock`, and `lastSealedBlock` are independent. In a 20-minute mainnet sample, the committed and executed frontiers were identical at every read. A distinct committed-but-not-executed state was not observable there. On testnet, the committed frontier ran about 90 blocks ahead of the executed one. Poll `zks_getFinalityStatus` and render all three values. \[Measured 2026-11-23]&#x20;

**Measuring the sealed-to-executed gap.** It varies. Do not publish a fixed duration. Take `lastSealedBlock` and `lastExecutedBlock` from `zks_getFinalityStatus`. Resolve both with `eth_getBlockByNumber`, then compare timestamps. Mainnet batches were sealed roughly hourly. Last-block timestamps of seven consecutive batches were 59.3 to 60.1 minutes apart. Testnet batches were sealed every few minutes. \[Measured 2026-11-23]&#x20;

**Operations that must wait for `executed`:**

* L2 to L1 withdrawals. The claim needs an executed batch. `zks_getL2ToL1LogProof` returns `null` before that.
* Bridge releases and any flow that releases value on Ethereum or another chain.
* Any action you cannot unwind if the block is replaced.
* Recommendation, not policy: crediting deposits against L2 state in a custodial system.

**Operations that may proceed at `latest`:** UI confirmation, non-custodial state changes, and indexing that can be rewound.

**Receipts do not report batches.** `eth_getTransactionReceipt` returns no `l1BatchNumber`, `l1BatchTxIndex`, `l1GasPrice`, or `l1GasUsed`. Use `zks_getTransactionFinality` or `zks_getBlockFinality`.

**Log queries.**  `eth_getLogs` accepts at most 10,000 blocks and 10,000 results per call on both networks. Over-range returns `-32602` with either `query exceeds max block range 10000` or `query exceeds max results 10000, retry with the range <from>-<to>`; the second form gives a range that fits. \[Measured 2026-11-23]

## The response shape

When non-null, `zks_getBlockFinality`, `zks_getTransactionFinality`, and `zks_getBatchFinality` return this object:

| Field         | Type           | Notes                                                                           |
| ------------- | -------------- | ------------------------------------------------------------------------------- |
| `stage`       | string         | `"pending"`, `"committed"`, or `"executed"`.                                    |
| `blockNumber` | number \| null | The L2 block. A batch can return `null` while linkage backfills.                |
| `batchNumber` | number \| null | The L1 batch. A block or transaction can return `null` while linkage backfills. |

## zks\_getBlockFinality

Finality of a block, by number or tag. `null` if the node does not have that block.

```bash
curl -s -X POST -H 'Content-Type: application/json' \
  --data '{"jsonrpc":"2.0","method":"zks_getBlockFinality","params":["0x10"],"id":1}' \
  https://rpc.ab.testnet.adifoundation.ai
```

```json
{"jsonrpc":"2.0","id":1,"result":{"stage":"executed","blockNumber":16,"batchNumber":1}}
```

## zks\_getTransactionFinality

Finality of a transaction, by hash. `null` if the node has not seen it.

```bash
curl -s -X POST -H 'Content-Type: application/json' \
  --data '{"jsonrpc":"2.0","method":"zks_getTransactionFinality","params":["0xabc...def"],"id":1}' \
  https://rpc.ab.testnet.adifoundation.ai
```

```json
{"jsonrpc":"2.0","id":1,"result":{"stage":"committed","blockNumber":42,"batchNumber":null}}
```

## zks\_getBatchFinality

Finality of an L1 batch, by number. This method always returns an object.

```bash
curl -s -X POST -H 'Content-Type: application/json' \
  --data '{"jsonrpc":"2.0","method":"zks_getBatchFinality","params":[3],"id":1}' \
  https://rpc.ab.testnet.adifoundation.ai
```

```json
{"jsonrpc":"2.0","id":1,"result":{"stage":"executed","blockNumber":55,"batchNumber":3}}
```

## zks\_getFinalityStatus

Every frontier the node tracks, in one call.

```bash
curl -s -X POST -H 'Content-Type: application/json' \
  --data '{"jsonrpc":"2.0","method":"zks_getFinalityStatus","params":[],"id":1}' \
  https://rpc.ab.testnet.adifoundation.ai
```

```json
{"jsonrpc":"2.0","id":1,"result":{"lastSealedBlock":368727,"lastCommittedBlock":368691,"lastCommittedBatch":11881,"lastExecutedBlock":368691,"lastExecutedBatch":11881}}
```

| Field                | Meaning                                     | Tag            |
| -------------------- | ------------------------------------------- | -------------- |
| `lastSealedBlock`    | newest block the sequencer sealed           | `latest`       |
| `lastCommittedBlock` | newest block committed to L1                | `safe`         |
| `lastCommittedBatch` | newest batch committed to L1                | not published  |
| `lastExecutedBlock`  | newest block whose proof was verified on L1 | see note above |
| `lastExecutedBatch`  | newest batch whose proof was verified on L1 | not published  |
