---
description: How to run a read-only ADI external node and monitor sync.
---

# Run Your Own Node

{% hint style="success" %}
External node version v0.19.0-b3 has been released on 3-June-2026

_It is backwards compatible with previous versions and contains minor fixes that improve resilience and stability when running against reth Ethereum nodes_
{% endhint %}

The [ADI Stack Setup repository](https://github.com/ADI-Foundation-Labs/ADI-Stack-EN-Setup-script) provides a one-command setup for running an ADI (external) node (ZK rollup follower):

* `external_node`: replays L2 blocks via P2P from peers, maintains local state/RPC, and serves JSON-RPC + status/metrics.

{% hint style="warning" %}
What it does **not** do:

* It does not generate new proofs or participate as a validator/sequencer. It locally replays and verifies the chain state and serves RPC for your own queries.
* It does not enable passing new transactions to the network currently, serving only as a read-only node
{% endhint %}

### Requirements

* Docker Engine with the `docker compose` plugin (or the legacy `docker-compose` binary).
* An archive-capable Ethereum L1 RPC endpoint (required to fetch historical state for genesis/upgrade discovery).
* Sufficient disk space for `<chain>_data` (RocksDB state).

### Firewall Ports

The following ports must be open on your host:

| Port           | Service            | Notes                                                    |
|----------------|--------------------|----------------------------------------------------------|
| `3060` TCP+UDP | P2P devp2p         | Peer discovery and block propagation (**required**)      |
| `3050`         | JSON-RPC           | `rpc_address`                                            |
| `3071`         | Health / status    | `status_server_address`                                  |
| `3312`         | Prometheus metrics | `observability_prometheus_port`                          |

Ensure port `3060` is open for both TCP and UDP inbound traffic — the node cannot connect to peers without it.

### Network Selection

By default, the script runs on **mainnet**. Use the `--testnet` flag to run on testnet:

```bash
# Mainnet (default)
./external-node.sh start --l1-rpc-url https://your-l1-endpoint

# Testnet
./external-node.sh --testnet start --l1-rpc-url https://your-l1-endpoint
```

Network-specific defaults (RPC URL, data directory, container prefix) are injected automatically.

| Network | Main RPC                                    | Data Directory    |
|---------|---------------------------------------------|-------------------|
| Mainnet | `https://rpc.adifoundation.ai`              | `./mainnet_data`  |
| Testnet | `https://rpc.ab.testnet.adifoundation.ai`   | `./testnet_data`  |

### Network Identity Setup

Every external node requires a secret key for P2P identity and a list of boot nodes for peer discovery. Complete this setup before starting the node.

#### 1. Generate the secret key (optional)

If `--external-network-secret-key` is not provided, the script generates one automatically. To keep a stable P2P identity across restarts, generate and save the key yourself:

```bash
openssl rand -hex 32
```

Keep this value private — it uniquely identifies your node in the P2P network.

```bash
export EXTERNAL_NETWORK_SECRET_KEY="<64-hex-chars>"
```

#### 2. Configure boot nodes (optional)

Boot nodes are already pre-configured per network inside the script. Use `BOOT_NODE_URLS` only to override the defaults. Format:

```
enode://<public-key>@<ip-or-host>:<port>
```

* `public-key` — 128 hex chars, no `0x` prefix
* `ip-or-host` — publicly reachable address of the boot node
* `port` — default `3060`

```bash
# Single boot node
export BOOT_NODE_URLS="enode://abcd1234...@1.2.3.4:3060"

# Multiple boot nodes (comma-separated)
export BOOT_NODE_URLS="enode://key1@host1:3060,enode://key2@host2:3060"
```

### Quickstart (mainnet)

#### Single Command

```bash
./external-node.sh start --l1-rpc-url <your-archive-l1-rpc>
```

#### Stepwise

1.  Export an archive L1 RPC URL:

    ```bash
    export GENERAL_L1_RPC_URL="<your-archive-l1-rpc>"
    ```
2.  Start the stack:

    ```bash
    ./external-node.sh start
    ```
3.  Watch logs (optional):

    ```bash
    ./external-node.sh logs
    ```
4.  Check RPC tip matches the canonical RPC:

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

```bash
./external-node.sh --testnet start --l1-rpc-url <your-archive-l1-rpc>
```

#### Stepwise

1.  Provide an archive L1 RPC URL:

    ```bash
    export GENERAL_L1_RPC_URL="<your-archive-l1-rpc>"
    ```
2.  Start:

    ```bash
    ./external-node.sh --testnet start
    ```

### Monitoring

* JSON-RPC: `http://localhost:3050`
  * `eth_syncing` → should return `false` when caught up.
  * `eth_blockNumber` → compare with the ADI RPC.
* Health check: `curl http://localhost:3071`
* Metrics: `http://localhost:3312/metrics` (Prometheus text format). Search for `replay`, `last_block`, `state_block_range`, or `tree_last_block` to infer replay progress.
* Logs: look for `Replay block <n>` lines; once they stop and `eth_blockNumber` matches, the node is at tip. A healthy node will show peer connections appearing in the logs within a few minutes of startup.

### Node Management

All commands support the `--testnet` flag for testnet operation.

```bash
# Check container status
./external-node.sh status

# Follow live logs
./external-node.sh logs

# Stop containers (data is preserved)
./external-node.sh stop

# Stop and remove containers (data is preserved)
./external-node.sh down

# Pull the latest image
./external-node.sh pull
```

### Data Locations

* `CHAIN_DATA_DIR` defaults per network (`./mainnet_data` or `./testnet_data`); overridable via env. Mapped to `/chain` in the containers.
* RocksDB state: `chain_data/db/node1/...`.

### Environment Variable Reference

| Variable                      | Required | Description                                                                                          |
|-------------------------------|----------|------------------------------------------------------------------------------------------------------|
| `GENERAL_L1_RPC_URL`          | Yes      | Ethereum L1 RPC endpoint                                                                             |
| `EXTERNAL_NETWORK_SECRET_KEY` | No       | P2P node identity key — auto-generated if not set; provide to keep a stable identity across restarts |
| `BOOT_NODE_URLS`              | No       | Override default boot node enode URLs (comma-separated)                                              |
| `CHAIN_DATA_DIR`              | No       | Host path for blockchain data (default: `./<network>_data`)                                          |
| `EN_VERSION`                  | No       | Docker image version override                                                                        |
| `DOCKER_COMPOSE_FILE`         | No       | Path to a custom docker-compose file                                                                 |

### Common Issues

* **Pruned L1 RPC**: startup panics with `... state at block is pruned ...` — use an archive L1 RPC.
* **Permissions**: ensure the host data directory is writable (the script sets permissive permissions automatically).
* **Behind on replay**: allow time to catch up (monitor via `eth_blockNumber` and logs).
* **No peer connections**: ensure port `3060` is open for both TCP and UDP traffic.

### What This Node Provides You

* A self-hosted, read-only ADI L2 RPC endpoint, backed by locally replayed and verified state.
* No participation in sequencing, proving, or validator duties; it is for verification, data availability, and private querying.