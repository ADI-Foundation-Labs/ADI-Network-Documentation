---
description: >-
  Supported account abstraction paths, paymaster options, and deployed entry
  points.
---

# Gas Abstraction & Account Abstraction

You can use either native EVM flows or overlay ERC-4337-style account abstraction.

Operational details on this page were measured 2026-11-23.

### Entry points

| Version | Address                                      | Both networks | Notes                                                                                            |
| ------- | -------------------------------------------- | ------------- | ------------------------------------------------------------------------------------------------ |
| v0.6    | `0x5FF137D4b0FDCD49DcA30c7CF57E578a026d2789` | yes           | Legacy, present but not recommended.                                                             |
| v0.7    | `0x0000000071727De22E5E9d8BAf0edAc6f37da032` | yes           | Byte-identical to the canonical Ethereum deployment. Source verified on Blockscout, solc 0.8.23. |
| v0.8    | `0x4337084d9e255ff0702461cf8895ce9e3b5ff108` | yes           | Deployed per network. Bytecode differs only in the embedded chain ID.                            |

### Bundler and paymaster support

* **ADI does not operate an ERC-4337 bundler, and the RPC is not a bundler endpoint.** `eth_sendUserOperation` and `eth_supportedEntryPoints` return `-32601 Method not found` on both mainnet and testnet. (Measured 2026-11-23)
* **ADI does not operate an official paymaster.** There is no ADI-funded sponsorship program.
* To use ERC-4337, run your own bundler or use a third-party provider. Before committing, verify support for chain ID `36900` on mainnet or `99999` on testnet. ADI does not confirm, endorse, or support any provider's integration, limits, or terms.
* ADI operates no bundler. There is no ADI rate limit, operator, or support model to document.

{% hint style="info" %}
`debug_traceCall` is disabled on mainnet. Measured 2026-11-23, it returns `-32601 Method disabled`. It works on testnet with `callTracer`. Bundlers that require it in safe mode fail on mainnet.
{% endhint %}

See [finality.md](../references/json-rpc-api/finality.md "mention") before releasing value against L2 state.

### EIP-7702

EIP-7702 is testnet-only pending the main-node upgrade. See [eip-7702-on-adi-chain.md](eip-7702-on-adi-chain.md "mention").
