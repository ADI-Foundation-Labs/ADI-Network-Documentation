---
description: Common issues and solutions for L3 deployments
---

# Troubleshooting

This page covers common issues encountered during L3 deployment and their solutions.

## Docker Connection Issues

**Error:** `Cannot connect to Docker daemon`

```bash
# Check Docker is running
docker info

# On Linux, ensure user is in docker group
sudo usermod -aG docker $USER
# Log out and back in

# On macOS, start Docker Desktop
open -a Docker
```

## Insufficient Funds

**Error:** `Insufficient funds for transfer`

Your funder wallet needs enough ETH to cover all wallet funding plus gas costs.

```bash
# Check funder balance
cast balance <FUNDER_ADDRESS> --rpc-url https://rpc.adi.network

# Run dry-run to see total required
adi deploy --dry-run
```

{% hint style="info" %}
The dry-run output shows the total ETH required across all wallets. Ensure your funder wallet has this amount plus a buffer for gas costs.
{% endhint %}

## Chain ID Already Exists

**Error:** `Chain ID already registered on settlement layer`

The chain ID must be unique. Check existing chains:

```bash
# List chains in your ecosystem
ls ~/.adi_cli/state/my_l3_ecosystem/chains/

# Choose a different chain ID
adi add --chain-name new-chain --chain-id 999
```

## RPC Connection Failures

**Error:** `Failed to connect to RPC endpoint`

```bash
# Test RPC connectivity
curl -X POST https://rpc.adi.network \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","method":"eth_chainId","params":[],"id":1}'

# Check for firewall/VPN issues
# Verify the RPC URL in your config
adi config
```

## Docker Image Pull Failures

**Error:** `Failed to pull image`

```bash
# Check Docker registry access
docker pull harbor-v2.dev.internal.adifoundation.ai/adi-chain/cli/adi-toolkit:v0.30.1

# If behind corporate firewall, configure Docker proxy
# Edit ~/.docker/config.json
```

## Next Steps

After successful deployment:

- **Accept Ownership** - Accept pending ownership transfers for deployed contracts
- **Transfer Ownership** - Transfer ownership to a multisig or governance contract
- **Run a Node** - Set up and run your L3 node
- **Verify Contracts** - Submit contracts for verification on block explorers
- **Server Parameters** - Get environment variables for running L3 server

{% hint style="info" %}
For conceptual background on L3 architecture, see [L3 Chains](../adi-network-components/l3-chains.md).
{% endhint %}
