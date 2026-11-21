# Primality Bench — optional visualization addendum

Version 1.0 · Prepared 21 November 2026 · Companion to `05-primality-bench-handoff.md`

## 1. Coding agent assignment

When this addendum is selected, add a visual explanation of primality-test evidence and a clear way to inspect benchmark results. The central question is: **“What did this computation establish, and has enough work been completed to justify its verdict?”** Preserve the original handoff's algorithm, hypothesis profiles, assurance categories, resource caps, and result states.

Implement **P1**, a bounded interactive witness replay and benchmark report panel, after the core is correct. **P2** is an optional 58-second Manim Community Edition explanation. Both are independent extensions to the original handoff. Manim, media rendering, and any narration service are optional development dependencies. This document describes work to build; it does not report an implemented app, measured speedup, or rendered video.

## 2. P1A: replay a witness, then inspect the conclusion

Build a small teaching panel with exact input, selected method, base, decomposition `n - 1 = 2^s d`, modular residues, and the method's actual final verdict. Each “Next step” advances one recorded arithmetic operation or decision. A text table contains the same information. Show the rule beside the relevant residue so viewers can understand why a round passes or fails.

Use the following independently calculable fixtures. Obtain the application's event stream from the real implementation and compare it to these values; do not create a separate teaching implementation that quietly diverges from the tested algorithm.

| Fixture | Exact arithmetic | Required interpretation |
| --- | --- | --- |
| `n = 2047`, base `2` | `2046 = 2 × 1023`; `2^1023 mod 2047 = 1` | This single strong probable-prime round passes; it does not prove primality |
| `n = 2047`, base `3` | `3^1023 mod 2047 = 1565`; `s = 1` | The residue is neither `1` nor `2046`, and no further squaring is permitted by this round; base 3 is a compositeness witness |
| `n = 17`, base `3` | `16 = 2^4 × 1`; residues `3, 9, 13, 16` | The round reaches `-1 mod 17` and passes; the selected method still determines what further work is required |
| `n = 17`, completed bounded method | `B = min(15, 2 × 5^2) = 15`; all bases `2..15`, 14 bases | A completed full-base run supports `prime` through the original handoff's `complete_base_range` route |
| `n = 257`, bounded method | `B = min(255, 2 × 9^2) = 162`; bases `2..162`, 161 bases | The short-bound route needs its documented hypothesis profile; interruption or an unresolved profile cannot yield `prime` |

The `2047` teaching preset begins with base 2, then reveals the base-3 witness. Show its factorization `23 × 89` afterward as a separate independently checkable fact. Do not imply that the Miller witness automatically produced these factors.

For `17`, distinguish the base-3 example from the complete ascending base sequence. A later “Replay full method” step should use an actual full-run trace. The original handoff specifies all required bases; do not replace its proof condition with a visually pleasing shortcut. For `257`, include two distinct presets if useful: an actual intentionally budget-limited run, and a completed arithmetic run with an unresolved hypothesis profile. Both can be inconclusive for different reasons, which the interface must explain.

Show a progress counter such as **“8 of 161 required bases completed.”** This measures work coverage, never probability of primality. For a fixed-base run, use the core's `probable_prime` result when appropriate. Do not attach a random-trial error probability to deterministically chosen bases. Preserve every core status: `prime`, `composite`, `probable_prime`, `not_prime`, `inconclusive`, and `error`.

## 3. Controls, playback, and scale

Provide the three small teaching presets, Previous/Next, Play/Pause, playback-speed selection, and an exact-value disclosure. “Play” replays recorded operations; it does not rerun a test or benchmark. Default to a paused view with the current result visible. The explanation clock is deliberately slower than computation; label it “Teaching playback.” Any measured runtime appears separately with its measurement context.

Render a readable sequence of residues and rule checks, not a large orbit whose visual density conceals the algorithm. Highlight the first decisive witness, or the point at which the method's complete-work and evidence conditions are satisfied. A passed base gets the label “Round passed”; reserve “Prime” for the final method result. Pausing or stopping teaching playback executes no additional arithmetic and does not change a recorded verdict. This is distinct from cancelling a core computation, which follows the original handoff's interruption semantics and cannot establish an unfinished prime decision.

Keep detailed replay bounded: begin with fixtures and optional local inputs up to 32 bits, at most 64 displayed operations per base, and 256 individually displayed bases. If a real run exceeds a display cap, summarize omitted operations and preserve the actual counts, verdict, and downloadable trace. Truncating the visualization must not truncate the core computation or relabel a partial trace as complete. A 4096-bit accepted input is not a promise that this method can prove it within the default budget.

