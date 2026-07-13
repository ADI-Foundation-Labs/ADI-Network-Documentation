---
description: Verify Solidity contracts on ADI Mainnet and ADI Network AB Testnet.
---

# Verify smart contracts

Use the working Blockscout explorers to publish and verify contract source code. Blockscout is open, so verification does not require an API key.

### Choose an explorer

| Network                                                                         | Verification status | Chain ID | API URL                                                |
| ------------------------------------------------------------------------------- | ------------------- | -------- | ------------------------------------------------------ |
| [Primary mainnet explorer](https://explorer.adifoundation.ai)                   | Broken              | `36900`  | —                                                      |
| [BLS mainnet explorer](https://explorer-bls.adifoundation.ai)                   | Working             | `36900`  | `https://explorer-bls.adifoundation.ai/api`            |
| [ADI Network AB Testnet explorer](https://explorer.ab.testnet.adifoundation.ai) | Working             | `99999`  | `https://explorer-api.ab.testnet.adifoundation.ai/api` |

Use the BLS explorer for ADI Mainnet. Use the testnet explorer for ADI Network AB Testnet. The primary mainnet explorer cannot verify contracts.

The BLS explorer runs Blockscout with the smart-contract-verifier and Sourcify backend.

### Verify in the explorer

1. Open the working explorer for your network.
2. Paste the deployed contract address into the search field.
3. Open the **Contract** tab.
4. Select **Verify & Publish**.
5. Select the compiler version and license.
6. Upload your source using one of these formats:
   * A single Solidity file.
   * Multiple source files.
   * Standard JSON input.

Use the exact compiler version, optimizer settings, and constructor arguments from deployment.

### Foundry

Configure the BLS explorer in `foundry.toml`. The empty `key` is intentional. Blockscout does not require an API key.

```toml
[etherscan]
adi = { key = "", url = "https://explorer-bls.adifoundation.ai/api", chain = 36900 }
```

Deploy and verify on ADI Mainnet:

```bash
forge create --verify --chain 36900 src/Contract.sol:Contract
```

Or explicitly select Blockscout:

```bash
forge create --verify --verifier blockscout \
  --verifier-url https://explorer-bls.adifoundation.ai/api \
  --chain 36900 src/Contract.sol:Contract
```

For ADI Network AB Testnet, use chain ID `99999` and its API endpoint:

```toml
[etherscan]
adiTestnet = { key = "", url = "https://explorer-api.ab.testnet.adifoundation.ai/api", chain = 99999 }
```

```bash
forge create --verify --verifier blockscout \
  --verifier-url https://explorer-api.ab.testnet.adifoundation.ai/api \
  --chain 99999 src/Contract.sol:Contract
```

### Hardhat

Configure `hardhat-verify` with custom ADI network definitions. Empty API keys are correct for Blockscout.

```javascript
require("@nomicfoundation/hardhat-verify");

module.exports = {
  networks: {
    adi: {
      url: "https://rpc.adifoundation.ai",
      chainId: 36900,
    },
    adiTestnet: {
      url: "https://rpc.ab.testnet.adifoundation.ai",
      chainId: 99999,
    },
  },
  etherscan: {
    apiKey: {
      adi: "",
      adiTestnet: "",
    },
    customChains: [
      {
        network: "adi",
        chainId: 36900,
        urls: {
          apiURL: "https://explorer-bls.adifoundation.ai/api",
          browserURL: "https://explorer-bls.adifoundation.ai",
        },
      },
      {
        network: "adiTestnet",
        chainId: 99999,
        urls: {
          apiURL: "https://explorer-api.ab.testnet.adifoundation.ai/api",
          browserURL: "https://explorer.ab.testnet.adifoundation.ai",
        },
      },
    ],
  },
};
```

Then run:

```bash
npx hardhat verify --network adi <CONTRACT_ADDRESS> <CONSTRUCTOR_ARGUMENTS>
```

Verify on ADI Network AB Testnet:

```bash
npx hardhat verify --network adiTestnet <CONTRACT_ADDRESS> <CONSTRUCTOR_ARGUMENTS>
```

The testnet network name is `ADI Network AB Testnet`. Its chain ID is `99999`.

You can also use the `blockscout-hardhat-verify` plugin. It supports Blockscout verification directly.

### Sourcify

Sourcify provides an additional verification backend for the BLS mainnet explorer. You can submit matching contract metadata and source files through Sourcify. The explorer can then display the verified result.

### Network endpoints

| Network                | RPC URL                                   | Explorer API URL                                       |
| ---------------------- | ----------------------------------------- | ------------------------------------------------------ |
| ADI Mainnet            | `https://rpc.adifoundation.ai`            | `https://explorer-bls.adifoundation.ai/api`            |
| ADI Network AB Testnet | `https://rpc.ab.testnet.adifoundation.ai` | `https://explorer-api.ab.testnet.adifoundation.ai/api` |
