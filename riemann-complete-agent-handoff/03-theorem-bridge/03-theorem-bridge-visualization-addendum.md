# Theorem Bridge — optional visualization addendum

Version 1.0 · Prepared 21 November 2026 · Companion to `03-theorem-bridge-handoff.md`

## 1. Coding agent assignment

When this addendum is selected, add a focused visual explanation of mathematical applicability to Theorem Bridge. The main question is: **“Why does this result apply to my inputs, and which evidence supports that conclusion?”** Build on the original handoff's deterministic engine, source ledger, typed hypothesis families, route semantics, and static website. Complete the core acceptance criteria before these extensions.

**P1**, a small interactive explanation panel, and **P2**, a 56-second Manim Community Edition introduction, are optional and separately selectable. Implement only the selected tier or tiers. If both are selected, complete P1 first; a P2-only delivery may consume validated fixture snapshots without implementing the interactive panel. Both use the same semantic export contract. The original product remains runnable without either extension, the animation package, the video renderer, or any AI service. This document specifies proposed work; it does not claim that the application, tour, or video already exists.

The visual contribution is making applicability understandable. It must not promote source-backed mathematics to locally verified formal mathematics, confer correctness on an implementation, or imply a performance improvement that has not been measured.

## 2. P1: an applicability route explorer

Use one consequence page with a compact route diagram and a synchronized explanation list. Draw requirements flowing into an implication and then into the result. Label this convention “requirements → consequence”; it is a presentation convention, not a universal convention for dependency graphs. Keep the result and its plain-language reason visible above the diagram.

Represent requirements within a route as **all required** (AND) and alternative routes as **any supported route suffices** (OR). Label these relationships explicitly. Clicking a requirement opens its exact statement, domain predicate, evidence links, and review state. A line indicates a catalogued mathematical relationship; an animation connecting two boxes creates no new relationship.

Start with three preset demonstrations:

| Preset | What the viewer changes or inspects | Teaching result |
| --- | --- | --- |
| Prime-count bound at `x = 1000000` | Compare the finite-range route and the global RH route | Both can support the same bound under the original handoff's evidence baseline; the finite route does not require full RH |
| Prime-count bound at `x = 10^100` | Change the exact input and inspect each route's domain check | The finite-range route is excluded by its upper limit; a supported global route can still establish applicability |
| A consequence requiring general Hecke L-functions | Inspect the required family beside the recorded Dirichlet family | A shared acronym is insufficient; preserve `needs_review` or the applicable unresolved state returned by the engine |

Construct the large-input fixture with Python `str(10**100)`, then serialize the complete integer as a decimal string. Scientific notation is acceptable for a display label with an exact-value disclosure. Never submit `1e100` through a binary floating-point field to the predicate engine.

Add a fourth, clearly marked **hypothetical evidence scenario** only if it is easy to support using the existing engine: treat the global RH requirement as an explicit assumption rather than supplied evidence. To demonstrate a conditional route, the isolated profile must leave no supported derivation of that RH requirement, including any catalogued implication from Dirichlet GRH, while retaining the evidence for the mathematical implication itself. Re-evaluate the complete route collection. The finite-range result at one million remains supported. At `10^100`, the global route is conditional only if its other requirements are satisfied and RH is its sole assumed requirement. Any remaining supported alternative still makes the consequence supported; otherwise preserve the engine's status precedence. This control changes an isolated scenario input; it does not edit the catalog, declare a historical event, or assert that the theorem is false. Keep “Hypothetical scenario” visible in screenshots and exports, and provide a reset to the recorded evidence profile.

Missing evidence is not automatically an assumed hypothesis. If the scenario removes evidence without explicitly supplying an assumption, retain the engine's `needs_review` or other correct unresolved result. Do not manufacture a simpler status solely to make a transition attractive.

## 3. Interaction and evidence presentation

Provide preset selection, an exact input field where supported, Previous/Next step controls, and an optional Play/Pause tour. Default to a static complete result. Preserve the selected route when possible across input changes; show the old and new predicate values when a route changes status. Use a short opacity or position transition without tweening truth states.

Show four independent evidence rows beside or below the diagram:

1. **Mathematical applicability:** the actual engine result and route explanation.
2. **Implementation checks:** recorded tests or verifier evidence for the selected implementation.
3. **Formalization:** the exact local or external formal status and scope.
4. **Measured performance:** an associated benchmark, or “No measurement recorded.”

