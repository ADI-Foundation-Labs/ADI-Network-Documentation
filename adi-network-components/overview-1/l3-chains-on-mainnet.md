---
description: Production L3 chains live on mainnet
---

# L3 Chains on Mainnet

## L3 chains on ADI Mainnet

ADI Mainnet hosts two production L3 chains. Both settle on ADI Mainnet, chain ID `36900`. Both use ADI as their native gas token.

<table><thead><tr><th>Chain</th><th width="115.8671875">Chain ID</th><th>Use case</th><th>Runtime</th></tr></thead><tbody><tr><td>Good Energy (Esyasoft)</td><td><code>999</code></td><td>Energy data anchoring</td><td><code>zksync-os/v0.13.0</code></td></tr><tr><td>Apeiro</td><td><code>37001</code></td><td>Healthcare claims verification</td><td><code>zksync-os/v0.13.0</code></td></tr></tbody></table>

### Good Energy (Esyasoft)

Good Energy is the first zkSync OS L3 to reach mainnet. It anchors energy data onchain. The sequencer is producing blocks.

| Property         | Value                         |
| ---------------- | ----------------------------- |
| Chain ID         | `999`                         |
| Settlement layer | ADI Mainnet, chain ID `36900` |
| Native gas token | ADI                           |
| Runtime          | `zksync-os/v0.13.0`           |

#### Network services

* **Explorer:** [Good Energy Explorer](https://goodenergyexplorer.com)
* **Explorer API:** [Good Energy Explorer API](https://api.goodenergyexplorer.com)
* **Bridge:** [Good Energy Bridge](https://bridge.goodenergyexplorer.com)

### Apeiro

Apeiro verifies healthcare claims on a dedicated L3. The application has active production traffic.

As of July 21, 2026, the chain had processed more than 2,700 transactions across more than 45 addresses.

| Property         | Value                         |
| ---------------- | ----------------------------- |
| Chain ID         | `37001`                       |
| Settlement layer | ADI Mainnet, chain ID `36900` |
| Native gas token | ADI                           |
| Runtime          | `zksync-os/v0.13.0`           |

#### Network services

* **Explorer:** [Apeiro Explorer](https://explorer.apeiro.adifoundation.ai)
* **Blockscout Explorer:** [Apeiro Blockscout](https://explorer-bls.apeiro.adifoundation.ai)
* **Bridge:** [Apeiro Bridge](https://bridge.apeiro.adifoundation.ai)

### Related documentation

See [L3 Chains](../l3-chains/overview.md) for L3 architecture and settlement flow. See [Configuration Reference](../l3-chains/run-a-rollup/configuration-reference.md) for the settlement-RPC configuration.
