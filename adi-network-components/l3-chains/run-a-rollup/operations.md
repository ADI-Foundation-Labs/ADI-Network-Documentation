---
description: >-
  Running, monitoring, GPU configuration, and troubleshooting your ADI Rollup
  deployment.
---

# Operations

## Running Everything Together

To bring up the full stack in one command:

```bash
# Core + Infrastructure
docker compose \
  -f docker-compose.yml \
  -f docker-compose.infra.yml \
  up -d

# Core + Infrastructure + Blockscout
docker compose \
  -f docker-compose.yml \
  -f docker-compose.infra.yml \
  -f docker-compose.blockscout.yml \
  up -d
```

### Rate limits and allowlisting

The mainnet RPC at `rpc.adifoundation.ai` enforces per-IP rate limits for L3 traffic.

If sequencer startup hits HTTP `429` with Cloudflare `1015`, the issue is infrastructure configuration, not chain logic. Contact ADI Ops to allowlist your outbound IP.

Testnet at `rpc.ab.testnet.adifoundation.ai` does not enforce these limits.

{% hint style="warning" %}
If you are hitting `429` during startup, set `restart: "no"` on the affected services until your IP is allowlisted. Keep `restart: unless-stopped` for transient upstream failures such as `502`.
{% endhint %}

***

## Port Reference

| Port   | Service       | Description                                               |
| ------ | ------------- | --------------------------------------------------------- |
| `3050` | Sequencer     | L2 JSON-RPC endpoint                                      |
| `3051` | External Node | L2 JSON-RPC (read replica)                                |
| `3053` | Sequencer     | Block replay server used by the External Node             |
| `3054` | External Node | Replay server override to avoid conflict on the same host |
| `3320` | Sequencer     | Prover API (internal)                                     |
| `3000` | Bridge        | dApp Portal / Bridge UI                                   |
| `3010` | Explorer App  | Block Explorer frontend                                   |
| `3020` | Explorer API  | Block Explorer REST API                                   |
| `3040` | Data Fetcher  | Explorer data fetcher                                     |
| `4000` | Blockscout    | Blockscout frontend (optional)                            |

***

## GPU Configuration

### Single GPU

If you have a dedicated GPU per prover, use the full GPU UUID:

```bash
nvidia-smi -L
# GPU 0: NVIDIA A100 80GB (UUID: GPU-12345678-abcd-...)
```

Set in `.env`:

```bash
GPU_DEVICE_FRI=GPU-12345678-abcd-...
GPU_DEVICE_SNARK=GPU-87654321-dcba-...
```

### Scaling Provers

For higher proving throughput, add more prover instances. Each needs a unique name and Prometheus port:

```yaml
  fri-prover-2:
    image: ${PROVER_FRI_IMAGE}
    container_name: ${CHAIN_SHORT_NAME}-fri-prover-2
    restart: unless-stopped
    network_mode: host
    depends_on: [server]
    environment:
      RUST_LOG: "info"
    volumes:
      - ./volumes/prover:/prover
    command:
      - "--base-url"
      - "http://127.0.0.1:3320"
      - "--app-bin-path"
      - "/multiblock_batch.bin"
      - "--enabled-logging"
      - "--prover-name"
      - "fri-prover-2"
      - "--prometheus-port"
      - "3125"
    deploy:
      resources:
        reservations:
          devices:
            - driver: nvidia
              device_ids: ["${GPU_DEVICE_FRI_2}"]
              capabilities: [gpu]
```

***

## Viewing Logs

```bash
# All services
docker compose logs -f

# Specific service
docker compose logs -f server
docker compose logs -f fri-prover
docker compose logs -f snark-prover

# Infrastructure stack
docker compose -f docker-compose.infra.yml logs -f explorer-worker
```

***

## Monitoring

| Endpoint                        | What It Shows                    |
| ------------------------------- | -------------------------------- |
| `http://localhost:3050`         | Sequencer JSON-RPC               |
| `http://localhost:3051`         | External Node JSON-RPC           |
| `http://localhost:3312/metrics` | Sequencer Prometheus metrics     |
| `http://localhost:3316/metrics` | External Node Prometheus metrics |
| `http://localhost:3124/metrics` | FRI Prover Prometheus metrics    |
| `http://localhost:3126/metrics` | SNARK Prover Prometheus metrics  |

Check sync status:

```bash
# Sequencer block number
curl -s -X POST http://localhost:3050 \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","method":"eth_blockNumber","params":[],"id":1}'

# GPU utilization
watch -n 1 nvidia-smi
```

***

## Restarting Services

```bash
# Restart a single service
docker compose restart server

# Restart the entire core stack
docker compose down && docker compose up -d
```

***

## Troubleshooting

| Error                                           | Likely cause                                                                                     | Resolution                                                                                                                                                                                                                                                                                                        |
| ----------------------------------------------- | ------------------------------------------------------------------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `failed to fetch finalized L1 state` with `429` | Your settlement RPC is rate-limiting L3 traffic                                                  | Check the upstream response. On mainnet, contact ADI Ops to allowlist your outbound IP. Set `restart: "no"` until allowlisting is complete.                                                                                                                                                                       |
| `failed to fetch finalized L1 state` with `502` | Upstream settlement RPC is unavailable or returning a transient gateway error                    | Keep `restart: unless-stopped`. Retry after the upstream recovers, or switch to a healthier RPC endpoint.                                                                                                                                                                                                         |
| `sender does not have enough funds`             | An operator wallet is missing settlement-layer gas funds                                         | Fund the commit, prove, and execute operator wallets with the settlement-layer gas token. If `base_token_address` was omitted at deploy time, that token is ADI.                                                                                                                                                  |
| `Existing batch first block does not match`     | Local node state does not match the chain you are trying to run                                  | Stop the node. Remove the affected node's RocksDB and replay data. Keep reusable wallet files only. Do not reuse `genesis.yaml` or `contracts.yaml` across chains.                                                                                                                                                |
| `L1 commit watcher component failed`            | Settlement RPC is wrong, unavailable, rate-limited, or paired with the wrong settlement chain ID | Verify `L1_RPC_URL` reaches your settlement layer. Confirm the generated `genesis.json` has the correct `l1_chain_id`. For ADI, use `36900` on mainnet or `99999` on testnet. If you use a public RPC, relax the watcher settings to `l1_watcher_poll_interval: "2s"` and `l1_watcher_max_blocks_to_process: 10`. |
