---
description: >-
  Explore the functionality of Block Explorer, a comprehensive tool for
  monitoring activities on the ADI Network.
---

# Block explorer

[The Block Explorer](https://explorer.adifoundation.ai/) is a tool designed to provide comprehensive insights into all activities occurring on the ADI Network. This tool is especially useful for users and developers who need to monitor or interact with the blockchain. Block Explorer consists of three main components:

* **Block Explorer App:** This is the user interface that enables users and developers to navigate and examine transactions, blocks, batches, contracts, tokens, and other elements within the ADI Network.
* **Block Explorer API:** This component provides an HTTP API for accessing structured data from the ADI Network. It retrieves data from the database maintained by the Block Explorer Worker.
* **Alternative Block Explorer URL:** [https://explorer-bls.adifoundation.ai/](https://explorer-bls.adifoundation.ai/)
* **Testnet Explorer API:** [https://explorer-api.ab.testnet.adifoundation.ai/](https://explorer-api.ab.testnet.adifoundation.ai/)

### Explorer APIs

Operational details in this section were measured 2026-11-23.

| Network                  | UI                                             | API base                                                  | API style                                                                                                                                                                                              |
| ------------------------ | ---------------------------------------------- | --------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| ADI Mainnet              | `https://explorer.adifoundation.ai`            | `https://explorer-api.adifoundation.ai`                   | Custom REST: `/blocks`, `/transactions`, `/batches`, `/address/{address}`, `/tokens`, `/stats`. Swagger at `/docs`, OpenAPI at `/docs-json`. Also an Etherscan-compatible subset at `/api?module=...`. |
| ADI Mainnet (Blockscout) | `https://explorer-bls.adifoundation.ai`        | `https://explorer-bls.adifoundation.ai/api` and `/api/v2` | Blockscout v12.0.4: Etherscan-compatible plus Blockscout v2.                                                                                                                                           |
| ADI Network AB Testnet   | `https://explorer.ab.testnet.adifoundation.ai` | `https://explorer-api.ab.testnet.adifoundation.ai`        | Same custom REST API as mainnet. No Blockscout `/api/v2` on this host.                                                                                                                                 |

**The testnet explorer is not Blockscout.**  `GET /api/v2/blocks` returns 404. Use `/blocks` or the Etherscan-style `/api?module=...` endpoints. (Measured 2026-11-23)

**Indexing delay.** The custom API was 0 to 2 blocks behind the RPC tip. Blockscout exposes its own lag. `GET https://explorer-bls.adifoundation.ai/api/health` returns `metadata.blocks.db.number`, `metadata.blocks.cache.number`, and `metadata.blocks.latest_block.node.number`. Compare `db.number` with `eth_blockNumber` to compute indexer lag. (Measured 2026-11-23)

**Health endpoints.**

* Custom API, mainnet and testnet: `GET /health` and `GET /ready`. Both return `{"status":"ok","info":{"database":{"status":"up"}}}`.
* Blockscout: `GET /api/health` returns `healthy` plus the node, database, and cache heights above. `GET /api/v2/health` returns 400 and is not a health check.
* RPC: there is no `/health` endpoint. Use `eth_blockNumber` or `zks_getFinalityStatus`.

**Rate limits and availability.** No rate limit or availability target is published for these endpoints. Blockscout returns `x-ratelimit-limit: -1`, and 100-request bursts against both APIs completed without a 429. This is an observation, not a guarantee. Cache responses, use conditional requests, and back off on 5xx. (Measured 2026-11-23)

**Pagination.**

* Custom API: `?page=N&limit=M`.  `limit` must not exceed 100. `limit=101` returns 400 `limit must not be greater than 100`. (Measured 2026-11-23)
* Etherscan-compatible subset: `page` × `offset` must not exceed 1000. Otherwise it returns `Result window is too large, PageNo x Offset size must be less than or equal to 1000`. (Measured 2026-11-23)
* Blockscout v2: follow `next_page_params`. `items_count` caps at 100. Larger values are silently clamped. (Measured 2026-11-23)

**Status.** `GET /stats` returns `lastSealedBlock`, `lastVerifiedBlock`, and `totalTransactions`. It is enough to plot tip lag and finality lag.

{% hint style="warning" %}
The explorer is a convenience indexer. For deposits, withdrawals, settlement, and any value-bearing decision, confirm through RPC: `eth_getLogs` with at most 10,000 blocks per call, `eth_getTransactionReceipt`, and `zks_getTransactionFinality`. Require `stage: "executed"` before releasing value. An explorer can lag, be rebuilt, or disagree with the node.
{% endhint %}

See [finality.md](../references/json-rpc-api/finality.md "mention") for finality stages. See [verify-smart-contracts.md](verify-smart-contracts.md "mention") for verification workflows.
