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

---

## Port Reference

| Port | Service | Description |
|------|---------|-------------|
| `3050` | Sequencer | L2 JSON-RPC endpoint |
| `3051` | External Node | L2 JSON-RPC (read replica) |
| `3053` | Sequencer | Block replay server used by the External Node |
| `3054` | External Node | Replay server override to avoid conflict on the same host |
| `3320` | Sequencer | Prover API (internal) |
| `3000` | Bridge | dApp Portal / Bridge UI |
| `3010` | Explorer App | Block Explorer frontend |
| `3020` | Explorer API | Block Explorer REST API |
| `3040` | Data Fetcher | Explorer data fetcher |
| `4000` | Blockscout | Blockscout frontend (optional) |

---

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

---

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

---

## Monitoring

| Endpoint | What It Shows |
|----------|---------------|
| `http://localhost:3050` | Sequencer JSON-RPC |
| `http://localhost:3051` | External Node JSON-RPC |
| `http://localhost:3312/metrics` | Sequencer Prometheus metrics |
| `http://localhost:3316/metrics` | External Node Prometheus metrics |
| `http://localhost:3124/metrics` | FRI Prover Prometheus metrics |
| `http://localhost:3126/metrics` | SNARK Prover Prometheus metrics |

Check sync status:

```bash
# Sequencer block number
curl -s -X POST http://localhost:3050 \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","method":"eth_blockNumber","params":[],"id":1}'

# GPU utilization
watch -n 1 nvidia-smi
```

---

## Restarting Services

```bash
# Restart a single service
docker compose restart server

# Restart the entire core stack
docker compose down && docker compose up -d
```