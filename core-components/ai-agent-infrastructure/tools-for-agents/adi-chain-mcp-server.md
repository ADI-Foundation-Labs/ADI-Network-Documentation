# ADI Chain MCP Server

## ADI Chain MCP Server

> An MCP server exposing ADI Chain bridging, funding-path planning, and network data as executable tools for AI agents. Canonical bridge (deposit/withdraw/claim), Chainlink CCIP transfers, and ERC-8004 agent profile reads, backed by a domain model verified against live chain state.

### Overview

The server encodes ADI's cross-chain domain knowledge as a typed model and exposes it through the Model Context Protocol. It exists because route selection on ADI is constraint-driven, not liquidity-driven: `nativeTokenBridgingOnly` on the Native Token Vault, a CCIP burn/mint USDC.e with no on-chain exit liquidity, and no DEX on ADI Mainnet. An agent without this model will route into dead ends. The server refuses them at the tool boundary and states why.

Two operating modes, enforced at registration:

* **Read-only** (default): 13 read/query/advisory tools. No signing capability exists; the write tools are not registered.
* **Signer** (`ADI_MCP_PRIVATE_KEY` env): adds four write tools. The key is held in memory for the process lifetime, never persisted, never logged.

### Trust model

| Concern                | Handling                                                                                                                                       |
| ---------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------- |
| Key storage            | In-memory only, from env. No file writes, no logs                                                                                              |
| Withdrawal claims      | Idempotent: `claim_withdrawal` checks finalization state via the Nullifier before submitting and skips already-claimed withdrawals             |
| Advisory vs executable | Swap services (SimpleSwap, LetsExchange, ChangeHero) are advisory only; the server cannot and does not execute them                            |
| Route guarantees       | `plan_funding_path` ranks routes with explicit caveats; constraints are encoded in the domain model, not prompted in                           |
| Address integrity      | All contract addresses and ABIs verified against live RPC (`zks_getBridgehubContract`), the Chainlink CCIP directory, and the bridge UI bundle |

### Networks

|                       | Mainnet                                      | Testnet (AB)                                   |
| --------------------- | -------------------------------------------- | ---------------------------------------------- |
| Chain ID              | `36900`                                      | `99999`                                        |
| L2 RPC                | `https://rpc.adifoundation.ai`               | `https://rpc.ab.testnet.adifoundation.ai`      |
| L1                    | Ethereum                                     | Ethereum Sepolia                               |
| Explorer              | `https://explorer.adifoundation.ai`          | `https://explorer.ab.testnet.adifoundation.ai` |
| L1 ADI (ERC-20)       | `0x8b1484d57abbe239bb280661377363b03c89caea` | `0x2a98B46fe31BA8Be05ef1cE3D36e1f80Db04190D`   |
| L1 Bridgehub          | `0x7a38c18a229ef8a0ae7104ba272a46280f2d59cb` | `0xFd3cE61C65dDd1039e6e9e07FB6D6e16388d1cc7`   |
| L1 Asset Router       | `0xf25227efad2046d19777a4ca540b5c016df7fe7a` | `0x5BA5B3a62745cD4eC5a59A22c988b2Dc2ae06573`   |
| L1 Native Token Vault | `0x0a0f8912162ff83a036883dbada42eff647a3065` | not published                                  |
| L1 Nullifier          | `0x5e5a72077dfb354dfe61200b8f31fa491f9b9cea` | not published; claims unsupported on testnet   |
| CCIP Router           | `0x010771998A1F4736BD844939d0bf01ac5cA0f8fa` | `0x5D445DF89674096B6A138565cAE955FF816f352D`   |
| CCIP Chain Selector   | `4059281736450291836`                        | `9418205736192840573`                          |

L2 system contracts are the standard zksync-os addresses: base token `0x…800A`, L2 Asset Router `0x…10003`, L2 Native Token Vault `0x…10004`.

Deployed configuration diverges from in-repo contract code; this domain model is maintained against live chain state, not repo source. Re-verify after any bridge protocol upgrade.

### Tool surface

#### Query

| Tool                    | Returns                                                                                                   |
| ----------------------- | --------------------------------------------------------------------------------------------------------- |
| `get_network_info`      | RPC, chain ID, explorer, bridge and CCIP contract addresses                                               |
| `get_token_info`        | Token addresses (L1/L2), decimals, bridge support                                                         |
| `get_balance`           | Native ADI or ERC-20 balance on ADI Chain                                                                 |
| `get_bridge_options`    | Viable bridges for a token route with per-bridge constraints                                              |
| `get_ccip_lanes`        | Live CCIP lanes and supported tokens per lane                                                             |
| `get_bridge_status`     | Transaction finality: pending (sealed) → committed (on L1) → executed                                     |
| `get_finality_status`   | Node finality frontiers: last sealed/committed/executed block and batch                                   |
| `estimate_deposit`      | L1 base cost (`mintValue`) of a canonical deposit, in ADI                                                 |
| `get_withdrawal_params` | Claim parameters reconstructed from an executed batch: batch number, message index, message, Merkle proof |
| `list_withdrawals`      | An address's L2→L1 withdrawals with stage and claim status; stateless, derived from on-chain state        |
| `get_agent_profile`     | ERC-8004 identity, reputation, and validation summary (mainnet only; testnet registries are not deployed) |

#### Advisory