Use keyboard-accessible controls, persistent step numbers, text labels alongside color, and a responsive vertical layout on phones. Reduced-motion mode changes directly between discrete states. Include a complete accessible table and a transcript of each curated lesson. No autoplay is necessary.

## 4. P1B: a benchmark view that preserves assurance

Build on the original report format. Use two coordinated panels rather than an animated race:

1. **Completed runtimes:** measured time by bit length, input class, implementation, and assurance category. Use a log time axis only with an explicit label and valid positive values. Permit tooltips and filtering, and show sample counts and the aggregation method.
2. **Completion and stop reasons:** counts or fractions of prime, composite, probable-prime, inconclusive, error, timeout, and work-cap outcomes as appropriate to the existing schema. Keep algorithm verdict and execution stop reason as distinct fields even when the display groups them for convenience.

A timeout is a censored observation, not a completed runtime of zero or a measured finish at the timeout threshold. Show its known elapsed lower bound with a distinct marker and label, or keep it in the outcome panel. Do not average timeout thresholds with completed runtimes as if they were finishes. A case skipped because the required work exceeded a preflight cap is “Not run: work cap,” not a timeout. Timeouts, excluded cases, and errors must remain in the reported denominator.

Separate exact decisions, probable-prime methods, and certificate-producing methods using facets or explicit filters. Users may compare their costs, but the report must retain their different guarantees. Display dataset digest, method configuration, software versions, host information, repetitions, and measurement overhead policy. A chart alone cannot support a general “faster algorithm” claim.

Prefer the report's existing plotting stack. If none exists, Plotly's Python HTML export is a reasonable optional choice for hover, zoom, and filtering [V4]. For an offline artifact, include the JavaScript locally or embed it; an external CDN dependency does not satisfy offline operation. Keep the underlying CSV/JSON and table available. Filtering an exported report must not silently launch fresh benchmarks.

## 5. One trace contract for all renderers

Add an optional event recorder at the algorithm boundary. It may record the decomposition, base selection, initial exponentiation, allowed squarings, round outcome, work-limit event, and final result. Capture exact integers as canonical decimal strings; machine-sized indices and counters may be JSON numbers. Reuse the core's reviewed logic and returned result.

The following is an illustrative shortened trace record, not a complete schema:

```json
{
  "schema_version": "1.0",
  "fixture_id": "2047-base3-witness",
  "n": "2047",
  "decomposition": {"d": "1023", "s": 1},
  "rounds": [{
    "base": "3",
    "events": [
      {"kind": "initial_power", "residue": "1565"},
      {"kind": "round_result", "result": "composite_witness"}
    ]
  }],
  "result": "composite",
  "witness": {"base": "3"},
  "trace_complete": true,
  "measured_runtime_ns": null
}
```

The production contract must additionally identify the method and revision, required/completed base counts when meaningful, actual hypothesis profile and supporting route, stop reason, runtime-measurement context, trace truncation details, and the core result digest. For a short composite run, early stopping on a valid witness is a complete decision; it need not test every base. For a claimed prime under the bounded method, every required base and applicable theorem condition must be satisfied.

Record scene/storyboard revisions and source data hashes in the media manifest. A renderer receives a validated read-only snapshot and may highlight a state; it cannot change the core verdict. Structural validation must reject an asserted `prime` state attached to incomplete required work or an unresolved necessary route.

Trace recording changes overhead. Keep explanatory captures separate from the uninstrumented benchmark timing runs, or instrument every method uniformly and report the overhead explicitly. Never present the teaching replay duration as a speed measurement. A `null` runtime means no measurement was recorded, not zero time.

## 6. P2: Manim storyboard

Use Manim Community Edition for an original `WhatACompositeWitnessShows` scene. The `manim` package and `from manim import ...` conventions belong to this edition. Grant Sanderson's original ManimGL repository uses different conventions; do not combine their installation or scene examples [V1–V3].

| Time | Visual action | Caption or teaching purpose |
| --- | --- | --- |
| 0–8 s | Show `2047` and the question “Prime?”; split `n-1` into `2 × 1023` | Establish the inputs and operation being tested |
| 8–19 s | Follow base 2 to residue 1; reveal “Round passed” | Passing one selected base does not prove the number prime |
| 19–32 s | Switch to base 3 and residue 1565; show the failed rule and decisive witness | One valid compositeness witness settles the composite decision |
| 32–39 s | Reveal `2047 = 23 × 89` as a separate check | The known factors confirm the fixture; they were not produced by the depicted modular test |
| 39–50 s | Move to `17`; briefly show the base-3 sequence and then a compact full-method completion record | A passing round and a completed proof procedure are different stages |
| 50–58 s | Show the budget-limited `257` record with “Inconclusive”; contrast its completed-work counter with the required total | Incomplete work must remain inconclusive |

