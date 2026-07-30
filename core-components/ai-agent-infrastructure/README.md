---
description: >-
  Trusted, verifiable AI agents and ERC-8004 registries for institutional
  deployments
---

# AI Agent Infrastructure

ADI Chain provides a production-ready infrastructure layer for trusted, verifiable AI agents operating across organizational boundaries. It combines private transaction flow, jurisdiction-aware L3s, and onchain registries for agent identity, reputation, and validation.

{% hint style="info" %}
Best for institutional decision-makers, enterprise architects, government technology teams, and compliance officers evaluating blockchain-based agent infrastructure.
{% endhint %}

## Getting started

#### Third-party testnet integration: Pearl Digital P3

Pearl Digital operates the Pearl Path Protocol (P3), a third-party test environment available on ADI Testnet (`99999`). P3 provides permissioned agent onboarding, Know Your Agent (KYA) controls, delegation credentials, agent discovery, reputation integration, rolling transfer limits, x402 payments, and test payment assets.

Developers can use the following Pearl-operated resources:

* [P3 Agentic Registry API](https://testagentic.pearldigital.com/docs)
* [Supported chains and deployed contract addresses](https://testagentic.pearldigital.com/api/v1/chains)
* [Pearl testnet faucet and onboarding service](https://faucet.pearldigital.com/)

{% hint style="warning" %}
Pearl P3 is a third-party testnet integration operated by Pearl Digital. Its contracts are separate from ADI Chain’s canonical ERC-8004 Identity, Reputation, and Validation registries. Availability, contract addresses, test assets, and access requirements may change while the service remains in public testnet.
{% endhint %}

**Running an x402 facilitator**

An x402 facilitator verifies signed payment authorizations and submits settlement transactions onchain. Facilitators are optional and independently operated. ADI Foundation does not operate a public facilitator.

Hosted x402 facilitators may not support ADI Testnet. Self-host a facilitator with the official [x402 reference implementations](https://github.com/x402-foundation/x402/tree/main/examples) when you need ADI support. x402 supports arbitrary EIP-155 networks; see the [network support](https://docs.x402.org/core-concepts/network-and-token-support) and [facilitator](https://docs.x402.org/core-concepts/facilitator) documentation.

For ADI Testnet, configure:

| Setting            | Value                                           |
| ------------------ | ----------------------------------------------- |
| Network identifier | `eip155:99999`                                  |
| RPC URL            | `https://rpc.ab.testnet.adifoundation.ai/`      |
| Gas token          | `ADI`                                           |
| Explorer           | `https://explorer.ab.testnet.adifoundation.ai/` |

Use a dedicated settlement signer with sufficient ADI for transaction fees. Start with assets that support EIP-3009 `transferWithAuthorization`, such as Pearl’s PRLUSD and FlashD test tokens. The x402 Permit2 settlement proxy contracts are not currently deployed on ADI Testnet. Do not assume arbitrary ERC-20 payments work through Permit2.

#### For institutions

1. Read the [ERC-8004 specification](https://github.com/ADI-Foundation-Labs/ERC-8004-Contracts/blob/main/ERC8004SPEC.md).
2. Explore the deployed contracts on [ADI Mainnet](../../adi-networks/adi-mainnet.md).
3. Check current availability for [ADI Testnet](../../adi-networks/adi-testnet.md).
4. Evaluate whether agents should run on ADI L2 or a dedicated [L3 chain](../../adi-network-components/l3-chains/overview.md).
5. Use the [ADI DLT Framework](../../adi-dlt-framework.md) to assess governance and operational controls.

#### For developers

* Register an agent with `register(agentURI)` on the Identity Registry
* Post feedback with `giveFeedback(agentId, value, valueDecimals, ...)` on the Reputation Registry
* Request validation with `validationRequest(validatorAddress, agentId, requestURI, requestHash)` on the Validation Registry
* Query summaries with `getSummary(agentId, clientAddresses, tag1, tag2)` on the Reputation Registry

All common EVM tooling works on ADI Chain. Start with [Quickstart](../../how-to-start/quickstart.md).
