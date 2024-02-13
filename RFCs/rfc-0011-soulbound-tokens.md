# RFC-0011: Soulboud Token Standard

- **Intent**: Create a standard for Soulbound Tokens on Mina
- **Submitted by**: Philipp Kant (email: philipp.kant@minafoundation.com, github: @kantp)
- **Submitted on**: [Insert Submission Date]

## Abstract

Soulbound Tokens (SBTs) are tokens that can be minted and burned, but not transferred. Thus, once created, they are bound to a specific account. They have been introduced in[^1], along with several ideas for applications that they enable.

This rfc proposes a set of standard interfaces that implementations of SBTs on Mina should adhere to. The reference implementation shows how SBTs that adhere to this standard can be implemented on Mina using `o1js`.

## Introduction

Soulbound tokens (SBTs) are non-fungible tokens that cannot be transferred. Once created, they are tied to a specific account (called a "Soul" in this context) for as long as they exist.

While this is a severe restriction compared to ordinary non-fungible tokens (NFTs), this restriction does enable a number of use cases. Think of tokens that are created to represent a certain fact about a person, such as a POAP that attests attendance at an event, or a KYC token stating that someone has undergone some know your customer. If those tokens can be transferred, they are much less significant, as it is straightforward to trade them. However, if such a token is implemented as an SBT, there is a system level guarantee that it has never been moved.

In addition to these attestations, SBTs also allow representing a class of real world contracts that grant the right to use an asset, without the possibility of reselling that right (such as an appartment lease).

There are different ways in which SBTs can be created. While some tokens might be created by a soul and issued to itself, the more useful case is when one soul (called issuer in the rest of this docuemt), creates a token that is bound to another soul (called the owner of that SBT below). In that case, the SBT can represent a given relationship between two parties on the chain. An employer, maintainer of an open source project, or club might issue a token to their employees/contributors/members. Other tokens might represent credentials from succeeding at university or online courses, winning a competition, etc.

Beyond simply tracking individual relationships such as having completed KYC, the paper[^1] suggests to use the collective of SBTs to establish _social provenance_, which enables a number of interesting use cases, including

- Community Recovery: a way of recovering your private key by having a qualified majority of related souls attest to a users identity
- Sybil protection in DAO governance: determining voting power as a function of the reputable SBTs of a soul, to dampen or remove the influence of bots
- Measuring decentralisation, including social dependencies
- Encouraging cooperation across different homogeneous groups

and many more.


All in all, soulbound tokens are an interesting and promising concept. This RFC intents to facilitate their use in the Mina ecosystem by establising a standard for how to create them and interact with them.

## Objectives

1. Establish a minimal standard for an API to create, verify, and revoke SBTs. Having a standard for this is cruciual for integration with wallets and any kind of zkApp might want to use SBTs. Standardisation is particularly important when it comes to capturing the collective of SBTs for the purpose of social provenance.
2. Provide a reference implementation for SBTs.

## Motivation and Rationale

Soulbound tokens are a fairly new concept, and currently, there is neither a standard nor an existing implementation of an SBT on Mina. Ethereum does have a standard in the form of ERC-5484[^2]. We should aim to create a standard on Mina as well, preferably before we get a number of implementations with different interfaces.

## Scenarios and Use Cases

TODO

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

## Design and User Flow

### On-chain smart contract and off-chain service

In accordance with the Mina design philosophy, we choose an architecture where we keep just enough state on chain to maintain consensus on the existing tokens.

Concretely, each kind of soulbound token is defined by a smart contract that can issue that kind of token. The contract maintains a `MerkleMap`, where each leaf represents an instance of the token. The key is derived from the metadata of the token, which includes the `PublicKey` that the soulbound token is issued to. A leaf can have one of three values, depending on whether a token with the given metadata has never been issued, is valid, or had been issued, but also revoked.

Only the root of the `MerkleMap` is stored on-chain as part of the contract state. Most operations do require knowledge of more than the root, specifically to create a merkle witness. Thus, the on-chain contract shall be augmented by an off-chain service.

How the off-chain service is run, and how the data is stored, is an implementation detail that is out of scope for this document; here, we only specify the functionality that it should provide, and how it should interact with the on-chain component.

We will refer to the contract defining a soulbound token as "the contract", and to the corresponding service that maintains the off-chain state and contructs witnesses as "the off-chain service" below.

### Token Metadata

The token metadata contains four fields

