# RFC-0014: Data Availability Integration into Mina

- **Intent**: To highlight the importance of including a DA (data availability) layer inside Mina and describe the possible methods of achieving such integration.
- **Submitted by**: Yunus Gürlek (GitHub: @yunus433, Email: yunus.gurlek@minaprotocol.com, Twitter: @yunusguerlek)
- **Submitted on**: Tuesday, April 23, 2024

## Abstract

This RFC is to explain the importance of a DA (data availability) layer for Mina, how it can be used by ZK developers, and how it may be possible to safely integrate a DA layer into Mina without creating any attack vectors.

## Introduction

### Decentralization

Decentralization is one of the most spoken concepts in the blockchain space, yet it is often misunderstood. There is no blockchain achieving 100% decentralization at the moment. Nevertheless, there are actually only two points to make a system 100% decentralized.

The first is the system to be trustless, meaning there are no trust assumptions in any party while making a computation. ZK (zero knowledge) achieves that with no exception by proving any computation by any party. When a ZKP (ZK proof) of a computation is provided, the verifier is convinced without doubt of the truthfulness of the process. ZK can be used outside of blockchains and does not require any on chain support to provide trustlessness.

However, being trustless is not enough to consider a system decentralized. The system and the data must be truly available for 100% decentralization. DA is the concept of being able to access data from everywhere at any time without the risk of censorship. If data can be censored, even for a limited period of time, then it cannot be considered fully available. DA’s details are discussed below.

If a system is able to provide DA and trustlessness at the same time, then it becomes 100% decentralized. Current zkApp implementations on Mina achieve trustless computation very well through ZK, but unfortunately it is not possible to have DA with Mina applications right now. This is the main concern of this proposal.

### DA (Data Availability)

DA is the concept of providing 100% liveness to the on chain stored data. In regular blockchains (meaning any blockchain but DA layers), TXs are sent to the chain through full nodes.

Full nodes are heavy structures and cannot be installed by every client of an application during the usage of a zkApp (or a dApp). Thus, RPC ports (like JSON rest APIs) are used to send TXs into nodes. This decreases the liveness of a system, as nodes can be offline temporarily or choose to censor a specific transaction.

In DA layers, a different kind of node is included in the system called light nodes. Light nodes are not a part of the consensus and are only responsible for checking the availability of some data to them through a method called sampling (details not discussed here). Light nodes can be installed in any machine and does not require synchronization with the blockchain.

In applications using a DA layer as the data storage, clients install a light node during the usage of the application (into their browser, for instance) and connect to the blockchain without any intermediary. This provides 100% liveness to the system.

### Mina Off Chain Storage Problem

Mina is a ZK verification layer and allows only a limited amount of data to be stored on chain. Most of the time, zkApp builders must choose an off chain storage layer to build their zkApps. There are 3 main ways of achieving such integration. Please note all of these solutions are fully trustless as a merkle root is stored on the Mina chain mapping to the data.

The first is to use client side data storage solutions (e.g. browser cookies). This approach gives very high liveness guarantees and performance, but it is only adaptable to a limited amount of cases. zkApps requiring their users to access some common data cannot use a client side storage solution.

The second is to use a central database. Central databases are very efficient in terms of performance, but they have very low liveness guarantees. A system using a central database can be considered only partially decentralized.

The third option is using another blockchain to store the data. If a conventional blockchain (like Ethereum) is chosen to store the data, then the zkApp can be considered only as decentralized as the chain used. As a result, this decreases the value proposition of Mina. If a regular blockchain is used, then the liveness of the system depends on this some other blockchain, which is clearly not preferable.

The only real solution is to use a modular DA blockchain as the storage layer. DA layers provide 100% liveness on the data, so they can be considered as an excellent data layer to be integrated into Mina.

If a DA layer is used with a Mina zkApp, then the application becomes 100% decentralized with no exception.

### Mina Concurrency Problem

Mina is designed as a ZKP verification layer, thus every TX is modeled as a ZKP sent to the chain. By definition, a ZKP can have multiple verifiers, but only a single prover. Thus, if multiple ZKPs (or equivalently Mina TXs) are sent in the same block updating the same state of a zkApp, then only one of these TXs are accepted by the chain. The first one to be accepted updates the chain, making all the ZKPs generated before this update fail.

