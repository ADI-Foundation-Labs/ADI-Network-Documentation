# Run Your Own Node

The [ADI Stack Setup repository](https://github.com/ADI-Foundation-Labs/ADI-Stack-EN-Setup-script) provides a one-command setup for running an ADI (external) node (ZK rollup follower) plus two helper sidecars:

* `external_node`: replays L2 blocks from the canonical main node, maintains local state/RPC, and serves JSON-RPC + status/metrics.
* `cloudflared-tcp-proxy`: forwards block replay data to the node.
* `proof-sync`: periodically syncs proving artifacts from the foundation-hosted Azure Blob storage into your local `<chain>_data/db/shared`.

{% hint style="warning" %}
What it does **not** do:&#x20;

* It does not generate new proofs or participate as a validator/sequencer. It locally replays and verifies the chain state and serves RPC for your own queries. Proofs are downloaded, not produced.
* It does not enable passing new transactions to the network currently, serving only as a read-only node
{% endhint %}

### Requirements

* Docker + Docker Compose.
* An archive-capable Ethereum L1 RPC endpoint (required to fetch historical state for genesis/upgrade discovery).
* Sufficient disk space for `<chain>_data` (RocksDB + synced proofs).

### Network Selection

* Default network: `mainnet`.
* Prefer CLI flags: `--testnet` / `--network testnet`. You may export `NETWORK=testnet` if you like, but the flag avoids lingering env state.
* Network-specific defaults (L2 RPC, replay host, proof storage URL, container prefix, data dir) are injected automatically; no need to swap compose files.

### Quickstart (mainnet)

#### Single Command

1.  Start the stack:

    ```bash
    ./external-node.sh start --l1-rpc-url <your-archive-l1-rpc>
    ```

#### Stepwise

1.  Export an archive L1 RPC URL:

    ```bash
    export GENERAL_L1_RPC_URL="<your-archive-l1-rpc>"
    ```
2.  (Optional) Prefetch proofs to speed first start:

    ```bash
    ./external-node.sh download
    ```

    If you skip this, the `proof-sync` sidecar will pull proofs automatically after start.
3.  Start the stack:

    ```bash
    ./external-node.sh start
    ```
4.  Watch logs (optional):

    ```bash
    ./external-node.sh logs
    ```
5.  Check RPC tip matches the canonical RPC (requires `ADI_RPC_URL`):

    ```bash
    curl -s -X POST -H 'Content-Type: application/json' \
      --data '{"jsonrpc":"2.0","method":"eth_blockNumber","params":[],"id":1}' \
      http://localhost:3050
    curl -s -X POST -H 'Content-Type: application/json' \
      --data '{"jsonrpc":"2.0","method":"eth_blockNumber","params":[],"id":1}' \
      $ADI_RPC_URL
    ```

    When both hex values match, you are synced.

### Quickstart (testnet)

Testnet may upgrade frequently; join [Discord](https://discord.gg/adi-foundation) for announcements.

#### Single Command

1.  Start with testnet flag:

    ```bash
    ./external-node.sh --testnet start --l1-rpc-url <your-archive-l1-rpc>
    ```

    (Optional) If you prefer envs: `export NETWORK=testnet` then run the same commands without the flag.

#### Stepwise

1.  Provide an archive L1 RPC URL:

    ```bash
    export GENERAL_L1_RPC_URL="<your-archive-l1-rpc>"
    ```
2.  (Optional) Prefetch proofs (respects the selected network):

    ```bash
    ./external-node.sh --testnet download
    ```
3.  Start:

    ```bash
    ./external-node.sh --testnet start
    ```

### Monitoring

* JSON-RPC: `http://localhost:3050`
  * `eth_syncing` → should return `false` when caught up.
  * `eth_blockNumber` → compare with the ADI RPC.
* Metrics: `http://localhost:3312/metrics` (Prometheus text format). Search for `replay`, `last_block`, `state_block_range`, or `tree_last_block` to infer replay progress.
* Logs: look for `Replay block <n>` lines; once they stop and `eth_blockNumber` matches, the node is at tip.

### Data Locations

* `CHAIN_DATA_DIR` defaults per network (`./mainnet_data` or `./testnet_data`); overridable via env. Mapped to `/chain` in the containers.
* Proofs: `chain_data/db/shared` (synced by `proof-sync`).
* RocksDB state: `chain_data/db/node1/...`.

### Common Issues

* **Pruned L1 RPC**: startup panics with `... state at block is pruned ...` then use an archive L1 RPC.
* **Permissions**: ensure the host data directory is writable (the script `ensure_container_dir` sets permissive permissions).
* **Behind on replay**: allow time to catch up (monitor via `eth_blockNumber` and logs).

### What This Node Provides You

* A self-hosted, read-only ADI L2 RPC endpoint, backed by locally replayed and verified state.
* Locally stored proofs (synced from the foundation blob), not generated locally.
* No participation in sequencing, proving, or validator duties; it is for verification, data availability, and private querying.
