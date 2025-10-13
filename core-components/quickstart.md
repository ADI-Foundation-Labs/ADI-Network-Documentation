# Development Engine

A modular zkEVM framework, based on the zkSync stack, powers ADI’s scalability, security, and Ethereum equivalence.

* Sequencer. Orders and executes transactions, runs in a high-availability setup with hot failover, and uses restricted networking to mitigate spam and denial attacks.
* Prover. Based on Airbender and using STARK to SNARK compression. It is GPU-accelerated and optimized to deliver sub-minute proofs, with the capability of sub-second generation for smaller batches.
* L1 Verifier & Bridge. Ethereum contracts verify submitted batches and update the state root. The Outbox/Inbox mechanism supports secure message execution, enabling near-instant withdrawals after proof acceptance.
