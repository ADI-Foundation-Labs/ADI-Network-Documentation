---
description: Full configuration reference for the ADI CLI
---

# CLI Configuration

Complete reference for `~/.adi.yml`. For a quick start, see the [minimal config](../adi-network-components/l3-chains/cli.md#minimal-config) in the CLI guide.

## Full Annotated Config

```yaml
# Where to store ecosystem state (wallets, contracts, chain configs)
# Default: ~/.adi_cli/state
state_dir: ~/.adi_cli/state

# Enable verbose logging for troubleshooting
# Default: false (can also use -d flag)
debug: false

# Default protocol version for toolkit Docker image
# Used by init, add, and deploy when --protocol-version is not provided
protocol_version: v0.30.1

# State storage backend (currently only "filesystem" is supported)
state_backend: filesystem

ecosystem:
  # Name used for the ecosystem directory and identification
  name: adi_ecosystem

  # Settlement layer network: localhost (Anvil) | sepolia | mainnet
  l1_network: sepolia

  # Settlement layer RPC endpoint
  # Anvil: http://host.docker.internal:8545
  # ADI Testnet: https://rpc.ab.testnet.adifoundation.ai
  rpc_url: https://rpc.ab.testnet.adifoundation.ai

  # Ecosystem-level ownership transfer after deploy
  # ownership:
  #   new_owner: "0x..."

  chains:
    - name: my-chain
      chain_id: 222

      # no-proofs: development/testing (fast, no real proofs)
      # gpu: production (requires GPU prover infrastructure)
      prover_mode: no-proofs

      # EVM bytecode emulator for unmodified Ethereum contracts
      evm_emulator: false

      # Blob-based pubdata (EIP-4844)
      # true: blobs (L2 settling on L1)
      # false: calldata (L3 settling on L2)
      blobs: false

      # Custom ERC20 token for gas payments (omit for native ETH)
      # base_token_address: "0x..."
      # base_token_price_nominator: 1
      # base_token_price_denominator: 1

      # Override generated operator addresses
      # operators:
      #   operator: "0x..."         # PRECOMMITTER, COMMITTER, REVERTER
      #   prove_operator: "0x..."   # PROVER
      #   execute_operator: "0x..." # EXECUTOR

      # Per-chain funding (ETH in ether)
      # funding:
      #   operator_eth: 30.0
      #   prove_operator_eth: 30.0
      #   execute_operator_eth: 30.0

      # Per-chain ownership
      # ownership:
      #   new_owner: "0x..."

# Ecosystem-level wallet funding during deployment
# A full deployment requires ~270 ADI tokens across all wallets.
# Testnet faucet: https://faucet.ab.testnet.adifoundation.ai/
funding:
  # SECURITY: use ADI_FUNDER_KEY env var instead
  # funder_key: "0x..."

  # Testnet (minimal):
  #   deployer_eth: 1.0
  #   governor_eth: 1.0
  #
  # Production:
  deployer_eth: 100.0
  governor_eth: 40.0
  governor_cgt_units: 5.0   # only if using custom base token

# Gas price multiplier percentage
# CLI fetches current gas price and multiplies by this value
# Recommended: 200 for Anvil, 300 for Sepolia/mainnet
gas_multiplier: 200

# S3 state backup
# s3:
#   enabled: true
#   tenant_id: my-tenant       # S3 key prefix
#   bucket: adi-state
#   region: us-east-1
#   endpoint_url: http://localhost:9000  # MinIO/LocalStack

# Override Docker toolkit image
# toolkit:
#   image_tag: "latest"
```

## Config Resolution Priority

Config file sources are mutually exclusive — only one file is loaded:

1. `--config` flag
2. `ADI_CONFIG` environment variable
3. `~/.adi.yml` (default)

Override sources are always applied on top:

4. `ADI__*` environment variables
5. CLI flags (highest priority)

## Environment Variables

| Variable | Purpose |
| --- | --- |
| `ADI_FUNDER_KEY` | Private key (hex) of the wallet that funds ecosystem wallets |
| `ADI_PRIVATE_KEY` | Private key (hex) for accepting ownership as new owner |
| `ADI_RPC_URL` | Settlement layer RPC endpoint |
| `ADI_EXPLORER_URL` | Block explorer API URL for contract verification |
| `ADI_EXPLORER_API_KEY` | Block explorer API key |
| `ADI_CONFIG` | Path to an alternative config file |
| `ADI__PROTOCOL_VERSION` | Default protocol version |
| `ADI__TOOLKIT__IMAGE_TAG` | Override Docker image tag |
| `ADI_OPERATOR` | Operator address (PRECOMMITTER, COMMITTER, REVERTER) |
| `ADI_PROVE_OPERATOR` | Prove operator address (PROVER) |
| `ADI_EXECUTE_OPERATOR` | Execute operator address (EXECUTOR) |
| `AWS_ACCESS_KEY_ID` | AWS access key for S3 state sync |
| `AWS_SECRET_ACCESS_KEY` | AWS secret key for S3 state sync |
| `ADI__S3__ENABLED` | Enable S3 sync (`true`/`false`) |
| `ADI__S3__TENANT_ID` | Tenant identifier for S3 key prefix |
| `ADI__S3__BUCKET` | S3 bucket name |
| `RUST_LOG` | Logging verbosity: `error`, `warn`, `info`, `debug`, `trace` |

Override any config value using the `ADI__` prefix with double underscores as path separators:

```bash
export ADI__ECOSYSTEM__NAME=production
export ADI__ECOSYSTEM__RPC_URL=http://localhost:8545
```

## S3 State Synchronization

When enabled, the CLI archives and uploads ecosystem state to S3 after write operations (init, deploy). Archives are stored at `s3://{bucket}/{tenant_id}/{ecosystem-name}.tar.gz`.

```bash
# Sync current state to S3
adi state sync --ecosystem-name my-ecosystem

# Restore state from S3
adi state restore --ecosystem-name my-ecosystem

# Force restore (overwrite local state)
adi state restore --ecosystem-name my-ecosystem --force
```

{% tabs %}
{% tab title="S3 Config" %}
```yaml
s3:
  enabled: true
  tenant_id: my-tenant
  bucket: adi-state
  region: us-east-1
```
{% endtab %}

{% tab title="MinIO (Local Dev)" %}
```bash
docker run -p 9000:9000 -p 9001:9001 minio/minio server /data --console-address ":9001"
```

```yaml
s3:
  enabled: true
  tenant_id: dev
  bucket: adi-state
  endpoint_url: http://localhost:9000
```

Set credentials via `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY`.
{% endtab %}
{% endtabs %}

## Deprecated Fields

These fields still work but will be removed in a future release:

| Field | Replacement |
| --- | --- |
| Top-level `ownership` | `ecosystem.ownership` or `ecosystem.chains[].ownership` |
| Top-level `operators` | `ecosystem.chains[].operators` |