A Mina zkApp can have a limited number of states stored on chain (more specifically 8), so no zkApp with multiple users interacting at the same block can use Mina TXs as they are because of the concurrency problem.

The only way of solving the concurrency issue is to combine multiple Mina TXs into a single ZKP through proof aggregation (a.k.a. recursively combining ZKPs) and updating the Mina state in a single TX. This does not put any trust assumptions or does not decrease privacy provided by any of the ZKPs combined. Once ZKPs are recursed into a single proof, they can be settled on Mina in a single TX. As a result we can say that for each state of a zkApp only a single TX in each block is enough to include multiple updates by different users.

However, recursing ZKPs coming from different users is tricky, as all these proofs must be available to a single source who is responsible for settling them on chain. There is a built-in `Actions` and `Reducer` library inside o1js to enable storing multiple TXs as actions and reducing them at once in each block. However, unfortunately, the library is not fully functional as of the time of this proposal, and only a very limited amount of TXs can be stored as actions. Thus, it is not a real solution for real life zkApps.

The other solution is to use a regular ZK rollup architecture. As a matter of fact, almost all zkApps working on Mina can be considered as a ZK rollup. Details and problems of ZK rollups are discussed below, but in simple terms rollups are responsible for storing TXs in some off chain storage and settling them as a single ZKP through proof aggregation for every block of the settlement layer (e.g. Mina). How a native DA integration can create a huge improvement on ZK rollups is also detailed below.

Please note that `Actions` and `Reducer` library is also essentially a ZK rollup: Actions are TXs stored on archive nodes. Archive nodes are a direct part of the Mina blockchain, but as they are not constant in block size, they cannot be considered as decentralized as the native Mina chain. As a result, it is a logical approach to say there is not too much difference between using an off chain ZK rollup or the native library. Thus, problems discussed for ZK rollups also extend to the `Actions` and `Reducer` library, even when the library is considered fully functional.

### ZK Rollups and their Problems

ZK rollups are structures designed above blockchains to combine multiple TXs into a single TX and settle it on their settlement layer (a.k.a. a blockchain as the consensus layer). As the process of combining TXs is proved through ZKPs, there is no trust in the settlement process. In regular ZK rollups, the rollup layer does not have its own consensus - they directly utilize the consensus of their settlement layer.

In ZK rollups on Mina, Mina is chosen as the settlement layer. As Mina is already a constant sized ZK L1 blockchain, it is a very suitable choice for ZKP settlement. Plus, o1js supports proof aggregation very well, allowing very high amounts of rollup TXs to be included in a single Mina block.

Rollups are very scalable, fast, and cheap as they divide the cost among multiple TXs. However, they decrease liveness guarantees significantly. Most rollups consist of a single (or a few) server (called the sequencer) responsible for the settlement. If the server becomes unreachable for a reason, then the entire access to the zkApp is censored.

Moreover, even though sequencers do not have the power to change a TX they receive in any way, they decide on the ordering of TXs happening in the same settlement block (thus the name “sequencer”). In financial applications, this creates a problem of arbitrage: Sequencers may put their TXs before some significant buy / sell order to gain considerable amounts of money. In a way, sequencers can see the near future and act accordingly.

The most efficient way of increasing the liveness of a rollup is to include a DA layer in the process. If users submit their TXs into a DA layer (through their own light client) and the sequencer is only responsible for submitting them on Mina, then the problem of censorship or arbitrage disappears. As said before, almost all zkApps on Mina are rollups, so integrating a DA layer natively into Mina is very important for zkApp developers.

## Objectives

This RFC is mainly to start a discussion around possible ways of integrating a DA into Mina and get the developer community involvement in the process. There are for sure a lot of unspoken advantages and bottlenecks missing in this RFC, and it is precisely the motivation to raise some discussion points around these issues.

## Motivation and Rationale

By integrating a DA layer into Mina, the following possible improvements may be achieved:

