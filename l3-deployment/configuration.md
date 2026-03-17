---
description: Configuration reference for adi-cli L3 deployments
---

# Configuration

Create `~/.adi.yml` with your L3 settings. Below is a complete example with all available options.

## Configuration File

```yaml
# ~/.adi.yml

# ============================================================================
# GLOBAL SETTINGS
# ============================================================================

# Protocol version determines which toolkit Docker image to use
# Format: v{MAJOR}.{MINOR}.{PATCH}
protocol_version: v0.30.1

# Directory where ecosystem state is stored
# Default: ~/.adi_cli/state
state_dir: ~/.adi_cli/state

# Enable verbose logging (equivalent to -d flag)
# Default: false
debug: false

# Gas price multiplier percentage
# The CLI fetches current gas price and multiplies by this value
# 200 = use 200% of estimated gas (100% buffer)
# Recommended: 200 for local/stable networks, 300 for congested networks
gas_multiplier: 200

# ============================================================================
# ECOSYSTEM SETTINGS
# ============================================================================

ecosystem:
  # Ecosystem name (used for directory naming)
  name: my_l3_ecosystem

  # ADI L2 RPC endpoint (settlement layer for your L3)
  # This is where your L3 contracts will be deployed
  rpc_url: https://rpc.adi.network

  # Network type identifier
  # Use "sepolia" for ADI L2 (even though it is not actual Sepolia)
  # Options: localhost, sepolia, mainnet
  l1_network: sepolia

  # ----------------------------------------------------------------------------
  # CHAIN CONFIGURATIONS
  # An ecosystem can have multiple chains. Each chain is an independent L3.
  # ----------------------------------------------------------------------------

  chains:
    - name: my-chain
      # Unique numeric identifier for this chain
      # Must not already exist on the settlement layer
      chain_id: 270

      # Proof generation mode
      # no-proofs: Fast, for development/testing (no real proofs generated)
      # gpu: Production mode (requires GPU prover infrastructure)
      prover_mode: no-proofs

      # Data availability mode
      # false = calldata mode (REQUIRED for L3 on ZKsync-based L2)
      # true = blob mode (for L2 on Ethereum L1)
      blobs: false

      # Enable EVM bytecode emulator
      # Allows running unmodified Ethereum contracts
      # Default: false
      evm_emulator: false

      # OPTIONAL: Custom ERC20 token for gas payments
      # Omit these fields to use native ETH
      # base_token_address: "0x..."
      # base_token_price_nominator: 1
      # base_token_price_denominator: 1

      # OPTIONAL: Predefined operator addresses
      # By default, the CLI generates random wallets for operators
      # Use these to specify your own addresses (e.g., for HSM or multisig)
      # operators:
      #   operator: "0x..."         # PRECOMMITTER, COMMITTER, REVERTER roles
      #   prove_operator: "0x..."   # PROVER role
      #   execute_operator: "0x..." # EXECUTOR role

      # OPTIONAL: Per-chain funding amounts (in ETH)
      # These override the global funding settings for this chain
      # funding:
      #   operator_eth: 30.0
      #   prove_operator_eth: 30.0
      #   execute_operator_eth: 30.0

      # OPTIONAL: Ownership transfer after deployment
      # ownership:
      #   new_owner: "0x..."

# ============================================================================
# FUNDING SETTINGS
# ============================================================================

funding:
  # SECURITY: Set via ADI_FUNDER_KEY environment variable instead
  # funder_key: "0x..."

  # ETH amounts for ecosystem-level wallets
  # These wallets are shared across all chains in the ecosystem

  # Deployer: Deploys all smart contracts
  # Needs enough for contract deployment gas costs
  deployer_eth: 1.0

  # Governor: Executes governance operations and upgrades
  # Needs enough for governance transactions
  governor_eth: 1.0

  # Custom gas token units (only if using custom base token)
  # governor_cgt_units: 5.0

# ============================================================================
# OPTIONAL: S3 STATE BACKUP
# ============================================================================

# s3:
#   enabled: true
#   tenant_id: my-tenant
#   bucket: adi-state
#   region: us-east-1
#   # endpoint_url: http://localhost:9000  # For MinIO

# ============================================================================
# OPTIONAL: TOOLKIT IMAGE OVERRIDE
# ============================================================================

# toolkit:
#   # Override the Docker image tag (defaults to protocol_version)
#   image_tag: "latest"
```

## Environment Variables

For sensitive data and per-session overrides, use environment variables.

{% hint style="warning" %}
Never commit private keys to configuration files. Always use `ADI_FUNDER_KEY` environment variable for the funder wallet.
{% endhint %}

| Variable | Purpose |
|----------|---------|
| `ADI_FUNDER_KEY` | Private key (hex) of the wallet that funds ecosystem wallets |
| `ADI_PRIVATE_KEY` | Private key for accepting ownership transfers |
| `ADI_RPC_URL` | Override RPC endpoint without editing config |
| `ADI_CONFIG` | Path to alternative config file |
| `ADI__PROTOCOL_VERSION` | Override protocol version |
| `ADI__TOOLKIT__IMAGE_TAG` | Override Docker image tag |
| `ADI_OPERATOR` | Override operator address |
| `ADI_PROVE_OPERATOR` | Override prove operator address |
| `ADI_EXECUTE_OPERATOR` | Override execute operator address |
| `ADI__S3__ENABLED` | Enable S3 sync (`true`/`false`) |
| `ADI__S3__TENANT_ID` | Tenant identifier for S3 |
| `AWS_ACCESS_KEY_ID` | AWS access key for S3 |
| `AWS_SECRET_ACCESS_KEY` | AWS secret key for S3 |
| `RUST_LOG` | Logging verbosity: `error`, `warn`, `info`, `debug`, `trace` |

### Path Separator Convention

Environment variables use `ADI__` prefix with double underscores as path separators:

```bash
# Override ecosystem name
export ADI__ECOSYSTEM__NAME=production

# Override RPC URL
export ADI__ECOSYSTEM__RPC_URL=https://other-rpc.example.com

# Set funder key (required for deployment)
export ADI_FUNDER_KEY="0x..."
```

## Next Steps

- [Deploy](deploy.md) - Initialize and deploy your L3 chains
