# RFC-0011: Soulboud Token Standard

- **Intent**: Create a standard for Soulbound Tokens on Mina
- **Submitted by**: Philipp Kant (email: philipp.kant@minafoundation.com, github: @kantp)
- **Submitted on**: [Insert Submission Date]

## Abstract

Soulbound Tokens (SBTs) are tokens that can be minted and burned, but not transferred. Thus, once created, they are bound to a specific account. They have been introduced in by [Weyl, Ohlhaver, and Buterin](https://deliverypdf.ssrn.com/delivery.php?ID=024094113006074084068090083083002102099074018037042059108095065004025006107091098074121060123119021098114124082003091083104116027080071064004069000082112003086113101067062033017010089116112119093027117126025089067099084001096084122028105088089029124073&EXT=pdf&INDEX=TRUE), along with several ideas for applications that they enable.

This rfc  proposes a set of standard interfaces that implementations of SBTs on Mina should adhere to. The reference implementation shows how SBTs that adhere to this standard can be implemented on Mina using `o1js`.

## Introduction

TODO

A detailed introduction to the RFC, providing context and background information. This should include:

- The problem or opportunity the RFC addresses.
- The relevance and importance of this RFC to the Mina ecosystem.

## Objectives

TODO

List the primary objectives and goals of the RFC. This section should be clear and concise, focusing on what the RFC aims to achieve.

## Motivation and Rationale

TODO

Provide a comprehensive explanation of why this RFC is necessary. This may include:

- Analysis of current challenges.
- Insights from relevant research or data.
- Connections to goals or priorities.

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
  <dt>`holderKey`</dt>
  <dd>The `PublicKey` that this token is issued to</dd>

  <dt>`issuedBetween`</dt>
  <dd>A range of slots in which the token has been issued</dd>

  <dt>`revocationPolicy`</dt>
  <dd>This determines who needs to agree to the revocation of the token. Depending on how the token is to be used, it can be sensible to require the token issuer, the token holder, or both to agree to the token being revoked. Furthermore, we allow for tokens that can never be revoked.</dd>

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

Verification of a token might or might not be a privileged operation requiring a signature from the token holder.

### Revoking a token

The contract should provide methods `revokeHolder`, `revokeIssuer`, and `revokeBoth`, for revoking tokens on behalf of the token holder, the issuer, or both. Given specific metadata, a corresponding witness, and signatures from the holder and/or issuer, those methods will check that the revocation policy of the token is compatible with the request, that the token currently exists, and that the provided signatures are valid.

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

List all references and sources cited in the RFC. Ensure proper formatting and attribution for each reference.
