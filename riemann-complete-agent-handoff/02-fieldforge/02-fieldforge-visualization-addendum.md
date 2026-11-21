# FieldForge — optional visualization addendum

**Prepared:** November 21, 2026. **Companion:** `02-fieldforge-handoff.md`. **Status:** design specification only; no application or video is created by this document.

## 1. Optional agent assignment

After the FieldForge core passes its existing acceptance criteria, add an interactive explanation of its parameter construction. Preserve the base handoff's mathematical contracts, status semantics, resource limits, and independent verifier. Implement priority P1 first: a candidate grid linked to an exact modular-power diagram. Priority P2 is an optional, locally rendered 45–60 second explanation using Manim Community Edition. Either priority may ship independently; neither blocks the core release.

Use visual explanation inspired by the clarity of 3Blue1Brown: concrete examples, purposeful movement, and equations connected to the objects they describe. Give FieldForge its own typography, colors, illustrations, and voice. Do not imply affiliation or reuse another channel's branding, characters, music, or footage.

This layer explains existing arithmetic. It does not add a transform engine, change the parameter-search algorithm, or turn animation into verification.

## 2. Learning questions and scope

The experience should answer four questions:

1. Why search primes of the form `m*N + 1`?
2. How does a quadratic nonresidue produce a root with the required order?
3. Why is returning to one after eight steps insufficient unless the halfway check also passes?
4. What did the independent verifier establish, and what does the mathematical search bound establish separately?

Use the exact example `p=17`, `N=8`, `z=3`, `omega=9`. Explain that `N` divides `p-1`, permitting the required multiplicative subgroup. The published Dirichlet-GRH consequences concern search bounds; the core's finite-domain primality and modular identity checks verify this output independently of that newly established hypothesis. Keep general Dedekind/Hecke statements outside the claim [S4, S5].

Successful understanding means a reader can reject an order-four substitute, identify an untested candidate, and distinguish a recorded demonstration from a live computation.

## 3. Exact fixtures and independent derivation

Use canonical decimal strings in presentation data. Recompute every displayed identity during the fixture build using Python integer arithmetic, and check the resulting artifact through the standalone verifier. Do not transcribe numbers from a rendered frame into the data source.

For request `k=3`, range `[3,100]`, the first twelve eligible candidates are:

```text
9, 17, 25, 33, 41, 49, 57, 65, 73, 81, 89, 97
```

A normal ascending run rejects 9, selects 17, and stops searching. The remaining cards are **untested in this run**, regardless of whether their mathematical status is familiar. An educational replay must preserve that distinction.

The nonresidue search gives `2**8 mod 17 = 1` and `3**8 mod 17 = 16`. Thus `z=3` passes Euler's criterion, and `omega = 3**((17-1)/8) mod 17 = 9`.

| Exponent j | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| `9**j mod 17` | 1 | 9 | 13 | 15 | 16 | 8 | 4 | 2 | 1 |
| `4**j mod 17` | 1 | 4 | 16 | 13 | 1 | 4 | 16 | 13 | 1 |

The valid root has `9**4 mod 17 = 16 = -1 mod 17` and `9**8 mod 17 = 1`. The substitute `4` passes the eighth-power check but fails the half-order check: `4**4 mod 17 = 1`. It returns too early. This counterexample is a deliberately invalid proposal, never a mutation of the accepted artifact.

Retain `omega_inverse=2`, `N_inverse=15`, and construction `m=2`. Bound metadata remains exactly as specified in the base: first-prime method `lls_integer_envelope_v1`, value `144`; nonresidue method `bit_length_squared_v1`, value `25`. The bound 144 does not promise a prime inside every user-selected subinterval.

## 4. P1: candidate grid and modular cycle

Use three coordinated regions: the request and candidate grid; the selected construction and modular diagram; the verification summary. Desktop can place the last two side by side. Mobile stacks them in that reading order.

Candidate cards show `m`, the resulting integer, and an explicit state: untested, current, composite, selected, or checking. A selected candidate is not yet a fully verified parameter set. Only the completed core verifier can supply that final status. Tooltips and a persistent text legend explain every state; color is supplementary.