| Tool                | Returns                                                               |
| ------------------- | --------------------------------------------------------------------- |
| `plan_funding_path` | Ranked routes onto ADI Chain with steps, costs, and caveats           |
| `get_swap_services` | Non-custodial swap services delivering ADI directly to an ADI address |

#### State-changing (signer required)

| Tool               | Effect                                                                                                                                                                            |
| ------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `bridge_deposit`   | Approves the Native Token Vault (ADI) or Asset Router (ERC-20), deposits via `requestL2TransactionDirect` on Bridgehub. Returns L1 and canonical L2 hashes. L2 finality in \~15 s |
| `bridge_withdraw`  | L2 → L1 withdrawal of native ADI or a bridged ERC-20. Commit → prove → execute, \~75 min, then L1 claim                                                                           |
| `claim_withdrawal` | Submits the claim on L1 via the Nullifier using params from `get_withdrawal_params` or rebuilt from a tx hash                                                                     |
| `ccip_transfer`    | `ccipSend` on the CCIP router for USDC.e, LINK (mainnet) or CCIP-BnM (testnet)                                                                                                    |

Deposit gas defaults: L2 gas limit 1,000,000 with `gasPerPubdataByteLimit` 800, matching the bridge UI. `estimate_deposit` returns the required L1 value before you commit funds.

### Flows

**Canonical funding cycle** (the reliable path to native ADI):

1. `plan_funding_path { network: "mainnet", target: "native-ADI", source: "USDC" }` returns ranked routes with cost and time estimates.
2. `bridge_deposit` moves L1 ADI ERC-20 to native ADI. Approve + deposit in one call; the canonical L2 hash tracks via `get_bridge_status`.
3. `bridge_withdraw` reverses direction. After \~75 min the batch executes on L1; `get_withdrawal_params` rebuilds the Merkle proof, `claim_withdrawal` settles through the Nullifier. `list_withdrawals` sweeps an address for claimable withdrawals without local state.

**CCIP lane transfer**: USDC.e and LINK move over mainnet lanes to Ethereum, Arbitrum One, Base, and Solana. The server rejects tokens without a CCIP pool rather than constructing a doomed `ccipSend`.

**The stablecoin constraint**: USDC.e on ADI Mainnet is a CCIP burn/mint token. The canonical bridge cannot touch it, and with no on-chain DEX liquidity it cannot be swapped to native ADI in place. `plan_funding_path` models this honestly: CCIP back to Ethereum → swap → canonical deposit, or an advisory swap service delivering ADI directly.

### Encoded constraints

Enforced in `src/domain.ts`; routing and error messages derive from these:

* Mainnet runs `nativeTokenBridgingOnly`: only ADI is registered in the Native Token Vault. Two-Bridges ERC-20 deposits are protocol-supported but unconfigured; the server rejects them and points to CCIP.
* USDC.e moves only via CCIP lanes. Canonical bridge interaction is impossible by token-pool design.
* No on-chain DEX with liquidity exists on ADI Mainnet.
* Testnet CCIP carries only CCIP-BnM. USDC is not on testnet lanes, so stablecoin test flows cannot run end-to-end via CCIP.
* Testnet is CCIP-only for writes. Canonical deposit, withdraw, and claim are rejected because the canonical bridge and L1 Nullifier are unpublished there (addresses are `0x0`).

### Architecture

```
src/
  cli.ts        stdio MCP server entry
  tools.ts      tool definitions, zod schemas, registration
  domain.ts     networks, tokens, bridges, constraints (source of truth)
  routing.ts    funding-path planner
  providers.ts  viem clients, Blockscout v1 API, zks_* RPC, L1 fallbacks
  bridge.ts     deposit/withdraw/claim construction
  ccip.ts       ccipSend construction
  erc8004.ts    ERC-8004 registry reads
  abis/         vendored ABIs, verified against deployed contracts
```

TypeScript on `@modelcontextprotocol/sdk` and viem. Runs over stdio; any MCP client works. Tests: unit (`bun test`), live smoke against mainnet RPC (`bun run test/smoke.ts`), write-path construction (`bun run test/write.ts`), and a live deposit → withdraw → claim round trip (`test/live.ts`).

### Install

Build from source off the tagged release — read the source before running it.

```bash
git clone https://github.com/ADI-Foundation-Labs/adi-mcp-server.git
cd adi-mcp-server
git checkout v1.0.1
bun i -g .          # or: npm i -g .
adi-mcp-server      # stdio MCP server
```

```json
{
  "mcpServers": {
    "adi": {
      "command": "adi-mcp-server",
      "env": { "ADI_MCP_PRIVATE_KEY": "0x..." }
    }
  }
}
```

Update: `git pull && git checkout v1.0.1 && bun i -g .`.

### Links

* Repository: [https://github.com/ADI-Foundation-Labs/adi-mcp-server](https://github.com/ADI-Foundation-Labs/adi-mcp-server)
* Release (v1.0.1): [https://github.com/ADI-Foundation-Labs/adi-mcp-server/releases/tag/v1.0.1](https://github.com/ADI-Foundation-Labs/adi-mcp-server/releases/tag/v1.0.1)
* Bridge UI: [https://bridge.adifoundation.ai](https://bridge.adifoundation.ai)
* Transporter (CCIP): [https://app.transporter.io](https://app.transporter.io)
* Explorer: [https://explorer.adifoundation.ai](https://explorer.adifoundation.ai)
