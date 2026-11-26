---
description: How to run an ADI external node, monitor its sync, and keep it current.
---

# Run Your Own Node

The [ADI Stack Setup repository](https://github.com/ADI-Foundation-Labs/ADI-Stack-EN-Setup-script) provides a one-command setup for running an ADI external node (an L2 replay follower). The stack is a single `external_node` service.

* `external_node`: replays L2 blocks from the canonical main node over devp2p, maintains local state, and serves JSON-RPC, a status server, and Prometheus metrics.

{% hint style="info" %}
What it does **not** do:

* It does not generate proofs, validate batches, or participate as a sequencer. It replays and serves state for your own queries. External nodes hold no proof artifacts: since the `v0.13.0 → v0.20.12` upgrade moved replay to devp2p, there is no proof-sync sidecar and no proof storage to populate.
* It does not sequence or produce blocks. That is the sequencer's role.

What it **does** do for transactions:

* It accepts `eth_sendRawTransaction` calls and adds transactions to its local mempool.
* It **forwards transactions to the main node/sequencer** through `general_main_node_rpc_url`.
* If the sequencer rejects a transaction, the node removes it from its mempool and returns the error. You can submit transactions to any external node and they will reach the network.
{% endhint %}

### Requirements

* Docker with the `docker compose` plugin.
* An archive-capable Ethereum L1 RPC endpoint. The node reads L1 logs and upgrade history, and from `v0.24.2` it also scans `eth_getLogs` from L1 block 0 on every start, so a pruned endpoint fails.
* Disk space for `<chain>_data` (RocksDB state and the local replay WAL).

### Network selection

* Default network: `mainnet`.
* Prefer CLI flags: `--testnet` or `--network <name>`. You may export `NETWORK=testnet` instead, but the flag avoids lingering env state.
* Network-specific defaults (main-node RPC, boot nodes, container prefix, data directory) are resolved by the script. You do not swap compose files by hand.

### Quickstart (mainnet)

#### Single command

```bash
./external-node.sh start --l1-rpc-url <your-archive-l1-rpc>
```

#### Stepwise

1. Export your archive L1 RPC URL:

   ```bash
   export GENERAL_L1_RPC_URL="<your-archive-l1-rpc>"
   ```

2. Start the stack:

   ```bash
   ./external-node.sh start
   ```

3. Watch logs (optional):

   ```bash
   ./external-node.sh logs
   ```

4. Compare your node's tip with the canonical RPC. `$ADI_RPC_URL` is `https://rpc.adifoundation.ai` on mainnet and `https://rpc.ab.testnet.adifoundation.ai` on testnet:

   ```bash
   curl -s -X POST -H 'Content-Type: application/json' \
     --data '{"jsonrpc":"2.0","method":"eth_blockNumber","params":[],"id":1}' \
     http://localhost:3050
   curl -s -X POST -H 'Content-Type: application/json' \
     --data '{"jsonrpc":"2.0","method":"eth_blockNumber","params":[],"id":1}' \
     $ADI_RPC_URL
   ```

   Both hex values should stay within a few blocks. A fresh node replays from genesis and takes hours to converge; a flat head is expected until it catches up.

### Quickstart (testnet)

```bash
./external-node.sh --testnet start --l1-rpc-url <your-archive-l1-rpc>
```

### Node identity

Both values below are optional and are generated or defaulted by the script, but you should pin the key.

* `--external-network-secret-key` (or `EXTERNAL_NETWORK_SECRET_KEY`): the node's P2P identity. If omitted, `start` generates one with `openssl rand -hex 32`, prints it, and uses it. Save that value and pass it on every restart. A new key means a new identity, and the node has to be treated as a new peer. To recover a key you did not save from a running container:

  ```bash
  docker inspect adi_mainnet_external_node \
    --format '{{range .Config.Env}}{{println .}}{{end}}' | grep network_secret_key
  ```

* `--boot-node-urls` (or `BOOT_NODE_URLS`): comma-separated `enode://<pubkey>@<host>:3060` URLs. Falls back to the per-network default when omitted. The key may carry a `0x` prefix.

### Transaction forwarding

The compose setting `general_main_node_rpc_url` controls the forwarding target and defaults to the ADI sequencer endpoint. When unset, the node accepts transactions into its local mempool but does not forward them, so they stay local.

### Monitoring

There are three surfaces, all read-only.

**Status server, port 3071**

| Endpoint                | What it tells you                                                                   |
| ----------------------- | ----------------------------------------------------------------------------------- |
| `GET /status/health`    | Liveness. Returns `{"healthy":true}` with 200 whenever the process is up. It does not mean the node is in sync. |
| `GET /status/ready`     | Readiness. Returns 200 once startup work completes, 503 before that.                 |
| `GET /status`           | Node status, including consensus state.                                              |
| `GET /status/pipeline`  | JSON array of every pipeline component with its state and the lag to its neighbour. The most useful signal for a slow node. |

```bash
curl -s http://localhost:3071/status/health
curl -s http://localhost:3071/status/pipeline
```

**JSON-RPC, port 3050**

```bash
# false means caught up with the sequencer
curl -s -X POST http://localhost:3050 -H 'Content-Type: application/json' \
  -d '{"jsonrpc":"2.0","method":"eth_syncing","params":[],"id":1}'

# compare against the canonical RPC to measure lag in blocks
curl -s -X POST http://localhost:3050 -H 'Content-Type: application/json' \
  -d '{"jsonrpc":"2.0","method":"zks_getFinalityStatus","params":[],"id":1}'
```

**Prometheus metrics, port 3312**

`GET http://localhost:3312/metrics` serves Prometheus text format. Useful families:

* `network.connected_peers` and the other `network.*` gauges. One peer is normal: replay traffic runs over a session to the main node, not a full mesh.
* `block_replay_storage_get_latency_seconds` and `block_replay_storage_set_latency_seconds` for replay-record I/O.
* `l1_sender_*`, `rocksdb_*`, and `zk_os_merkle_tree_*` for the rest of the pipeline.

Logs remain the ground truth for replay progress. Look for replay requests and persisted blocks:

```bash
docker logs --tail 200 adi_mainnet_external_node 2>&1 | grep -Ei 'replay|persisted block|panicked'
```

Judge sync by `eth_blockNumber` against the reference RPC, never by L1 watcher lines. `zksync_os_l1_watcher` messages describe L1's view of executed batches, not how far this node has replayed L2.

### Data locations

* `CHAIN_DATA_DIR` defaults to `./mainnet_data` or `./testnet_data` and is overridable via env. It is mounted at `/chain` in the container.
* RocksDB state lives under `chain_data/db/node1/`.
* The replay WAL is stored alongside it. Deleting the directory forces a full resync from genesis.

### Common issues

* **Pruned L1 RPC**: the node panics at startup with `state at block is pruned`. Use an archive endpoint.
* **`eth_getLogs` refused on startup**: from `v0.24.2` the interop-roots watcher reads L1 logs from block 0 in ranges of 1,000 blocks with no pause. Free or heavily rate-limited endpoints fail this. Set `l1_watcher_max_blocks_to_process` to use smaller ranges, at the cost of more calls.
* **Permissions**: the host data directory must be writable. The script's `ensure_container_dir` sets permissive modes for you.
* **Behind on replay**: keep waiting. Compare `eth_blockNumber` with the reference RPC and check that the number is still moving.
* **Container name**: `adi_<network>_external_node`, or `${CONTAINER_PREFIX}_external_node` if you set `CONTAINER_PREFIX`.

### Ports

| Port           | Service            | Notes                                 |
| -------------- | ------------------ | ------------------------------------- |
| `3050`         | JSON-RPC           | `rpc_address`                         |
| `3060` TCP+UDP | P2P devp2p         | peer sessions and block replay        |
| `3071`         | Health / status    | `status_server_address`               |
| `3312`         | Prometheus metrics | `observability_prometheus_port`       |

Port `3060` must be open for both TCP and UDP inbound.

### Managing the node

```bash
./external-node.sh status   # compose service status
./external-node.sh logs     # follow logs
./external-node.sh stop     # stop, containers preserved
./external-node.sh down     # stop and remove containers
./external-node.sh pull     # pull the image for the pinned version
```

`stop` preserves the containers and `down` removes them. Neither touches the data directory.

### Keeping the node current

External node upgrades are coordinated, network-wide events. Nodes on different versions do not peer with each other, and a node that has stored blocks from a newer main node cannot go back to an older version. Check for updates periodically, then follow the upgrade procedure. Do not upgrade on the strength of a repository tag alone.

#### Check what exists

```bash
# What this checkout would deploy, and what upstream now pins
cd ~/ADI-Stack-EN-Setup-script && git fetch --quiet origin
grep -o 'EN_VERSION:-[^}]*' docker-compose.mainnet.yml
git show origin/main:docker-compose.mainnet.yml | grep -o 'EN_VERSION:-[^}]*'
git log --oneline HEAD..origin/main        # empty when the checkout is current

# What this host runs
docker ps --format '{{.Names}}\t{{.Image}}' | grep external_node

# What the cluster runs, as a live signal that it has moved
curl -s -X POST https://rpc.adifoundation.ai -H 'Content-Type: application/json' \
  -d '{"jsonrpc":"2.0","method":"web3_clientVersion","params":[],"id":1}'
```

Read the working-tree compose file **and** `origin/main`. They differ whenever your checkout is behind, and a stale checkout reports an old version as though it were current. Image tags are also readable from the registry without an account:

```bash
TOKEN=$(curl -s 'https://harbor.sde.adifoundation.ai/service/token?service=harbor-registry&scope=repository:adi-public/chain/server:pull' | sed -E 's/.*"token":"([^"]+)".*/\1/')
curl -s -H "Authorization: Bearer $TOKEN" \
  'https://harbor.sde.adifoundation.ai/v2/adi-public/chain/server/tags/list?n=100'
```

A published tag newer than your running image means an upgrade exists. Images are often published before the setup repo pins them, and `latest` must never be used for an external node.

The main node's `web3_clientVersion` and the external node's image tag are numbered on different tracks: at the time of writing, mainnet answers `zksync-os/v0.21.1` while the external node image is `v0.24.2-b7`. Compare `major.minor.patch` only and trust the repository pin for what the external node should actually run.

#### Read the release before you pull

Announcements land in GitHub releases on `ADI-Foundation-Labs/ADI-Stack-Server` and `ADI-Foundation-Labs/ADI-Stack-EN-Setup-script`, in the [changelog](../references/changelog.md), and in the ADI Discord. Operators additionally receive email and Telegram notices. The changelog entry and the matching guide under `upgrades/` in the setup repository are the authoritative instructions; the checklist below is the shape, not a substitute.

```bash
gh release list -R ADI-Foundation-Labs/ADI-Stack-Server --limit 5
git -C ~/ADI-Stack-EN-Setup-script log --oneline -5 origin/main
```

#### Upgrade procedure

1. **Confirm the main node has moved first, unless the release says otherwise.** Some releases allow upgrading external nodes early, and the guide for that version says so explicitly. If it does not, stop here.
2. **Record the current state**: the image tag from `docker ps`, the output of `./external-node.sh status`, and your network secret key.
3. **Stop the node**: `./external-node.sh stop`.
4. **Update the checkout**: `git pull origin main` in `~/ADI-Stack-EN-Setup-script`, then `./external-node.sh pull`.
5. **Recreate with the same key**: `./external-node.sh start --l1-rpc-url <url> --external-network-secret-key <key>`. Compose files are the source of truth for any environment changes between versions. Do not hand-edit the version; the pin is a repository change.
6. **Verify**: RPC answers on 3050, `eth_blockNumber` advances and closes on the reference RPC, `network.connected_peers` is at least 1, and the logs contain no `verifier authorization failures` and no panics.

#### Roll back

If the node will not come back, stop it, `git checkout <previous ref>` in the setup repository, `pull`, and start again with the same secret key. The data directory and key are untouched, so this returns the node to its prior state. This works only before the node has stored blocks from a newer main node, which is why the upgrade order matters.

### Related

* [Changelog](../references/changelog.md) for the release history and deployed versions.
* [JSON-RPC API](../references/json-rpc-api/README.md) and [Finality](../references/json-rpc-api/finality.md) for what the node serves.