For the default fixture, arrange the sixteen nonzero residues modulo 17 around a circle. Highlight the current residue and an arrow from `r` to `9*r mod 17`. Keep the exact multiplication immediately beside the drawing. A visible caption must say: **“Residues arranged around a circle; these positions are schematic.”** This is not a plot of complex roots of unity. Never use complex-plane axes, an `exp(iθ)` formula, or geometric distances as evidence for modular multiplication.

Provide Previous, Next, Play/Pause, and Reset controls for the nine exponent states. Highlight the halfway and full-cycle identities when those states are reached. Choosing “Compare root 4” displays the invalid proposal alongside the accepted root, with the same exponent index. At exponent four, show the early return explicitly. Keep an equivalent exponent/residue table accessible at all times.

The default is a labeled replay of a recorded, verified fixture. A live mode can consume actual core events, but display time must not stand in for computation time. Never delay real completion to simulate a difficult calculation, invent intermediate primality-test activity, or animate an unperformed independent check. “Replay step 3” and “Verifier completed” describe different facts.

## 5. Interaction limits and accessibility

The core supports large 64-bit values; the visualization must not attempt to draw their complete residue sets. Full residue circles are allowed only for `p <= 31`. For larger inputs, use a bounded power-sequence table with at most sixteen transitions and a separate exact order-check card. Label a prefix as a prefix: it cannot visually establish the complete period.

Show at most 24 candidate cards at once, with a bounded page of recorded data. Never enumerate a huge interval merely to fill a drawing. Display actual recorded counts; do not extrapolate a density, fill omitted cards with invented outcomes, or imply continuous search coverage from a sample. The base handoff's cancellation and budget states remain authoritative.

Keep integer values as strings throughout transport. Any browser arithmetic uses `BigInt`; only bounded drawing coordinates and local step indices use JavaScript numbers. Recomputing a browser label does not upgrade verification status.

All controls must be reachable by keyboard with visible focus. Provide arrow-key stepping when the diagram control has focus, without capturing page-wide shortcuts. At 320 CSS pixels, numeric labels must remain readable; replace the circle with the table when necessary. Provide text alternatives for arrows and equations. Avoid status announcements on every animation frame; announce only meaningful steps and outcomes.

Respect reduced-motion settings by using static state changes, disabling automatic playback, and preserving every mathematical explanation. Video never autoplays. No drag-only interaction, flashing emphasis, or color-only validity indicator is acceptable.

## 6. Immutable scene data and status separation

Build a versioned presentation packet from immutable core artifacts and recorded evidence. Required fields are:

| Field | Purpose |
|---|---|
| `schema_version`, `scene_id`, `fixture_revision` | Reproducible scene identity |
| `mode` | `fixture_replay` or `live_run` |
| `artifact_digest`, `verification_report_digest` | Actual source references, or null when unavailable |
| `request`, `parameters`, `bound_objects` | Validated copies of base-schema values; request is required, while parameters and bound objects may be null until actually available during a live run |
| `events` | Ordered observations with sequence, type, phase, value, and evidence reference |
| `power_states` | Exact exponents and residues with derivation version |
| `verification_state` | `not_run`, `running`, `passed`, `failed`, or `interrupted` |
| `presentation_state` | Selected exponent, comparison selection, and playback position |

Presentation state is mutable; evidence and mathematical payloads are not. Derive event ordering from sequence numbers, and retain actual timestamps only when recorded. The player may change playback speed without altering either. A hash identifies content; it is not proof that a check occurred. Missing verification evidence means the interface cannot claim independently verified output.

Cache derived packets by source digests and derivation version. Bind all events to their run identifier so a delayed result from a previous request cannot color the current diagram. Treat an imported packet as untrusted: validate size, decimal caps, schema, and references, and use the core verifier before promoting a record. Progress cannot be imported to resume or establish previous search coverage.

Reverification establishes the imported arithmetic tuple and metadata consistency only; it does not authenticate the claimed event history, timing, or earliest-candidate status. Show imported observations as producer-reported unless locally replayed. Set the current `verification_state` to `passed` only after a fresh local verifier report matching the artifact digest; retain any producer-reported verification status separately. While parameters are null, show search progress without a fabricated modulus, root, or nonresidue bound.

