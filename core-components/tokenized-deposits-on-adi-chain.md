---
description: Architecture considerations for banks issuing tokenized deposits on ADI Chain.
---

# Tokenized Deposits on ADI Chain

A stablecoin is a claim on its issuer's reserve. A tokenized deposit is a claim on the issuing bank's balance sheet.

Both can use the same settlement rail. Their liability, ledger integration, and regulatory treatment remain distinct.

{% hint style="warning" %}
Regulatory classification, licensing requirements, and accounting treatment require bank and regulatory sign-off for each deployment.
{% endhint %}

### What a tokenized deposit is

A tokenized deposit represents a deposit liability of the issuing bank. The bank maintains the relationship between the token and its deposit ledger.

This model can keep client funds as bank deposits until exchange. It can also support settlement outside banking hours without prefunding a separate settlement asset.

The token contract should reflect the bank's product terms, eligibility rules, and redemption process. Legal, accounting, and regulatory treatment remain the bank's responsibility.

### The control model

The bank owns its token contract and its administrative permissions. It controls issuance, redemption, holder eligibility, and transfer rules.

ADI network operators run network infrastructure. They perform upgrades, batching, and proving. They cannot mint, freeze, or move a bank's token without authority granted by the bank's contract.

The bank depends on ADI for network availability. It retains control over its own liability and token permissions.

[How ADI Chain is Validated](how-adi-chain-is-validated.md) explains the sequencer, proving, and Ethereum settlement roles.

### Atomic exchange against DDSC

A deposit token and DDSC can exchange on the same execution layer. Contract logic executes both legs together or reverts both legs.

Eligibility checks run at the moment of exchange. The transaction completes only when each leg satisfies its rules.

For example, a corporate payment at 21:00 on Friday can exchange a bank-issued deposit token against DDSC. Both legs settle atomically on ADI Chain, rather than entering a bank processing queue for Monday.

DDSC is listed in [Mainnet Network Contracts](../how-to-start/network-contracts.md#tokens) at `0x1211f0cfe66739433c1330e21f4951B80E813479`.

{% hint style="warning" %}
The availability and legal settlement effect of out-of-hours payment workflows require product, operational, and regulatory sign-off.
{% endhint %}

### Finality-stage selection

ADI confirmations arrive in stages. A bank's ledger policy must define which stage marks a payment as settlement-final.

| Stage     | Block tag   | Use in a bank workflow                        |
| --------- | ----------- | --------------------------------------------- |
| Pending   | `latest`    | Near-instant inclusion for client experience. |
| Committed | `safe`      | The batch is committed to Ethereum.           |
| Executed  | `finalized` | L1 execution provides irreversible finality.  |

[Finality](../references/json-rpc-api/finality.md) describes the `pending`, `committed`, and `executed` stages and their matching block tags.

{% hint style="warning" %}
Choose ledger posting, reconciliation, and exception handling rules with the bank's risk, operations, and compliance teams.
{% endhint %}

### L2 vs dedicated L3

Place a deposit token on the shared L2 when counterparties and DDSC already transact there. This keeps the exchange on one execution layer.

Use a dedicated L3 when operating data must remain isolated from other participants. Examples include bilateral terms, client position data, and internal liquidity workflows. This adds a settlement hop between the L3 and L2.

ADI supports three L3 infrastructure models: ADI-managed, client-operated, and hybrid. Chain contract ownership can transfer to the client.

[ADI Network components](../adi-network-components/overview.md) introduces the execution layer. [L3 Chains](../adi-network-components/l3-chains/overview.md#infrastructure-models) details the operating models and ownership options.

{% hint style="warning" %}
Confirm data visibility, privacy controls, and cross-layer settlement requirements for the selected topology.
{% endhint %}

### Fee sponsorship

Counterparties do not need to hold ADI directly. The bank can fund transaction fees through a treasury balance or sponsor fees through an ERC-4337 paymaster.

ADI does not provide an audited default paymaster. A bank can integrate external account-abstraction infrastructure or deploy and own its own paymaster stack.

[Gas Abstraction & Account Abstraction](../how-to-start/gas-abstraction-and-account-abstraction.md) covers supported account-abstraction paths and deployed entry points.
