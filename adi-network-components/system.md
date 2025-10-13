---
description: Learn about the system
---

# System

The _System_ is passed to all EEs by the bootloader. It provides access to memory, IO, and oracles. Oracles are the interface through which the system accesses outside information.

The System struct is generic over memory, IO, oracles and allocators and provides little else besides access to those and block metadata. The _Concrete system implementations_ are two instances of those components to result in full system implementations, one for the sequencer and the other for the prover (as mentioned in the [Overview](overview.md)).&#x20;

#### System Struct

For convenience, the system is parameterized with `S: SystemTypes`, which bundles all the types the system is generic over.

#### Limited Capabilities

`SystemTypes` only requires `IO` to be an `IOSubsystem`. To access the all io functionality, `IOSubsystemExt` is required. The weaker version is meant to be passed to Execution environments, while the bootloader may use the stronger version of the trait by requiring `S::IO: IOSubsystemExt`.

#### Resources and IO types configuration

The system is generic in its representation of computational resources. It provides a reference implementation for

EE (an amount of ergs, consumed as gas in Ethereum) and native resources (used for proving). More details about resource accounting can be found in Double resource accounting.

It is also generic over the types used for IO—such as addresses, storage keys and values, balances, etc.

While many parts of the codebase require EthereumLikeTypes, maintaining type-level abstraction ensures that components not dependent on these specific types remain isolated and do not require modification when type definitions change.

#### System Functions

The system also describes a set of pure functions that should be part of any system implementation.

Most of them are used to implement precompiles as [System Hooks](system-hooks.md).

#### IO and Memory Subsystems

Most of the functionality of the system is related to IO or memory. These two functionalities are defined as their own subsystems.

* **The IO** subsystem is designed to abstract over the underlying storage implementation. This decision allows BoojumOS to be instantiated with different storage implementations depending on the chain needs. Each implementation is responsible for charging resources, and the charging can be different for each Execution Environment type.
* **The Memory Subsystem** provides execution environments with heap buffers in an efficient manner. The heaps are valid for the lifetime of the execution environment, so return data needs to be moved to a dedicated memory area.

#### User-Facing Methods

* Start and finish io-frames (frames without separate memory, like near-calls in Era),
* Query the block metadata from the oracle.

If you have access to a System with the \*Ext versions of the subsystems available, you can also use the following methods.

* Start and finish global execution frames (memory and io),
* Query the oracle to read the next transaction,
* Deploy bytecode,
* Initialize the system from the oracle,
* Finalize the system after the block processing is done.

#### Implementations of the System's Parts

These parts differ between forward-running and proving:

* Allocator
* Stack implementations
* IOOracle
* Logger, a simple trait to log data about the system execution, omitted in proving.
