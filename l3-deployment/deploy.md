---
description: Commands for initializing and deploying L3 chains
---

# Deploy

This page covers the CLI commands for initializing your ecosystem and deploying L3 chains.

## Initialize Ecosystem

The `init` command creates the ecosystem infrastructure and your first L3 chain.

### What Happens During Init

1. **Validates protocol version** - Checks the version format is valid
2. **Pulls Docker image** - Downloads the toolkit image if not present locally (may take a few minutes first time)
3. **Runs zkstack** - Executes `zkstack ecosystem create` inside a container to generate configuration files
4. **Imports state** - Copies generated files to your state directory
5. **Generates wallets** - Creates cryptographic keys for all ecosystem wallets (deployer, governor, operators)

### Command

Using config file defaults:

```bash
adi init
```

With explicit flags:

```bash
adi init \
  --protocol-version v0.30.1 \
  --ecosystem-name my_l3_ecosystem \
  --l1-network sepolia \
  --chain-name my-chain \
  --chain-id 270 \
  --prover-mode no-proofs
```

### Flags Reference

| Flag | Short | Description | Default |
|------|-------|-------------|---------|
| `--protocol-version` | `-p` | Toolkit version (e.g., `v0.30.1`) | From config |
| `--ecosystem-name` | | Name for ecosystem directory | `adi_ecosystem` |
| `--l1-network` | | Settlement layer type | `sepolia` |
| `--chain-name` | | Name for first L3 chain | From config |
| `--chain-id` | | Unique numeric chain ID | `222` |
| `--prover-mode` | | `no-proofs` or `gpu` | `no-proofs` |
| `--operator` | | Operator address | Generated |
| `--prove-operator` | | Prove operator address | Generated |
| `--execute-operator` | | Execute operator address | Generated |

### Using Predefined Operator Addresses

If you need specific operator addresses (e.g., for HSM or external key management):

```bash
adi init \
  --protocol-version v0.30.1 \
  --operator 0x1234567890abcdef1234567890abcdef12345678 \
  --prove-operator 0xabcdef1234567890abcdef1234567890abcdef12 \
  --execute-operator 0x7890abcdef1234567890abcdef1234567890abcd
```

Or via environment variables:

```bash
export ADI_OPERATOR="0x..."
export ADI_PROVE_OPERATOR="0x..."
export ADI_EXECUTE_OPERATOR="0x..."
adi init -p v0.30.1
```

### Generated Directory Structure

After initialization, your state directory contains:

```
~/.adi_cli/state/my_l3_ecosystem/
├── ZkStack.yaml              # Ecosystem metadata
├── configs/
│   └── wallets.yaml          # Ecosystem wallets (deployer, governor)
└── chains/
    └── my-chain/
        ├── ZkStack.yaml      # Chain metadata
        └── configs/
            └── wallets.yaml  # Chain wallets (operators)
```

## Add More Chains

Use the `add` command to create additional L3 chains in your ecosystem. Each chain is independent with its own chain ID, operators, and configuration.

### When to Use Multiple Chains

- **Tenant isolation** - Separate chains for different clients or applications
- **Environment separation** - Dev, staging, production chains in one ecosystem
- **Performance isolation** - High-throughput applications on dedicated chains
- **Regulatory compliance** - Chains with different data residency requirements

### Command

```bash
adi add \
  --chain-name second-chain \
  --chain-id 271
```

With all options:

```bash
adi add \
  --protocol-version v0.30.1 \
  --chain-name second-chain \
  --chain-id 271 \
  --prover-mode no-proofs \
  --evm-emulator \
  --yes
```

### Flags Reference

| Flag | Short | Description | Default |
|------|-------|-------------|---------|
| `--protocol-version` | `-p` | Toolkit version | From config |
| `--chain-name` | | Unique name for the chain | From config |
| `--chain-id` | | Unique numeric identifier | From config |
| `--prover-mode` | | `no-proofs` or `gpu` | `no-proofs` |
| `--base-token-address` | | Custom ERC20 for gas | ETH |
| `--base-token-price-nominator` | | Price ratio numerator | `1` |
| `--base-token-price-denominator` | | Price ratio denominator | `1` |
| `--evm-emulator` | | Enable EVM emulator | `false` |
| `--yes` | `-y` | Skip confirmation prompt | `false` |
| `--force` | `-f` | Overwrite existing chain | `false` |

### Custom Base Token Example

To use a custom ERC20 token for gas payments:

