# RFC-0011: Formal Verification of Kimchi and Pickles in Lean 4

**Intent:** Core Grant Proposal

**Submitted by:** Onyeka Obi, PhD Mathematics, Independent Researcher, Washington State, USA

**Submitted on:** 2026-03-25

---

## Abstract

This RFC proposes the first formal verification of Mina's Kimchi proof system and Pickles recursive composition layer using the Lean 4 theorem prover. Mina's defining property (a constant ~22KB blockchain state) depends entirely on the correctness of these two cryptographic subsystems, yet no mechanized verification of either system exists in any theorem prover. This project will construct formal models of Kimchi's PLONKish arithmetization (including custom gates, Plookup integration, and the inner-product-argument-based polynomial commitment scheme) and Pickles' recursive proof composition (including deferred accumulator verification over the Pasta curve cycle). The deliverables are machine-checked Lean 4 proofs of soundness for each critical component, published as open-source public goods for the Mina ecosystem.

---

## Introduction

Mina Protocol achieves its succinct blockchain property through a layered proof architecture. At the base, Kimchi provides the core SNARK proving system built on PLONKish arithmetization with 15 witness columns, custom gates for foreign field arithmetic, Keccak, Poseidon, elliptic curve operations, and boolean logic, plus Plookup-based lookup arguments. On top of Kimchi, Pickles implements recursive proof composition: proofs that verify other proofs, operating across the Pasta curve cycle (Pallas and Vesta) with a mirrored circuit structure and deferred accumulator commitment verification. This recursion is the mechanism that compresses the entire blockchain into approximately 22KB.

Despite the critical role these systems play, no formal verification effort has been undertaken for Kimchi or Pickles in Lean 4, Coq, Isabelle, or any other proof assistant. The existing assurance comes from code review, testing, and informal reasoning. While valuable, these methods cannot provide the mathematical certainty that formal verification delivers.

Lean 4 is the ideal tool for this effort. Its Mathlib library provides mature formalizations of finite fields, polynomial rings, and elliptic curves; the algebraic infrastructure required to model Kimchi's constraint system and the Pasta cycle. Lean 4's metaprogramming capabilities (tactics, macros, elaboration hooks) enable domain-specific proof automation for constraint-system reasoning. Furthermore, Lean 4's compiled performance makes it practical to work with the scale of definitions required by a production proof system.

This project would produce the first formal model of a recursive SNARK composition in any theorem prover: a contribution not only to Mina but to the broader field of verified cryptography.

---

## Objectives

1. **Formalize Kimchi's constraint system in Lean 4.** Define the PLONKish arithmetization (15 columns, permutation argument, vanishing polynomial construction) and prove that the constraint system is complete and sound with respect to intended gate semantics.

2. **Verify custom gate correctness.** For each of Kimchi's custom gates (foreign field addition, foreign field multiplication, Keccak round, Poseidon sponge, XOR, generic arithmetic, elliptic curve addition/doubling), prove that the polynomial constraint equations correctly enforce the intended computation.

3. **Formalize and verify the Plookup argument.** Model the lookup argument used for range checks and random-access arrays; prove soundness (a cheating prover cannot fake membership) and completeness (honest execution always succeeds).

4. **Model the Pasta curve cycle.** Formalize the Pallas and Vesta curves, their relationship as a 2-cycle, and the field arithmetic that underpins both Kimchi circuits.

5. **Formalize Pickles' recursive composition.** Model the deferred accumulator commitment verification pattern, the mirrored circuit structure operating over both curves, and prove that recursive composition preserves soundness across proof layers.

6. **Verify the Fiat-Shamir transcript construction.** Model the transcript protocol used to derive verifier challenges and prove that it correctly instantiates the random oracle model (i.e., all prover messages are absorbed before challenges are squeezed).

7. **Publish all results as open-source Lean 4 libraries** under a permissive license, with documentation sufficient for independent review and extension.

---

## Motivation and Rationale

**Mina's security model concentrates trust in Kimchi and Pickles.** Unlike blockchains where nodes independently re-execute transactions, Mina nodes verify a single succinct proof. If Kimchi's constraint system has a soundness bug, an attacker could forge proofs of invalid state transitions. If Pickles' recursion has a flaw, the entire chain of recursive proofs could be undermined. The 22KB property that defines Mina is simultaneously its greatest strength and its most concentrated point of risk.

**Informal verification has known limitations.** Custom gates involve complex polynomial identities over large finite fields. The foreign field multiplication gate alone requires multi-limb decomposition with carry propagation, range checks, and native-vs-foreign field boundary management. Human review of these constraints is error-prone, and testing can only cover a finite subset of the input space. Formal verification provides exhaustive coverage: a machine-checked proof that the constraints hold for all valid inputs and reject all invalid ones.

**Pickles' deferred computation model is subtle and unprecedented.** The decision to defer accumulator commitment verification (never proving it inside the circuit) is central to Pickles' efficiency, but it introduces a non-obvious trust assumption. Formalizing this mechanism would make the assumption explicit and machine-checkable, giving the Mina community a precise characterization of what Pickles does and does not prove at each recursion step.

