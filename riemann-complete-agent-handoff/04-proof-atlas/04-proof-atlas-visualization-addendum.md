# Proof Atlas — Optional Visualization Addendum

**Prepared:** November 21, 2026.  
**Applies to:** `04-proof-atlas-handoff.md`.  
**Status:** Optional implementation specification; no application changes or video renders are requested by this document alone.

## Assignment and priority

Extend a completed Proof Atlas locally with a guided educational tour of its small licensed Lean fixture. Preserve the base application’s extraction, verification, identity, licensing, and isolation contracts. First deliver **P1: an interactive tour using the existing graph renderer**. Subsequently, if selected, deliver **P2: a 45–60 second Manim Community Edition video generated from the same validated tour snapshot**. Keep both optional; normal installation, tests, proof checking, and API use must work without Manim.

This adds explanation of what graph relationships mean. The base application already draws dependencies. The educational question is: **“What must I inspect before trusting an explanation of this theorem?”** A learner should distinguish a symbol needed to state a claim, a lemma referenced by its proof, an unused module import, and an explanation that became stale after a change.

Use original visual composition and assets. Grant Sanderson recommends Community Edition for newcomers; his own `3b1b/manim` development line is separate. Install the `manim` Community Edition package and follow its documentation throughout; do not mix ManimCE instructions with ManimGL APIs. [3Blue1Brown’s explanation](https://www.3blue1brown.com/about/) and [Manim Community repository](https://github.com/ManimCommunity/manim).

## 1. P1 — A guided tour with evidence

Use the base project’s real extracted arithmetic fixture. The author chooses its Lean identifiers, then binds the following **pedagogical roles** to actual dataset IDs. These role names are not assertions that particular Lean declarations already exist.

| Step | Fixture role and evidence | Action and educational point |
|---|---|---|
| 1. Read the claim | `target`; exact statement and source hash | Present the claim before its graph. Explain what is being asserted, using a reviewed gloss or a visible draft label. |
| 2. Decode the statement | `definition`; extracted statement edge | Highlight the relevant expression and definition. Explain why a declaration can be needed to express the theorem. |
| 3. Inspect proof references | `support_a`, `support_b`; extracted proof edges | Highlight both referenced prerequisites. Keep their statements available together. Show how the authored explanation relates them to the proof. |
| 4. Separate imports | `unused_module`; actual module-header edge | Move to the module layer. Explain that importing a module does not demonstrate use of each theorem it contains. |
| 5. Inspect the boundary | Actual external reference or unresolved stub | Show extraction coverage and what remains outside this slice. Keep a context-only endpoint visibly distinct from a newly discovered result. |
| 6. Compare a revision | Same logical key in two licensed snapshots | Change the proof while preserving the theorem statement, where the fixture author can do so correctly. Show unchanged statement, changed proof hash, and the resulting stale explanation review binding. |

Choose a fixture whose actual extracted edges support this sequence. If the baseline fixture lacks a needed relationship, add a separate small licensed teaching fixture and extract it; preserve the baseline proof and ordinary verification workflow. Never draw the desired edge over unsupported data. If a complete external declaration is available, do not fabricate an unresolved stub merely to fit the lesson. Demonstrate unresolved coverage separately using an explicitly labelled partial extraction.

The initial tour contains at most six steps, eight focused entities per step, and 40 distinct entities overall. Each step offers Back, Next, Read source, and an equivalent dependency table. Learners advance manually; there is no autoplay. “Why this highlight?” opens the edge kind, evidence record, and pinned source anchor.

## 2. Preserve mathematical meaning

Keep the base direction convention: **A → B means A references B**. A pedagogical sequence describes the order in which a reader learns; it does not reverse formal edges or create an inference chain. Store tour order separately from graph edges.

Dependencies of a represented inference are joint requirements, not alternative routes. Never animate a token choosing one branch and arriving at “proved.” Highlight all prerequisite references relevant to the current explanation together, and leave a persistent “selected slice” indicator when coverage is partial. Ordinary dependency extraction alone does not identify a complete logical inference rule; do not label a collection of references as an AND-inference unless the underlying proof evidence supports that interpretation.

Statement references, proof references, and imports retain different text labels and line patterns. A reused node appearing as context on another page is the same declaration, not a second proof. Render missing records as explicit stubs without generated statements. Preserve separate states for unknown axioms, extracted axioms, local checks, external verification reports, and reviewed prose.

The RH profile stays `metadata_only`. It may explain that no local graph is available and link to verification evidence, but it cannot launch a tour depicting an imagined RH argument. The checked scope remains RH and Dirichlet GRH, as described by the [MIT report](https://math.mit.edu/~drew/grh/); reuse restrictions remain those of the [community verification repository](https://github.com/leanprover-community/riemann-hypothesis).

## 3. Tour contract and snapshot binding

Add a versioned JSON schema, validated with the base project’s strict models. Required top-level fields are `schema`, `tour_id`, `title`, `dataset_ids`, `snapshot_sha256`, `layout_sha256`, `review_status`, `review_record_id`, and `steps`. Limit `dataset_ids` to one primary snapshot plus one comparison snapshot.

Every step requires the following shape. Angle-bracket strings are documentation placeholders; the builder must replace them with real IDs and hashes before validation succeeds.

```json
{
  "id": "inspect-proof-references",
  "kind": "proof_references",
  "dataset_id": "<actual immutable dataset ID>",
  "focus_node_ids": ["<target ID>", "<support A ID>", "<support B ID>"],
  "highlight_edge_ids": ["<proof edge A ID>", "<proof edge B ID>"],
  "evidence_anchors": [{
    "declaration_id": "<target ID>",
    "statement_sha256": "<actual hash>",
    "proof_sha256": "<actual hash>",
    "source_file_sha256": "<actual hash>",
    "source_span": {"line_start": 12, "line_end": 17}
  }],
  "explanation_id": "<actual explanation ID>",
  "caption": "Inspect both references used by this proof.",
  "camera": {"mode": "fit_ids", "duration_ms": 180},
  "coverage_label": "selected_slice"
}
```

Allowed `kind` values are `claim`, `statement_reference`, `proof_references`, `module_import`, `coverage`, and `revision_compare`. Comparison steps additionally identify the second dataset, both declaration IDs, the shared logical key, and the base application’s comparison/review-reuse record. No fuzzy matching is permitted.

The snapshot contains only permitted source material, required entities/edges, explanations, review records, and coverage metadata. Calculate its digest using the base canonical JSON convention over an explicit payload that excludes the digest itself and render observations. Reject missing anchors, mismatched hashes, wrong edge kinds, unlicensed material, or unsupported placeholders.

Use the UI state enum `loading`, `ready`, `active`, `completed`, `blocked_missing`, `blocked_stale`. Draft prose can be explored with its status visible; changed evidence blocks the current binding until rebuilt. A historic stale review intentionally depicted in the comparison lesson is evidence, not automatically a stale tour: the tour's own bindings must still match their pinned snapshots. Formal proof checks never automatically approve the tour. Reuse reviews only under the base handoff’s unchanged-content rules.

## 4. Stable interaction and accessibility

Keep positions fixed throughout a tour, using a stored layout keyed to the snapshot. Camera focus fits the selected IDs into a bounded viewport without changing edge direction, node identity, or graph coordinates. Do not run a force simulation during playback or move nodes to imply that the proof is being executed.

Animate only short focus transitions, at most 250 milliseconds. Under `prefers-reduced-motion`, apply the next state immediately. Preserve manual pan/zoom until the learner explicitly selects “Focus step.” Returning to a step restores the same evidence, highlights, and deterministic focus box.

The text/table view must convey every teaching point without seeing motion or color. Supply visible focus, keyboard controls, meaningful headings, screen-reader step announcements, and patterns alongside colors. Do not move keyboard focus on every graph highlight. Keep full statements selectable, and provide readable wrapped text on narrow screens instead of shrinking a large graph to illegibility.

## 5. P2 — A 54-second Manim storyboard

Render the same approved snapshot and explanation records used by P1. If no genuine human review exists, a local preview must remain labelled draft and cannot claim approval. Use short original captions; narration is optional and no voice service is required.

| Time | Visual and caption | Evidence requirement |
|---|---|---|
| 0–7 s | Show the small fixture’s statement: “What supports this claim?” | Exact target statement, snapshot identity, fixture label. |
| 7–15 s | Highlight one symbol, then its definition: “Needed to state it.” | Actual statement-reference edge and source anchor. |
| 15–26 s | Reveal both proof references together: “Used in this proof.” | Both extracted proof edges; no animated path implying sufficiency. |
| 26–35 s | Show the separate import layer: “Imported does not mean used here.” | Module-header evidence and the selected theorem’s extracted reference set. |
| 35–43 s | Hold a partial-boundary view: “This view has a boundary.” | Real coverage metadata; stubs only where actually unresolved. |
| 43–54 s | Compare the two fixture versions: same statement, changed proof, stale review. | Logical-key mapping and content hashes; finish with a link back to the interactive tour. |

Use one camera frame per teaching point with restrained transitions. Retain a visible legend and adequate dwell time; do not rapidly morph mathematical expressions in ways that suggest unproved equality. A diagram can simplify layout while preserving every relationship it claims to show.

## 6. Architecture, media outputs, and limits

Reuse the browser graph renderer. Add a small tour controller and static validated manifests; do not replace interactive graph exploration with Manim. Proposed paths are `tours/`, `web/src/tours/`, `media/proof_tour_scene.py`, and `tests/test_tour_contract.py`.

ManimCE is an optional development dependency used by an offline CLI. The web server never executes scene Python, Lean, arbitrary TeX, or rendering commands. Scene code is authored and versioned; captions are treated as text. Only explicitly authored, reviewed mathematical expressions may use a TeX renderer. Render in the base worker isolation model, with pinned fonts and dependencies.

Produce MP4, a poster PNG, step-end PNGs, a plain-text transcript, WebVTT captions, and a media manifest. Record tour/snapshot/layout hashes, scene revision, ManimCE version, dimensions, frame rate, duration, asset/font licenses, review status, and output hashes. Use original assets; an MIT animation library does not license third-party music, footage, logos, or fonts. ManimCE’s repository provides its license information and rendering commands. [Repository](https://github.com/ManimCommunity/manim).

Start with a low-quality preview; inspect step-end frames before a production render. Suggested final output is 1920×1080 at 30 fps. Begin with one render worker, two CPU cores, 4 GiB memory, 1 GiB output allowance, and a 20-minute wall timeout; measure and document actual needs. Do not silently increase budgets. Cache by snapshot, layout, scene, toolchain, font, and render-setting hashes. No rendering occurs during ordinary page requests or default test runs.

## 7. Acceptance and bounded completion

P1 passes when every step resolves to the real fixture, explanations retain their status, and the six teaching points work through keyboard and table navigation. Test wrong edge kinds, missing IDs, stale hashes, partial closure, repeated context nodes, version mismatch, and reduced motion. A metadata-only RH record must never acquire a synthetic proof tour.

P2 additionally requires identical evidence binding to P1, accurate captions, complete licensing metadata, and visual inspection of every step-end PNG plus the production video. Check clipping, overlapping labels, contrast, readable statements, legend persistence, duration, and caption synchronization. A mathematical reviewer must confirm that the sequence does not turn a dependency path into a claimed proof.

Deliver the optional source, schemas, fixtures, measurements, and review results locally. Stop after one useful tour and its optional video. Do not add a general video editor, automated RH explanation generator, public publishing flow, or per-user rendering service.
