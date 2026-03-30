---
description: >-
  Complete environment variable and CLI argument reference for all ADI Rollup
  components.
---

# Configuration Reference

This page covers the most important environment variables and CLI arguments for each component. For the full list of available options, see the linked source repositories.

---

## Sequencer Configuration

The sequencer uses environment variables in `component_setting_name` format (lowercase, underscore-separated). All settings are defined via the [`smart_config`](https://github.com/ADI-Foundation-Labs/ADI-Stack-Server/tree/main/node/bin/src/config) framework.

### General

| Variable | Default | Description |
|----------|---------|-------------|
| `general_l1_rpc_url` | `http://localhost:8545` | Ethereum L1 JSON-RPC endpoint |
| `general_rocks_db_path` | `./db` | Path to RocksDB state storage |
| `general_main_node_rpc_url` | — | Main node RPC URL (required for External Nodes) |
| `general_min_blocks_to_replay` | `10` | Minimum blocks to replay on restart |
| `general_force_starting_block_number` | — | Force replay from a specific block number |

### Sequencer

| Variable | Default | Description |
|----------|---------|-------------|
| `sequencer_block_time` | `250ms` | Block production interval |
| `sequencer_max_transactions_in_block` | `1000` | Maximum transactions per block |
| `sequencer_block_gas_limit` | `100000000` | Gas limit per block |
| `sequencer_block_pubdata_limit_bytes` | `110000` | Pubdata size limit per block |
| `sequencer_block_dump_path` | `./db/block_dumps` | Path to dump block data for replay |
| `sequencer_block_replay_server_address` | `0.0.0.0:3053` | Replay server address used by External Nodes |
| `sequencer_block_replay_download_address` | — | Replay source address; setting this enables External Node mode |
| `sequencer_fee_collector_address` | `0x3661...c049` | Address that collects transaction fees |

### Sequencer Fee Overrides

| Variable | Default | Description |
|----------|---------|-------------|
| `sequencer_base_fee_override` | — | Override base fee (hex, e.g. `0x3e8`) |
| `sequencer_pubdata_price_override` | — | Override pubdata price (hex) |
| `sequencer_native_price_override` | — | Override native token price (hex) |

### RPC

| Variable | Default | Description |
|----------|---------|-------------|
| `rpc_address` | `0.0.0.0:3050` | JSON-RPC listen address |
| `rpc_max_connections` | `1000` | Maximum concurrent connections |
| `rpc_eth_call_gas` | `10000000` | Gas limit for `eth_call` |
| `rpc_max_logs_per_response` | `20000` | Maximum log entries returned per request |
| `rpc_max_request_size` | `15` | Maximum request payload size (MB) |
| `rpc_max_response_size` | `24` | Maximum response payload size (MB) |

### L1 Sender

| Variable | Default | Description |
|----------|---------|-------------|
| `l1_sender_operator_commit_pk` | — | Private key for committing batches to L1 |
| `l1_sender_operator_prove_pk` | — | Private key for submitting proofs to L1 |
| `l1_sender_operator_execute_pk` | — | Private key for executing batches on L1 |
| `l1_sender_max_fee_per_gas` | `100 Gwei` | Maximum gas price for L1 transactions |
| `l1_sender_max_fee_per_gas_gwei` | — | Same setting expressed with a Gwei suffix for easier Docker env usage |
| `l1_sender_max_priority_fee_per_gas` | `1 Gwei` | Maximum priority fee for L1 transactions |
| `l1_sender_max_priority_fee_per_gas_gwei` | — | Same setting expressed with a Gwei suffix |
| `l1_sender_max_fee_per_blob_gas` | `1 Gwei` | Maximum blob gas price |
| `l1_sender_max_fee_per_blob_gas_gwei` | — | Same setting expressed with a Gwei suffix |
| `l1_sender_pubdata_mode` | — | Pubdata availability mode — `Blobs` or `Calldata` |
| `l1_sender_enabled` | `true` | Enable L1 settlement transactions |
| `l1_sender_fusaka_upgrade_timestamp` | `18446744073709551615` | Switch to Fusaka blob format after this Unix timestamp |

### Prover API (on Sequencer)

| Variable | Default | Description |
|----------|---------|-------------|
| `prover_api_address` | `0.0.0.0:3124` | Prover API listen address |
| `prover_api_fake_fri_provers_enabled` | `true` | Use fake FRI provers (set `false` for production) |
| `prover_api_fake_snark_provers_enabled` | `true` | Use fake SNARK provers (set `false` for production) |
| `prover_api_fri_job_timeout` | `300s` | Timeout before reassigning a FRI job |
| `prover_api_snark_job_timeout` | `300s` | Timeout before reassigning a SNARK job |
| `prover_api_max_fris_per_snark` | `10` | Maximum FRI proofs aggregated per SNARK proof |
| `prover_api_object_store_file_backed_base_path` | `./db/shared` | Base path for file-backed prover proof storage |

### Prover Input Generator

| Variable | Default | Description |
|----------|---------|-------------|
| `prover_input_generator_maximum_in_flight_blocks` | `16` | Number of blocks the prover input generator can process concurrently |

### Batching

| Variable | Default | Description |
|----------|---------|-------------|
| `batcher_batch_timeout` | `1s` | Maximum time to wait before sealing a batch |
| `batcher_blocks_per_batch_limit` | `10` | Maximum blocks per batch |

### Genesis

| Variable | Default | Description |
|----------|---------|-------------|
| `genesis_chain_id` | — | L2 chain ID |
| `genesis_bridgehub_address` | — | L1 Bridgehub contract address |
| `genesis_bytecode_supplier_address` | — | L1 bytecode supplier contract address |
| `genesis_genesis_input_path` | — | Path to `genesis.json` file |

### Observability

| Variable | Default | Description |
|----------|---------|-------------|
| `observability_prometheus_port` | `3312` | Prometheus metrics port |
| `status_server_address` | `0.0.0.0:3071` | Status server address; set a different port on the External Node when both run on one host |
| `RUST_LOG` | — | Log level filter (e.g. `info`, `debug`, `warn`) |

> Full server configuration reference: [ADI-Stack-Server/node/bin/src/config](https://github.com/ADI-Foundation-Labs/ADI-Stack-Server/tree/main/node/bin/src/config)

---

## FRI Prover Configuration

The FRI prover is configured via CLI arguments. Environment variables control logging only.

| Argument | Default | Description |
|----------|---------|-------------|
| `--base-url` / `--sequencer-urls` | `http://localhost:3124` | Sequencer prover API URL(s), comma-separated for round-robin |
| `--sequencer-urls-file` | — | Path to file with URLs (one per line), takes precedence over `--sequencer-urls` |
| `--app-bin-path` | `../../multiblock_batch.bin` | Path to the RISC-V binary |
| `--circuit-limit` | `10000` | Maximum MainVM circuits to instantiate |
| `--iterations` | — | Number of proving iterations before exiting (unlimited if omitted) |
| `--prover-name` | `unknown_prover` | Prover identifier for monitoring |
| `--prometheus-port` | `3124` | Prometheus metrics port |
| `--enabled-logging` | `false` | Enable structured logging |
| `--request-timeout-secs` | `2` | HTTP request timeout to sequencer |

| Env Variable | Description |
|--------------|-------------|
| `RUST_LOG` | Log level filter (`debug`, `info`, `warn`, `error`) |

> Full FRI prover source: [ADI-Stack-Airbender-Prover/crates/zksync\_os\_fri\_prover](https://github.com/ADI-Foundation-Labs/ADI-Stack-Airbender-Prover/tree/main/crates/zksync_os_fri_prover)

---

## SNARK Prover Configuration

The SNARK prover uses the `run-prover` subcommand with CLI arguments.

| Argument | Default | Description |
|----------|---------|-------------|
| `--sequencer-url` / `--sequencer-urls` | — | Sequencer prover API URL(s) |
| `--sequencer-urls-file` | — | Path to file with URLs |
| `--binary-path` | — | Path to the RISC-V binary |
| `--trusted-setup-file` | — | Path to the trusted setup key (`setup_compact.key`) |
| `--output-dir` | — | Output directory for SNARK proofs |
| `--iterations` | — | Number of proving iterations before exiting |
| `--prover-name` | — | Prover identifier for monitoring |
| `--prometheus-port` | `3124` | Prometheus metrics port |
| `--disable-zk` | `false` | Disable zero-knowledge property (testing only) |

| Env Variable | Default | Description |
|--------------|---------|-------------|
| `RUST_MIN_STACK` | — | Thread stack size in bytes (recommended: `267108864` = 256 MB) |
| `RUST_LOG` | — | Log level filter |

> Full SNARK prover source: [ADI-Stack-Airbender-Prover/crates/zksync\_os\_snark\_prover](https://github.com/ADI-Foundation-Labs/ADI-Stack-Airbender-Prover/tree/main/crates/zksync_os_snark_prover)

---

## Block Explorer Configuration

The Block Explorer consists of four services. Key environment variables for each:

### Worker (Indexer)

| Variable | Default | Description |
|----------|---------|-------------|
| `BLOCKCHAIN_RPC_URL` | — | L2 JSON-RPC endpoint to index |
| `DATA_FETCHER_URL` | — | URL of the data fetcher service |
| `DATABASE_HOST` | `localhost` | PostgreSQL host |
| `DATABASE_USER` | `postgres` | PostgreSQL user |
| `DATABASE_PASSWORD` | — | PostgreSQL password |
| `DATABASE_NAME` | `block-explorer` | PostgreSQL database name |
| `SETTLEMENT_RPC_URL` | — | L1 RPC URL for batch settlement tracking |
| `DIAMOND_PROXY_ADDRESS` | — | L1 Diamond Proxy contract address |
| `BASE_TOKEN_SYMBOL` | `ETH` | Base token ticker symbol |
| `BASE_TOKEN_NAME` | `Ether` | Base token display name |

### API

| Variable | Default | Description |
|----------|---------|-------------|
| `PORT` | `3020` | HTTP listen port |
| `DATABASE_HOST` | `localhost` | PostgreSQL host |
| `DATABASE_USER` | `postgres` | PostgreSQL user |
| `DATABASE_PASSWORD` | — | PostgreSQL password |
| `DATABASE_NAME` | `block-explorer` | PostgreSQL database name |
| `BASE_TOKEN_SYMBOL` | `ETH` | Base token ticker |
| `BASE_TOKEN_L1_ADDRESS` | — | L1 address of the base token |

### App (Frontend)

| Variable | Default | Description |
|----------|---------|-------------|
| `APP_NETWORK_NAME` | — | Network display name |
| `APP_L2_CHAIN_ID` | — | L2 chain ID |
| `APP_RPC_URL` | — | L2 JSON-RPC URL (used by frontend) |
| `APP_API_URL` | — | Block Explorer API URL |
| `APP_BRIDGE_URL` | — | Bridge UI URL |
| `APP_L1_EXPLORER_URL` | — | L1 block explorer URL |
| `APP_BASE_TOKEN_ADDRESS` | — | Base token contract address on L2 |

### Data Fetcher

| Variable | Default | Description |
|----------|---------|-------------|
| `PORT` | `3040` | HTTP listen port |
| `BLOCKCHAIN_RPC_URL` | — | L2 JSON-RPC endpoint |
| `RPC_BATCH_MAX_COUNT` | `50` | Maximum batch size for RPC calls |

> Full Block Explorer source: [matter-labs/block-explorer](https://github.com/matter-labs/block-explorer)

---

## Bridge (dApp Portal) Configuration

The Bridge is configured via a `RUNTIME_CONFIG` JSON environment variable. Key fields:

| Field | Description |
|-------|-------------|
| `nodeType` | Must be `"hyperchain"` for custom rollups |
| `hyperchainsConfig[].network.id` | L2 chain ID |
| `hyperchainsConfig[].network.name` | Network display name |
| `hyperchainsConfig[].network.rpcUrl` | L2 JSON-RPC endpoint |
| `hyperchainsConfig[].network.blockExplorerUrl` | Block Explorer frontend URL |
| `hyperchainsConfig[].network.blockExplorerApi` | Block Explorer API URL |
| `hyperchainsConfig[].network.l1Network.rpcUrls` | L1 RPC URLs |
| `hyperchainsConfig[].tokens` | List of bridgeable tokens with addresses and metadata |

> Full dApp Portal source: [matter-labs/dapp-portal](https://github.com/matter-labs/dapp-portal)