**Lean 4's ecosystem is ready.** Mathlib's formalization of finite fields (`ZMod`, `GaloisField`), polynomial rings (`MvPolynomial`, `Polynomial`), and elliptic curve arithmetic provides the algebraic foundation. Recent work on formalized cryptography in Lean 4 (hash functions, commitment schemes) demonstrates the feasibility of protocol-level verification.

**This is a public good.** The Lean 4 formalization would serve as a machine-readable specification for Kimchi and Pickles, useful to auditors, researchers, and developers building on Mina. It would also serve as a reference implementation for anyone formalizing recursive SNARK systems in other provers.

---

## Scenarios and Use Cases

### Scenario 1: Custom Gate Soundness Verification

| Field | Detail |
|---|---|
| **Description** | Formalize the polynomial constraint equations for Kimchi's custom gates and prove that each gate's constraints are sound (no invalid witness can satisfy them) and complete (all valid witnesses do satisfy them). Priority targets: foreign field multiplication, Poseidon, and elliptic curve operations. |
| **Requirements** | Access to Kimchi gate specifications (available in the public kimchi repository and Mina documentation). Lean 4 with Mathlib. Definitions of Pallas/Vesta base and scalar fields. |
| **Expected Outcome** | Lean 4 theorems of the form: for each gate type, the constraint polynomial evaluates to zero if and only if the witness columns encode a valid instance of the intended operation. |
| **Impact Analysis** | Provides mathematical proof that the most complex components of Kimchi's constraint system are correct. Eliminates an entire class of potential vulnerabilities (constraint under/over-specification). |

### Scenario 2: Plookup Argument Verification

| Field | Detail |
|---|---|
| **Description** | Formalize the Plookup-based lookup argument used in Kimchi for range checks and random-access array operations. Prove extraction soundness: if the verifier accepts, then the looked-up values are indeed present in the table. |
| **Requirements** | Plookup protocol specification. Polynomial commitment scheme interface (abstracted). Schwartz-Zippel lemma formalization (available in Mathlib or derivable). |
| **Expected Outcome** | Lean 4 proof that the Plookup argument is sound in the algebraic group model with negligible soundness error bounded by the field size. |
| **Impact Analysis** | Secures the lookup-based optimizations that Kimchi depends on for efficient range checks, Keccak, and other table-driven computations. |

### Scenario 3: Pickles Recursive Composition Soundness

| Field | Detail |
|---|---|
| **Description** | Formalize Pickles' recursive proof composition, including the dual-curve circuit structure and the deferred accumulator commitment verification. Prove that if the base Kimchi proofs are sound, then the recursively composed proof is sound, subject to an explicitly stated assumption about deferred verification. |
| **Requirements** | Pickles specification and OCaml source code (public). Formalized Pasta curve cycle. Inner product argument specification. |
| **Expected Outcome** | Lean 4 theorem: recursive composition preserves knowledge soundness under the discrete log assumption on the Pasta cycle, with the deferred verification assumption stated as an explicit axiom that can be discharged separately. |
| **Impact Analysis** | First formal model of recursive SNARK composition in any theorem prover. Provides machine-checked assurance for the mechanism that gives Mina its 22KB property. |

### Scenario 4: Fiat-Shamir Transcript Integrity

| Field | Detail |
|---|---|
| **Description** | Formalize the Fiat-Shamir heuristic as applied in Kimchi's interactive-to-non-interactive transformation. Verify that the transcript construction absorbs all prover messages before deriving each verifier challenge, preventing related-challenge attacks. |
| **Requirements** | Kimchi's sponge/transcript implementation (public OCaml/Rust source). Poseidon sponge specification. |
| **Expected Outcome** | Lean 4 proof that the transcript protocol is a valid instantiation of the Fiat-Shamir transform: every challenge depends on all prior prover messages. |
| **Impact Analysis** | Eliminates the risk of transcript manipulation, a vulnerability class that has affected other ZK implementations in practice. |

### Scenario 5: Pasta Curve Arithmetic Verification

| Field | Detail |
|---|---|
| **Description** | Formalize the Pallas and Vesta elliptic curves, their 2-cycle relationship, and the field arithmetic operations used throughout Kimchi and Pickles. Verify curve parameter correctness and group law completeness for the short Weierstrass form used. |
| **Requirements** | Pasta curve specifications (public). Mathlib elliptic curve definitions. |
| **Expected Outcome** | Lean 4 definitions of Pallas and Vesta with proofs that they form a 2-cycle (the base field of each is the scalar field of the other) and that the implemented group operations are complete (no exceptional cases for valid curve points). |
| **Impact Analysis** | Provides a verified algebraic foundation for all higher-level proofs. Any error in curve arithmetic would propagate through the entire proof system. |

---

## Open Issues and Discussion Points

1. **Scope prioritization.** A full formal verification of Kimchi and Pickles is a multi-year effort. This proposal targets 12 months and must prioritize. The proposed ordering is: Pasta curve formalization (foundation), then custom gate verification (highest-risk components), then Plookup, then Fiat-Shamir, then Pickles recursion (most complex). The Foundation may wish to adjust these priorities based on internal risk assessment.

