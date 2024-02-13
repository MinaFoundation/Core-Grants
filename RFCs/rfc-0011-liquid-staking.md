# RFC-0011: 

- **Intent**: Solicit feedback from relevant stakeholders and understand how to move forward on liquid staking
- **Submitted by**: 45930, contact@45930.xyz
- **Submitted on**: February 12, 2024

## Abstract

With Mina's unique combination of latent delegated staking and high inflation, there is a high disincentive to provide liquidity to any ZkApps.  This is a timely problem with main net release around the corner.  A mechanism by which to allow token holders to keep pace with inflation while being allowed to move their tokens across app chains and different ZkApps on the network is necessary.

## Introduction
### How Delegated Staking Works on Mina
#### Staking Ledger
Mina is divided into epochs that last approximately 2 weeks.  At the end of one epoch, a snapshot is taken of delegated balances, called the staking ledger.  That ledger is referenced two epochs later in order to calculate rewards.  Block producers (BPs) pay out a share of stake to every delegator starting two epochs after they first delegate.  When a delegator chooses to undelegate, they will still be paid out for their inclusion in the staking ledger for two more epochs.  In this way, the Mina staking delegation has a very high lag, and 2-week discrete steps of time.
#### Payouts
Payouts made by staking pools to their delegators are made manually in good faith.  There is not a protocol-level payout feature.  Staking pool operators run scripts to calculate reward payouts and issue payments, but this can take hours or days for large pools.  According to [Mina's Documentateion](https://docs.minaprotocol.com/mina-protocol/sending-a-payment#sending-many-transactions), there is a 10tx/15 second rate limit on transactions imposed by the node software.  Extrapolating, that's 2400 per hour.  According to [Mina Explorer](https://minaexplorer.com/staking), Auro Wallet's pool currently has ~17k delegators.  Assuming best-case conditions, that's 7 hours straight of sending payments required every 2 weeks.
### How Liquid Staking works on Other Chains
On Ethereum, [Lido](https://lido.fi/) is an example LST provider where the company operates many nodes, and issues a token representing a share of their collective operation of the nodes.  Ownership of the liquid token determines the destination of rewards from the Lido staking pool.
### Other Recommended Reading
- [Defending Against LST Monopolies, Callum Waters, NashQueue, Jacob Arluck, Barry Plunkett, and Sam Hart](https://forum.celestia.org/t/defending-against-lst-monopolies/1554)
- [Mina Liquid Staking Proposal, Staketab Team](https://docs.google.com/document/d/1Lf0JMgAArAavUxs7Ef5RUJfMr2ES6qJNcy10ErGmqiQ/edit?usp=sharing)
## Objectives
- Attain buy-in from community about the need liquid staking
- Solicit comments from community leaders, developers, tokenomics experts about requirements and obstacles not addressed in the original RFC
- Forge a clear path for a successful RFP
## Motivation and Rationale

### Token Holder
As a token holder, I want access to the highest risk-free return to avoid being diluted by inflation.  I also want to be able to move my tokens freely around the Mina ZkApp ecosystem without thinking about putting tokens back in my wallet for the staking snapshot.  I believe that my tokens are mine and eligible for reward payments regardless of whether they are locked in an app chain, L2, ZkApp voting contract, or any other mechanism.  Furthermore, I expect that if I hold Mina in my wallet for 13 days, and on the 14th day I move it to an exchange, I am entitled to 13/14ths of that epoch's staking rewards, even if the staking snapshot was taken on the 14th day.
### ZkApp Developer
As a ZkApp developer, I expect the protocol not to punish users for engaging with my service.  Especially as an app chain which requires locking funds in a contract on the L1, I expect that users in my ecosystem will not miss out on staking rewards for using my app.
### Staking Pool Operator
As a staking pool operator, I want a consistently high stake delegated to me by as few individual delegators as possible.
## Open Issues and Discussion Points
### One month delegation lag
The lag between staking and receiving rewards is a critical concern to this discussion.  One idea I've seen proposed is that app chains and zkapps which store funds will simply delegate the funds and pay out their users accordingly.  Essentially they would act as mini LSTs specific to their own app and users.  But with the one-month lag, the zkapp becomes responsible for maintaining a copy of the staking ledger and processing payouts manually.  This procedure is not considered very easy or convenient by existing staking pools.  If an app ignores the staking lag, then bad actors will pile money in before payout time, but after the staking ledger snapshot is taken in order to steal extra rewards.

A freely-traded LST would price in the upcoming rewards payments efficiently like a dividend-paying stock.  So a potential approach would be to wrap the latency in the on and off ramps to the system.  2 epochs after depositing Mina, a user would be eligible to mint their LST.  During those 2 epochs they will receive staking rewards on their previous wallet.  After the LST is minted, rewards from epcoch - 2 can always be paid out instantly with no latency.  To redeem the LST for Mina, a user would enter an exit queue where they can burn the LST and receive their Mina immediately, but get an IOU for the additional 2 epochs of staking rewards which they can cash in after the 2 epochs have passed.  Something like this removes the temporal lag from the LST, but may introduce other problems.
### Selection of staking pools
A successful LST would enrich the staking pools which are selected to do the underlying staking.  Selection seems like a task for a DAO, but patterns for DAOs don't yet exist on Mina.  There is also the technical challenge of interfacing many staking pools with a single logical entity (the LST "contract").  In fact, one account may only delegate to a single pool, so there would have to be at least one contract per supported staking provider.  This seems like an obvious centralizing influence on the network which would ideally be avoided.

A potential solution to this problem would be for LST to be a protocol which any staking pool can implement.  In the same way that every pool has to manage payout today, every pool could also manage their own LST smart contract which implements whatever standards the community agrees on.  In this way there's no centralizing force other than the stakers' own quality and marketing efforts.  The decentralization would likely also lead to more scams, however.
### Compatibility with ecosystem
Just like DAOs and DEXs, Mina is also lacking a highly-adopted token standard.  There is a risk that a certain team or party implements LST in a way that is suitable only to them, or simply accidentally does not work well with some other part of the ecosystem.  The L1 ZkApp token standard seems like a front-runner for the solution, but has the downside of costing 1 Mina to create an account.

By opening this RFC, I hope to gain insight from block producers, L2 developers and zkapp developers about the feasibility and compatibility of different approaches.
## Conclusion
There is a strong need for LST on Mina, but there is not an obvious trivial implement.  The community ought to dedicate resources to understanding this problem and building a solution.