- Creating a stable and live off chain storage environment;
- Decreasing decentralized off chain storage costs to its minimum (DA layers are usually very affordable for data storage as their main concern is data storage);
- Making it possible to implement decentralized ZK rollups on top of Mina through:
  - Increasing liveness guarantees of sequencers to its full,
  - Solving the arbitrage problem caused by the ordering of TXs by sequencers;
- Solving the concurrency issue fully through decentralized ZK rollups.

## Scenarios and Use Cases

### Example Scenario 1: Off Chain Storage Integration

| Aspect           | Description |
|------------------|-------------|
| **Description**  | When an external storage layer is implemented to a blockchain, the decentralization of the original blockchain depends on the liveness of the storage layer, which is not ideal if the storage does not have very high liveness guarantees. To not decrease Mina's decentralization and liveness, DA layers can be utilized. Just like Mina, DAs can be accessed through light clients, and users can submit their data directly into the storage layer from their browser without needing an RPC. This makes sure that a zkApp working on Mina is always reachable to its users. For more long term data, more stable data storage blockchains can be preferred (like IPFS), but DA is the perfect solution for short term liveness. |
| **Requirements** | DA layers do not usually include some kind of authentication on the submitted TXs, so anyone can push a new block in the DA layer without updating the corresponding Mina merkle tree. However, this does not risk the usage of zkApp, as there is only a limited amount of TXs that can be sent during a Mina block (considering the block time of DA ~10s, it is about 18 TXs per Mina block). Moreover, the time required to find the last corresponding block to Mina can be improved even further by implementing an ancestor tree structure (O(lg(n)) time complexity) or cryptographic hash of history on Celestia blocks (O(1) time complexity). Moveover, it is not a feasible attack vector to sensor the usage of zkApp through consistently submitting data on the DA layer, as DA has its own consensus and tokenomics model, so it is too costly to submit continuous blocks. <br> However, this is not the case for updating Mina without a corresponding DA block. If the Mina merkle tree is updated, and there is no corresponding DA block, then it is impossible to retrieve the original merkle tree. The only way of preventing such an attack is to check inside Mina that the DA is already updated before accepting the update to the Merkle tree. There are mainly two approaches to achieve such control. <br> The first is a cryptographic solution that includes checking DA validator signatures inside Mina zkApp. Of course, this requires the merkle tree roof of the set of validator public keys of DA to be stored on Mina, but these keys can also be updated by signatures, as all changes in the DA is basically a block hash signed by validator public keys. Required incentives can be provided by Mina Foundation or zkApp developers to make sure the validator set is always up to date. <br> However, verifying DA signatures can be costly on Mina. Mina uses Poseidon hash to create its signatures, and even though other more commonly used cryptographic libraries exist in Mina, they are not as efficient as Poseidon inside o1js proofs. This is a possible limitation on this approach. <br> The second approach is using an external node set as a DA proof verification layer. These nodes would be responsible for getting DA signatures, verifying their correctness (through their own light clients to not increase trust assumptions), sign the new DA block hash with their own Mina private key, gossip signed block hashes among them, and eventually settle the block hash on Mina when a required minimal percentage of signatures is obtained (like 66%). This approach is very similar to the first, the only difference is that Mina zkApp is used to verify this layer's Mina private keys instead of custom DA signatures. Moreover, the same layer can be used to integrate multiple DA layers into Mina without increasing costs. However, this approach is more expensive than the first, as this external layer should also be incentivized. This layer's liveness and trustlessness can be considered complete, as they are only light nodes, anyone can join the network without a synchronization process (nodes do not store data, but only responsible for DA signature verification), and nodes are properly incentivized to settle on Mina as fast as possible. Please note that this structure is very generic, and such blockchain verification abstraction layers can be used to integrate any kind of layer into Mina in a fully decentralized manner. |
| **Expected Outcome** | Allowing client side off chain storage solutions through DA would allow implementation of 100% decentralized zkApps on Mina with the needs of high data storage. |
| **Impact Analysis** | At the time of this proposal, there are not many ZK applications allowing direct connection of users to a storage layer, and the existence of light nodes in Mina makes it the perfect chain to create these kinds of fully live applications. Thus, it is a reasonable assumption that Mina ecosystem will see new applications and technologies that do not exist in any other blockchain with the implementation of a propoper off chain storage solution. <br> Including a DA layer as an off chain storage solution is very doable, and even individual applications implement their own solutions without waiting for a native integration on Mina side. It may be a logical move to motivate developers to work more in this direction. |

