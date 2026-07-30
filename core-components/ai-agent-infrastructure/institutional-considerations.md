# Institutional Considerations

### Institutional considerations

#### Compliance by design

ADI Chain is built for regulated environments. For agent infrastructure, that means:

* **Private mempool** that reduces front-running, sandwiching, and pending-order leakage
* **Jurisdiction-aware L3s** that enforce local policy while inheriting ADI settlement security
* **Ex-ante compliance** through integrations such as SettleMint DALP or Zoniqx zCompliance
* **Onchain auditability** for registrations, trust signals, and validation events

#### Trust model selection

Institutions can choose a trust model that matches the value at risk.

| Value at risk | Recommended trust model               | How it works                                                      |
| ------------- | ------------------------------------- | ----------------------------------------------------------------- |
| Low           | Reputation only                       | Trust comes from historical feedback by known counterparties.     |
| Medium        | Reputation + TEE attestation          | The agent runs in a TEE and the attestation is recorded onchain.  |
| High          | Reputation + stake-secured validation | Validators stake capital and face slashing for bad validation.    |
| Maximum       | Reputation + zkML + TEE               | Multiple independent validation paths reduce trust concentration. |

#### Data privacy

* Registration files can use IPFS or HTTPS URIs
* Sensitive endpoints can stay off the public record and be shared bilaterally
* Detailed feedback context can remain offchain and be referenced by hash
* Validation evidence can point to encrypted payloads with controlled access
* The private mempool reduces transaction visibility during submission

#### Governance

* ERC-8004 registries are upgradeable through the Diamond Proxy pattern
* Registry-level upgrades are managed by the contract owner
* Agent-level governance remains with token owners and delegated operators
* L3s can extend base registries with jurisdiction-specific logic
* The [ADI DLT Framework](../../adi-dlt-framework.md) defines the broader governance and operational control model
