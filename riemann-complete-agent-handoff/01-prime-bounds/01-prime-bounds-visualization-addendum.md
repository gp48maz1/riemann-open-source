# Prime Bounds — Optional Visualization Addendum

**Prepared:** November 21, 2026  
**Paired specification:** `01-prime-bounds-handoff.md`  
**Status:** Proposed work after the numerical core passes its acceptance gates. No application, animation, or video is implemented by this document.

## Agent assignment and purpose

> Add the optional P1 interactive explanation and, subsequently, the P2 offline Manim sequence described here. Preserve every input, theorem-domain, arithmetic, serialization, resource, and claim-level contract in the paired handoff. Generate visual data from the reviewed numerical core; neither the browser nor the animation may invent results. Complete the mathematical and visual acceptance checks before presenting outputs as accurate. These additions do not authorize publishing, deployment, paid services, or source-license changes.

The learning question is: **What does a bound on prime counts tell us, and what uncertainty remains?** Borrow the useful explanatory principles associated with 3Blue1Brown: connect an equation to a visible quantity, preserve object identity through a transformation, and introduce one relationship at a time. Create original artwork and captions.

Use browser SVG, optionally with locally bundled D3-style utilities, for direct manipulation. Use **Manim Community Edition** for an optional prerecorded explanation and still frames. Grant Sanderson's [FAQ recommends Community Edition for newcomers](https://www.3blue1brown.com/about/). His original [3b1b/manim project](https://github.com/3b1b/manim) is the distinct ManimGL variant; its installation, CLI, and `manimlib` imports must not be mixed with CE's `manim` package. Browser Manim execution is unnecessary.

## Three connected views

### A. Exact staircase and analytic envelope

Plot the exact count `pi(x)` as a cyan, right-continuous staircase and the unoffset `li(x)` as a contrasting dashed curve. Start with the integer window `[2500, 3000]`. At a prime `p`, an open marker identifies the previous count and a filled marker identifies `pi(p)`. Vertical connectors illustrate jumps; they do not represent intermediate count values.

For `x >= 2657`, show the envelope `li(x) ± E(x)`, where `E(x) = sqrt(x) log(x)/(8 pi)`. Mask the envelope below `2657` and place a labeled threshold line. The exact staircase and, for `x >= 2`, `li` remain available there; this particular bound does not. At zero and one, show only the exact count and never evaluate `li`. Use uniform transparent fill and outlined edges, with no probability gradient.

The paired handoff's finite-range Johnston route applies throughout the proposed sieve-backed demonstration. The provenance panel must display that route, not suggest that these modest examples became computable only after the new RH result. The [Johnston paper](https://link.springer.com/article/10.1007/s11139-022-00616-x) supplies the finite-range theorem.

### B. Error measured against the bound

For valid inputs, display

\[
r(x)=\frac{\pi(x)-\operatorname{li}(x)}{E(x)}.
\]

This is a derived presentation of the same inequality. Label the horizontal rails `−1` and `+1`, with a central zero line. The theorem gives `−1 < r(x) < 1` where its domain applies. Use a dashed rail style to reinforce the strict inequalities.

Each integer sample uses an exact sieve count and an Arb enclosure for the expression. A marker may show the enclosure midpoint as a display approximation; retain its numerical enclosure in the detail panel. Do not clamp a point or its enclosure to the rails. If numerical uncertainty overlaps a rail, increase precision within the inherited budget or show an unresolved enclosure. Do not offer this view without an exact count.

### C. Subtracting uncertain endpoint counts

Two endpoint cards display `[La, Ua]` and `[Lb, Ub]`. Below them, a horizontal bracket represents the possible count in `(a, b]` using the paired contract:

`[max(0, Lb − Ua), min(b − a, Ub − La)]`.

A separate exact-count marker appears only when both endpoints have supported exact counts. The teaching example `(2657, 10000]` has exact count `1229 − 384 = 845`; analytic brackets come from the core. Shrinking the interval can leave a zero lower bound even when the exact count is positive. That conveys limited information from this bound, not a probability of finding no primes.

## P1: One interactive panel