### Example Scenario 2: Concurrency Issue Fix with Availability Guarantees

| Aspect           | Description |
|------------------|-------------|
| **Description**  | As described above, the concurrency issue in Mina is solvable through an off chain aggregator layer who is responsible for recursively combining ZKPs and making sure that at most 1 TX is sent for each state of a zkApp in every block. However, off chain layers usually lack availability guarantees and thus prevents Mina zkApps to be 100% decentralized, as in general sequencers allow potential arbitrage. |
| **Requirements** | The historical data of a settled block can be stored in various storage layers (e.g. IPFS). Once a data point is sent to a distributed storage layer (either by the client or a sequencer etc.), anyone can verify its correctness at any time by comparing with the merkle root stored on chain. This case is equivalent to the [first example usage](./rfc-0011-data-availability.md#example-scenario-1-off-chain-storage-integration) described above. <br> However, there is another issue raised when off chain aggregators are used, which is the problem of ordering. ZKPs can make sure that there is no trust during the settlement process, but the sequencer is free to choose whatever ordering it wishes inside the settlement block. DA also provides a solution to this problem. <br> In this architecture, the DA layer consensus is responsible for the ordering inside the settled block, while Mina is again used as the verification layer. Instead of sending their TXs to the sequencer, clients directly submit their ZKP to the DA layer from their own light client. Then, a sequencer node gets block hashes from the DA layer, aggregates blocks as a new Mina state, and submits the settlement TX. There are two important points to notice here. First, as block heights are also a part of the DA signatures, sequencer has no longer the responsibility of ordering the settled TXs. Moreover, the so-called "sequencer" in this architecture can actually be any party running an aggregator service. As settlement is always incentivized, it can safely be assumed that this system has very high liveness guarantees and can be considered 100% decentralized. |
| **Expected Outcome** | By enabling native DA integration, it will be easy to implement this architecture, thus it is hoped that applications with this kind of very high liveness and trustlessness needs will adapt this model.  |
| **Impact Analysis** | As of the time of this proposal, there is no application in the blockchain with this kind of a technology: This architecture allows fully trustless and private applications working only on distributed layers with Web2 performance (as the aggregation is done by external servers and o1js is fast enough to be used in the client side). It supports client side computation, and it is a fully distributed system with no central mechanisms. Adapting this solution can be revolutionary for financial applications. <br> However, it is important to note that this is a very new concept, and there may be unnoticed problems in this proposal. Nevertheless, it is certain that native DA integration of the settlement layer will enable some kind of improvements in the sequencer architecture. |

## Open Issues and Discussion Points

The following questions must be answered for this RFC:

- What are the liveness guarantees of DA layers for historical data? Can we consider past data to be as available as the current data, or DA should only be used with more static storage layers (like IPFS)? Or in simpler terms, can we consider DA as the ultimate solution of the off chain storage problem, or is it helpful up to a point?
- How does block ordering work in DA consensus (the answer of this question may differ based on the DA layer considered)? Can it be easily changed by DA validators, or can it be used as a distributed ordering mechanism?
- If a modular rollup architecture is implemented, does the arbitrage problem continue through DA validators? More precisely, can DA validators use MEV for Mina zkApps?
- In this architecture, the bottleneck of scalability is the maximum number of DA blocks that can be included in a settlement block. Is it possible to improve this limitation? Potentially by using multiple DA layers for a single Mina zkApp, but this is probably impossible without creating another layer connecting the two DA chains.
- Would it be possible to get rid of the sequencer as a whole and use client side ZKP aggregation to make users using the zkApp responsible for the settlement process? It clearly decreases the performance of proof aggregation (as a custom sequencer can use OCaml etc to improve the performance while the client is restricted by the browser JVM), but it can be interesting in terms of liveness guarantees.
- How should the system be incentivized to guarantee 100% liveness? As an initial product, Mina Foundation can fund the DA integration layer, but in the long run users of the DA bridge should bring the incentives to the system.
- How Mina's hard finality time would affect modular ZK rollup structures? This is not a direct concern of this RFC, but must definitely be answered if a modular ZK rollup will be implemented.

## Conclusion

To conclude, the following points can be given as a summary:

- DA brings a very strong solution to some ongoing problems in the Mina ecosystem.
- As DA allows light nodes, just like Mina, it supports Mina's architecture without decreasing Mina's value proposition.
- DA should only be considered as a storage layer, but also as a distributed ordering mechanism that will allow 100% decentralized ZK rollups structures.
- Once the DA layer is integrated into the chain, the same architecture can easily be used to power rollups working on the Mina. Any data can be modeled as a TX, thus there is no difference between using this system to settle off chain data or off chain rollup TXs.

Finally, it is important to emphasize the following points:

- The implementation of a proof of concept is not hard to achieve, and it may inspire a lot of developers on how different chains can be integrated into Mina through decentralized sequencers.
- To put the integration in production, all of the functionality (both zkApp clients and sequencers) must be wrapped in NPM packages, allowing developers to easily use or join the DA layer.

## References

### Mina Related Resources

These resources are fundamental for open questions and implementation details.

- [Mina Off Chain Storage Tutorial](https://docs.minaprotocol.com/zkapps/tutorials/offchain-storage)
- [Mina Concurrency Issue Initial RFC](https://github.com/o1-labs/o1js/issues/265)
- [o1js Actions & Reducers Library](https://docs.minaprotocol.com/zkapps/o1js/actions-and-reducer): This is the native solution of the concurrency issue, but it has some problems as said before.
- [Mina Light Nodes](https://openmina.com/web-node)
- [Mina Hard Finality Time](https://docs.minaprotocol.com/mina-protocol/lifecycle-of-a-payment): This may be relavant for modular rollup structures.
- [o1js Merkle Trees](https://docs.minaprotocol.com/zkapps/o1js/merkle-tree)
- [o1js Recursion](https://docs.minaprotocol.com/zkapps/o1js/recursion)
- o1js Cryptographic Libraries: These may be relevant for implementation details.
  - [o1js ECDSA](https://docs.minaprotocol.com/zkapps/o1js/ecdsa)
  - [o1js Keccak](https://docs.minaprotocol.com/zkapps/o1js/keccak)
  - [o1js SHA256](https://docs.minaprotocol.com/zkapps/o1js/sha256)

### DA Related Resources

These may be helpful to better understand DA and the underlying technology.

- [DA Explanation](https://coinmarketcap.com/academy/article/what-is-data-availability)
- [DA Light Clients Example](https://blog.celestia.org/celestia-mvp-release-data-availability-sampling-light-clients/)
- [DA Light Node & Sampling White Paper](https://arxiv.org/abs/1809.09044)
- [DA Sampling](https://celestia.org/glossary/data-availability-sampling/)

### Protokit & Zeko Resources

Protokit is a ZK rollup SDK settling on Mina. Zeko is an ZK rollup L2 setting on Mina.

These sources on them may be releavant for modular proof aggregation considerations in rollups.

- [Protokit Website](https://protokit.dev/)
- [Protokit Twitter](https://twitter.com/proto_kit)
- [Protokit GitHub](https://github.com/proto-kit/)
- [Protokit Discord](http://discord.gg/xdGf2ucppM)
- [Protokit Online Playground](https://stackblitz.com/github/proto-kit/starter-kit?file=src%2FBalances.ts&startScript=test)

<br>

- [Zeko Website](https://zeko.io/)
- [Zeko Twitter](https://twitter.com/ZekoLabs/)
- [Zeko Lite Paper](https://docsend.com/view/f9a6kgdr4tjwuqng)
- [Lumina Twitter](https://twitter.com/LuminaDEX): Lumina is a ZK DEX (Decentralized Exchange) on top of Zeko. It may use a DA integration to increase availability guarantees.
- [Lumina Website](https://luminadex.com/)
- [Lumina Lite Paper](https://docsend.com/view/5tviuhs8cqditskh)
- [Lumina GitHub](https://github.com/Lumina-DEX)
- [Lumina Introductory Article](https://medium.com/luminadex/what-compliance-means-for-lumina-e84f14aa2089)