Use actual fixture exports for every arithmetic value and verdict. The final `257` scene must name the stopped run's real completed count rather than invent a plausible count. If the preset does not exist yet, create it using the core's deterministic work cap, validate it, and export it before storyboarding the final frame.

Deliver an MP4, poster PNG, captions, plain-text transcript, and source manifest. Narration is optional; the captions and geometry should already explain the lesson. Render offline and cache assets by fixture digest, scene code revision, storyboard revision, and render configuration. Do not render on page view, on each base selection, or during benchmark timing.

## 7. Implementation files and delivery sequence

Suggested modules fit within the original repository:

| Path | Responsibility |
| --- | --- |
| `src/primality_bench/trace.py` | Event schema and optional recorder |
| `src/primality_bench/visual_export.py` | Validated snapshots and report metadata |
| `reports/witness-replay.*` | Browser controls and accessible table |
| `reports/benchmark-view.*` | Existing-stack chart integration |
| `fixtures/visuals/` | Reproducible core-generated traces |
| `animations/composite_witness.py` | Optional Manim scene |
| `docs/visualizations.md` | Teaching script, commands, provenance, limits |

First capture and validate the small traces. Then implement the replay and benchmark panels. Review the mathematical captions and only then render the optional film. Keep Manim dependencies in an optional group with compatible pinned versions. A first render budget is ten minutes, two CPU cores, and 4 GiB RAM per scene; allow documented overrides. Use 720p/30 fps for development and optionally 1080p/30 fps for release. Optional render failures must not affect the classifier, CLI, or ordinary benchmark report.

Write original code and captions; retain dependency notices. No copied 3Blue1Brown voice, music, mascot, or video assets are required.

## 8. Acceptance criteria

1. Independently confirm the residue fixtures above, including that `2047` passes base 2 and fails base 3. The browser's final result equals the actual core result.
2. The full run for `17` has 14 required bases and uses the complete-base route even if the short-witness profile is unresolved. For `257`, the short route has 161 required bases and cannot claim primality without its required evidence.
3. A valid early composite witness remains decisive. An interrupted or capped prime candidate remains inconclusive; finishing the animation changes no status.
4. JSON round trips preserve exact integers above `2^53`. Truncated replay has a visible disclosure and retains the true total work counts.
5. Fixed-base success is not labelled a deterministic prime proof or assigned an unjustified random-trial probability. The progress bar describes work, not confidence.
6. Completed-runtime summaries exclude censored observations as finishes. Timeout/cap/error counts and denominators remain visible. No report uses illustrative or invented benchmark numbers without an explicit synthetic-data label.
7. Trace capture and timing runs have a documented overhead policy. Every reported timing has a real source record; the default teaching fixtures can have no runtime measurement.
8. Keyboard, mobile, reduced-motion, table, and caption views communicate the same verdicts. Inspect major PNG frames for clipped integers, readable exponents, and status labels that remain correct throughout transitions.

The completion report must state the tier delivered, fixture generation and checks, evidence profile, run commands, optional renderer setup, report provenance, and remaining limitations. A storyboard alone is not a completed animation.

## 9. Sources

The original handoff's source ledger governs primality, witness-bound, and probability claims. Reuse its reviewed algorithm and scope rather than deriving a new test in the presentation layer.

- **V1:** [3Blue1Brown FAQ/about](https://www.3blue1brown.com/about/): Manim and the Community Edition recommendation.
- **V2:** [Manim Community repository](https://github.com/ManimCommunity/manim) and [documentation](https://docs.manim.community/en/stable/): current scene, rendering, and configuration APIs.
- **V3:** [Original Manim repository](https://github.com/3b1b/manim): distinct ManimGL toolchain.
- **V4:** [Plotly interactive HTML export](https://plotly.com/python/interactive-html-export/): browser interaction and self-contained exports.
- **V5:** [Victor Shoup, A Computational Introduction to Number Theory and Algebra](https://www.shoup.net/ntb/ntb-v2_1.pdf), chapter 10, especially section 10.5: mathematical foundation used by the original handoff.

Tooling sources checked 21 November 2026. Concrete small modular-arithmetic values above were independently computed for this addendum. Timings in the storyboard and resource/display limits are proposed design choices, not benchmark measurements.