## 7. P2: optional 58-second Manim film

Render a recorded-fixture explanation, clearly identified as such. Use this storyboard as a target; adjust pacing within 45–60 seconds after visual review.

| Time | Visual and learning point |
|---|---|
| 0–6 s | “We need a cycle of exactly eight steps.” Show `N=8` and the request. |
| 6–13 s | Reveal candidates 9 and 17 from the recorded run; 9 is composite and 17 is selected. Later cards remain untested. |
| 13–23 s | Show the two nonresidue checks, then `omega=3**2 mod 17=9`. Connect each symbol to its displayed value. |
| 23–36 s | Advance through the exact power sequence. Hold at exponent four with value 16, and at exponent eight with value 1. |
| 36–47 s | Compare proposed root 4. It returns to 1 after four steps, illustrating why the half-order check is necessary. |
| 47–58 s | Reveal the recorded independent-verification result and exported constants. Explain that search bounds and checked output are separate guarantees. |

Use concise captions and an optional original narration track. Deliver a transcript and captions if narration is added. A “verifier completed” event must come from an actual successful report prepared before rendering; timeline placement is an explanatory reveal, not a claim that verification took eleven seconds.

Manim Community Edition is the default offline renderer. The 3Blue1Brown FAQ recommends the Community Edition for newcomers [S1]. Its repository is distinct from Grant Sanderson's original ManimGL repository [S2, S3]. Do not mix `manim` installation examples with `manimgl` commands or `manimlib` imports.

Keep rendering dependencies in an optional environment with versions actually resolved and locked. The browser receives cached MP4/WebM output through ordinary video playback; it does not invoke Manim, LaTeX, FFmpeg, or a render worker during user interaction. The core installation and P1 must work without this environment.

## 8. Implementation milestones and acceptance

**V0 — fixtures and derivation.** Add exact scene fixtures and a converter from core artifacts. Completion: every table value and identity is independently recomputed; the invalid proposal is rejected; missing evidence never produces a verified badge.

**V1 — interactive explanation.** Implement the SVG diagram, candidate cards, table, comparison, keyboard controls, and reduced-motion behavior. Completion: all states agree with the evidence packet, mobile is readable, and large inputs stay bounded without full residue enumeration.

**V2 — optional film.** Add ManimCE scenes, renderer lockfile, caption/transcript assets, poster, and cached video. Completion: visual review covers all identity reveals, no labels overlap, timing is 45–60 seconds, and playback requires no rendering service.

**V3 — integration review.** Test out-of-order live events, cancelled verification, missing reports, tampered integers, changed artifacts, reduced motion, and no-renderer installation. Completion: no presentation action changes mathematical validity, and the base application remains usable with all optional visuals disabled.

Suggested additions are `visuals/fixtures/`, `visuals/schema/`, `visuals/derive.py`, `web/field-cycle/`, and optional `visuals/manim_ce/`. Cache renders using fixture, scene-code, renderer-version, font, and render-setting fingerprints. Add an NTT butterfly demonstration only as a separate later lesson using checked existing examples; it must not create a new core transform engine.

## 9. Sources and handoff boundaries

Sources checked November 21, 2026:

- **[S1]** 3Blue1Brown, About and FAQ: https://www.3blue1brown.com/about/ — visual-explanation context and Community Edition recommendation.
- **[S2]** Manim Community Edition: https://github.com/ManimCommunity/manim — renderer identity and current installation guidance.
- **[S3]** Original ManimGL: https://github.com/3b1b/manim — distinct project, package, and imports.
- **[S4]** MIT verification scope: https://math.mit.edu/~drew/grh/ — ordinary RH and Dirichlet statements.
- **[S5]** Lamzouri–Li–Soundararajan: https://www.math.ksu.edu/about/personal/xiannan/papers/QuaResL1ver2Sound.pdf — mathematical search bounds and their conditions.

The base handoff governs arithmetic, sources, permissions, and release criteria. This addendum adds optional explanation and locally reviewable media; it does not authorize hosting, publication, or external distribution.
