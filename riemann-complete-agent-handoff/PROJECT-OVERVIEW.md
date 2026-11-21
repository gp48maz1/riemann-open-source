# Riemann project coding agent handoffs

Prepared 21 November 2026

This package contains five independent implementation specifications for open source projects that use the Riemann Hypothesis, the checked Dirichlet L-function generalization, or the formal proof as a software use case. Each handoff includes the background needed by a coding agent without access to the original conversation.

These are implementation briefs. The applications have not been built by preparing these documents. Acceptance tests, benchmark runs, and release gates described inside are work for the implementing agents. Small mathematical fixtures and the specifications' contracts were reviewed during preparation; that review is not validation of future application code.

## The five handoffs

| File | Application | First useful release |
| --- | --- | --- |
| [01-prime-bounds-handoff.md](01-prime-bounds/01-prime-bounds-handoff.md) | Rigorous bounds on prime counts | Python package, CLI, and local UI with exact small counts, analytic enclosures, interval queries, and mathematical provenance |
| [02-fieldforge-handoff.md](02-fieldforge/02-fieldforge-handoff.md) | Reproducible finite-field parameters | Bounded 64-bit generation, an independent verifier, and portable constant exports |
| [03-theorem-bridge-handoff.md](03-theorem-bridge/03-theorem-bridge-handoff.md) | A map from theorems to usable algorithms | Curated catalog, deterministic scope checking, searchable site, and two runnable examples |
| [04-proof-atlas-handoff.md](04-proof-atlas/04-proof-atlas-handoff.md) | Exploration of formal proof structure | Genuine small Lean fixture, bounded graph browser, provenance, and explanation review records |
| [05-primality-bench-handoff.md](05-primality-bench/05-primality-bench-handoff.md) | Honest evaluation of primality methods | Reference deterministic algorithm, independent checks, budgeted comparison runner, and offline reports |

## How to assign the work

Give one coding agent the complete file for the project it will implement. Each document opens with an assignment and supplies the mathematical contract, architecture, interfaces, milestones, validation plan, source references, and definition of done. The five applications can be built in separate repositories and do not require one another to ship.

Suggested launch instruction:

```text
Implement the attached handoff in the repository I provide, or create a local
repository if none exists. Read the repository's instructions first. Follow the
MVP and milestones, make ordinary implementation choices autonomously, and keep
the mathematical and output guarantees intact. Produce working software and
run the specified meaningful validation. Finish with exact local run commands,
test outcomes, measured limitations, and any unresolved issue. Do not claim
features or verification that you did not actually implement or run.
```

Add the target repository path and any actual publication/deployment authorization separately. No provider account, API key, or paid model is required by the default scope of these briefs. If the surrounding repository already fixes a language, toolchain, or frontend framework, adapt routine choices while preserving the mathematical contract and record the decision.

## Recommended order

Start with **Prime Bounds** for the smallest direct application of RH. It establishes a useful pattern: return a result, a precise guarantee, and evidence supporting the guarantee. **FieldForge** can proceed independently and gives developers concrete reusable parameters.

**Theorem Bridge** is especially suitable for an agent focused on structured data and developer interfaces. Its first runnable examples are specified locally, so it does not depend on the other two projects. Later, it can link their released versions as implementations of cataloged consequences.

**Proof Atlas** is a separate formal-methods application. Its first release uses an original licensed fixture and an external RH metadata profile. Full processing of the enormous proof is a later resource-intensive operation, not a prerequisite for the fixture-based application.

**Primality Bench** is a separate correctness and measurement project. It may later provide reproducible performance evidence for catalog entries, but it must not be turned into a marketing exercise that assumes a speed improvement.

## Decisions shared across the specifications

- Mathematical facts, numerical arithmetic, code validation, formal verification, and measured performance have separate evidence fields.
- Large mathematical integers cross JSON and browser boundaries as exact decimal strings.
- Runtime queries do not need language-model inference or a live source lookup.
- Interrupted computation retains an explicit incomplete outcome.
- Actual installed versions and source revisions are captured during implementation; these briefs do not invent them.
- Public proof source is not automatically treated as permissively licensed code.
- Each application can be useful without copying or recompiling the entire RH proof.

These are design requirements intended to make the projects dependable. The handoffs explain their project-specific implementation consequences.

## The source snapshot

The mathematical baseline is the evidence reviewed on 21 November 2026. The [Lean and Mathlib community statement](https://leanprover-community.github.io/blog/posts/riemann-hypothesis/) endorses the proof, while [MIT's verification page](https://math.mit.edu/~drew/grh/) specifies ordinary RH and the Dirichlet L-function statements that were checked. General number-field and other L-function hypotheses must be examined separately.

The [community verification repository](https://github.com/leanprover-community/riemann-hypothesis) describes the original submission as published without a license. The code implementing these applications can be independently authored and licensed; the briefs keep external proof source separate. Each handoff contains the relevant technical sources and tells the implementing agent to record what it actually inspected.

## Handoff review completed during preparation

The preparation review checked mathematical domain conditions, theorem-family separation, exact integer serialization, resource-limit outcomes, fixture consistency, and independence of verification paths. It also resolved several implementation traps:

- Strict inequalities and outward numerical bounds must be converted into inclusive integer answers correctly.
- A first-prime theorem does not establish a prime in every arbitrary bit window.
- A valid finite-field tuple does not prove its producer's claimed search history.
- A supported route to a consequence must not be overridden by an unsupported alternative route.
- A full-base primality run has a different dependency profile from a shorter theorem-based run.
- Proof graph data, synthetic fixtures, and informal explanations must keep distinct provenance.

The next validation belongs in the implementation repositories, against real dependency versions and actual executable code.
