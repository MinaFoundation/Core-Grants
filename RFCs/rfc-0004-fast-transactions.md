# RFC-0004: Decreasing the minimum interval between SmartContract's Transactions from 3 minutes to 1 second by providing a fast, universal L2 ephermal blockspace with no dedicated infrastructure

- **Intent**: Implement fast, universal L2 ephermal blockspace that requires no dedicated infrastructure to reduce the minimum transaction interval from 3 minutes to 1 second.
- **Submitted by**: DFST ([Github](https://github.com/dfstio), [Twitter](https://twitter.com/dfst_io), Discord: dfst.io)
- **Date**: Thursday, December 07, 2023 as amended on Tuesday, December 12, 2023
- **Requires**: Changes to the o1js library and Mina protocol node

![Second image](https://docs.minanft.io/rfc4/second.png)

## Abstract

The speed and cost of sending the transactions to the blockchain are of utmost importance to end users, organizations, and developers and significantly impact the adoption of the blockchain technology and Total Value Locked in the blockchain.

This RFC aims to provide each zkApp a personal, fast, and universal on-demand L2 blockspace (ephermeral blockchain) with no dedicated infrastructure to improve the Mina Protocol's developer and user experience by enabling fast transaction processing in SmartContracts.

Currently, a minimum interval of 3 minutes is required between SmartContract transactions that use preconditions and alter the state, contrasting with other blockchains like Tron (3 seconds) and Ethereum (12 seconds). This proposal intends to shorten this interval to 1 second, allowing rapid transaction execution with state preconditioning and changing.

The focus of this proposal is to provide speed capabilities equal to the usual L2 but without the need to significantly change the code or do some integrations with third-party dedicated L2 infrastructure by describing a light ephermal L2 blockspace with a life span of about 3 minutes that, at the end of its life, as a new L1 block comes, transfers all the assets and state to the L1 and disappears to be born again when needed by zkApp.

This L2 ephermal blockspace is designed to allow proof calculations and verifications to be done in parallel, thus opening the possibilities for sub-second intervals between transactions for zkApps and blockchain nodes that support parallel proof calculations and verifications.

Having sub-second intervals possible with Mina protocol and ephermal blockspace removes the barrier for blockchain adoption by removing the bottleneck in transaction processing speed while adding significant value due to the first-in-class features of Mina protocol, including private inputs, recursive proofs, direct connection to the blockchain and TypeScript programming with formal verification of the contracts hopefully added in the future.

Although computing resources required to run transaction processing on Mina protocol are significantly higher than in software systems that do not use blockchain, they are on par with computing resources used by AI and are considered normal in current business and development environments.

## Objectives

- Remove barriers to Mina protocol adoption and TVL increase by achieving transaction processing speeds comparable to those of usual non-blockchain software.
- Provide the technology that can be easily integrated into existing SmartContracts without requiring dedicated L2 infrastructure.

## Motivation

The need for rapid transaction processing is driven by:

- Current system limitations: Transactions can take 3 to 20 minutes.
- Real-world applications: Many scenarios require transaction intervals of only a few seconds.
- Ecosystem growth: Faster transactions open up more use cases and attract developers.

The end customers hardly know and do not care what is under the hood, but they do care about two things:

- Speed of sending the transaction
- Cost of sending the transactions

You can see how Tron TRC-20 tokens took the leading role in money transactions mainly because the transfers are fast and cheap, and Tron TVL is now 4 times higher than Polygon TVL, notwithstanding the fact of $2B investments into Polygon and many advanced ZK technologies integrated into Polygon.

![Tron TVL table](https://docs.minanft.io/rfc4/speed.png)

It applies to all types of applications. The attention span of the people now is 8 seconds, and being able to deliver the result within those 8 seconds leads to a conversion rate increase for zkApp. If the result is delivered after 8 seconds, many people have already forgotten what they would be doing and switched their attention to the next thing.

The integration of blockchain technology poses a significant challenge for most enterprises and government bodies, primarily due to a lack of expertise in building and managing blockchain systems. This gap hinders the widespread adoption and utilization of blockchain's potential and shows the need for tools that allow for easy integration into existing business workflows while maintaining business transaction processing speed on par with the existing systems.

Looking at the rollup market, we will see that 80% of it uses optimistic rollup technology. One of the factors why this technology is being chosen by market participants is the speed.

![Market](https://docs.minanft.io/rfc4/market.png)

The developers want to satisfy the customers first, so the motivation described above is paramount for them.
They have a wide range of tools available to them, and one of the tools they have used in recent years, as I see from my experience, is not to use the blockchain at all and base the design on old architecture to meet their business objectives, so the first factor we are speaking about is the adoption of the blockchain technology.

The other tools as L2, recursive proofs, Actions/Reducer are available for them, and they will choose the tool based on the following criteria:

- Easy to use
- Ability to meet the objectives
- Cost
- SLA levels

All those solutions are complementary.

Actions/Reducer is easy to use and does not require external infrastructure and additional cost, but you cannot update Merkle Map with it. According to the Mina documentation, “because multiple actions are handled concurrently in an undefined order, it is important that actions commute against any possible state to prevent race conditions in your zkApp”.

Therefore, it is possible to use operations like sum and multiply to calculate the new state, but you cannot prove the change of the Merkle Map root that is stored in one of eight state Fields as the root depends upon the order of the inserts into the Merkle Map. If previous inserts differ, the root will be different.

On the other hand, you cannot reconstruct the Merkle Map in the reduce(). To do this, the information on all the elements of the Merkle Map is needed, and this information is stored off-chain.

Although changes can be made in the Actions and Reducer mechanism, such as having the same order of actions provided by the archive node and provided to the reducer during on-chain calculation or allowing to provide arguments to the reducer to use additional information and proofs during reduce() call, this discussion is out of scope of the current RFC.

Recursive proofs are very powerful but require a significant CPU and memory. Fast calculation and merging of even 32 proofs in a reasonable time requires the computer to have the parameters that exist only in a cloud/server environment. Setting up such an environment is difficult for the developer who just came to the ecosystem. It can be solved by providing a cloud proof calculation service in the Mina ecosystem.

L2 is probably the best solution, but the questions of data availability, messaging, and asset transfer between L1 and L2 always complicate the development. Still, non-ephermal L2 can be the best choice for many use cases.

The experienced developer can handle all this, but if we are speaking about a junior developer, he is looking to implement his idea based on a slight modification of the example from the tutorial and is not looking to integrate with L2 or cloud infrastructure except when this integration is very simple and already included into example))

Many corporate developers also want to implement a solution quickly without integrating with dedicated L2 infrastructure.

Mina Protocol's SmartContract transactions, authenticated through proof, must meet these criteria:

- Matching account update preconditions with the current account state.
- Proof verification against the SmartContract's verification key written to the account state.
- Fee alignment with the market rate for the next block to not be evicted from mempool or wait a long time for the inclusion in the block
- There should not be other transactions that will replace or will be selected by the node instead of this one till the inclusion in the block time.

Once conditions are met, the transaction's inclusion in the block is assured, bypassing the need for blockchain-wide recalculations. Despite a known post-verification state, SmartContracts must wait about 3 minutes for state reflection in the new block before initiating another transaction.

Current approaches to multiple transactions within a block include:

- Non-use of preconditions and nonce manipulation (for example, in the case of the stateless contract).
- Recursive proofs via ZkProgram for bundled transactions.
- Using Actions and Reducer

These methods, especially recursive proofs, pose significant entry barriers due to complexity and infrastructure demands.

It also should be noted that there is a limit of about 120 zkApp transactions per block, which is shared between all zkApps. Therefore, even if zkApp can send many transactions per block using stateless contracts and nonce increases, it will compete with other zkApps for the limited space in the block. This RFC aims to shift this limit for zkApp by using compound transactions.

## Specification

### Blockspace

Blockspace is an ephermal L2 blockchain run inside the L1 BlockSpace node with a lifetime span comparable to the time between L1 blocks. At the end of its lifetime, this ephermal L2 blockchain transfers all the state to the L1 and disappears to be born again when needed by zkApp.

![L2 drawing](https://docs.minanft.io/rfc4/teddy.png)

From the [Robert Habermeier blog](https://www.rob.tech/blog/polkadot-blockspace-over-blockchains/) :

“Blockspace is an ephemeral good. When you intend to commit an operation to a chain you need blockspace in-the-moment: not yesterday’s, not tomorrow’s. Blockspace is either utilized or it is not.

One parallel for thinking about the design of blockspace allocation products is the cloud computing market. Cloud computing business models often have two key features: reserved instances and spot instances. Reserved instances are cheaper but guaranteed for a prolonged period of time. Spot instances are more costly, available on-demand, and ephemeral.

Furthermore, by changing our perspective from blockchain-centric to blockspace-centric it becomes clear that there is no reason a blockchain or state machine should run forever. **Ephemeral blockchains are an interesting use-case I believe is highly under-explored - longer running processes should be able to offload their computations or protocols to short-lived chains just like programs running on a PC can offload computations or work to background threads**.

In my opinion, the blockchain ecosystem has been thinking too small about the multi-chain world. Blockchains that start and run indefinitely with a steady pulse are an evidently inefficient mechanism. **The multi-chain of tomorrow consists of blockchains that scale and shrink on demand. It contains ephemeral chains spawned by on-chain factories, spun up by contracts and imbued with automated purpose - to complete their work a few hours later and disappear.**

Our goal in Web3 is not to maximize the number of blockchains. Maximizing the number of blockchains is something I would state explicitly as a non-goal, as it primarily benefits validator cartels seeking to extract value. Our goal in Web3 is to maximize the amount of blockspace that exists and ensure it is allocated to the state machines which need it most at any time: a constant generation and allocation of global consensus resources to those who need it the most. An enterprise without waste. In other words: the most effective blockspace producer in the world.”

### BlockSpace Node

BlockSpace Node is a node that, in addition to the usual services of the Mina protocol node, provides the services for running blockspace - an L2 epithermal blockchain. It is up to the node to decide whether it wants to run the blockspace service, whether it is being provided to every zkApp or just to the zkApps whitelisted by this specific node, and what the fees are for running blockspace services.

Since the transaction replacement mechanism is used to update the blockspace state, BlockSpace Node can set the minimum fee increase for the replacement transaction, thus being reimbursed for additional computational and storage resources necessary to run the blockspace service.

Every zkApp can use only one node as a BlockSpace Node and should send all the transactions exclusively through this node. To switch to another BlockSpace/Non-BlockSpace Node, the blockspace should be committed to the L1 and closed.

Although it is possible to use other nodes to get information about the blockspace state, this design pattern is not recommended as it increases the load for the inter-node communication that does not have enough resources to support many blockspace state changes of the different zkApps.

The BlockSpace Node provides sequencer services to the zkApp by putting the transactions that have arrived simultaneously in order and deciding what transaction to accept and what to refuse in case two transactions referring to the same guaranteed state arrived simultaneously.

![Nodes](https://docs.minanft.io/rfc4/node.png)

Some use cases where you can send many transactions in the same block are already possible without this RFC. You can now calculate recursive proofs with ZkProgram and include many proofs in one block. You can send many non-contract transactions or contract transactions not relying on the precondition state by increasing the nonce by one each time, and all those transactions will be included in the same block.

What is different with SmartContract is that the precondition is checked to include the transaction to the block. This precondition is now checked against the last block account state, so you cannot include several transactions that rely on different precondition states, you should wait for the next block. That is why we need a guaranteed state and a calculated state:

### The guaranteed state

The “guaranteed new state” is very similar to finality, but there are edge cases when they are different, so I prefer to use a different term than finality.

The guaranteed new state is guaranteed to zkApp to become final, so if the architecture of zkApp previews that guaranteed new state should always become final, it is possible, and those two terms are equal.

Still, there is a technical possibility for zkApp to replace the transaction with the same nonce, not using the new state but using the old state, effectively rolling back the transactions not included yet in the Mina blockchain block.

If we formulate what a guaranteed state means in the contract terms, it will lead us to the following wording:

### **Terms of the State Guarantee Contract**

- **Guarantee Issuer**: BlockSpace Node

- **Beneficiary**: zkApp

- **Condition Precedent**: zkApp is obliged at all times during the term of this agreement to maintain at its own expense the fee for the transactions that is necessary for avoiding the eviction of the transaction from the mempool and is enough for the inclusion of the transaction in the block during the maximum time that transaction is allowed to be in the mempool.

- **Term**: The time that starts when zkApp sends the transaction to the BlockSpace Node to be committed to the L1 blockchain and ends on the earliest of:

  - The transaction is included in the L1 block
  - ZkApp calls off the transaction by sending the replacement transaction
  - The transaction is excluded from the mempool due to the low fee or due to the maximum time reached for the transaction to be in the mempool as the result of the violations by zkApp of the Condition Precedent.

- **Rights of Third Parties**: Third Parties, including, but not limited to, other zkApps and Accounts, are NOT beneficiaries of the State Guarantee Contract and can rely on this guaranteed state only in the optimistic sense by also relying on the architecture of the zkApp and not on this Guarantee. It is recommended for third parties to wait for BlockSpace to commit to the L1 before executing any material transaction that relies on the state, such as asset withdrawal.
- **Applicable law**: Mina protocol
- **Arbitrage**: O(1)Labs
- **SLA**: TBD

### The calculated state

zkApp can calculate the state that will be the result of the transaction in a way similar to how Mina.transaction() does (or by using Mina.transaction() without calling transaction.prove())

This calculated state can be used immediately by the zkApp to start creating and proving the next transaction without waiting for the previous transaction.prove() to be finished, enabling parallel proof calculations.

The calculated state is not guaranteed and can be considered simply as one of the variables zkApp uses. To use it, Mina.transaction() arguments have to be amended to accept calculated or guaranteed state to be used as precondition input instead of the default precondition account state returned by fetchAccount() and referring to the last block account state.

The calculated state will become a guaranteed state in an optimistic sense and, later, a final state in an optimistic sense.

### Creating and verifying compound transactions

There are several ways we can create and verify compound transactions.

The most obvious, but hardly feasible way would be to put all AccountUpdates from two transactions to the compound transaction. If we repeat this operation many times, it can lead us to the transaction with 100 AccountUpdates that will require significant time for proof verification and will take a lot of memory in the mempool.

Currently, performance and memory issues are already the bottleneck in the mempool management and block creation, and various techniques are being used:

- Limiting the [number of AccountUpdates that can be included in one transaction](https://github.com/bkase/MIPs/blob/66c60c48bc4d0710202f7573765ad526ca74905a/MIPS/mip-zkapps.md)
- Limiting the [number of zkApp commands in the block](https://github.com/MinaProtocol/mina/blob/develop/rfcs/0054-limit-zkapp-cmds-per-block.md)
- Skipping proof verifications after the transaction has been added to the transaction pool

I’ve also noticed that the probability of seeing the 10-minute interval instead of the 3-minute interval between blocks is higher when I send many zkApp transactions to the same block on TestWorld2.

According to the [zkApp MIP 2](https://github.com/bkase/MIPs/blob/66c60c48bc4d0710202f7573765ad526ca74905a/MIPS/mip-zkapps.md):

"The size heuristic involves three limits: a limit on the number of field elements in actions, a limit on the number of field elements in events, and a limit on the cost of the account updates. These three limits are fixed numbers in the protocol. The cost of the account updates is calculated from the number of proofs and signatures contained in them, subject to a grouping used to minimize the number of SNARKs needed to prove the transaction. That grouping sometimes pairs signatures as one element contributing to the cost. The number of proofs, signatures, and signature pairs are multiplied by factors determined empirically to yield a valid cost metric. These limits are subject to be tuned during the incentivized testnet if this MIP passes, but in the prototype are set to any set of account updates that satisfies this equation:

```
  np := proof account updates
  n2 := signedPair account updates
  n1 := signedSingle account updates

  formula used to calculate how expensive a zkapp transaction is
  10.26*np + 10.08*n2 + 9.14*n1 < 69.45
```

The transaction pool maintains a queue of pending transactions for each fee payer, and checks the applicability of transactions considering nonces and balances, before accepting transactions into the pool.

If added to the pool, zkApp transactions are selected according to fee, just as for payments and delegations.

...

Proofs inside of account updates are checked when zkApp transactions are added to the transaction pool against some known potential future verification key as described in Mitigation of Attack 2: Verification Key Superposition. **When a block is created, the proofs are not re-checked because they were already checked when added to the pool.** When a block is received, all checks required to verify that the sender hasn't manipulated the payload are re-verified, but the proof is not explicitly checked in all cases.

Transaction pool

To keep the transaction pool simple, only fee payers of zkApp transactions are checked for balance and nonce validity. Signatures and proofs of all account updates must be verified before adding a zkApp transaction to the pool. This introduces an additional snark verification step in the transaction pool which until now checked only signatures. **After verified in the pool, the proofs are assumed to be valid during block creation and don't require checking again, similar to signatures for signed commands. Proofs within a transaction and across multiple transactions can be batched to make the verification step slightly more efficient.** Failing batch verification involves additional verification to identify the faulty proof or transaction and so, worst case each transaction are actually a bit slower. Additionally, hashing zkApp transactions impact the pool's performance. Care should be taken to hash a transaction only once and use it everywhere in the pool and throughout the protocol."

The better way would be to prepare the compound transaction in the BlockSpace node.

BlockSpace node can use parallel/serverless processing to verify the AccountUpdate proofs and create a proof for the compound transaction, or it can outsource proof creation to the Snarketplace to use third-party SNARK workers. In this way, the number of AccountUpdates in the compound transaction will be the same or almost the same as in the usual transaction.

## How it works

### **Simple example**

As usual, the zkApp prepares and sends transaction No. 1 to the node. The node checks the transaction and returns a new guaranteed contract state No 1. zkApp uses this state as an argument to Mina.transaction of fetchAccount() to immediately prepare and send another transaction No 2 to the node.

Having received transaction No. 2, the node verifies it by checking the proof and preconditions against the guaranteed state No. 1 and creates a compound transaction that replaces the previous one with the relevant fee increase and the same nonce, returning guaranteed state No. 2

As soon as a new block comes, the new compound transaction, containing compound AccountUpdates reflecting AccountUpdates from transaction No. 1 and transaction No. 2, is included in the block. At this stage, guaranteed state No. 2 is the same as the account state in the last block.

In this example, the node executes the role of the Sequencer and will decide, in case several blockspace transactions arrive, which one should be included and which should be refused immediately.

![Simple example](https://docs.minanft.io/rfc4/simple.png)

### **Advanced example**

zkApp, instead of waiting for the guaranteed state, can calculate it and use it to create new transactions even before tx.prove() finishes working. It will allow zkApp to calculate in parallel many proofs.

By doing so, zkApp is optimistic about the fact that the calculated state will be equal to the guaranteed state and then to the final state.

For this optimism to be substantiated, zkApp, in this advanced example, should take a sequencer role and put all the transactions from the users in order and finalize them before sending them to the node.

It is quite easy in some cases, such as the AccountAbstraction contract or NFT contract, where each NFT is minted as a separate account, and more complicated in other cases, such as multi-user DEX.

It should be noted that although in this example zkApp takes a sequencer role, the node and blockchain play critical roles by guaranteeing that all transactions are executed in accordance with the program that matches the verification key written to the blockchain, thus ensuring that all rules and contraints are met.

![Advanced example](https://docs.minanft.io/rfc4/advanced.png)

### API example

```typescript
interface  BlockSpaceOptions {
  isEnabled?: boolean;
  commitMode?: 'immediate' | 'lazy' | 'manual';
  lazyCommitInterval?: number;
  commitFeePayer: PrivateKey;
}

const blockspaceOptions : BlockSpaceOptions = {
    isEnabled: true,
    commitMode: 'manual',
    commitFeePayer: deployerPrivateKey
}

const network = Mina.Network({
  mina: TESTWORLD2,
  archive: TESTWORLD2_ARCHIVE,
  blockspaceOptions
});
Mina.setActiveInstance(network);

// first transaction
const transaction1 = await Mina.transaction(
  { sender, fee },
  () => {
    zkApp.addVoteToMerkleTree(...);
  }

// second transaction
const calculatedState = transaction1.calculateState();
const transaction2 = await Mina.transaction(
  { sender, fee, state: calculatedState },
  () => {
    zkApp.addVoteToMerkleTree(...);
  }
await transaction1.prove();
await transaction1.sign(...).send();
await transaction1.prove();
await transaction1.sign(...).send();
const guaranteedState = await blockSpace.commit();
const transaction3 = await Mina.transaction(
  { sender, fee, state: guaranteedState },
  () => {
    zkApp.addVoteToMerkleTree(...);
  }
  await transaction3.prove();
  await transaction3.sign(...).send();
  await blockSpace.close(); // commit and close blockspace
```

### Implementation

- BlockSpace functionality needs to be added to the Mina protocol node
- o1js should be amended to add BlockSpace functionality to Mina.transaction, Transaction, fetchAccount, and Mina.Network

### Proposed stages of implementation

- **Stage 1**: research and tests. Developing middleware serverless node to be inserted between o1js and node to emulate some parts of blockspace behavior according to zkApp commands
- **Stage 2**: POC code using forked o1js and node repos
- **Stage 3**: Pull requests for o1js and node, preliminary testing
- **Stage 4**: Code changes according to the results of the Stage 3, code review by O(1)Labs
- **Stage 5**: Integration into Experimental namespace of o1js and node
- **Stage 6**: Testing on Berkeley/TestWorld/Lightnet
- **Stage 7**: Integration to the mainnet
- **Stage 8**: Developing additional features not included in previous stages and improving performance:
  - add parallel optimizations
  - integration with light node
  - integration with a cloud proof computing service

## Feasible minimum time between transactions

Let’s consider several scenarios to define the minimum feasible time between transactions.

- **Usual heavy** SmartContract needs approximately 30 seconds to calculate the proof. In the current scenario, sending several transactions in raw will spend 30 seconds calculating the proof and 150 seconds (3 minutes - 30 seconds) waiting for the block. With blockspace, it will be able to send 5-6 transactions per block with 1-second minimum transaction interval and 4-5 transactions with 10 seconds minimum interval, which is 20% difference

- **Usual Minimum** [SmartContract](https://github.com/dfstio/minanft-lib/blob/rfc4/experimental/rfc.nonce.test.ts) spends about 10 seconds to calculate the proof and send the [transactions toTestWorld2](https://minascan.io/testworld/account/B62qryiea4rVrTCuSaUXNxebVtj7NUQgUWMi1WV9t91YDESXj2wa2ka/zkApp?type=zk-acc). It will be able to send 16 transactions per block with a 1-second minimum transaction interval and 9 transactions with 10-second minimum interval; it is a 77% difference

- **Advanced** RealTimeVoting SmartContract that calculates proofs in parallel for the purposes of the real-time voting (example code: [jest test](https://github.com/dfstio/minanft-lib/blob/rfc4/experimental/rfc.api.test.ts) and [backend code](https://github.com/dfstio/minanft-api/blob/rfc4/src/api/rfc4.ts)) can calculate 128 proofs in 166 seconds (1.3 seconds per proof) and can [send transactions](https://minascan.io/testworld/account/B62qibzrmvWg3M7BS7aVzfvTZgjqTrsTnXEcaFCF1ZK9Q9yvbNRTEVC/zkApp?type=zk-acc) to TestWorld2 using 0.65 seconds per transaction:

```
[10:13:55 PM] Nonce: 330
[10:14:04 PM] api call result {
  success: true,
  jobId: '6459034946.1702318443674.eawb55rpojy9q5fkqmsjd4h3x38y3e34',
  error: undefined
}
[10:16:50 PM] Time spent to calculate 128 proofs: 166102 ms
[10:16:50 PM] Billed duration 3410970 ms
[10:16:50 PM] Duration 165527 ms
[10:16:58 PM] Downloaded transactions: 128
[10:16:59 PM] Sending 128 transactions...
[10:17:00 PM] MinaNFT first transaction sent: 5Jup9Wkc6tkFecuxiw9LbHJABnCXcA1soU8iZPefYetNWvxyHU2X
[10:17:00 PM] Waiting for a new block to put the remaining transactions in one block...
[10:19:34 PM] Transaction wait time: 2:33.856 (m:ss.mmm)
[10:20:57 PM] sent 127 transactions: 1:23.556 (m:ss.mmm)
```

This example RealTimeVoting zkApp has used MinaNFT backend for parallel proof calculation and was able to include 122 zkApp transactions emulating blockspace work in TestWorld2 block 18241:
https://minascan.io/testworld/block/3NKCH5552TNFAGhCbprddsbSbLwc4YqimqLVjeepY9Hg2Kh9Darf/zk-txs

For this test, the argument to the method emulates the guaranteed state. It also shows that some zkApps can generate and send proofs to the blockchain at the rate of approx. one transaction per second.

To calculate the proofs in parallel, zkApp has to calculate in advance the state used as a precondition, which is easy in case the state is the root of the Merkle Tree where elements are being inserted one by one and use this state in Mina.transaction().

This contract can send 138 transactions every 3 minutes with a 1-second minimum transaction interval and 18 transactions with 10 seconds minimum interval, which is a 1280% difference.

Given that we see performance differences for all types of contracts, it is feasible to target the 1-second minimum transaction interval.

## Lower theoretical bound of the interval between transactions and why it matters

Let’s analyze what can be a lower bound of the interval between transactions. We have here two types of calculations:

- State calculations cannot run in parallel in general, but they are fast
- Proof calculations are slow, but they can be run in parallel.
  The proofs calculations from the RealTimeVoting example above use one sequencer and 128 lambdas calculating proofs. By using 10 or 20 sequencers and increasing the number of lambdas, we can increase the rate of proof calculation from one proof every 1.3 seconds to 1 proof every 0.1 seconds.

State calculations cannot be run in parallel and require in my tests 18.285 s for calculating 128 states, or 0.14 s per state.

Therefore, if we account for some overhead, the theoretical minimum interval between transactions in the Mina protocol + blockspace setup can be 0.2 seconds, leading us to a theoretically achievable rate of 5 transactions per second per zkApp.

By splitting the contract between several contracts or zkApps, in some cases (for example, by deploying the separate contract for each trading pair in DEX), it will be theoretically possible to multiply this rate by some coefficient in the range of 2-10, which leads us to a theoretical 10-50 TPS per zkApp group.

**This rate, although hard to achieve in real-life systems due to non-blockchain software, CPU, memory, and network issues, is very important because it is comparable with rates for many non-blockchain software systems and, therefore, having such a high TPS rate eliminates the barrier for Mina protocol adoption**

## Scenarios and Use Cases

Almost any application can benefit from high TPS. The example of such applications are:

- Games
- Trading (and to withdraw material amounts safely, you still need to wait for an L1 block, but trading can be done within the block)
- Account Abstraction contracts
- Putting live data into the blockchain in real-time (oracles for quotes, legally important audio, other real-time data streams)
- Real-time voting in the shareholder and public hearing meetings
- Any transactions where identity should be present to the zkApp and relied on to make the transaction, such as including the person in a white list before making the transaction.

### Scenario 1: High-Frequency Transaction SmartContract

Example: A chess blitz game SmartContract needs several transactions per minute.

This contract has to calculate the new state almost in real-time, with a very short time between the moves. It is a one-user game, and the contract can use a calculated state that takes 0.2 seconds to calculate.

Then, in the background, zkApp can calculate the proofs for each move and send them to the BlockSpace node that is configured to commit the state to L1 in manual mode. As soon as the blitz party is finished and all proofs are calculated, and all transactions are sent to the BlockSpace node, the zkApp issues blockSpace.close() command to commit the compound transaction to the transaction pool and close blockspace.

### Scenario 2: RealTimeVoting SmartContract Requiring Voting Results in real-time.

The RealTimeVoting contract used in the example above is responsible for counting and keeping the public votes at the shareholder’s meeting of the company or at the public hearing in the local municipality.

All the votes have to be inserted into the Merkle Tree, and the root of the Merkle Tree should be kept on the zkApp state. The results are expected to be available immediately.

The zkApp first can do the process of calculating the state as soon as new votes become available by processing them by the zkApp sequencer. All the new transactions are sent immediately for parallel proof calculations, with the delay between the vote arrival and proof readiness being about one minute. The transactions with proofs are then sent in batches to the BlockSpace node to get the guaranteed state.

The delay between the last vote and the guaranteed state reception can be 1-2 minutes in this scenario, with the final state to be included in the L1 block within 3 minutes, total of 3-5 minutes delay for the final state.

## Open Issues and Discussion Points

- The calculation and verification of any proof takes some time, and the previous transaction can be included in the block at that time, so the replacement transaction will fail. There will be needed a mechanism for automatic increase of the nonce in this case to be able to send this transaction as new or other ways how to handle this situation.
- To act in the most efficient way, the node should support parallel processing or even be running on the cluster or server+serverless environment. This requirement is optional but recommended for high-performant blockspace.
- It is not clear how actions and events should be handled in blockspace. As handling actions and events is not the role of a regular node, it should be considered during the research phase.
- Transaction Pool and block creation algorithms must be analyzed during the research phase to find the most efficient way to build and verify compound transactions. This will also affect the BlockSpace fee structure and can require a deposit from the zkApp to the BlockSpace node in some cases to offset SNARK work fee.

## Conclusion

Fast transaction capabilities in Mina protocol and o1js mark a significant advancement in developer and user experience, potentially catalyzing the adoption and growth of the Mina ecosystem and can significantly increase the blockchain's Total Value Locked (TVL).