Use one panel titled **“What the prime-count bound guarantees”**, with three tabs: Counts, Relative error, and Interval. Preserve the selected integer across tabs. Show the plot, a compact exact-value table, and expandable theorem/numerical provenance.

| Control | Behavior |
|---|---|
| Integer input and previous/next buttons | Move a shared crosshair; submit only canonical integer strings |
| Window preset | Select `[2500,3000]`, `[9000,10000]`, or another cached window inside the exact range |
| Display layers | Independently toggle staircase, estimate, and valid envelope; labels remain explicit |
| Interval endpoint inputs | Validate `0 <= a <= b`; clearly display `(a,b]` |
| Explain numerical precision | Compare cached valid precision runs and display numerical enclosure widths separately from analytic `E(x)` |
| Data/source buttons | Download canonical JSON and open theorem details |

A short guided sequence highlights four actions: inspect a prime jump, cross the `2657` threshold, switch to relative error, and compare uncertain endpoint subtraction with the exact answer. Users can advance or reverse each step. Avoid autoplay.

Exact integers stay strings or `BigInt` in the browser. Only bounded display coordinates may become ordinary floating-point values. When hovering between integers, snap the readout to a labeled integer rather than reporting an interpolated count. Tooltips distinguish exact counts, approximate display values, numerical enclosures, and theorem-derived bounds.

On narrow screens, stack plot, controls, then value table; use a minimum useful plot height and horizontal label abbreviation without truncating the accessible value. Every interaction needs a keyboard equivalent. Pair colors with line styles and text. Support reduced motion by switching tabs and guide steps instantly, provide a persistent pause control for any animation, and include a table alternative. Do not require hover to discover the result.

## Shared numerical data contract

Create one immutable dataset consumed by SVG and Manim. The exporter uses the existing core, including a separate forced-analytic call where the normal `auto` path returns an exact sieve result. It must not replace the main API's method choice. Expose reviewed calculation diagnostics through an optional exporter; do not duplicate `li`, `E`, rounding, or theorem routing in presentation code.

| Data component | Required content |
|---|---|
| Manifest | Schema version, dataset ID, actual code/dependency revisions, source dates, exact range, requested view, precision, generation settings |
| Prime events | Integer-string prime positions and integer-string cumulative counts; enough to reconstruct every jump in the visible window |
| Sample row | Integer-string `x`, exact count or null, `li` enclosure or null (null at `x=0,1`), `E` enclosure or null, analytic result or null, normalized residual enclosure or null |
| Domain record | Validity flag, threshold, theorem ID, and reason when a layer is unavailable |
| Interval record | Original core endpoint results, derived count bounds, and optional exact count |
| Display record | Approximate normalized coordinates, units, interpolation description, and a link back to the authoritative sample |

Represent exact numerical endpoints as dyadics: `{ "mantissa": "…", "exponent": "…" }`, meaning `mantissa * 2**exponent`. All mathematical integers remain strings. If using decimal display strings, label them approximate and retain dyadic source endpoints. Graphics never become the source of a mathematical result.

The visible staircase includes every prime event; no smooth interpolation is allowed. Analytic curves are sampled renderings: label them as such. Connecting sampled envelope edges does not itself certify intermediate points. The theorem supplies the continuum statement; the graphic illustrates it. Explicitly state **“Displayed samples illustrate the theorem; they do not prove it.”**

Outside the configured sieve range, exact count and normalized residual fields are null unless a separately supported exact backend produced them. Never fill missing counts with rounded `li`. A higher-precision calculation can reduce numerical enclosure width while leaving the analytic error essentially unchanged; explain that in two separately labeled rows, not with a single shrinking uncertainty band.

## P2: A 56-second Manim explanation

Render an optional silent explanation with readable captions, plus a transcript and WebVTT captions for a player. Narration can be added later without changing the mathematical sequence. Retain recurring colors, show the quantity before its notation, and use clear transitions rather than decorative camera movement.