```bash
adi add \
  --chain-name token-chain \
  --chain-id 272 \
  --base-token-address 0x1234567890abcdef1234567890abcdef12345678 \
  --base-token-price-nominator 1 \
  --base-token-price-denominator 1000
```

### Chain Isolation

Each chain added gets its own:
- Directory in `~/.adi_cli/state/<ecosystem>/chains/<chain-name>/`
- Set of operator wallets
- Contract deployments (Diamond proxy, validators)
- Chain ID on the settlement layer

Chains share ecosystem-level infrastructure (Bridgehub, Governance, StateTransitionManager).

## Deploy Contracts

The `deploy` command funds ecosystem wallets and deploys smart contracts to the settlement layer.

### Deployment Phases

{% tabs %}
{% tab title="Phase 1: Wallet Funding" %}
The CLI calculates how much ETH each wallet needs, checks current balances, and transfers funds from your funder wallet.

**Ecosystem-level wallets:**

| Wallet | Purpose | Typical Funding |
|--------|---------|-----------------|
| Deployer | Deploys all smart contracts | 1-2 ETH |
| Governor | Executes governance operations | 1-2 ETH |

**Per-chain wallets:**

| Wallet | Purpose | Typical Funding |
|--------|---------|-----------------|
| Operator | Commits transaction batches (PRECOMMITTER, COMMITTER, REVERTER) | 5-30 ETH |
| Prove Operator | Submits validity proofs (PROVER) | 5-30 ETH |
| Execute Operator | Executes verified batches (EXECUTOR) | 5-30 ETH |
{% endtab %}

{% tab title="Phase 2: Ecosystem Contracts" %}
Deploys shared infrastructure used by all chains:

- **Bridgehub** - Central registry for all chains
- **Governance** - Timelock-based governance contract
- **StateTransitionManager** - Manages chain state transitions
- **ValidatorTimelock** - Enforces delay on validator operations
- **Verifier** - ZK proof verification contract
{% endtab %}

{% tab title="Phase 3: Chain Contracts" %}
For each chain in the ecosystem:

- **Diamond Proxy** - Main chain entry point (upgradeable via facets)
- **Chain Admin** - Administrative operations for the chain
- **DA Validators** - Data availability validation
{% endtab %}

{% tab title="Phase 4: DA Configuration" %}
When `blobs: false` (default for L3), the CLI configures calldata-based data availability:

- Calls `setDAValidatorPair()` on the Diamond proxy
- Sets pubdata source to calldata mode
- Configures the L1 DA validator

{% hint style="info" %}
This step is required because ZKsync-based L2s (like ADI) do not support EIP-4844 blobs.
{% endhint %}
{% endtab %}
{% endtabs %}

### Command

**Always start with a dry run:**

```bash
adi deploy --dry-run
```

This shows the funding plan without executing any transactions:

```
Funding Plan
────────────────────────────────────────
Wallet              Current    Required    Transfer
deployer            0.00 ETH   1.00 ETH    1.00 ETH
governor            0.00 ETH   1.00 ETH    1.00 ETH
operator            0.00 ETH   30.00 ETH   30.00 ETH
prove_operator      0.00 ETH   30.00 ETH   30.00 ETH
execute_operator    0.00 ETH   30.00 ETH   30.00 ETH
────────────────────────────────────────
Total:              92.00 ETH
```

**Execute deployment:**

```bash
export ADI_FUNDER_KEY="0x..."
adi deploy
```

With explicit flags:

```bash
adi deploy \
  --protocol-version v0.30.1 \
  --gas-multiplier 300 \
  --yes
```

### Flags Reference

| Flag | Short | Description | Default |
|------|-------|-------------|---------|
| `--protocol-version` | `-p` | Toolkit version | From config |
| `--dry-run` | | Preview without executing | `false` |
| `--skip-funding` | | Skip wallet funding phase | `false` |
| `--skip-deployment` | | Only fund, do not deploy | `false` |
| `--gas-multiplier` | | Gas price buffer % | `200` |
| `--blobs` | | Override blobs mode | From chain config |
| `--yes` | `-y` | Skip confirmations | `false` |

### Gas Multiplier Guidelines

| Network | Recommended Value | Notes |
|---------|-------------------|-------|
| Local (Anvil) | 200 | Stable gas prices |
| ADI L2 Testnet | 200-300 | Generally stable |
| ADI L2 Mainnet | 300 | Buffer for congestion |

## Next Steps

- [Operations](operations.md) - Verify your deployment and manage state
