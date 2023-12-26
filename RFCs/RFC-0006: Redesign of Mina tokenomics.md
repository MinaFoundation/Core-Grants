# RFC-0006: [Review and Redesign of Mina Tokenomics]

- **Intent**: [To request that Mina Foundation commission a reputable research team to redesign Mina tokenomics to best incentivise protocol developments.]
- **Submitted by**: [lamps (discord: lamps6718, twitter: lamps945); Pete(discord: flushthefashion, twitter: minacryptocom) ]
- **Submitted on**: [27 Dec 2023]

## Abstract

Mina’s tokenomics, especially the high inflation brought by supercharged rewards, has been the concern of many community members, as evident by the number of posts on the [MIP channel of Mina Research](https://forums.minaprotocol.com/c/mips-mina-improvement-proposals/24). The upcoming hardfork is set to introduce zkApps (smart contracts), layer 2s, and many other exciting but complicated features to Mina, which also makes the original tokenomics design - a linear emission model - outdated. In this RFC idea, instead of proposing a tokenomics model directly, we outline the importance, opportunity and difficulty to design a successful model for Mina, and propose that Mina Foundation commission a thorough review and redesign of Mina’s tokenomics post hardfork, taking in account the critical aspects as outlined here.

## Introduction

**'Incentives are superpowers; set them carefully.'** - [Sam Altman](https://blog.samaltman.com/what-i-wish-someone-had-told-me).

Tokenomics is as important as the consensus algorithm to a blockchain. On one hand, it contributes to the security by making attacking the chain more costly; on the other hand, it creates incentives for people to participate in the operation of the chain in various roles, from miners/block producers, to ecosystem builders, to end users.

**Mina’s tokenomics is long outdated** Mina’s current tokenomics was designed prior to the mainnet launch, and supercharged rewards (24% APR) for unlocked tokens was introduced to temporarily incentivize token holders to stake Mina. It worked at the time, but supercharged rewards were planned to be gradually wound down from 24% to 12% (equal to the APR for locked tokens) and eventually removed within 16 months of mainnet launch (see Section 6 of [this blog post](https://minaprotocol.com/blog/mina-token-distribution-and-supply)). This did not get implemented, though, as the hardfork of smart contract was delayed by almost two years, and the inflation has been continuously running at >20% the whole time. This hyper inflation has been a concern for the community for a long time. Even though an MIP has been passed to remove supercharged rewards at the planned hardfork, this theoretical inflation will still be 8.8% ([calculations by Gareth](https://forums.minaprotocol.com/t/mip-to-review-and-redesign-mina-tokenomics/6131/6): max of 7140 slots and 75% fill rate so 5355 max blocks. 5355 x 720 = 3.8556 million MINA increase an epoch. The current supply is 1.0724 billion, so that’s an increase of 0.00359517041% an epoch, annualized that’s 8.8%. If considering an active stake of 77.71%, the inflation rate would be ~8.0%). In comparison, the inflation rates of major L1 chains, for example, Ethereum, Solana and Cardano, are all < 7%.

**Upcoming zkApps present overhaul of token utilities - and opportunities for innovation.** The hardfork will introduce zkApps as well as layer 2s on top of Mina. As a result, the Mina token’s utility will go from solely for paying for transactions, to multi-functions including paying for zkApp transactions, issuing custom tokens, and bridging to other chains and L2s and securing these bridges. These added utilities provide a golden opportunity for a tokenomics overhaul. If designed perfectly, it will allow the Mina token to capture these wonderful new value creations on top of Mina, which will then attract more talented builders and enthusiastic participants to the ecosystem, thus creating a positive spiral effect to grow the Mina brand, ecosystem and community.

## Objectives

List the primary objectives and goals of the RFC. This section should be clear and concise, focusing on what the RFC aims to achieve.

## Motivation and Rationale

Provide a comprehensive explanation of why this RFC is necessary. This may include:

- Analysis of current challenges.
- Insights from relevant research or data.
- Connections to goals or priorities.

## Scenarios and Use Cases

In this section, describe specific scenarios and use cases that the proposed solution should satisfy. This helps to illustrate the practical application of the solution and ensures it meets the needs of users or stakeholders. For each scenario, include:

- **Scenario Title**: A short, descriptive title of the scenario or use case.
- **Description**: A detailed explanation of the scenario, outlining the context and specific conditions.
- **Requirements**: List the specific requirements or criteria that the solution must meet in this scenario. User stories and behavior driven development formats like Cucumber are welcome.
- **Expected Outcome**: Describe the desired outcome or result after applying the proposed solution in this scenario.
- **Impact Analysis**: Analyze the impact of the solution on the scenario, including benefits and potential challenges.

### Example Scenario 1: [Scenario Title]

| Aspect           | Description |
|------------------|-------------|
| **Description**  | _Provide a detailed explanation of the scenario, outlining the context and specific conditions._ |
| **Requirements** | _List the specific requirements or criteria that the solution must meet in this scenario._ |
| **Expected Outcome** | _Describe the desired outcome or result after applying the proposed solution in this scenario._ |
| **Impact Analysis** | _Analyze the impact of the solution on the scenario, including benefits and potential challenges._ |

## Open Issues and Discussion Points

List any remaining open issues or questions that need to be resolved. Encourage discussion and feedback on these points.

## Conclusion

Summarize the key points of the RFC and reiterate its importance and expected outcomes.

## Appendices

Include any additional material that supports the RFC, such as data tables, detailed analysis, or extended examples.

## References

List all references and sources cited in the RFC. Ensure proper formatting and attribution for each reference.