| Time | Scene and action | Learning question / caption |
|---|---|---|
| 0–7 s | Build a small staircase from prime events; hold at an actual jump | “Counting primes gives an exact staircase.” |
| 7–15 s | Introduce the dashed `li(x)` curve; a vertical segment marks the discrepancy | “How far can a smooth estimate miss?” |
| 15–25 s | Reveal `E(x)` and the envelope only from `2657`; keep the earlier region masked | “This bound has a stated domain.” |
| 25–35 s | Re-express sampled discrepancies in units of `E(x)`; axes visibly relabel to `−1, 0, +1` | “The same bound becomes a limit of one in either direction.” |
| 35–46 s | Replace the plot with endpoint cards for `(2657,10000]`; derive the bracket and place exact `845` | “Subtracting bounds preserves uncertainty.” |
| 46–56 s | Hold a final card showing exact count, analytic width, and numerical width as separate concepts | “More numerical precision does not remove the theorem's error bound. Samples illustrate; the theorem guarantees.” |

The transformation into the residual view uses precomputed data for both endpoints of the animation. Intermediate animation shapes are explanatory transitions, not new measurements; dim numeric readouts until the transformed axes settle. Keep the final qualification visible long enough to read.

Produce one 16:9 master, a poster frame, and six labeled stills matching the beats. A later vertical version must reflow labels and equations rather than crop the master. Use original or permissively licensed fonts; no music, voice service, or API key is required.

## Optional engineering work and limits

Add `visualization/export.py`, `visualization/schema.py`, `static/prime_bounds_explorer.js`, `videos/prime_bounds_story.py`, `videos/manim.cfg`, `docs/visualization.md`, and focused visual-contract tests. Keep CE in an optional render dependency group; its absence must not affect the library, CLI, or interactive panel. Install the selected CE release's actual native/text-rendering dependencies and lock compatible versions. Do not claim a render succeeded before executing it.

The [CE output guide](https://docs.manim.community/en/stable/tutorials/output_and_config.html) documents low-quality previews, still-frame export, and output configuration. Start with a low-quality preview and static keyframes; render the accepted master at an explicitly configured 1080p/30 fps. Cache rendered assets by dataset, scene, font, configuration, and dependency revision. Videos are built offline once, never on each mathematical query or mouse movement.

Initial display limits: exact inputs at most the core's one-million sieve ceiling, window width at most 2000 integers, at most 2001 integer sample rows, and a five-megabyte dataset budget. These are proposed engineering caps, not measured performance promises. Reuse cached fixtures when possible; arbitrary query processing retains the core's time/precision limits. A render worker needs its own timeout and disk cap. Normal viewing works offline with locally bundled assets.

## Acceptance criteria

1. Every sample count and analytic bound matches its original core result; `2657` has exact count `384`, and `(2657,10000]` has exact count `845`.
2. Below `2657`, envelope and normalized residual layers are unavailable. No plotted line, tooltip, or animation frame quietly extends their guarantees.
3. The staircase jumps at primes and is right-continuous. Missing exact data suppresses exact markers and residuals.
4. Residuals use enclosure arithmetic and are never visually clamped. Uniform interval shading has no confidence-percentage or probability interpretation.
5. Higher numerical precision does not animate the analytic width toward zero. A zero interval lower bound is never captioned “no primes.”
6. Browser and video consume the same dataset revision. Large integers survive downloads and accessible labels unchanged.
7. Review all keyframes and the complete video for equation clipping, false intermediate labels, threshold masking, and caption readability. Check the interactive panel at mobile and desktop sizes, with keyboard navigation and reduced motion.
8. Confirm that no Manim installation, network call, model key, or per-query rendering is required to use the existing core or view cached material.

Primary implementation references, checked November 21, 2026: [Manim Community repository](https://github.com/ManimCommunity/manim), [CE output/configuration guide](https://docs.manim.community/en/stable/tutorials/output_and_config.html), [3Blue1Brown FAQ](https://www.3blue1brown.com/about/), [Python-FLINT Arb semantics](https://python-flint.readthedocs.io/en/stable/arb.html), and [Johnston's theorem](https://link.springer.com/article/10.1007/s11139-022-00616-x). Preserve the paired handoff's complete source and claim-level records.
