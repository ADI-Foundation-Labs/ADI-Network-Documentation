---
description: >-
  Deploy Blockscout as an optional block explorer with smart contract
  verification for your ADI Rollup.
---

# Blockscout Explorer (Optional)

[Blockscout](https://www.blockscout.com/) provides an alternative block explorer with smart contract verification (Solidity + Vyper), Solidity-to-UML visualization, function signature decoding, and chain statistics.

> **Warning:** Blockscout is entirely optional and runs independently from the Block Explorer. The Bridge requires the **Block Explorer** ([Infrastructure Stack](infrastructure-stack.md)) — not Blockscout. You can run both explorers side by side.

---

## Docker Compose

Create `docker-compose.blockscout.yml`:

```yaml
services:
  # ─────────────────────────────────────────────
  # Blockscout — PostgreSQL
  # ─────────────────────────────────────────────
  blockscout-db:
    image: postgres:15-alpine
    container_name: ${CHAIN_SHORT_NAME:-adi}-blockscout-db
    restart: unless-stopped
    command: postgres -c max_connections=300
    environment:
      POSTGRES_PASSWORD: ${BLOCKSCOUT_DB_PASSWORD:-postgres}
      POSTGRES_USER: blockscout
      POSTGRES_DB: blockscout
    volumes:
      - blockscout_db_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U blockscout -d blockscout"]
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 10s

  # ─────────────────────────────────────────────
  # Blockscout — Redis
  # ─────────────────────────────────────────────
  blockscout-redis:
    image: redis:7-alpine
    container_name: ${CHAIN_SHORT_NAME:-adi}-blockscout-redis
    restart: unless-stopped
    command: redis-server --maxmemory 256mb --maxmemory-policy allkeys-lru
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 5s
      retries: 5

  # ─────────────────────────────────────────────
  # Blockscout — Backend
  # ─────────────────────────────────────────────
  blockscout-backend:
    image: ghcr.io/blockscout/blockscout:${BLOCKSCOUT_VERSION:-latest}
    container_name: ${CHAIN_SHORT_NAME:-adi}-blockscout-backend
    restart: unless-stopped
    command: >
      sh -c "bin/blockscout eval \"Elixir.Explorer.ReleaseTasks.create_and_migrate()\"
      && bin/blockscout start"
    environment:
      DATABASE_URL: postgresql://blockscout:${BLOCKSCOUT_DB_PASSWORD:-postgres}@blockscout-db:5432/blockscout
      ECTO_USE_SSL: "false"
      POOL_SIZE: "80"
      POOL_SIZE_API: "10"
      ETHEREUM_JSONRPC_HTTP_URL: http://${RPC_HOST:-host.docker.internal}:${RPC_PORT:-3050}
      ETHEREUM_JSONRPC_TRACE_URL: http://${RPC_HOST:-host.docker.internal}:${RPC_PORT:-3050}
      ETHEREUM_JSONRPC_TRANSPORT: http
      ETHEREUM_JSONRPC_VARIANT: geth
      CHAIN_ID: ${CHAIN_ID:-444}
      COIN: ${CHAIN_CURRENCY_SYMBOL:-ADI}
      COIN_NAME: ${CHAIN_CURRENCY_NAME:-ADI Token}
      INDEXER_DISABLE_INTERNAL_TRANSACTIONS_FETCHER: "true"
      INDEXER_DISABLE_PENDING_TRANSACTIONS_FETCHER: "true"
      INDEXER_BEACON_BLOB_FETCHER_DISABLED: "true"
      PORT: 4000
      ACCOUNT_ENABLED: "false"
      DISABLE_MARKET: "true"
      SECRET_KEY_BASE: ${SECRET_KEY_BASE:?Run openssl rand -hex 32 and set SECRET_KEY_BASE in .env}
      ACCOUNT_REDIS_URL: redis://blockscout-redis:6379/0
      MICROSERVICE_SC_VERIFIER_ENABLED: "true"
      MICROSERVICE_SC_VERIFIER_URL: http://blockscout-verifier:8050
      MICROSERVICE_SC_VERIFIER_TYPE: sc_verifier
      MICROSERVICE_VISUALIZE_SOL2UML_ENABLED: "true"
      MICROSERVICE_VISUALIZE_SOL2UML_URL: http://blockscout-visualizer:8050
      MICROSERVICE_SIG_PROVIDER_ENABLED: "true"
      MICROSERVICE_SIG_PROVIDER_URL: http://blockscout-sig-provider:8050
    extra_hosts:
      - "host.docker.internal:host-gateway"
    depends_on:
      blockscout-db:
        condition: service_healthy
      blockscout-redis:
        condition: service_healthy
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:4000/api/v2/stats"]
      interval: 30s
      timeout: 10s
      retries: 5
      start_period: 120s

  # ─────────────────────────────────────────────
  # Blockscout — Frontend
  # ─────────────────────────────────────────────
  blockscout-frontend:
    image: ghcr.io/blockscout/frontend:${BLOCKSCOUT_FRONTEND_VERSION:-latest}
    container_name: ${CHAIN_SHORT_NAME:-adi}-blockscout-frontend
    restart: unless-stopped
    ports:
      - "4000:3000"
    environment:
      NEXT_PUBLIC_APP_HOST: localhost
      NEXT_PUBLIC_APP_PORT: 4000
      NEXT_PUBLIC_APP_PROTOCOL: http
      NEXT_PUBLIC_API_HOST: blockscout-backend
      NEXT_PUBLIC_API_PORT: 4000
      NEXT_PUBLIC_API_PROTOCOL: http
      NEXT_PUBLIC_API_BASE_PATH: /
      NEXT_PUBLIC_API_WEBSOCKET_PROTOCOL: ws
      NEXT_PUBLIC_NETWORK_NAME: "${CHAIN_NAME:-ADI Chain}"
      NEXT_PUBLIC_NETWORK_SHORT_NAME: "${CHAIN_SHORT_NAME:-adi}"
      NEXT_PUBLIC_NETWORK_ID: ${CHAIN_ID:-444}
      NEXT_PUBLIC_NETWORK_CURRENCY_NAME: "${CHAIN_CURRENCY_NAME:-ADI Token}"
      NEXT_PUBLIC_NETWORK_CURRENCY_SYMBOL: ${CHAIN_CURRENCY_SYMBOL:-ADI}
      NEXT_PUBLIC_NETWORK_CURRENCY_DECIMALS: 18
      NEXT_PUBLIC_IS_TESTNET: "true"
      NEXT_PUBLIC_VISUALIZE_API_HOST: http://blockscout-visualizer:8050
      NEXT_PUBLIC_STATS_API_HOST: http://blockscout-stats:8050
      NEXT_PUBLIC_AD_BANNER_PROVIDER: none
    depends_on:
      blockscout-backend:
        condition: service_healthy
    healthcheck:
      test: ["CMD", "wget", "-q", "--spider", "http://localhost:3000"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 30s

  # ─────────────────────────────────────────────
  # Blockscout — Smart Contract Verifier
  # ─────────────────────────────────────────────
  blockscout-verifier:
    image: ghcr.io/blockscout/smart-contract-verifier:${BLOCKSCOUT_VERIFIER_VERSION:-latest}
    container_name: ${CHAIN_SHORT_NAME:-adi}-blockscout-verifier
    restart: unless-stopped
    environment:
      SMART_CONTRACT_VERIFIER__SERVER__HTTP__ADDR: 0.0.0.0:8050
      SMART_CONTRACT_VERIFIER__SERVER__HTTP__ENABLED: "true"
      SMART_CONTRACT_VERIFIER__SOLIDITY__ENABLED: "true"
      SMART_CONTRACT_VERIFIER__SOLIDITY__COMPILERS_DIR: /tmp/solidity-compilers
      SMART_CONTRACT_VERIFIER__VYPER__ENABLED: "true"
      SMART_CONTRACT_VERIFIER__VYPER__COMPILERS_DIR: /tmp/vyper-compilers
      SMART_CONTRACT_VERIFIER__SOURCIFY__ENABLED: "true"
      SMART_CONTRACT_VERIFIER__SOURCIFY__API_URL: https://sourcify.dev/server/
    healthcheck:
      test: ["CMD", "wget", "-qO-", "http://localhost:8050/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 30s

  # ─────────────────────────────────────────────
  # Blockscout — Visualizer (Solidity UML)
  # ─────────────────────────────────────────────
  blockscout-visualizer:
    image: ghcr.io/blockscout/visualizer:${BLOCKSCOUT_VISUALIZER_VERSION:-latest}
    container_name: ${CHAIN_SHORT_NAME:-adi}-blockscout-visualizer
    restart: unless-stopped
    environment:
      VISUALIZER__SERVER__HTTP__ADDR: 0.0.0.0:8050
    healthcheck:
      test: ["CMD", "wget", "-qO-", "http://localhost:8050/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 30s

  # ─────────────────────────────────────────────
  # Blockscout — Signature Provider
  # ─────────────────────────────────────────────
  blockscout-sig-provider:
    image: ghcr.io/blockscout/sig-provider:${BLOCKSCOUT_SIG_PROVIDER_VERSION:-latest}
    container_name: ${CHAIN_SHORT_NAME:-adi}-blockscout-sig-provider
    restart: unless-stopped
    environment:
      SIG_PROVIDER__SERVER__HTTP__ADDR: 0.0.0.0:8050
    healthcheck:
      test: ["CMD", "wget", "-qO-", "http://localhost:8050/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 30s

  # ─────────────────────────────────────────────
  # Blockscout — Stats Database
  # ─────────────────────────────────────────────
  blockscout-stats-db:
    image: postgres:15-alpine
    container_name: ${CHAIN_SHORT_NAME:-adi}-blockscout-stats-db
    restart: unless-stopped
    environment:
      POSTGRES_PASSWORD: ${BLOCKSCOUT_DB_PASSWORD:-postgres}
      POSTGRES_USER: stats
      POSTGRES_DB: stats
    volumes:
      - blockscout_stats_db_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U stats -d stats"]
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 10s

  # ─────────────────────────────────────────────
  # Blockscout — Stats Service
  # ─────────────────────────────────────────────
  blockscout-stats:
    image: ghcr.io/blockscout/stats:${BLOCKSCOUT_STATS_VERSION:-latest}
    container_name: ${CHAIN_SHORT_NAME:-adi}-blockscout-stats
    restart: unless-stopped
    environment:
      STATS__DB_URL: postgresql://stats:${BLOCKSCOUT_DB_PASSWORD:-postgres}@blockscout-stats-db:5432/stats
      STATS__BLOCKSCOUT_DB_URL: postgresql://blockscout:${BLOCKSCOUT_DB_PASSWORD:-postgres}@blockscout-db:5432/blockscout
      STATS__CREATE_DATABASE: "false"
      STATS__RUN_MIGRATIONS: "true"
      STATS__BLOCKSCOUT_API_URL: http://blockscout-backend:4000
      STATS__SERVER__HTTP__ENABLED: "true"
      STATS__SERVER__HTTP__ADDR: 0.0.0.0:8050
      STATS__SERVER__HTTP__CORS__ENABLED: "true"
      STATS__SERVER__HTTP__CORS__ALLOWED_ORIGIN: http://localhost:4000
      STATS__FORCE_UPDATE_ON_START: "false"
    depends_on:
      blockscout-stats-db:
        condition: service_healthy
      blockscout-backend:
        condition: service_healthy
    healthcheck:
      test: ["CMD", "wget", "-qO-", "http://localhost:8050/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 30s

volumes:
  blockscout_db_data:
    name: ${CHAIN_SHORT_NAME:-adi}_blockscout_db_data
  blockscout_stats_db_data:
    name: ${CHAIN_SHORT_NAME:-adi}_blockscout_stats_db_data
```

---

## Start

> **Note:** The default RPC host is `host.docker.internal`, which works out of the box with the `extra_hosts` entry on Linux and on Docker Desktop for macOS/Windows. If your node runs on a different machine, set `RPC_HOST` and `RPC_PORT` in `.env` and consider an SSH tunnel.

```bash
docker compose -f docker-compose.blockscout.yml up -d
```

> **Tip:** Blockscout will be available at `http://localhost:4000`. The initial sync may take a few minutes depending on your chain's block count.
