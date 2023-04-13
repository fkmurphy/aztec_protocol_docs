---
title: Architecture
---

:::caution
We are building Aztec 3 as transparently as we can. The documents published here are merely an entry point to understanding. These documents are largely complete, but unpolished.

If you would like to help us build Aztec 3, consider reviewing our [GitHub](https://github.com/AztecProtocol) to contribute code and joining our [forum](https://discourse.aztec.network/) to participate in discussions.
:::

Aztec3 describes a programmable [ZK Rollup](https://ethereum.org/en/developers/docs/scaling/zk-rollups/) operating on Ethereum as an L2.

A "smart contract" is defined as a set of public and private functions that operate on public and private state respectively. Each function is represented as a ZK SNARK verification key, where the contract is uniquely described by the set of its verification keys.

## Heuristics that differ from Aztec Connect

**A contract cannot manipulate another contract's state**

In A.C. we had multiple circuits that could each create/destroy a unified set of value notes. This was acceptable because all A.C. circuits had a single author (us!).

In A3 our architecture must process arbitrary circuits written by potentially dishonest actors. Contract state must therefore be siloed at the architecture level similar to Ethereum. Fortunately, this does not require splitting up the anonymity set.

**Anonymity set must be shared across all contracts**

In A.C. observers knew when different note types were being created (value note, account note etc). This cannot be the case in A3 as we want to provide strong privacy gaurantees to all private contracts even if they have few transactions interacting with their contract.

**Support for call semantics**

If a contract can only modify its own state, we need a way for a contract to "call" another contract to request state modifications.

## Aztec3 Actors

**User**

Generates proofs-of-correctness for private functions.

**Sequencer**

Sequences a block of Aztec transactions.

**Prover**

Constructs a proof-of-correctness for a block of Aztec transactions. Computes proofs-of-correctness for public functions within the block.

## Enabling Transaction Semantics: The Aztec Kernel

Aztec3 posesses 2 kernel circuits (a private kernel and a public kernel) that each validate the correct execution of a function call.

The kernel circuits verify the correct execution of a single function call. A key data structure is the call stack - a [FIFO](<https://en.wikipedia.org/wiki/FIFO_(computing_and_electronics)>) queue containing pending function calls. There are two call stacks, one for private calls and one for public calls.

One iteration of a kernel circuit will pop a call off of the stack and execute the call. If the call triggers subsequent contract calls, these are pushed onto the stack.

Private kernel proofs are generated first. The transaction is ready to move to the next phase when the private call stack is empty.

The public kernel circuit takes in proof of a public/private kernel circuit with an empty private call stack, and operates recursively until the public call stack is also empty.

A transaction is considered complete when both call stacks are empty.

The only information leaked about the transaction is:

1. The number of private state updates triggered
2. The set of public calls generated

The addresses of all private calls are hidden from observers.

## Programming contracts with Noir

[Noir](https://noir-lang.org) is a programming language designed for converting high-level programs into ZK circuits. Based on Rust, the goal is to present an idiomatic way of writing private smart contracts that is familiar to Ethereum developers. Future versions will include concepts such as contracts, functions and storage variables.

The end goal is a language that is intuitive to use by web3 developers with no cryptographic knowledge.

There are no plans to achieve EVM compatibility or support Solidity in Aztec 3. The privacy-first nature of Aztec 3 is fundamentally incompatible with the EVM architecture and Solidity's semantics. In addition the heavy use of client-side proof construction makes this impractical.

## Private state semantics

Private state must be treated differently to public state and this must be expressed in the semantics of the Noir language.

Private state is encrypted and therefore is "owned" by a user or a set of users (via shared secrets).

Private state is represented in an append-only database since changing or deleting a record would leak information about the transaction graph.

The act of "deleting" a private state variable can be represented by adding an associated nullifier to a nullifier set. The nullifier is generated such that, without knowing the decryption key of the owner, an observer cannot link a state record with a nullifier.

Modification of state variables can be emulated by nullifying the a state record and creating a new record to represent the variable. Private state has an intrinsic UTXO structure and this must be represented in the language semantics of manipulating private state.

## Core components of Aztec3

- Contract programming language (Noir)
- ZK SNARK proof system (Honk)
- Kernel circuits
- Rollup circuits
- Private client
- Public client
- Transaction mempool + p2p messaging network
- Sequencer client
- Prover client
- L1 rollup smart contracts