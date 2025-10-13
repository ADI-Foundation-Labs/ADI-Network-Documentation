---
description: Introduction to Airbender
---

# Airbender

* ZKsync Airbender is a new ZK proof system.
* [**ADI Network**](overview.md) utilized an **“operating system”** based on ZKsync O&#x53;**.**

### What is Airbender?

Airbender is ZKsync's next-generation proof system, purpose-built to enable efficient ZK proofs of RISC-V bytecode execution. Built on the foundation of highly optimized STARK/FRI implementations, Airbender is designed to support long-term scaling strategy by being fast, cheap, and flexible to a wide range of use cases (without compromising security).

### Key Features

* RISC-V 32I+M instruction set
* AIR constraints
* Optimized DEEP STARK/FRI proofs
* Mersenne31 prime field
* Blake2s + Blake3 hash function

The name “Airbender” is a reference to AIR constraints (Arithmetic Intermediate Representation).

### Proving Pipeline

The proving process follows a structured six-stage pipeline designed for optimal performance and resource utilization:

* **Stage 1 - Witness Generation**: Computes Low-Degree Extensions (LDEs) of witness data and generates trace commitments. This stage establishes the foundational data structures for the proof.
* **Stage 2 - Lookup and Memory Argument**: Configures lookup tables and memory arguments, establishing the infrastructure for efficient verification of memory operations and table lookups.
* **Stage 3 - STARK Quotient Polynomial**: Computes the primary STARK quotient polynomial, which encodes the constraint satisfaction proof for the main circuit.
* **Stage 4 - DEEP Polynomial**: Implements FRI batching optimizations to reduce the overall proof size and verification complexity.
* **Stage 5 - FRI IOPP Generation**: Produces the final Interactive Oracle Proof of Proximity (FRI proof) that enables efficient verification.
* **Stage 6 - SNARK Wrapper:** Wrap the final FRI proof into a final FFLONK proof, which gets posted and verified on-chain.
