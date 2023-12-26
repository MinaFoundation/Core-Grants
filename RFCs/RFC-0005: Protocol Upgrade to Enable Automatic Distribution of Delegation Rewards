# RFC-0005: [Protocol Upgrade to Enable Automatic Distribution of Delegation Rewards]

- **Intent**: [To request a protocol upgrade for automatic distribution of delegation rewards.]
- **Submitted by**: [lamps, discord: lamps6718, twitter:lamps945 ]
- **Submitted on**: [26 Dec 2023]

## Abstract

This RFC aims to address one big pain point affecting all node operators on Mina network, which is the distribution of delegation rewards. The current mechanism relies the node operators to voluntarily calculate the rewards and send them back to the delegators. It is error-prone for the calculations and, more importantly, relies complete trust of the node operator, which hinders the decentralisation of the network. The RFC requests a redesign of the mechanism and achieve one that is similar to the Cosmos chains of Cardano, where the rewards are not accessible to the node operators, and the delegators can check and claim the rewards any time.   

## Introduction

A big pain point for all node operators on Mina is how delegation rewards are distributed. Currently, all block rewards go to operators' wallet, and it relies on the operators to calculate the amounts and send them to the delegators. The calcualtion is not trivial and can lead to mistakes, especially when one receives delegations from both locked and unlocked accounts and only the latter is eligible for supercharged rewards (to be removed in hardfork). Moreover, this entire process is voluntary and there has been one or two cases right after mainnet launch in 2021, where the operator ran away with the foundation delegation rewards, and nobody could do anything about it. 

Two major issues come with this relatively rudimentary design: one is the congestions of the network at the end of each epoch when the snapshot of ledger is taken, which resulting in most operators choosing to send transaction during such times. The other is that the Mina holders will only delegate to the operators who they can completely trust, to avoid risking loss of rewards, which hurts the decentralisation of the protocol and results in the funds being concentrated to a small number of reputable operators, instead of having a long tail distribution of decent-sized operators and enthusiasts.

## Objectives

A redesign of the mechanism and achieve one that is similar to the Cosmos chains of Cardano, where the rewards are not accessible to the node operators, and the delegators can check and claim the rewards any time. 

This will help resolve both issues mentioned above: the reward claim transactions can be much evenly distributed in time during each epoch (instead of congestion towards the end/beginning), and the necessity to trust a node operator is removed.  

## Motivation and Rationale

As mentioned in the Introduction section.

## Scenarios and Use Cases

### Example Scenario 1: [User delegates and claims rewards from wallet/explorer]

| Aspect           | Description |
|------------------|-------------|
| **Description**  | Node operators set the delegation charge (e.g. 5 percents) onchain, which is viewable on wallet/explorer interface to inform users when choosing who to delegate to. For claiming the rewards, a section shows 'Claimable staking reward: $amount' and a button 'Claim all' in the wallet and explorer UI, with $amount calculated from the delegation charge and the proportion of total reward. Under the hood, the node client exposes an API for calculating the claimable amount for display, and when the user hits 'Claim all' button, the wallet/explorer submit a transaction onchain to claim the rewards. |
| **Requirements** | _Protocol redesign to allow the new reward distribution mechanism, node client implementation to expose relevant API endpoints, wallet and explorer integration._ |
| **Expected Outcome** | _as described above._ |
| **Impact Analysis** | _A much smoother UX for general Mina users; more evenly distributed and less congested transactions on L1, remove uncessary trust in the delegation process._ |

## Open Issues and Discussion Points

Community member and MinaExplorer operator, garethdavies, has conducted some research and given an example of doing pool payouts with a zkApp: [1]. https://hackmd.io/@garethtdavies/BJH3xMpFs. To quote him: 'there are numerous issues with this, not least the reliance on an oracle for block history and currently no way to bind the coinbase receiver to the zkApp.'

## Conclusion

This RFC seeks a solution to the current rudimentary distribution mechanism of delegation rewards on Mina. An automatic claim mechanism, similar to what is employed on Cosmos chains or Cardano, would reduce congestions on L1, remove unncessary trust requirements, and considerably improve the user experiences on Mina. 


## References

[1]. https://hackmd.io/@garethtdavies/BJH3xMpFs.
