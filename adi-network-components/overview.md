---
description: Introduction to ADI Network
---

# Overview

**ADI Network**, utilizing the ZKsync stack and based on two complementary pillars: \
\
[**ZKsync Airbender**](airbender.md), a next-generation proof system and a modular execution layer based on the ZKsync OS stack.

In this section, we will deep dive into the ADI Network execution layer.

{% hint style="warning" %}
The ADI network and ZKsync OS are under active development; you can track the updates for [**ZKsync OS**](https://docs.zksync.io/zksync-protocol/zksyncos/overview) here.&#x20;
{% endhint %}

### ADI Network: The Execution Layer

ADI Network execution layer is a system-level implementation for the state transition function.

It takes block data and an initial state as input and computes the new state after the application of the block.

ADI Network execution layer is implemented as a Rust program that will be compiled to two targets. This first one, x86, is used for running in the sequencer. The second, RISC-V, is fed as an input to the Airbender prover to produce the validity proof of the state transition.

### Components of ADI Network execution layer

The main components of the ADI Network execution layer are:

* [**Bootloader**](bootloader.md): The entry point program. It initializes the system and then runs transactions using two components: the system and the execution environment interpreters.
* [**Execution Environment**](execution-environment.md): Regular interpreters that take bytecode, calldata, resources (similar to gas), and some other call context values as its input. Interpreters are instantiated with some local state to execute a frame. When an interpreter sees a call to another contract, return/revert from the current frame, or contract creation, it triggers special functionality to process it, as a potentially different interpreter should be run.
* [**System**](system.md): Provides an abstract interface for low-level handling of IO (storage, events, L1 messages, oracles) and memory management. The system communicates with the external oracle (non-determinism source), which is needed to read block data , and also for some IO operations, e.g., to perform the initial read for a storage slot.

#### Running Environments

The two running environments are:

1. **Forward Running Mode:** To be used in the sequencer. In such mode, we expect code to be run on the usual platform with OS, so the default memory allocator can be used (as it’s part of the OS). For non-determinism source, we can pass Oracle's Rust implementation as a bootloader input. Some code can also be skipped in this mode (e.g., Merkle proof verification for storage reads).
2. **Proving Running Mode:** To be used during proof generation. The code runs on a pure RISC-V platform without an execution environment, so memory management must be handled manually. Also, special care is needed to pass external data into the RISC-V machine because of the absence of standard non-determinism sources. All behavior must be fully deterministic and provable.

***

### System Resources

In ADI Network, the concept of "resources" is required to limit and charge for both computation (primarily proving) and data usage. This is more complex than it may initially appear: ADI Network is designed to be EVM gas-equivalent, meaning that EVM code execution should follow the same gas schedule as on Ethereum.

However, the EVM gas schedule does not accurately reflect the cost of ZK proof generation. To address this mismatch, we are using [double accounting](double-resource-accounting.md): tracking both Execution Environment (EE) is gas-equivalent to EVM gas, and a "native" computational resource that models the cost of proving.

### L1 Integration

ADI Network, is designed for use in ZK rollups and validiums, where state transition correctness must be verified on the settlement layer (referred to here as L1 for simplicity).

Specifically, a state commitment is stored on L1. For each block or batch, a proof is generated to verify that a valid state transition has occurred from a known L1 state commitment to a new one based on some set of inputs. This means the state pre and post transition must be included in the public inputs (or preimage) of the ZK proof.

In addition, the public input will include other components required for:

* Messaging
* Data availability (DA) validation
* Input validation
