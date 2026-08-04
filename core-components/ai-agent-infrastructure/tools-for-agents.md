---
description: MCP servers, agent frameworks, and protocols for building agents on ADI Chain.
---

# Tools for Agents

ADI Chain is EVM-equivalent. Every MCP server and agent framework built for Ethereum works here by pointing it at an ADI RPC endpoint.

This page covers agent-specific tooling not documented elsewhere. For network setup, see [ADI Testnet](../../adi-networks/adi-testnet.md).

{% hint style="info" %}
Use existing Ethereum integrations without an ADI-specific adapter. Configure the RPC URL and chain ID.
{% endhint %}

### MCP servers

Model Context Protocol (MCP) servers give agents blockchain data and onchain actions. Any EVM-compatible MCP server works with ADI Chain.

| MCP server                                                            | What it does                                                                                                                         | ADI setup                                                                                                                |
| --------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------ |
| [EVM MCP Server](https://github.com/mcpdotdirect/evm-mcp-server)      | Provides 22 tools across 60+ EVM networks. Read and write contracts, transfer tokens, query balances, and fetch ABIs from explorers. | Add ADI Testnet (`99999`) or Mainnet (`36900`) through `CUSTOM_NETWORKS`. Use `https://rpc.ab.testnet.adifoundation.ai`. |
| [EVM MCP](https://github.com/JamesANZ/evm-mcp)                        | Provides 20+ RPC methods, contract calls, gas estimation, and transaction simulation with state overrides.                           | Set `CUSTOM_PROVIDERS` to an ADI RPC URL. No chain-specific configuration is needed.                                     |
| [Ethereum JSON-RPC MCP](https://github.com/john0n1/ethereum-mcp)      | Proxies an Ethereum JSON-RPC endpoint as MCP tools. Includes admin, debug, and txpool tools.                                         | Point it at an ADI RPC endpoint.                                                                                         |
| [Blockscout MCP](https://github.com/blockscout/mcp-server)            | Exposes balances, tokens, NFTs, transactions, and contract verification.                                                             | Use the [ADI Testnet explorer](https://explorer.ab.testnet.adifoundation.ai).                                            |
| [The Graph MCP](https://thegraph.com/docs/en/subgraphs/querying/mcp/) | Queries indexed blockchain data through subgraphs.                                                                                   | Deploy or use subgraphs indexing ADI Chain contracts.                                                                    |
| [GOAT SDK MCP](https://github.com/goat-sdk/goat)                      | Provides 200+ onchain actions across EVM chains.                                                                                     | Point it at an ADI RPC endpoint.                                                                                         |

#### Run EVM MCP Server on ADI Testnet

```json
{
  "mcpServers": {
    "evm": {
      "command": "npx",
      "args": ["-y", "@mcpdotdirect/evm-mcp-server"],
      "env": {
        "CUSTOM_NETWORKS": "[{\"name\":\"ADI Testnet\",\"slug\":\"adi-testnet\",\"chainId\":99999,\"rpc\":\"https://rpc.ab.testnet.adifoundation.ai\",\"explorer\":\"https://explorer.ab.testnet.adifoundation.ai\"}]",
        "DEFAULT_NETWORK": "adi-testnet"
      }
    }
  }
}
```

#### Build a custom MCP server

For deeper integration, wrap ERC-8004 registry interactions with the MCP SDK. The `@adi-devtools/sdk` provides typed providers and contract ABIs.

Expose tools such as:

* `register_agent(uri)` — register an agent on the Identity Registry.
* `query_reputation(agentId, tags)` — query the Reputation Registry.
* `request_validation(validator, agentId, uri, hash)` — request validation.
* `get_balance(address)` — check an ADI balance.
* `call_contract(address, abi, function, args)` — call any contract.

See [Registration & Agent Lifecycle](registration-and-agent-lifecycle.md) for registry behavior and deployed Mainnet contracts.

### Agent frameworks and protocols

| Framework                                                                                                       | ADI support                          | Notes                                                                                                                          |
| --------------------------------------------------------------------------------------------------------------- | ------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------ |
| [Eliza](https://github.com/elizaOS/eliza)                                                                       | Full EVM support                     | Configure the character file with an ADI Testnet RPC endpoint and funded wallet. Agents can interact with ERC-8004 registries. |
| [LangChain](https://github.com/langchain-ai/langchain) / [LangGraph](https://github.com/langchain-ai/langgraph) | Use `ethers` or `viem` tools         | Point the tool integration at an ADI RPC endpoint. No adapter is needed.                                                       |
| [CrewAI](https://github.com/crewAIInc/crewAI)                                                                   | Custom EVM tools                     | Add a tool that wraps `ethers` or `viem` calls to ADI Chain.                                                                   |
| [A2A](https://a2a-protocol.org/)                                                                                | Identity Registry endpoint discovery | Advertise an A2A endpoint URI in the registration file. Counterparties discover it through `agentURI` resolution.              |
| MCP                                                                                                             | Registration-file endpoint URIs      | Agents can expose MCP servers and discover each other's MCP endpoints through the Identity Registry.                           |

{% hint style="info" %}
Registration files support A2A, MCP, DID, ENS, and email service endpoints. Publish these endpoints with the agent identity.

For agents using ERC-8004 and `did:adi` (Apeiro/IDA), cross-link through the registration file's `services[]` array and the DID Document's `service[]` array. See the [ERC-8004 specification](https://eips.ethereum.org/EIPS/eip-8004) for the registration file format.
{% endhint %}
