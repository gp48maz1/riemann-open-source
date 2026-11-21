# Optional visualization addenda for the five project handoffs

Prepared 21 November 2026

## How to use these documents

Give a coding agent the original handoff and the matching addendum below. Each addendum is a self-contained extension for that project. The original handoff governs the mathematics, core product, numerical behavior, source scope, and release criteria. The addendum specifies optional explanatory work after the core is working.

These documents are design specifications. They do not contain implemented applications, completed animations, measured benchmarks, or new mathematical results. The existing handoffs remain separate. The ZIP contains this guide and the five addenda.

| Original handoff | Matching addendum | Main explanatory question |
| --- | --- | --- |
| `01-prime-bounds-handoff.md` | [01-prime-bounds-visualization-addendum.md](01-prime-bounds/01-prime-bounds-visualization-addendum.md) | What does a prime-count bound guarantee, and how much uncertainty remains? |
| `02-fieldforge-handoff.md` | [02-fieldforge-visualization-addendum.md](02-fieldforge/02-fieldforge-visualization-addendum.md) | Why does this parameter construction produce a cycle of exactly the requested length? |
| `03-theorem-bridge-handoff.md` | [03-theorem-bridge-visualization-addendum.md](03-theorem-bridge/03-theorem-bridge-visualization-addendum.md) | Which precise theorem and input conditions support this result? |
| `04-proof-atlas-handoff.md` | [04-proof-atlas-visualization-addendum.md](04-proof-atlas/04-proof-atlas-visualization-addendum.md) | What does a proof reference mean, and when does an explanation become stale? |
| `05-primality-bench-handoff.md` | [05-primality-bench-visualization-addendum.md](05-primality-bench/05-primality-bench-visualization-addendum.md) | What did a primality test establish, and was enough work completed? |

Suggested instruction to accompany any pair:

> Implement the original handoff first. Then implement the optional P1 visualization described in its addendum. Use the application's actual results and evidence as the source of visual data. Treat P2, the rendered Manim explanation, as a separate optional milestone. Preserve the mathematical, scope, status, and resource contracts throughout. Report what you implemented and checked, and distinguish proposed work from completed work.

If both tiers are wanted from the outset, replace the P2 sentence with “After P1 passes its acceptance checks, also implement P2.” This is a selection of deliverables, not a requirement to stop repeatedly for routine design decisions.

## Where Manim fits

