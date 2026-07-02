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
| Sequencer      | `v0.13.0-b1`    | `v0.13.0-b4`    |
| External nodes | `v0.13.0-b4`    | `v0.13.0-b4`    |

### Release notes

## [v0.13.0-b4](https://github.com/ADI-Foundation-Labs/ADI-Stack-Server/releases/tag/v0.13.0-b4)

_2026-06-30 - latest_

L1 watcher hardening.

**Fixes**

* **l1-watcher** - leave 2 confirmation blocks unprocessed, matching the upgrade watcher's behavior, to avoid `block range extends beyond current head block` errors.
* **tests** - add the missing private API server config and bind it to a dynamically allocated port.

## [v0.13.0-b3](https://github.com/ADI-Foundation-Labs/ADI-Stack-Server/releases/tag/v0.13.0-b3)

_2026-06-11_

The `zks` finality RPC and ADI-native CI/CD.

**Features**

* **rpc(finality)** - add `zks_getBlockFinality`, `zks_getTransactionFinality`, `zks_getBatchFinality`, and `zks_getFinalityStatus`, reporting how far a block, transaction, or batch has moved through L1 finality (`pending` -> `committed` -> `executed`), plus a single-call snapshot of every finality frontier. See the [JSON-RPC API](json-rpc-api.md#finality).

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
