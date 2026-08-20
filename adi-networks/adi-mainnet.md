---
description: Get connected to ADI Network Mainnet
---

# ADI Mainnet

{% hint style="success" %}
**Mainnet upgrade complete.** \
The ADI mainnet was successfully upgraded to the new P2P (devp2p) protocol on August 20, 2026 at 15:00 UTC. External nodes still running the old HTTP-replay version have stopped syncing and must be updated to the P2P-enabled version to reconnect.&#x20;

See the [v0.13.0 → v0.20.12 migration guide](https://github.com/ADI-Foundation-Labs/ADI-Stack-EN-Setup-script/blob/main/upgrades/v0.13.0_to_v0.20.12.md) for instructions.
{% endhint %}

<a href="https://adi-foundation-labs.github.io/adi-mainnet-connect/connect.html" class="button primary" data-icon="wallet">Add ADI Network to Wallet</a>

### Manually Add ADI Network

To manually add ADI Network Mainnet as a custom network in your wallet, follow these steps:

1. Find the “Add Network” option in your wallet (in MetaMask, you can find this in the networks dropdown).
2. Click on “Add Network" and then "Add network manually."
3. Fill in the following details for the ADI Network mainnet environment:

#### Network details

<table><thead><tr><th width="267.19921875">PROPERTY</th><th>VALUE</th></tr></thead><tbody><tr><td><strong>Network Name</strong></td><td><code>ADI Network</code></td></tr><tr><td><strong>RPC URL*</strong></td><td><a href="https://rpc.adifoundation.ai/">https://rpc.adifoundation.ai/</a></td></tr><tr><td><strong>Chain ID</strong></td><td><code>36900</code></td></tr><tr><td><strong>Currency Symbol</strong></td><td><code>ADI</code></td></tr><tr><td><strong>Block Explorer URL</strong></td><td><a href="https://explorer.adifoundation.ai/">https://explorer.adifoundation.ai/</a></td></tr><tr><td><strong>Alternative Block Explorer URL</strong></td><td><a href="https://explorer-bls.adifoundation.ai/">https://explorer-bls.adifoundation.ai/</a></td></tr><tr><td><strong>L1 ADI token contract</strong></td><td><a href="https://etherscan.io/address/0x8b1484d57abbe239bb280661377363b03c89caea"><code>0x8b1484d57abbe239bb280661377363b03c89caea</code></a></td></tr><tr><td><strong>Bridgehub</strong></td><td><a href="https://etherscan.io/address/0xcf1c73439c85f7eb9d4439daf398fd6392d176e6"><code>0xcf1c73439c85f7eb9d4439daf398fd6392d176e6</code></a></td></tr><tr><td><strong>Asset Router</strong></td><td><a href="https://etherscan.io/address/0x47eec6f57c7e84391ba6c9ac976537d0db0bb257"><code>0x47eec6f57c7e84391ba6c9ac976537d0db0bb257</code></a></td></tr><tr><td><strong>Protocol version</strong></td><td><code>v0.30.1</code></td></tr><tr><td><strong>Sequencer version</strong></td><td><code>v0.20.12-b1</code></td></tr><tr><td><strong>External node version</strong></td><td><code>v0.20.12-b1</code></td></tr></tbody></table>

\*RPC endpoint is rate-limited. You can also access the RPC through [Alchemy](https://dashboard.alchemy.com/?utm_source=chain_partner\&utm_medium=referral\&utm_campaign=adi) or [NowNodes](https://nownodes.io/nodes/adi-adi). NowNodes does not support WebSocket for ADI. It may return HTTP status errors instead of JSON-RPC error codes.

{% hint style="info" %}
**Building an L3 on ADI?** Send us the IP addresses of your sequencer and prover, and we will whitelist them for full-speed RPC access. See [L3 Chains](../adi-network-components/l3-chains/overview.md).
{% endhint %}

#### Bridging ADI and ETH

You can use the [Bridge](https://bridge.adifoundation.ai/) to bridge ADI and ETH to mainnet.

### Important notes

{% hint style="warning" %}
What is not supported currently:

* ERC-7702
* ERC-4844
* `debug_traceCall` RPC call with custom tracers
{% endhint %}
