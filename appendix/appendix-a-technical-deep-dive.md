# Appendix A - Technical Deep Dive

#### A1. Rollup Specification (Summary)&#x20;

ADI Network is a zkRollup: A Layer-2 technology that uses zero-knowledge validity proofs to guarantee the correctness of every state transition. All transactions are executed on ADI L2 and collected into batches. For each batch, a zkProof is generated, demonstrating that the resulting state is valid.

This proof is then submitted to Ethereum L1, where it is verified by a dedicated contract. Only if the proof is valid does Ethereum finalize the new state, which means invalid batches can never be accepted.&#x20;

#### A2. Transaction Lifecycle — From ADI to Ethereum Finality&#x20;

<figure><img src="../.gitbook/assets/unknown (1).png" alt=""><figcaption></figcaption></figure>

#### A3. Key Features of the ADI zkRollup Architecture&#x20;

* Scalability. Each ADI L2 instance supports roughly 2,000–10,000 TPS. Where additional throughput or specialization is needed, separate L3 chains can be deployed on top.
* Gas Fees. By batching transactions and compressing proofs, the system reduces fees by 90–95% compared with L1 Ethereum. Gas is paid in ADI via the Custom Gas Token model, removing the need to manage ETH separately.\
  ZK Finality & Security. Soft confirmations by Sequencer. Final settlement on Ethereum once the proof is verified.&#x20;
* Ethereum Compatibility & Bridging. Contracts, tooling, and EVM standards work without modification, ensuring smooth deployment and migration. ADI supports seamless bridging and messaging between L1 and L2, and includes trust-minimized pathways.

#### A6. Cross‑Chain Messaging

A unified bridging portal interface allows users to deposit or withdraw assets between Ethereum (L1), ADI (L2), and specialized L3s. All communication is trust-minimized: messages are proven as part of the ZK proof, ensuring rapid finality when moving data or assets back to L1.

#### A7. Enterprise Middleware (Hyperledger FireFly)&#x20;

Middleware is included to simplify integration for enterprises and governments. It auto-generates REST APIs for deployed smart contracts, provides reliable event streaming with guaranteed delivery, and integrates with DID/KYC services.

#### A8. Modularity & L3 Expansion&#x20;

The network supports additional L3 chains that applications, technical needs, privacy needs, or jurisdictions can tailor. These L3s inherit ADI L2 security while offering compliance segmentation, higher privacy, or performance tuning. This approach allows governments, enterprises, or sector-specific networks to operate their own domain while remaining part of the ADI ecosystem.<br>