These rows are ordinary evidence details, not a second elaborate diagram. A supported mathematical route must not turn the other rows into success indicators. “Source reviewed,” “externally reported,” “locally checked,” and “unknown” retain different meanings.

Use labels and shapes in addition to color. For example, a check and the word “supported” can accompany a solid outline; “out of domain” can use a crossed predicate; “needs review” can use a labelled outline. Do not rely on green versus red. All nodes and source disclosures must be keyboard accessible. Provide a linear route list with identical semantics, visible focus, and a text export.

At narrow widths, stack routes vertically and let details appear beneath the selected item. Respect reduced-motion preferences and turn the tour into instantaneous discrete steps. Avoid autoplay, pulsing backgrounds, animated particle flows, or essential information conveyed only through motion.

## 4. Semantic snapshot and renderer boundary

The Python applicability engine remains the sole evaluator. Export a versioned, immutable explanation snapshot containing inputs, catalog/evidence revisions, complete route decisions, predicate results, and source identifiers. The browser and Manim scene consume this snapshot. They may lay it out differently but must show identical decisions for the same snapshot.

An illustrative adapter object follows. The ids and filenames are proposed interfaces; use real generated revisions and hashes in actual artifacts.

```json
{
  "schema_version": "1.0",
  "scenario_id": "prime-count-million-recorded-evidence",
  "scenario_kind": "recorded_evidence",
  "consequence_id": "prime_count.schoenfeld",
  "inputs": {"x": "1000000"},
  "applicability": "supported",
  "routes": [
    {
      "id": "finite-range",
      "status": "supported",
      "requirements_mode": "all",
      "predicate_results": [
        {"kind": "integer_gte", "expected": "2657", "passed": true},
        {
          "kind": "integer_lte",
          "expected": "110100000000000000000000000",
          "passed": true
        }
      ],
      "evidence_ids": ["johnston-finite-range"]
    }
  ],
  "alternative_routes_mode": "any",
  "illustrative_excerpt": true
}
```

This example intentionally shows one route. Production snapshots must include all evaluated alternatives and must not use `illustrative_excerpt` to conceal missing evidence. Add the actual catalog digest, evidence profile revision, source-snapshot date, implementation evidence references, and canonical snapshot digest. Unknown metadata is `null`; never invent a commit hash. Keep counters as small machine integers and scientific integers as canonical strings.

Store pedagogical instructions separately: ordered step ids, referenced semantic ids, camera framing, caption text, and optional source anchors. A step may highlight a known predicate or route; it cannot set its status. Schema validation must reject references to absent ids. For a live evaluator, generate a new snapshot when inputs change; for a static build, ship the bounded preset snapshots and clearly indicate that only those inputs are interactive.

Do not duplicate the predicate engine in JavaScript. Do not send user-authored code or arbitrary LaTeX to a renderer. Use curated captions and mathematical labels from the reviewed catalog.

## 5. P2: Manim storyboard

Use Manim Community Edition (`manim`, imports from `manim`) for an optional original scene named `WhyThisResultApplies`. Grant Sanderson's original repository uses a separate implementation commonly called ManimGL; its installation and import conventions must not be mixed with the Community Edition [V1–V3].

| Time | Visual action | Caption or intended explanation |
| --- | --- | --- |
| 0–7 s | Present the prime-count bound and input one million | “A guarantee has both a statement and conditions.” |
| 7–19 s | Reveal the finite-range and global routes; highlight their domain predicates | “Different established routes can support the same result.” |
| 19–31 s | Replace the input with `10^100`; mark the finite route out of domain; retain the supported global route | “A failed alternative does not invalidate a route that still applies.” |
| 31–43 s | Switch to the scope-review example; highlight Dirichlet and general Hecke family labels | “The exact family of functions matters.” |
| 43–56 s | Reveal the four evidence rows; leave unrecorded software or performance evidence unfilled | “Mathematical support, implementation checks, formalization, and performance are separate evidence.” |

All values and states come from fixture exports. If a source profile changes, regenerate or retire the affected scene. The film is an explanation of a dated snapshot, not a proof replay. A hypothetical scenario, if used in a later scene, must be identified throughout; keep it out of the initial 56-second scene to reduce cognitive load.

