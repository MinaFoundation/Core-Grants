# RFC-0006: [Review and Redesign of Mina Tokenomics]

- **Intent**: [To request that Mina Foundation commission a reputable research team to redesign Mina tokenomics to best incentivise protocol developments.]
- **Submitted by**: [lamps (discord: lamps6718, twitter: lamps945); Pete(discord: flushthefashion, twitter: minacryptocom) ]
- **Submitted on**: [26 Dec 2023]

## Abstract

Mina’s tokenomics, especially the high inflation brought by supercharged rewards, has been the concern of many community members, as evident by the number of posts on the [MIP channel of Mina Research](https://forums.minaprotocol.com/c/mips-mina-improvement-proposals/24). The upcoming hardfork is set to introduce zkApps (smart contracts), layer 2s, and many other exciting but complicated features to Mina, which also makes the original tokenomics design - a linear emission model - outdated. In this RFC idea, instead of proposing a tokenomics model directly, we outline the importance, opportunity and difficulty to design a successful model for Mina, and propose that Mina Foundation commission a thorough review and redesign of Mina’s tokenomics post hardfork, taking in account the critical aspects as outlined here.

## Introduction

**'Incentives are superpowers; set them carefully.'** - [Sam Altman](https://blog.samaltman.com/what-i-wish-someone-had-told-me).

Tokenomics is as important as the consensus algorithm to a blockchain. On one hand, it contributes to the security by making attacking the chain more costly; on the other hand, it creates incentives for people to participate in the operation of the chain in various roles, from miners/block producers, to ecosystem builders, to end users.

**Mina’s tokenomics is outdated.** Mina’s current tokenomics was designed prior to the mainnet launch, and supercharged rewards (24% APR) for unlocked tokens was introduced to temporarily incentivize token holders to stake Mina. It worked at the time, but supercharged rewards were planned to be gradually wound down from 24% to 12% (equal to the APR for locked tokens) and eventually removed within 16 months of mainnet launch (see Section 6. Initial distribution Categories, 1d)Supercharged rewards of [this blog post](https://minaprotocol.com/blog/mina-token-distribution-and-supply)). The winding down and removal of supercharged rewards did not get implemented, though, as the hardfork of smart contract was delayed by more than two years, and the inflation has been continuously running at >20% during the whole period. This unplanned high inflation has been a concern for the community (N.B.: Gareth pointed out that the combined inflation rate has historically been around 12%, which is actually very close to the original tokenomics target. Though this was not really by design, but a combination of fixed coinbase and not all MINA staked). 

Even though an MIP has been passed in early 2023 to remove supercharged rewards at the planned hardfork, which is expected in 2024, by the time the unplanned inflation will have run more than 15 months, and the theoretical inflation will still be 8.8% ([calculations by Gareth](https://forums.minaprotocol.com/t/mip-to-review-and-redesign-mina-tokenomics/6131/6): max of 7140 slots and 75% fill rate so 5355 max blocks. 5355 x 720 = 3.8556 million MINA increase an epoch. The current supply is 1.0724 billion, so that’s an increase of 0.00359517041% an epoch, annualized that’s 8.8%. If considering an active stake of 77.71%, the inflation rate would be ~8.0%). In comparison, the inflation rates of major L1 chains, for example, Ethereum, Solana and Cardano, are all < 7%.

**Upcoming zkApps present overhaul of token utilities - and opportunities for innovation.** The hardfork will introduce zkApps as well as layer 2s on top of Mina. As a result, the Mina token’s utilities will go from solely for paying for transactions on the base layer, to multi-functions including paying for zkApp transactions, issuing custom tokens, and bridging to L2s and other chains, and securing these bridges. These added utilities provide a great opportunity for a tokenomics overhaul. If designed perfectly, it would allow the Mina token to capture these new value creations on top of Mina, which will then attract more talented builders and enthusiastic participants to the ecosystem, and spur network growth, thus creating a positive spiral effect to grow the Mina brand, ecosystem and community.

## Objectives
An ideal tokenomics design for Mina needs to take considerations of all the following aspects and perhaps some more:

- Security of the blockchain. In addition to the POS considerations on Mina's native chain, one additional thing that could work in Mina's favour is that it could inherent security from Ethereum, once its state can be verified on Ethereum via the Mina -> Ethereum bridges worked by Nil Foundation and Lambda Class. See discussions [here](https://polynya.medium.com/nil-foundations-trustless-bridges-9d205929f69).  
- Value capture
- Incentives, for people to participate in and to build on top of the chain. It can only be achieved by striking a delicate balance between inflation and deflation mechanisms, and token prices.
- Mina's unique design of off-chain computation & on-chain verification, which means the verification 'gas fees' are the same no matter how complex the off-chain computation is.
- Utility with zkApps, eg custom token issuance (similar to ERC20 tokens using Ether) and settling the zkApp transactions on the base layer. 
- Symbiosis with Layer 2s on top of Mina: ZekoLabs, Protokits, Anomix etc.

## Proposed approach
To achieve all these design goals at the same time is obviously a multidisciplinary task involving many sub fields of computer sciences and economics. While MF and o1Labs have established themselves at the forefront zk, in our opinion the design of the tokenomics would benefit significantly from engaging with other leading experts encompassing the broad fields with proved successful experiences. Therefore, through this RFC, we would like to request that Mina Foundation commission a world leading team to conduct a comprehensive review and redesign Mina’s tokenomics. As far as the author’s knowledge, the following teams would be good choices (disclaimer: we are not associated with these teams in any way):

- Gauntlet (https://www.gauntlet.xyz/ 11 ) led by Tarun Chitra, are a leading expert team for tokenomics design. Coincidentally, Gauntlet also reviewed Mina’s original tokenomics upon mainnet launch in 2021.
- Delphi Digital. Most famously designed Axie Infinity’s tokenomics in 2021 which was a great success.
- Blockworks Research

## Initial comments from the MinaResearch post

We had positive feedbacks from the community, with most people supporting the review and redesign a long-term plan of tokenomics. See the details on [the link to the post](https://forums.minaprotocol.com/t/mip-to-review-and-redesign-mina-tokenomics/6131). Here I copy over the replies from Evan (then CEO of Mina Foundation):

>Hey, yeah I would be supportive of this too.
>
>My personal take (not formally speaking on behalf of MF either) would be reducing block rewards to 500, plus 33% fee burning, I think that would be a good step forward for the next HF after zkApps (pending perhaps seeing if fee burning is complex to implement). Some thoughts on why for each of these too. (Ultimately, I’d like to emphasize that this is up to the Mina community and would love to see an MIP on it)
>
> - 500 block reward - We want to make sure staking participation stays strong for network security, so a modest decrease from the current 720 probably makes sense. ~5% annual inflation towards network security has some precedence on other networks too such as Ethereum.
>
> - 33% fee burning - this probably won’t do very much until network activity goes up, but having this agreed and implemented would be a strong statement I think about what to expect from the tokenomics, vs it just being planned or being discussed. This also now has some precedence with Ethereum too with their fee market & burning.
>
>I’d also be supportive of some kind of tokenomics work / report too on different options and tradeoffs, given its a big decision I think. I’d be curious if there are any groups that have looked at L1s in particular, as opposed to most of their work being on the DeFi side.
>
>Ecosystem partners are heads down on the upcoming hardfork, but I think as a next step, after the hardfork MF could put forward an RFP, and the community could decide via an on-chain-vote on selecting the right service provider if folks want to go that route?

and Emre (CEO of o1Labs):

>[speaking from personal perspective and not representing O(1) Labs’ view]
>I am supportive of investing more time and energy into revamping Mina’s tokenomics. Both Gauntlet and Delphi are well reputed teams so supportive of working with them too. To me there are two aspects however to any research and proposal:
>
> - Model: Do we stick to current fixed inflation; or have something more sophisticated and dynamic like Ethereum’s. Personally I would be strongly in favor of a fee burn model that allows the supply to go deflationary based on increasing usage of block space.
>
> - Implementation: How does the Model above get implemented into the protocol. e.g. if kept at fixed inflation, is there a need for protocol to dynamically adjust block rewards to actually target fixed inflation (based on fill rate / slot time / other factors); or is the community ok with something more simple like what there is today.

## Open Issues and Discussion Points

From the MinaResearch post, a few points were brought forward for discussion: 

- **How to decide which service provider to choose.** It might be more efficient to omit MIP process in the RFP stage, but relies on MF/ o1's experience with the said providers and their proposals to decide who to go for (could be beneficial to have two teams working on it). 
- **How to decide which new design to implement to the protocol.** After the review and redesign processes are done, an MIP could be used to decide which one to implement. The flow looks like: RFC/RFP to redesign Mina tokenomics → getting proposals and designs from Gauntlet/Delphi etc → MIP to vote on the new tokenomics to be integrated onchain. 
- **Whether a redesign of tokenomics is needed at all.** This was brought by Talha in [his reply](https://forums.minaprotocol.com/t/mip-to-review-and-redesign-mina-tokenomics/6131/13). It is entirely possible that the current tokenomics work well post hardfork, but in that case the review would conclude so, therefore the review is still worth commissioning.
- **parametric design of tokenomics with a governing council to oversee the parameter configurations.** This was brought up by Jonathan in his reply to the post, see the [original idea](https://forums.minaprotocol.com/t/mip-to-review-and-redesign-mina-tokenomics/6131/18) and [further explanation](https://forums.minaprotocol.com/t/mip-to-review-and-redesign-mina-tokenomics/6131/22).  

## Conclusion

We have presented the historical issues of Mina's tokenomics and the imminent oppotuinities presented by the hardfork, and propose that a review and redesign of the tokenomics is commissioned to a reputable team within the crypto space. We also reviewed the objectives of the redesign, and presented comments from the Mina community and key discussion points for the RFC. 


## References

[1]. https://forums.minaprotocol.com/t/mip-to-review-and-redesign-mina-tokenomics/6131