<dl>
  <dt>`ownerKey`</dt>
  <dd>The `PublicKey` that this token is issued to</dd>

  <dt>`issuedBetween`</dt>
  <dd>A range of slots in which the token has been issued</dd>

  <dt>`burnAuth`</dt>
  <dd>This determines who needs to agree to the revocation of the token. Depending on how the token is to be used, it can be sensible to require the token issuer, the token owner, or both to agree to the token being revoked. Furthermore, we allow for tokens that can never be revoked. See [ERC 5485](https://eips.ethereum.org/EIPS/eip-5484)</dd>

  <dt>`attributes`</dd>
  <dd>his array of values of type `Field`, holds custom data, according to the specific use case.</dd>
</dl>

### Issuing a token

The contract defining the token should provide a method `issue` that creates a token and binds it permanenty to a `PublicKey` on Mina. The caller provides the metadata, as well as a `MerkleMapWitness` that should prove that the token has not been issued yet.

Constructing the witness will require knowledge of the current `MerkleMap`, abd thus will relyt on the off-chain service.

The contract will verify that the token has not existed previously. It will also evaluate a business logic particular to the concrete type of token, to determine whether the token should indeed be issued.

Once those checks pass, the contract will modify the root of the `MerkleMap` to include the token. The off-chain service will modify it's `MerkleMap` once the call to the contract succeeds.

### Verifying a token

The contract should provide a method `verify`. Given some metadata and a corresponding `MerkleMapWitness`, it shall validate that the witness is indeed a valid witness for an existing, non-revoked token with the given metadata, and matches the on-chain merkle root.

Constructing the witness shall be performed by the off-chain service. The service should also return that witness to the user, so that they can reproduce the proof of inclusion against the merkle root.

Verification of a token might or might not be a privileged operation requiring a signature from the token owner.

### Revoking a token

The contract should provide methods `revokeOwner`, `revokeIssuer`, and `revokeBoth`, for revoking tokens on behalf of the token owner, the issuer, or both. Given specific metadata, a corresponding witness, and signatures from the owner and/or issuer, those methods will check that the revocation policy of the token is compatible with the request, that the token currently exists, and that the provided signatures are valid.

If all checks succeed, the contract will update the merkle root to reflect revocation, and the off-chain service will update its `MerkleMap`.

Again, construction of the witness is enabled by the off-chain service.

### Events

Every time the merkle root is updated, the contract might emit an event publishing the new value of the root. This allows users to reproduce (off-chain) a proof that they held a specific token at a given time.

### Temporary Forks

As a proof of stake system, Mina guarantees eventual consistency, but any change that is committed to the chain has a finite probability to be reverted during resolutiuon of a temporary fork.

This probability has to be accounted for in a production implementation of soulbound tokens. In particular, a disagreement between the on-chain contract and the off-chain service on the current state of the `MerkleMap` has to be excluded.

This is a common problem when dealing with blockchains or other distributed systems, and there are a number of ways in which the off-chain service can assure consistency with the contract:

1. Keeping Snapshots
   The off-chain system does not only store the most recent value of the map, but a number of snapshots. Whenever there is a rollback of a temporary fork that has an effect on the map, it reverts to the appropriate snapshot.
2. Event Log Style
   Going all-in on the idea of snapshots, the service could store the map as an event log, keeping all the actions that modified it to arrive at the current state. A rollback in this model would be handled by ignoring the tail of the event log.
3. Public Event Log
   Taking this idea even further, the contract could me modified to emit events that completely describe each modification of the merkle map. This approach has the advantage of allowing anyone observing those events to replicate the off-chain service, removing a potential point of centralisation.
4. Retrying Transactions
   In addition, the service can resubmit transactions that were lost in a rollback. This can improve the user experience, but in itself is not sufficient to ensure consistency. In particular, some transactions might be rejected at resubmission because the time interval to issue a token has passed. Furthermore, attempts to issue/validate/revoke a token would fail in the time between the rollback and the reinclusion of the relevant transactions.

## API Specification

## Reference Implementation

The repository https://github.com/MinaFoundation/soulbound-tokens contains a reference implementation of a contract defining a soulbound token. As part of the tests, it contains a simple example of an off-chain service that keeps the map in memory.

## Open Issues and Discussion Points

List any remaining open issues or questions that need to be resolved. Encourage discussion and feedback on these points.

## Conclusion

Summarize the key points of the RFC and reiterate its importance and expected outcomes.

## Appendices

Include any additional material that supports the RFC, such as data tables, detailed analysis, or extended examples.

## References

[^1]: Ohlhaver, Puja and Weyl, Eric Glen and Buterin, Vitalik, Decentralized Society: Finding Web3's Soul (May 10, 2022). Available at SSRN: https://ssrn.com/abstract=4105763 or http://dx.doi.org/10.2139/ssrn.4105763

[^2]: Buzz Cai (@buzzcai), "ERC-5484: Consensual Soulbound Tokens," Ethereum Improvement Proposals, no. 5484, August 2022. [Online serial]. Available: https://eips.ethereum.org/EIPS/eip-5484.