[Manim](https://github.com/3b1b/manim) is the open source animation project associated with 3Blue1Brown. [Manim Community Edition](https://github.com/ManimCommunity/manim) is a separately maintained fork; [Grant Sanderson's FAQ](https://www.3blue1brown.com/about/) recommends it as a starting point for new users. Use Community Edition for these proposed scenes and follow its documentation consistently. Its `manim` package and imports differ from the original project's ManimGL conventions.

Use two complementary outputs:

- **Interactive explanation:** a browser-native plot, SVG diagram, existing graph view, or exported report where the user can change a bounded input, inspect evidence, or step through a recorded calculation.
- **Short explanation film:** an optional original Manim scene rendered offline from the same validated data. Include captions, a transcript, a poster, and reproducible source. Serve the output as ordinary cached media.

The browser interaction should respond immediately without launching a renderer. The film gives a carefully paced introduction, while the interactive view lets someone inspect a particular value or relationship. Neither requires an LLM, paid narration, or a live rendering service.

## Recommended first visual for each project

| Project | P1 interactive view | P2 film target | Useful future extension |
| --- | --- | --- | --- |
| Prime Bounds | Exact staircase, domain-aware analytic envelope, normalized residual, and uncertain endpoint subtraction in one panel | 56 seconds | A separate lesson on very large inputs and relative versus absolute error, without inventing exact counts |
| FieldForge | Candidate cards linked to a modular-power cycle; compare valid root 9 with an order-four substitute modulo 17 | 58 seconds | A checked NTT butterfly lesson after the parameter explanation is complete |
| Theorem Bridge | A compact applicability diagram with explicit all-required conditions and alternative supported routes | 56 seconds | Additional source-reviewed catalog lessons |
| Proof Atlas | A guided tour through real statement/proof/import references and a controlled version comparison | 54 seconds | More licensed teaching fixtures with independently bound explanations |
| Primality Bench | A witness replay plus measured-runtime and completion panels that retain assurance categories | 58 seconds | Additional real traces and deeper inspection of benchmark workloads |

If only one demonstration is built initially, FieldForge offers a particularly compact mathematical story: the user can see an early-return counterexample and understand the purpose of the half-order check. Prime Bounds is another strong starting point for explaining an abstract guarantee through a familiar plot. These are product-design recommendations, not claims about measured development effort or user demand.

## Shared architecture principle

Each project should export a validated, versioned snapshot of the facts it already computes. The interactive view and Manim scene consume that snapshot. Presentation code may change layout, emphasis, and pacing. The mathematical result, input, evidence scope, and computation status come from the core.

Keep these distinctions visible where they matter:

| Distinction | Consequence for the visual |
| --- | --- |
| Exact result versus sampled illustration | A plotted sample or interpolated edge never becomes a proof |
| Analytic bound versus numerical rounding | Increasing arithmetic precision does not erase the theorem's error term |
| A mathematical theorem versus an implementation | A supported theorem does not automatically certify code or benchmark speed |
| Passing one round versus completing a method | A progress animation cannot turn incomplete work into a prime verdict |
| Recorded playback versus live execution | Animation duration and reveal order are not computation timings |
| Diagram layout versus mathematical structure | Circle positions and camera motion carry no extra mathematical claims |

Use canonical decimal strings for arbitrary-size integers. Store source references and actual dataset/code revisions. Unknown metadata remains unknown. Keep rendering dependencies optional and assets cached. Respect keyboard use, narrow screens, reduced motion, and text/table alternatives.

## Illustrative Manim integration pattern

This small example shows the intended boundary. `load_validated_trace` is a proposed adapter the Primality Bench coding agent would implement as part of its exporter; it is not an existing API. The adapter must enforce the addendum's trace and evidence contract before returning the record. The scene reads the result and does not implement another primality test.

```python
from pathlib import Path

from manim import DOWN, FadeIn, Scene, Text, VGroup
from primality_bench.visual_export import load_validated_trace


class CompositeWitnessCard(Scene):
    def construct(self):
        trace = load_validated_trace(
            Path("fixtures/visuals/2047-base3-witness.json")
        )
        base = trace["witness"]["base"]
        lines = VGroup(
            Text(f"Input: {trace['n']}"),
            Text(f"Witness base: {base}"),
            Text(f"Recorded result: {trace['result']}"),
        ).arrange(DOWN)

        self.play(FadeIn(lines))
        self.wait(2)
```

After implementing that adapter and resolving a compatible Manim version, a development render follows the documented CLI pattern:

```bash
manim -ql animations/composite_witness_card.py CompositeWitnessCard
```

This is a minimal integration illustration, not the complete storyboard. The full scene should teach the modular arithmetic and final-decision distinction described in the project addendum. The [Community Edition repository](https://github.com/ManimCommunity/manim) and [output/configuration documentation](https://docs.manim.community/en/stable/tutorials/output_and_config.html) are the implementation references.

## Definition of a useful addendum implementation

A completed P1 delivers one understandable interaction, its accessible alternative, exact fixture provenance, and focused checks for the misleading states that matter to that project. A completed P2 additionally delivers actual rendered media, reviewed frames, captions/transcript, original scene source, and its data/version manifest. A storyboard by itself completes design work, not rendering work.

Start small, preserve the explanation's mathematical meaning, and stop after the selected tier works. The aim is to help a developer, researcher, or learner understand a result they can inspect.
