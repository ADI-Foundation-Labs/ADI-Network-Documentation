---
description: Learn about Bootloader, the entry point of the system
---

# Bootloader

The bootloader is the component responsible for implementing the general blockchain protocol. Roughly, this means:

1. Initializing the system.
2. Reading the block context from the oracle.
3. Reading and parsing the first transaction.
4. Validating the transaction.
5. Executing the transaction.
6. Saving the transaction result.
7. Repeat from 3 until there are no more transactions.
8. Finalizing the block.

#### Configuration

The bootloader can be configured with the following parameters:

* `ONLY_SIMULATE`: skips the [validation](processing-of-transactions.md#validation) step when processing a transaction. Used for call simulation in the node.
* `IS_PROVING_ENVIRONMENT`: to skip some checks during sequencing.
* `SPECIAL_ADDRESS_SPACE_BOUND`: the range of addresses (starting from 0) where system contracts can be deployed.
* `AA_ENABLED`: whether native account abstraction is enabled.

In addition, the `basic_bootloader` crate has the following compilation flags:

* `code_in_kernel_space`: to enable normal contract execution for addresses in the range `[0, SPECIAL_ADDRESS_SPACE_BOUND]`.
* `transfers_to_kernel_space`: to enable token transfers to addresses in the range `[0, SPECIAL_ADDRESS_SPACE_BOUND]`. Note: the bootloader itself has a special formal address (0x8001) that is always allowed to receive token transfers. This is used to collect fees.
* `charge_priority_fee`: to enable charging for the EIP-1559 tip (priority fee) on top of the base fee.
* `evm-compatibility`: enables all the previous flags, needed for the EVM test suite.

#### Code Execution

For transaction execution, the bootloader has to execute some contract code. This contract code corresponds to one of the supported VMs, and is executed through the [Execution Environment](execution-environment.md) (EE) module.

A contract call is executed through an interplay between the bootloader and (potentially different) execution environments. Indeed, a contract executing in a given EE can call to contracts that run on a different EE or to a [System Hook](system-hooks.md). This interplay is described under [Runner Flow](runner-flow.md).

#### Block Header

At the end of the execution, the bootloader outputs the block header.

For the block header, the Ethereum format will be used. However, some fields will be set differently in the initial version for simplification.

The block header should fully determine the block, i.e., include all the necessary inputs to execute the block.

| Ethereum Field Name | Ethereum value                                                                                 | ADI Network value                                                                 | Comments                                |
| ------------------- | ---------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------- | --------------------------------------- |
| parent\_hash        | previous block hash                                                                            | previous block hash                                                               |                                         |
| owners\_hash        | <p>0x1dcc4de8dec75d7aab85b567<br>b6ccd41ad312451b948a7413f<br>0a142fd40d49347 (post merge)</p> | <p>0x1dcc4de8dec75d7aab<br>85b567b6ccd41ad312451b<br>948a7413f0a142fd40d49347</p> | hash of empty RLP list                  |
| beneficiary         | block proposer                                                                                 | Operator (fee) address                                                            |                                         |
| state\_root         | state commitment                                                                               | 0                                                                                 |                                         |
| transactions\_root  | transactions trie (Patricia Merkle tree) root                                                  | transactions rolling hash                                                         |                                         |
| receipts\_root      | receipts trie (Patricia Merkle tree) root                                                      | 0                                                                                 |                                         |
| logs\_bloom         | 2048-bit bloom filter over logs’ addresses and topics                                          | 0                                                                                 |                                         |
| difficulty          | 0 (post merge)                                                                                 | 0                                                                                 |                                         |
| number              | block number                                                                                   | block number                                                                      |                                         |
| gas\_limit          | block gas limit                                                                                | constant, not defined yet, 10–15 M most likely                                    |                                         |
| gas\_used           | block gas used                                                                                 | block gas used                                                                    | TBD — with or without pubdata           |
| timestamp           | block timestamp                                                                                | block timestamp                                                                   |                                         |
| extra\_data         | any extra data included by proposer                                                            | TBD, possibly gas\_per\_pubdata                                                   |                                         |
| mix\_hash           | beacon-chain-provided random, (post merge)                                                     | 0                                                                                 | after consensus will be provided random |
| nonce               | 0 (post merge)                                                                                 | 0                                                                                 |                                         |
| base\_fee\_per\_gas | base\_fee\_per\_gas                                                                            | base\_fee\_per\_gas                                                               |                                         |
