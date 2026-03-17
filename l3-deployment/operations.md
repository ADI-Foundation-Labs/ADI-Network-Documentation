---
description: Verify deployments and manage ecosystem state
---

# Operations

This page covers post-deployment verification and state management.

## Verify Deployment

After deployment, verify your ecosystem is correctly set up.

### View Ecosystem Information

```bash
# Ecosystem-level contracts
adi ecosystem
```

Output:

```
Ecosystem: my_l3_ecosystem
Settlement Layer: https://rpc.adi.network
Protocol Version: v0.30.1

Contracts:
  Bridgehub:              0x1234...abcd
  Governance:             0x5678...efgh
  StateTransitionManager: 0x9abc...ijkl
  ValidatorTimelock:      0xdef0...mnop
  ...
```

```bash
# Include chain-level contracts
adi ecosystem --chain my-chain
```

### View Contract Owners

```bash
# Ecosystem contract owners
adi owners

# Include chain contract owners
adi owners --chain my-chain
```

Output:

```
Contract                 Owner
───────────────────────────────────────
Governance               0x1234...abcd (governor)
Bridgehub                0x1234...abcd (governance)
ValidatorTimelock        0x1234...abcd (governor)
...
```

## State Directory

All ecosystem state is stored in `~/.adi_cli/state/`. Understanding this structure helps with debugging and backup.

### Directory Structure

```
~/.adi_cli/state/
└── my_l3_ecosystem/
    ├── ZkStack.yaml                    # Ecosystem metadata
    ├── configs/
    │   ├── wallets.yaml                # Ecosystem wallets (deployer, governor)
    │   ├── contracts.yaml              # Deployed contract addresses
    │   ├── initial_deployments.yaml    # Deployment configuration
    │   └── erc20_deployments.yaml      # Token deployments
    └── chains/
        ├── my-chain/
        │   ├── ZkStack.yaml            # Chain metadata
        │   └── configs/
        │       ├── wallets.yaml        # Operator wallets
        │       ├── contracts.yaml      # Chain contract addresses
        │       ├── genesis.yaml        # Chain genesis config
        │       └── general.yaml        # General settings
        └── second-chain/
            └── ...
```

### Key Files

| File | Contains | Security |
|------|----------|----------|
| `wallets.yaml` | Wallet addresses and private keys | **Keep secure** |
| `contracts.yaml` | Deployed contract addresses | Public after deployment |
| `ZkStack.yaml` | Ecosystem/chain metadata | Public |
| `genesis.yaml` | Chain genesis configuration | Public |

{% hint style="warning" %}
The `wallets.yaml` files contain private keys. Ensure proper file permissions and never commit these to version control.
{% endhint %}

## Backup and Restore

### Manual Backup

```bash
tar -czf ecosystem-backup.tar.gz ~/.adi_cli/state/my_l3_ecosystem/
```

### S3 Sync

With S3 sync enabled in your [configuration](configuration.md):

```bash
# Manually sync to S3
adi state sync --ecosystem-name my_l3_ecosystem

# Restore from S3
adi state restore --ecosystem-name my_l3_ecosystem

# Force restore (overwrite local)
adi state restore --ecosystem-name my_l3_ecosystem --force
```

## Next Steps

- [Troubleshooting](troubleshooting.md) - Common issues and solutions
- [L3 Chains](../adi-network-components/l3-chains.md) - Architecture reference
