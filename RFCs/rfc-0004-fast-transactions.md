# RFC-0004: Decreasing the minimum interval between SmartContract's Transactions from 3 minutes to 1 second with Recursive Proofs

- **Intent**: Implement recursive proofs in SmartContract transactions to reduce the minimum transaction interval from 3 minutes to 1 second.
- **Submitted by**: DFST ([Github](https://github.com/dfstio), [Twitter](https://twitter.com/dfst_io), Discord: dfst.io)
- **Date**: Thursday, December 07, 2023

## Abstract

This RFC aims to improve the Mina Protocol's developer and user experience by enabling faster transaction processing in SmartContracts. Currently, a minimum interval of 3 minutes is required between SmartContract transactions that have preconditions and alter the state. This proposal intends to shorten this interval to 1 second, allowing rapid transaction execution with state preconditioning and changing.

## Introduction

Mina Protocol's SmartContract transactions, authenticated through proof, must meet these criteria:

- Matching account update preconditions with the current account state.
- Proof verification against the SmartContract's verification key.
- Fee alignment with the market rate.

Once verified, the transaction's inclusion in the block is assured, bypassing the need for blockchain-wide recalculations. Despite a known post-verification state, SmartContracts must wait about 3 minutes for state reflection in the new block before initiating another transaction.

Current approaches to multiple transactions within a block include:

- Non-use of preconditions and nonce manipulation (for example, in the case of the stateless contract).
- Recursive proofs via ZkProgram for bundled transactions.
- Using Actions and Reducer

These methods, especially recursive proofs, pose significant entry barriers due to complexity and infrastructure demands and do not allow for many uses, as in the case of Actions and Reducer.

**Problem**: The Mina blockchain's slow transaction pace (3 to 20 minutes) hinders rapid processing, contrasting with other blockchains like Tron (3 seconds) and Ethereum (12 seconds).

**Relevance**: Accelerating transaction speed can greatly enhance the blockchain's Total Value Locked (TVL) and user experience, fostering Mina's ecosystem growth.

## Objectives

- Reduce SmartContract transaction intervals from 3 minutes to 1 second.
- Introduce and utilize the concept of "a guaranteed new state".
- Enable immediate use of the new guaranteed state post-verification.
- Simplify SmartContract interaction with the new state.
- Integrate the changes to the new light web node to receive a new guaranteed state within 50-100 ms.

## Motivation and Rationale

The need for rapid transaction processing is driven by:

- Current system limitations: Transactions can take 3 to 20 minutes.
- Real-world applications: Many scenarios require transaction intervals of only a few seconds.
- Ecosystem growth: Faster transactions open up more use cases and attract developers.

## Possible Implementations

### Variant 1: Node-side Recursive Proof Computation

Node, after transaction verification, computes the new account state, sign it, and returns it to the zkApp. The zkApp can utilize this state (for example, by providing it as an argument to the fetchAccount()) to create new immediate transactions. The node can replace the existing mempool transaction with a new composite one, reflecting the initial transaction's preconditions and the final state of the last transaction.

### Variant 2: zkApp-side Recursive Proof Computation

SmartContracts gain the ability to merge multiple transaction proofs. The first transaction proceeds normally, with subsequent transactions being based on the internal state maintained by o1js and the calculation of the merged proofs. The composite transaction, combining initial preconditions, merged proof, and final state, is sent for replacement to the node.

## Scenarios and Use Cases

### Scenario 1: High-Frequency Transaction SmartContract

Example: A chess blitz game SmartContract needing several transactions per minute. This feature would enable high-frequency transactions without complex proof infrastructure.

### Scenario 2: Multi-User SmartContract Requiring Immediate Final State

Example: A multi-user SmartContract that operates on simultaneous requests and requires immediate access to the final state post-transaction. It can use Actions and Reducer to collect user requests, then send a reduce transaction and receive the final state to be used in the following user's transactions immediately.

## Open Issues and Discussion Points

- The calculation of any proof takes some time, and the previous transaction can be included in the block at that time, so the replacement transaction will fail. There will be needed a mechanism for automatic increase of the nonce in this case to be able to send this transaction as new or other ways how to handle this situation.
- Although the good way to use a new signed state is to feed it as an argument to fetchAccount(), the other ways that would allow use of this feature without any changes to the existing codebase should be considered
- This addition, in case of the implementation in accordance with variant 1, requires changes to the node code and should be tested extensively on the testnet. The timing of this change should be coordinated with the mainnet launch, with the mainnet launch being the obvious priority to distribute engineering and testing resources wisely.

## Conclusion

Fast transaction capabilities in Mina protocol and `o1js` mark a significant advancement in developer and user experience, potentially catalyzing the adoption and growth of the Mina ecosystem and can greatly enhance the blockchain's Total Value Locked (TVL).
