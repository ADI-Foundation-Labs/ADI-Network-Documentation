---
description: Verify Solidity contracts on ADI Mainnet and ADI Network AB Testnet.
---

# Verify smart contracts

Verification is specific to each explorer. It is not a property of the chain.

Operational details on this page were measured 2026-11-23.

### Choose an explorer

| Network                                                                         | Explorer type                                                          | Chain ID | Verification API                                                                  |
| ------------------------------------------------------------------------------- | ---------------------------------------------------------------------- | -------- | --------------------------------------------------------------------------------- |
| [Primary mainnet explorer](https://explorer.adifoundation.ai)                   | Custom ADI explorer                                                    | `36900`  | `https://explorer-api.adifoundation.ai/api?module=contract&action=...`            |
| [BLS mainnet explorer](https://explorer-bls.adifoundation.ai)                   | Blockscout v12.0.4 with smart-contract-verifier and a Sourcify backend | `36900`  | `https://explorer-bls.adifoundation.ai/api`                                       |
| [ADI Network AB Testnet explorer](https://explorer.ab.testnet.adifoundation.ai) | Custom ADI explorer                                                    | `99999`  | `https://explorer-api.ab.testnet.adifoundation.ai/api?module=contract&action=...` |

Only `explorer-bls.adifoundation.ai` is Blockscout. The testnet explorer and primary mainnet explorer expose an Etherscan-compatible `/api?module=contract&action=...` surface. They have no Blockscout `/api/v2`.

EntryPoint v0.7, `0x0000000071727De22E5E9d8BAf0edAc6f37da032`, is verified on the BLS explorer and unverified on `explorer-api.adifoundation.ai`. Use the BLS explorer for mainnet verification.

The primary mainnet API accepts `POST /api?module=contract&action=verifysourcecode` and requires no API key. So does the testnet API. Both expect `contractname` to be fully qualified as `path/File.sol:ContractName`. A bare contract name is rejected with `Missing Or invalid contractname`. Standard Foundry and Hardhat verification requests already send the qualified name, so they work unmodified against all three explorers. (Measured 2026-11-23)

Both verification endpoints respond and require no API key. No rate limit is published. Blockscout returns `x-ratelimit-limit: -1` and absorbed a 100-request burst without throttling. This is an observation, not a guarantee. (Measured 2026-11-23)

Blockscout v2 list endpoints cap `items_count` at 100. The Etherscan-compatible subset caps `page` × `offset` at 1000. The verifier supports Solidity, Vyper, multi-part, standard JSON input, and flattened source. Supported solc versions run up to `v0.8.39`. (Measured 2026-11-23)

### Verify in the explorer

1. Open the explorer for your network: the BLS explorer for mainnet, or the ADI Network AB Testnet explorer for testnet.
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

If `--verifier blockscout` fails against the testnet endpoint, post directly to `https://explorer-api.ab.testnet.adifoundation.ai/api?module=contract&action=verifysourcecode`. It is live and returns Etherscan-style JSON.

### Hardhat

Configure `hardhat-verify` with custom ADI network definitions. Empty API keys are correct for these endpoints.

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

### Sourcify

{% hint style="info" %}
Submit Sourcify metadata for portability. The BLS explorer runs a Sourcify backend.
{% endhint %}

### Network endpoints

| Network                | RPC URL                                   | Explorer API URL                                       |
| ---------------------- | ----------------------------------------- | ------------------------------------------------------ |
| ADI Mainnet            | `https://rpc.adifoundation.ai`            | `https://explorer-bls.adifoundation.ai/api`            |
| ADI Network AB Testnet | `https://rpc.ab.testnet.adifoundation.ai` | `https://explorer-api.ab.testnet.adifoundation.ai/api` |