Deliver an MP4, a poster PNG, caption file, plain-text transcript, and a manifest linking them to the scene source and input snapshot. The transcript must communicate the relationships without requiring the viewer to distinguish colors or read tiny formulae.

## 6. Engineering plan and practical limits

Reuse the existing website. A small SVG diagram with HTML controls and accessible details is sufficient. Do not introduce a graph database or a second graph visualization stack for this panel. Start with at most 24 visible nodes and 40 edges; collapse additional requirements behind labelled counts and an explicit expanded list. A partial display must say it is partial and still expose the complete engine trace for download.

Suggested additions:

| Path | Responsibility |
| --- | --- |
| `src/theorem_bridge/explanation_export.py` | Validate and serialize engine decisions for presentation |
| `web/applicability-view.*` | Interactive view using the existing frontend conventions |
| `examples/visuals/*.json` | Reproducibly generated scenario snapshots |
| `animations/why_this_result_applies.py` | Optional Manim scene |
| `animations/storyboards/` | Reviewed captions and step references |
| `docs/visualizations.md` | Commands, accessibility, source provenance, limitations |

Resolve and pin compatible versions during implementation. Keep Manim and its rendering dependencies in an optional development environment. Build curated clips offline; serve them as ordinary cached assets. No render starts on a page view or every input change. The first render profile can use 720p at 30 fps; final release media can use 1080p at 30 fps. Set a configurable per-scene budget, initially ten minutes, two CPU cores, and 4 GiB RAM. A failed optional render must leave the core CLI and static explanation usable.

Cache by semantic snapshot digest, storyboard revision, scene code revision, and rendering configuration. Record the observed Manim version and relevant font dependencies in the manifest. A visual-only edit need not rerun mathematical examples if their canonical snapshot is unchanged; a mathematical change must regenerate the snapshot and affected assets.

Use original scene code and original prose. Preserve dependency license notices. No copied narration, music, mascot, or branding is needed to obtain the explanatory benefits of Manim.

## 7. Acceptance criteria and handoff report

Validate the meaningful failure modes for the selected tier or tiers. Shared mathematical and data invariants apply to every delivered output; criteria referring to an unselected panel, tour, or video do not require that output to be built.

1. The finite-range and global routes at one million display the same statuses returned by the engine. At `10^100`, the finite route is excluded without overriding a supported global route.
2. Exact inputs survive JSON export, display, and copy without digit changes. The finite endpoint is checked as an integer, including equality and its immediate successor.
3. An unsupported general-Hecke requirement cannot inherit support from the Dirichlet profile. A hypothetical assumption cannot mutate the recorded profile.
4. A supported mathematical result does not fabricate test, formalization, or benchmark evidence. Every evidence link resolves to a real catalog object or clearly reports unavailability.
5. The diagram, accessible list, exported snapshot, transcript, and rendered scene agree on result and route states. An edited storyboard cannot override a decision.
6. Keyboard navigation, narrow layout, reduced motion, and captions preserve the complete explanation. Long labels and missing-source states remain readable.
7. Inspect at least one static frame for each major scene state. Check clipping, font size, edge direction, exact family labels, and that excluded routes never flash a success state during transitions.

The implementation report should list the completed tier, run commands, selected dependencies, fixtures, checked source revisions, validation results, media locations, and any remaining limitations. Do not claim P2 completion from a storyboard alone.

## 8. Sources

The original handoff's primary source ledger governs the mathematics and evidence baseline. Reuse those identifiers and revisions rather than creating a parallel authority list.

- **V1:** [3Blue1Brown FAQ/about](https://www.3blue1brown.com/about/): Manim's purpose and the recommendation that new users consider the Community Edition.
- **V2:** [Manim Community repository](https://github.com/ManimCommunity/manim) and [documentation](https://docs.manim.community/en/stable/): supported scene, installation, output, and configuration APIs.
- **V3:** [Grant Sanderson's Manim repository](https://github.com/3b1b/manim): original project and distinct ManimGL conventions.
- **V4:** [Johnston, improving bounds on prime counting functions](https://link.springer.com/article/10.1007/s11139-022-00616-x): the finite-range route used in the original brief.
- **V5:** [Andrew Sutherland's verification report](https://math.mit.edu/~drew/grh/): the exact externally reported scope recorded in the original brief; recheck the source when implementing.

Tooling sources checked 21 November 2026. UI limits, timings, architecture, and pedagogical choices above are proposed design decisions.
