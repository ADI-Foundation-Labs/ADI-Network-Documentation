---
description: >-
  Release history for the ADI Server - the node behind ADI's sequencer and
  external nodes.
---

# Changelog

### Deployed versions

| Component      | Mainnet version | Testnet version |
| -------------- | --------------- | --------------- |
| Protocol       | `v0.30.1`       | `v0.30.1`       |
| Sequencer      | `v0.21.1-b4`    | `v0.21.1-b4`    |
| External nodes | `v0.21.1-b4`    | `v0.21.1-b4`    |

### Release notes

## [v0.21.1-b4](https://github.com/ADI-Foundation-Labs/ADI-Stack-Server/releases/tag/v0.21.1-b4)

_2026-09-07 - latest_

Current sequencer and external node version on testnet and mainnet, superseding `v0.13.0-b4`. This covers everything ADI changed across the `v0.13` -> `v0.21` line, plus the one upstream change that every node operator has to act on: block replay moved off HTTP onto devp2p.

**Breaking changes**

* **network** - block replay no longer runs over HTTP. External nodes now sync over [devp2p](https://github.com/ethereum/devp2p) on port `3060` (TCP for RLPx, UDP for peer discovery) instead of pulling replay records from the sequencer's HTTP replay server on port `3053`. An external node left on the HTTP-replay build stops syncing and cannot peer with the network.
  * Removed config: `sequencer_block_replay_download_address`, `sequencer_block_replay_server_enabled`, `sequencer_block_replay_server_address`.
  * New required config on external nodes: `general_node_role: external`.
  * The cluster needs at least one node reachable at a stable address for ENs to peer against, and networking must be enabled on main and external nodes alike.
  * See the [v0.13.0 -> v0.20.12 migration guide](https://github.com/ADI-Foundation-Labs/ADI-Stack-EN-Setup-script/blob/main/upgrades/v0.13.0_to_v0.20.12.md) for the full rollout.

**Features**

* **network** - resolve hostnames in `enode` URLs, so boot nodes can be configured by DNS name instead of a hardcoded IP (#16).
* **network** - raise the P2P active-connection cap from 25 to 100, apply it to inbound connections, and disable peer rotation, so a growing external-node fleet is not churned off the sequencer.
* **l1** - retry L1 requests in the L1 sender and in the commit and execute watchers, instead of failing the operation on the first transient RPC error (#17).
* **rocksdb** - cap open files at 4096 and move SST index and bloom filters into a sized block cache (2 GiB repository, 2 GiB state, 1 GiB preimages) on read-heavy databases. The previous defaults pinned every SST file's index and filter in RAM permanently, so memory grew without bound as the database grew.
* **memory** - use jemalloc as the global allocator. On mainnet, app plus allocator overhead measured ~22 GiB on the glibc build versus ~3 GiB on the jemalloc build under the same database and workload, because jemalloc returns freed memory to the OS.
* **prover** - support circuit-soundness proving versions V8 (protocol v30.2) and V9 (protocol v31.1).

**Fixes**

* **network** - an external node whose replay connection goes quiet now disconnects and re-requests instead of stalling indefinitely. The tolerated silence is configurable via `en_replay_inactivity_timeout` (default 300s); it must comfortably exceed the chain's block cadence, so raise it on chains that can legitimately idle for longer (#1513).
* **sequencer** - apply runtime fee overrides before the mempool reads the base fee, so an override takes effect on the next block instead of one block late.
* **rpc(fees)** - propagate a config-override base fee to every fee-exposing RPC endpoint, derive `native_price` from the base fee when it is not set explicitly, and default `pubdata_price` to `0` unless overridden, so the base fee covers all additional cost.
* **rpc(estimation)** - `eth_estimateGas` caps the gas limit by what the sender can afford, but divided the balance by the block base fee while validation charges the effective gas price. Any request carrying a priority fee therefore produced an unaffordable cap and failed with `LackOfFundForMaxFee` for every account below `fee_cap * block_gas_limit`. The allowance is now divided by the request's fee cap, which is affordable by construction.
* **merkle\_tree** - handle empty batch proofs instead of failing on them (#1535).
* **rpc** - log config-override messages at debug instead of info.

**Images**

* `harbor.sde.adifoundation.ai/adi-public/chain/server:v0.21.1-b4`

## [v0.13.0-b4](https://github.com/ADI-Foundation-Labs/ADI-Stack-Server/releases/tag/v0.13.0-b4)

_2026-06-30_

L1 watcher hardening.

**Fixes**

* **l1-watcher** - leave 2 confirmation blocks unprocessed, matching the upgrade watcher's behavior, to avoid `block range extends beyond current head block` errors.
* **tests** - add the missing private API server config and bind it to a dynamically allocated port.

**Images**

* `harbor.sde.adifoundation.ai/adi-public/chain/external-node@v0.13.0-b4`
* `harbor.sde.adifoundation.ai/adi-public/chain/server@v0.13.0-b4`

## [v0.13.0-b3](https://github.com/ADI-Foundation-Labs/ADI-Stack-Server/releases/tag/v0.13.0-b3)

_2026-06-11_

The `zks` finality RPC and ADI-native CI/CD.

**Features**

* **rpc(finality)** - add `zks_getBlockFinality`, `zks_getTransactionFinality`, `zks_getBatchFinality`, and `zks_getFinalityStatus`, reporting how far a block, transaction, or batch has moved through L1 finality (`pending` -> `committed` -> `executed`), plus a single-call snapshot of every finality frontier. See the [JSON-RPC API](https://github.com/ADI-Foundation-Labs/ADI-Network-Documentation/blob/main/references/json-rpc-api.md#finality).

**CI/CD**

* Switch to ADI-native CI/CD workflows (multi-arch container image build and push to GHCR).

## [v0.13.0-b2](https://github.com/ADI-Foundation-Labs/ADI-Stack-Server/releases/tag/v0.13.0-b2)

_2026-06-03_

L1 block-finality hardening.

**Fixes**

* **l1** - add 2 confirmation blocks to mitigate reorg issues in L1 block finality.

## [v0.13.0-b1](https://github.com/ADI-Foundation-Labs/ADI-Stack-Server/releases/tag/v0.13.0-b1)

_2026-02-04_

Replay-connection resilience.

**Fixes**

* **replay** - retry on a dropped replay connection and raise the initial delay to 500s (#12).

## [v0.13.0-b](https://github.com/ADI-Foundation-Labs/ADI-Stack-Server/releases/tag/v0.13.0-b)

_First ADI beta_

First ADI beta over upstream `v0.13.0` - a private runtime-config RPC plus RPC and mempool fixes.

**Features**

* **rpc(private)** - private RPC API for runtime configuration overrides, with on-disk persistence, `config_removeOverrides`, and `pubdata_price = 0` support.

**Fixes**

* **rpc** - fix gas estimation for accounts with small balances.
* **mempool** - use the pending base fee on canonical-state change (#9); add a transaction fee cap (#8).
* **fees** - fee estimation now honours config overrides (#7).

**Chores**

* Integrate ADI repositories and improvements; refresh dependencies, README, licenses, and authors.
