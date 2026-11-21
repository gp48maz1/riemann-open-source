# Complete coding-agent handoff package

Prepared November 21, 2026

This ZIP contains the five original project specifications and their matching optional visualization addenda, organized so that each project folder can be handed to an independent coding agent.

**These are implementation specifications, not implemented applications.** The acceptance tests, benchmarks, visualizations, and video renders described in them are work for the implementing agent. Preparing this package does not establish that any application or animation has been built or validated.

## Assign one project folder per agent

Each folder contains exactly two documents. Read its original numbered handoff first, then its matching visualization addendum. The projects can be implemented independently in separate repositories; one project does not need the others to ship.

| Project folder | Read first: original handoff | Then: optional visualization addendum |
| --- | --- | --- |
| `01-prime-bounds/` — Prime Bounds | [01-prime-bounds-handoff.md](01-prime-bounds/01-prime-bounds-handoff.md) | [01-prime-bounds-visualization-addendum.md](01-prime-bounds/01-prime-bounds-visualization-addendum.md) |
| `02-fieldforge/` — FieldForge | [02-fieldforge-handoff.md](02-fieldforge/02-fieldforge-handoff.md) | [02-fieldforge-visualization-addendum.md](02-fieldforge/02-fieldforge-visualization-addendum.md) |
| `03-theorem-bridge/` — Theorem Bridge | [03-theorem-bridge-handoff.md](03-theorem-bridge/03-theorem-bridge-handoff.md) | [03-theorem-bridge-visualization-addendum.md](03-theorem-bridge/03-theorem-bridge-visualization-addendum.md) |
| `04-proof-atlas/` — Proof Atlas | [04-proof-atlas-handoff.md](04-proof-atlas/04-proof-atlas-handoff.md) | [04-proof-atlas-visualization-addendum.md](04-proof-atlas/04-proof-atlas-visualization-addendum.md) |
| `05-primality-bench/` — Primality Bench | [05-primality-bench-handoff.md](05-primality-bench/05-primality-bench-handoff.md) | [05-primality-bench-visualization-addendum.md](05-primality-bench/05-primality-bench-visualization-addendum.md) |

## Scope and implementation order

1. **Implement the base project first.** Its mathematical, input/output, evidence, resource, licensing, and acceptance contracts remain authoritative.
2. **P1 is optional interactive explanation.** Select it when assigning the work. It uses the application's actual results and evidence, with accessible alternatives and the addendum's validation checks.
3. **P2 is optional rendered media.** Select it separately. It uses Manim Community Edition to produce the specified prerecorded explanation from validated data. Ordinary application use must not require Manim or a live renderer.
4. **Preserve the base guarantees.** An optional visual layer may explain a result; it cannot replace verification, weaken a domain condition, fabricate evidence, or silently change the project's core scope.

An agent assigned only a base project should not treat either visualization tier as a required release gate. An agent assigned a tier should complete that tier's additional acceptance checks and report what actually ran. Routine implementation decisions do not require repeated confirmation.

## Suggested assignment

```text
Implement the original handoff in the attached project folder in the repository
I provide, or create a local repository if none exists. Read repository
instructions first. Build and validate the base MVP before adding selected
visualization work. Use the matching addendum for any selected P1 or P2 tier;
those tiers are optional unless included in this assignment. Preserve the base
mathematical, evidence, status, resource, and acceptance contracts. Complete the
specified meaningful checks and report exact run commands, actual test results,
measured limitations, and remaining issues. Do not claim work or verification
that you did not perform. Do not publish or deploy without separate authorization.
```

Add the repository path and explicitly select **base only**, **base plus P1**, or **base plus P1 and P2** when sending the assignment. The full original and matching addendum remain together so the agent has the complete context for that project.

## Package-wide guides

- [PROJECT-OVERVIEW.md](PROJECT-OVERVIEW.md) preserves the original overview, project comparisons, assignment guidance, and source snapshot.
- [VISUALIZATION-GUIDE.md](VISUALIZATION-GUIDE.md) preserves the visualization overview, tier guidance, and shared rendering pattern.

The ten project documents are unchanged copies of their source files. The two package-wide guides retain their original wording, with relative links updated for this folder layout. This START-HERE document is the navigation and assignment entry point for the combined package.
