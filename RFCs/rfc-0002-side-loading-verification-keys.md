# RFC-0002: Generic Recursive Proving Primitives (Side-loading Verification Keys)

- **Intent**: To introduce side-loading of verification keys in `o1js` smart contracts and provable-programs, enhancing flexibility and introducing new design patterns.
- **Submitted by**: Theodore Pender (Github: @teddyjfpender, email: theodore.pender@minaprotocol.com, Twitter/X: @franklyteddy)
- **Submitted on**: Monday, November 27, 2023

## Abstract

This RFC proposes an enhancement to the current developer experience of working with verification keys in `o1js` for example in smart contracts and provable-programs. Currently users are unable to provide a specific verification key for the contract or program to use when verifying a proof. By enabling the side-loading of verification keys, this approach allows for greater flexibility, permitting a prover to use any verification key from a tree of accepted keys, rather than being constrained to a single pre-defined key.

## Introduction

In `o1js` smart contracts and provable-programs do not allow for verification keys directly to be side-loaded into their methods. This rigidity limits the adaptability and potential scope of these contracts. The proposal aims to address this limitation by enabling the side-loading of verification keys.

- *Problem*: Verification keys are static and compiled at the time of smart contract or provable-program creation, limiting the potential for dynamic interaction between different contracts or programs.
- *Relevance to Mina Ecosystem*: Enhancing the flexibility and capability of `o1js` smart contracts and provable-programs within the Mina ecosystem, leading to broader applications, use-cases, and design patterns.

## Objectives

- Introduce the concept of side-loading verification keys.
- Illustrate the benefits of this update over the current implementation.
- Outline the potential impact on the Mina ecosystem.
- Elicit feedback from the Mina ecosystem on requirements and use-cases to appropriately design an implementation.

## Motivation and Rationale

The need for side-loading verification keys arises from several factors:

- Limitation of Current Systems: Embedded verification keys restrict the flexibility of smart contracts and provable-programs.
- Research and Insights: 
    - [o1js issue #673](https://github.com/o1-labs/o1js/issues/67)
    - [o1js issue #1088](https://github.com/o1-labs/o1js/issues/1088)

## Scenarios and Use Cases

### Example Scenario 1: Multi-Verification-Key Smart Contract

Consider a KYC-transaction scenario with a smart contract. Users are required to submit a `Proof` of KYC-issuance in their transaction, which attests that their account public key (the transaction `sender`) has undergone a KYC process. This `Proof` must be generated by a provable program, either executed by the user or by the KYC-issuer. In a conventional setup, where the smart contract is compiled with a single, fixed KYC-issuer's provable-program verification key, it's impossible to validate proofs from new KYC-issuers. However, if the system allows side-loading of verification keys, then the system can verify `n` verification keys from `m` different KYC-issuer (where `n` >= `m`) provable-programs. This set of verification keys can form a tree structure with a root committed on-chain, enabling the contract to verify if the side-loaded verification keys are in the tree of acceptable verification keys. The KYC-transaction contract becomes capable of accommodating proofs from multiple issuers.

| Aspect           | Description |
|------------------|-------------|
| **Description**  | A smart contract (or provable program) enables users to side-load verification keys (that can be validated from a defined tree of verification keys) to validate user-submitted proofs. This capability allows the contract to adapt to various verification needs without being confined to the verification keys set at compile time. |
| **Requirements** | The smart contract must be able to dynamically accept and process different verification keys (logic of the contract must check if the verification key is in the tree of keys). It should ensure the integrity of the proof verification process. |
| **Expected Outcome** | The aim is a flexible, robust smart contract system that can verify proofs with various keys sourced from different provable programs (i.e. unique circuits). This enhancement would increase the utility and upgradability of the contract, simplifying the process for users who need to provide proofs to the contract's methods and expanding the contract's application range; reducing the complexity that manifests in the user experience. |
| **Impact Analysis** | The introduction of side-loading verification keys in this scenario would significantly increase the versatility and adaptability of smart contracts and not bind them to verification keys defined at compile-time. It would allow for more complex and varied use cases. |

### Example Scenario 2: Mutually Recursive Provable-Programs

Originally mentioned [here](https://github.com/o1-labs/o1js/issues/673#issue-1522203294). Consider a scenario involving two provable-programs, `Program A` and `Program B`, which are designed to verify each other's proofs in a mutually recursive manner. Under the current system, a strict compilation order is required: if `Program A` needs to verify proofs from `Program B`, the verification key for `Program B` must already be available at the time of `Program A`'s compilation. This limits the potential for these programs to function in a mutually dependent manner.

| Aspect           | Description |
|------------------|-------------|
| **Description**  | This scenario demonstrates the need for side-loading verification keys to enable mutually recursive provable-programs. By allowing each program to dynamically accept verification keys, they can verify proofs generated by one another, even if these keys were not available at compile time. |
| **Requirements** | Both programs must be capable of dynamically accepting and processing verification keys from each other. This requires a flexible system that can handle verification keys not predetermined during the initial compilation, ensuring both security and functionality. |
| **Expected Outcome** | The goal is to allow provable-programs to mutually verify each other's proofs without being constrained by the order of compilation. This would allow for a more interconnected and dynamic system of provable-programs, enhancing their functionality and potential use cases. |
| **Impact Analysis** | Implementing side-loading verification keys in mutually recursive provable-programs would significantly improve their interactivity, removing barriers imposed by static compilation orders. This would open up possibilities for more complex provable-programs interactions and dependencies. |

## Open Issues and Discussion Points

- Other scenarios to satisfy
- Further requirements elicitation
- Implementation discussion

## Conclusion

The introduction of side-loading verification keys in `o1js` represents a stride towards enhancing the adaptability and scope of smart contracts and provable programs within the Mina ecosystem. This RFC has outlined a method to overcome the current limitations posed by static, compile-time embedded verification keys, with a more dynamic and flexible approach.

Through detailed scenarios, we've demonstrated how this feature can revolutionize the way contracts and programs interact and verify proofs, notably in contexts like KYC transactions and mutually recursive provable-programs. This flexibility not only broadens the scope of potential applications but also introduces new paradigms in contract and program design, significantly benefiting developers and users alike.

While this opens up a realm of possibilities, it also presents challenges and open issues that require further discussion and input from the community. The development and implementation of this feature will be an evolving process, reliant on collaboration and feedback from developers, users, and all within the Mina ecosystem.