2. **Abstraction boundaries.** Some components (e.g., the polynomial commitment scheme's binding property) will initially be axiomatized rather than fully verified. Each axiom will be explicitly documented with a justification for why it is a reasonable assumption and a plan for future discharge. The proposal takes a "correct by layers" approach: verify higher-level protocol logic first, then drill into lower-level primitives.

3. **Specification source.** Kimchi and Pickles lack a single canonical written specification. The formalization will be derived from the OCaml/Rust source code in the mina and proof-systems repositories, the Kimchi documentation site, and the Plonk/Plookup academic papers. Where ambiguities arise, the implementation will be treated as authoritative, with discrepancies documented.

4. **Lean 4 and Mathlib stability.** Lean 4 and Mathlib are under active development, which occasionally introduces breaking changes. The project will pin to specific Lean toolchain and Mathlib versions per milestone, upgrading between milestones. This is standard practice in the Lean formalization community.

5. **Relationship to existing audit work.** This effort complements but does not replace traditional code audits. Formal verification operates on a mathematical model of the protocol; it proves properties of the design. Code audits verify that the implementation matches the design. Ideally, future work would connect the Lean 4 model to the production code via extraction or differential testing.

6. **Community review process.** All Lean 4 code will be developed in a public repository from day one. Milestone deliverables will include written summaries accessible to readers without Lean expertise. The proposer welcomes the Foundation establishing a technical review committee for ongoing feedback.

---

## Conclusion

Kimchi and Pickles are the cryptographic foundation on which Mina's entire value proposition rests. The 22KB succinct blockchain, recursive proof composition, and efficient verification all depend on the correctness of these systems. Yet no formal verification of either system exists.

This proposal addresses that gap directly. By building machine-checked proofs in Lean 4, the project would provide the Mina ecosystem with the strongest available assurance that its core proof infrastructure is sound. The work would also represent a first in the broader ZK ecosystem: a formal model of recursive SNARK composition in a modern theorem prover.

The 12-month scope, $96,000 budget ($8,000/month), and milestone-based structure are designed to be appropriate for the Mina Core Grants program while delivering meaningful verified results at each stage. All deliverables will be open-source public goods.

---

## References

1. Mina Protocol. "Kimchi: The Latest Update to Mina's Proof System." Mina Protocol Blog. https://minaprotocol.com/blog/kimchi-the-latest-update-to-minas-proof-system

2. Gabizon, A., Williamson, Z., and Ciobotaru, O. "PLONK: Permutations over Lagrange-bases for Oecumenical Noninteractive arguments of Knowledge." IACR ePrint 2019/953.

3. Gabizon, A. and Williamson, Z. "plookup: A simplified polynomial protocol for lookup tables." IACR ePrint 2020/315.

4. Bowe, S., Grigg, J., and Hopwood, D. "Recursive Proof Composition without a Trusted Setup." IACR ePrint 2019/1021.

5. Pasta Curves. https://github.com/zcash/pasta_curves

6. O(1) Labs. "Pickles: Recursive SNARKs for Mina." Mina Protocol Documentation. https://docs.minaprotocol.com

7. O(1) Labs. Proof Systems Repository. https://github.com/o1-labs/proof-systems

8. The Lean 4 Theorem Prover. https://lean-lang.org

9. Mathlib4: The Math Library for Lean 4. https://github.com/leanprover-community/mathlib4

10. Fiat, A. and Shamir, A. "How to Prove Yourself: Practical Solutions to Identification and Signature Problems." CRYPTO 1986.

11. Bootle, J., Cerulli, A., Chaidos, P., Groth, J., and Petit, C. "Efficient Zero-Knowledge Arguments for Arithmetic Circuits in the Discrete Log Setting." EUROCRYPT 2016.

12. Ben-Sasson, E., Chiesa, A., Tromer, E., and Virza, M. "Succinct Non-Interactive Zero Knowledge for a von Neumann Architecture." USENIX Security 2014.

---

## Budget Summary

| Item | Detail |
|---|---|
| **Total Duration** | 12 months |
| **Monthly Rate** | $8,000 |
| **Total Budget** | $96,000 |
| **Payment Structure** | Milestone-based, paid upon deliverable acceptance |

### Milestone Breakdown

| Milestone | Timeline | Deliverable | Amount |
|---|---|---|---|
| M1: Foundations | Months 1-2 | Lean 4 project scaffold; Pasta curve formalization (Pallas/Vesta definitions, 2-cycle proof, group law completeness) | $16,000 |
| M2: Core Gates | Months 3-5 | Formal verification of foreign field multiplication, Poseidon, and generic arithmetic gates | $24,000 |
| M3: Lookups and Remaining Gates | Months 6-8 | Plookup argument soundness proof; verification of remaining custom gates (Keccak, XOR, EC operations) | $24,000 |
| M4: Protocol Layer | Months 9-10 | Fiat-Shamir transcript verification; polynomial commitment scheme interface formalization | $16,000 |
| M5: Recursive Composition | Months 11-12 | Pickles recursion model; recursive soundness theorem (with explicit deferred-verification axiom); final documentation and library packaging | $16,000 |
