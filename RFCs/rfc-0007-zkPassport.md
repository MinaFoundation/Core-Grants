# RFC-0007: zkPassport

- **Intent**: Create a private identity primitive for Mina and connected ecosystems that is secure and accessible.
- **Submitted by**: Evan Shapiro (Github: @es92, Twitter: @evanashapiro)
- **Submitted on**: January 8th, 2024

## Abstract

This RFC proposes a grant to build a private proof-of-passport attestation (zkPassport) leveraging smartphone NFC, ZKPs, and Mina, with an aim to provide a private identity primitive for Mina, blockchains that are connected to it, and ultimately web2 applications.

## Introduction

A secure, widely accessible implementation of private identity (proof-of-personhood) would be very useful for the blockchain ecosystem and for digital systems broadly.

This could be leveraged by applications on Mina, applications on any bridged chains (eg Ethereum), or any web3-connected web2 applications.

Use cases include:
* Proving proof of uniqueness, to apply to use cases such as one person one vote, one person one account, one person one airdrop, etc.
* Proving facts about identity (nationality, age, etc).

Historically, this has been challenging to bootstrap and make accessible without some prior information to "anchor" digital identity to.

One widely accessible place that identity information has been stored, including with cryptographic signatures and roots of trust, is passports. It seems that for decades now, passports have come with (1) NFC, (2) digital representations of their data, and (3) cryptographic signatures by their host countries[[1](https://www.icao.int/Security/FAL/PKD/Pages/ePassport-Basics.aspx),[2](https://en.wikipedia.org/wiki/Biometric_passport)]. As of writing, it seems there are currently over 140 countries issuing biometric passports.

Using this information directly online would expose passport information online, which would not be viable.

However, with ZKPs, users could provide proofs about their passport information, without exposing the passport information itself.

Additionally, because passports following this standard support NFC, which is present on most smartphones, users can obtain this data themselves from their own passports relatively easily. Here for example is an open library that reads passports for iOS[[3](https://github.com/AndyQ/NFCPassportReader)].

Properties stored in the standard include:
* documentType
* documentSubType
* documentNumber
* issuingAuthority
* documentExpiryDate
* dateOfBirth
* gender
* nationality
* lastName
* firstName
* names
* placeOfBirth
* residenceAddress
* phoneNumber

## Objectives

We propose a grant taking advantage of this, implementing, at a high level, the following:

1. An iOS & Android App that uses NFC to allow users to get the data from their own passports, and store that data in their MINA wallet for secure use by web3 applications through the (in-progress) Mina Attestation API Standard.
2. An o1js library that uses the Mina Attestation API Standard to create proofs of various components of one's zkPassport identity, for use in proof-of-unique-user and proof of nationality & age use cases.

## Motivation and Rationale

This would unlock many use cases for web3 and web2 applications connected to Mina.

For applications on Mina, they can directly take advantage of this data through nullifiers for proof-of-uniqueness and other identity related proofs (see both scenarios below).

For other chains, using Mina's zk state bridge, they could also leverage these proofs in their applications.

Web2 applications may find use in this functionality as well. This seems particularly relevant as AI systems become increasingly powerful, and the risk this could pose to new and existing digital systems being overwhelmed by fake users.

## Scenarios and Use Cases

### Example Scenario 1: Proof-of-Uniqueness

| Aspect           | Description |
|------------------|-------------|
| **Description**  | A zkApp wants a user to prove they are unique. |
| **Requirements** | An implementation of zkPassport in the Attestation API Standard (see objectives), plus an on-chain nullifier that users can use with the zkApp. zkPassport nullifier proofs should be sure to have a *context* input salt that is hashed with the unique zkPassport identifier to ensure identities are unique across their context, but cannot be associated cross-context unless a proof is provided by the user. |
| **Expected Outcome** | Via leveraging a simple library, users can make zkApps that are sybil-resistant. |
| **Impact Analysis** | This would unlock many applications, such as one-person-one-vote, one-person-one account, Sybil-Resistant airdrops, etc. |

### Example Scenario 2: Proof-of-nationality

| Aspect           | Description |
|------------------|-------------|
| **Description**  | A zkApp wants a user to prove facts about what country they are / are not from. |
| **Requirements** | An implementation of zkPassport in the Attestation API Standard (see objectives). |
| **Expected Outcome** | Via leveraging a simple library, zkApps can require a user to prove facts about their nationality. |
| **Impact Analysis** | This would be useful across web3, to make applications that are regulatory-compliant. |

## Open Issues and Discussion Points

A remaining question is exactly what new cryptography primitives would need to be implemented in o1js, if any new ones are needed. It looks like RSA, DSA, ECDSA, and Sha-xxx are used (see pages 15, 16, and 17 here[[4](https://www.icao.int/publications/Documents/9303_p12_cons_en.pdf)]).

It seems countries keep databases of lost, stolen, and revoked travel documents[[5](https://www.icao.int/publications/Documents/9303_p2_cons_en.pdf)]. It doesn't seem like this is public however, so there may also have to be logic for invalidating one's old passport if one gets a new passport (due to the old one either being lost, stolen, revoked, or just expired).

Lastly, a passport isn't something that everyone has or is able to get, so this is unlikely to work as a universal solution for proving personhood. That said, it could be a useful "anchor", for decentralized social-graph based approaches, which I think could be promising future work.

This could be done via passport-holders verifying non-passport-holders. One way this could work is requiring multiple passport-holders, say 3, to attest to the validity of a non-passport-holder, and limiting the number of times they can do this (if 3 passport-holders are required, and each passport-holder gets 3 verifications, this would mean each passport-holder could verify up to one non-passport-holder on average).

Applications using zkPassport should be very clear what exactly it is they are proving - specifically, proof of having had access, at some point during a passports lifetime, to the data contained on the passport's NFC chip.

Users of applications should be careful to keep their passports secure. Note that passport RFIDs can only be read while the passport is open and a portable reader is within 6" of the passport, somewhat mitigating this risk.

However if a user suspects someone may have had access to their passport they should consider expiring it and getting a new one (open question: how hard is this to do?), and an implementation of zkPassport should include logic for expiring old passports and replacing them with new ones. 

Additionally, there are various social and privacy risks to enabling digital composable identity. Including a "context"-specific salt in nullifiers is mentioned above as one way to ensure users have choice over what to expose about their cross-context identity-specific operations. But other risks and mitigations should be considered as well with this new technology.

## Conclusion

This RFC proposes an implementation of a private identity system leveraging passports (zkPassport) for use in the Mina Ecosystem and any connected chains.

## Appendices



## References

International Standard Docs[[6](https://www.icao.int/publications/pages/publication.aspx?docnum=9303)].

See other links in-line.
