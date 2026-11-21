# Primality Bench coding agent handoff

Version 1.0 · Prepared 21 November 2026 · Proposed repository `primality-bench` · Primary language Python

## 1 Agent assignment

Build an open source primality-testing workbench with a readable bounded deterministic Miller implementation, an independent correctness oracle for practical inputs, backend adapters, and reproducible benchmark reports. Its central question is which workloads benefit from the now-supported Dirichlet-GRH route to deterministic primality testing, and how those results compare with established methods at the same level of assurance.

This is a complete standalone specification. Work in the operator's chosen repository, read its applicable instructions, implement milestones in order, and finish with a usable CLI, an offline HTML report, documented APIs, meaningful tests, and reproducible results. Make ordinary engineering decisions autonomously and record consequential deviations. No language model is needed to run the software. Public GitHub publication, hosted execution, paid compute, or deployment are separate operator actions unless already authorized. Completion is local working software and an accurate implementation report, not a performance claim.

Do not use this brief as permission to implement a new cryptographic key generator. The project benchmarks mathematical operations on public test inputs. Do not implement APRCL or ECPP from scratch; use documented existing backends when available and report missing capabilities explicitly.

## 2 Users and value

The users are computational number theorists, maintainers of mathematical libraries, educators, and developers evaluating assurance/performance tradeoffs. They need a result that distinguishes a compositeness witness, a probable-prime outcome, a complete deterministic decision, and an interrupted computation. They also need fair measurements that do not hide preprocessing, timeouts, certificate verification, or differences in guarantee.

The useful contribution is a transparent reference implementation plus a trustworthy comparison harness. The established theorem changes a correctness guarantee; whether the resulting implementation is fast enough to use is an empirical question. A report finding no practical advantage is a valid successful project result.

## 3 Mathematical and source baseline

The dated baseline is 21 November 2026. The Lean/Mathlib community endorsement and MIT verification report support ordinary RH and the stated generalization for all Dirichlet L-functions [S1, S2]. They do not establish every broader hypothesis historically called GRH or ERH. Record exact theorem scope rather than relying on an acronym.

Shoup's section 10.5 describes the Miller/Bach result: under the relevant generalized hypothesis, a composite integer has a small Miller witness bounded by `2*(ln n)^2` [S3]. The implementation agent must check the precise reduction and its assumptions, preserve the source citation, and distinguish this established mathematical implication from formal verification of the program. The software's passing tests do not constitute a newly formalized proof.

At project start, record source observation dates and resolved dependency versions. Do not invent proof hashes, installed versions, expert reviews, or local verification runs. Normal execution must use a pinned evidence profile and work offline. Source freshness and validity are separate fields; temporary network failure does not automatically invalidate the stored mathematics.

The original formal proof is not needed at runtime. Its community mirror reports no license on the submission [S4]; link to it instead of bundling it. Use an explicit open source license for this application's own code and preserve licenses of reused dependencies.

## 4 First release scope

The required release includes:

1. A pure-Python strong probable-prime round with visible arithmetic and witness output.
2. A deterministic driver checking every required small base up to a conservative integer bound.
3. A fixed-base probable-prime mode that never upgrades itself to a proof because it passed.
4. A small independent sieve oracle for correctness testing and corpus labeling.
5. At least one existing exact primality backend for cross-checking and comparison, with its actual guarantees documented.
6. A deterministic, versioned corpus and raw measurement schema.
7. A benchmark runner with hard per-case budgets, truthful timeout handling, and separate assurance categories.
8. CLI commands and a self-contained offline HTML report generated from recorded data.
9. Tests, a methods document, lockfiles, and a reproducible completion report.

Optional later adapters can expose ECPP certificate generation/verification, APRCL, optimized GMP arithmetic, or a rigorously tighter logarithmic cutoff. Public compute endpoints, distributed benchmarking, GPU code, and an interactive cloud service are outside the first release. The report can be served as static files.

## 5 Integer domain and result semantics

Accept nonnegative integers. Reject negative values, booleans masquerading as integers, floating-point input, malformed decimal strings, and oversized inputs before expensive processing. Permit decimal text and an optional explicitly documented hexadecimal CLI syntax. Over JSON, all candidate integers, factors, bases, residues, and mathematical bounds are canonical decimal strings. Small counters and bit lengths may be JSON numbers.

Use these result categories:

| Verdict | Required meaning |
| --- | --- |
| `prime` | The chosen exact method completed all required work and its mathematical dependency profile is supported |
| `composite` | A valid factor or Miller witness establishes compositeness |
| `probable_prime` | A specified probabilistic or fixed-base test passed; this alone is not a proof |
| `not_prime` | The input is 0 or 1, which is neither prime nor composite |
| `inconclusive` | Budget exhausted, dependency unresolved, or exact method did not finish |
| `error` | Backend failure, malformed internal output, or violated invariant |

Keep `execution_status`, `verdict`, and `assurance` separate. `inconclusive` carries a structured reason such as `time_limit`, `base_limit`, `memory_limit`, or `unsupported_dependency`. A partially executed deterministic run must never return `prime`. It may retain the fact that tested bases passed in diagnostic fields without promoting that into its main verdict.

A composite witness remains valid independently of GRH when checked by exact arithmetic. A complete small-bound prime decision records the source profile supporting its stopping bound. If that hypothesis profile is deliberately marked unresolved in a test or alternate deployment and B is less than n-2, return `inconclusive` with a conditional mathematical note, not an unconditional prime result. Trivial cases, exhaustive trial division, and a completed full-base range with B equal to n-2 can still return their unconditional outcomes.

## 6 Reference deterministic algorithm

### Preprocessing and bound

Handle 0 and 1 as `not_prime`; 2 and 3 as `prime`; an even n at least 4 as `composite` with factor 2. For an odd n at least 5, write `n-1 = d*2**s`, with d odd.

Let `b = n.bit_length()` and choose:

```python
B = min(n - 2, 2 * b * b)
```

Because `ln(n) < b`, this integer bound is conservatively above the analytic `2*(ln n)^2` bound unless clipped to the complete candidate-base range. Test every integer base from 2 through B, inclusive. Do not test only prime bases without a separately justified algorithm and source. The conservative cutoff has extra work relative to a tightly evaluated logarithmic cutoff; show the actual bound policy in every benchmark.

When B equals n-2, the run covers all nontrivial residues used by the test; this complete-base route can be documented separately from reliance on the small-witness theorem. The default implementation must remain simple enough to audit.

### One Miller round

This is reference pseudocode; production code adds structured witnesses and budget accounting.

```python
from math import gcd

def passes_strong_round(n: int, a: int, d: int, s: int) -> bool:
    # Driver guarantees odd n >= 5 and 2 <= a <= n - 2.
    if gcd(a, n) != 1:
        return False
    x = pow(a, d, n)
    if x == 1 or x == n - 1:
        return True
    for _ in range(s - 1):
        x = (x * x) % n
        if x == n - 1:
            return True
        if x == 1:
            return False
    return False
```

The deterministic driver returns `composite` immediately on a failing round, with a factor if the gcd exposes one or a replayable witness otherwise. It returns `prime` only after coverage of the full required range and successful invariant checks, with either an accepted small-witness dependency profile or the unconditional `complete_base_range` route when B equals n-2. Increment `bases_completed` only after a round finishes. Record the inclusive upper base and the next untested base for interruption diagnostics.

The exact witness checker must recompute the arithmetic from n and a. Do not trust a serialized list of residues. Distinct code paths for generating and verifying a witness reduce shared-bug risk; at minimum the verifier must not call the same high-level verdict function that emitted the result.

## 7 Probable-prime modes and probability claims

Provide a fixed-base mode for transparent comparisons. It records the exact ordered base list and returns `probable_prime` when all rounds pass, except trivial primes independently recognized by preprocessing. Require at least one valid nontrivial base after any normalization; reject an empty effective vector before testing. Repeating a base does not increase the independent evidence. A base outside the valid range must be normalized according to a documented policy or rejected; avoid treating a zero residue as a meaningful passing round.

A randomized Miller–Rabin extension may use independent uniform bases for its usual worst-case error analysis [S3]. A reproducible pseudorandom benchmark schedule is not automatically an adversary-independent random experiment. The MVP should label saved base vectors as fixed vectors and leave a numerical probabilistic guarantee unset. If randomized mode is added, record the sampling method and assumptions, and never apply a Miller–Rabin error bound to BPSW or another algorithm with different theory.

Benchmark repeatability comes from storing inputs and base vectors, not from making unsupported security claims about a seed. No secret values are involved in the corpus.

## 8 Backend adapters and independent checks

Define an adapter protocol with `capabilities()`, `classify()`, and optionally `verify_certificate()`. Its manifest includes library name/version, arithmetic backend, supported size, verdict semantics, whether randomness is used, whether certificates exist, and which setup costs are measured. Resolve these details from current official documentation at implementation time.

FLINT documents existing primality-proving methods including APRCL and ECPP [S5]. Python-FLINT and SymPy are candidate integrations, but the implementation must verify the selected version's exact API and guarantees [S6, S7]. Some APIs are deterministic only below a size threshold or expose probable-prime results separately. Never convert a nonzero status code into `prime` without reading its documented meaning.

Use an independently implemented sieve to establish all labels through 100000 in the test suite. For larger fixture primes, retain an independent exact result or a verified certificate. For composite fixtures, preserve an explicit factorization or a validated witness. Exact comparison data must not be generated by the new algorithm being tested.

If an optional ECPP backend is absent, report `unavailable` for that benchmark cell. Do not silently substitute another algorithm and keep the original label. Certificate generation time and verification time are separate measurements. A hash of an output file is provenance, not a primality certificate.

## 9 Corpus specification

Store small authored fixtures in version-controlled JSONL with source or derivation. Generate larger corpora using a script that records its version, settings, seed, and full resulting input list. Keep generation and exact labeling outside timed regions.

Include these classes:

- Small boundary values and consecutive integers.
- Primes across several bit sizes.
- Even composites and composites with small factors.
- Odd prime powers and squares.
- Products of two similarly sized primes.
- Carmichael numbers and strong pseudoprimes to selected bases.
- Inputs that intentionally exceed default time/base limits.

Initial correctness fixtures:

| Input | Required outcome or property |
| --- | --- |
| 0, 1 | `not_prime` |
| 2, 3, 5, 17, 97, 65537 | `prime` under a completed exact mode |
| 4, 9, 25, 49 | `composite` |
| 561, 1105, 1729 | `composite`; include Carmichael classification in corpus metadata |
| 2047 | Composite, 23 × 89; passes the base-2 strong round |
| 1373653 | Composite, 829 × 1657; fixture for a number surviving selected small bases |
| 341550071728321 | Composite regression case; establish its witness or factors independently when importing |

Verify all fixture annotations in the implementation rather than trusting this prose. For large cases, a mislabeled dataset item must fail validation instead of being used to score the implementation. Benchmarks report class composition explicitly; they do not present an unlabeled average over arbitrary inputs.

## 10 Benchmark methodology

The correctness suite runs before timings. Benchmark a fixed corpus revision and report hardware, OS, Python/compiler versions, dependency versions, thread count, arithmetic backend, method settings, warmup policy, timer, and input digest. Use a monotonic high-resolution wall clock. Run algorithms sequentially on the same machine by default to reduce contention.

Separate at least three groups in the report: exact decisions, probable-prime screening, and certificate operations. A faster probable-prime screen cannot be presented as winning an exact-proof comparison. Show per-input data and grouped summaries by bit size and input class.

Use a small, documented warmup for loaded backends; distinguish process-start cost from steady-state operation time. Exclude corpus generation and oracle labeling from the algorithm timer, but report preprocessing costs performed by the algorithm itself. If trial division is used as a front end, record its policy for every adapter and offer comparable settings where possible.

Retain every timeout as a censored observation. Report completion counts and timeout rates beside median and tail timing summaries. Do not replace timeouts with zero, drop them silently, or compute an apparent speedup using only one algorithm's easy successes. When comparing completed paired cases, identify that subset and its coverage.

A valid first report can use 16-, 32-, and 64-bit inputs plus a deliberately small larger-input sample. Any default workload expansion must have a resource estimate. Never run an unbounded suite simply to obtain an impressive plot. Performance thresholds in CI should test gross regressions or budgets, not unstable microsecond differences on shared runners.

## 11 Data and API contracts

Public Python interfaces:

```python
def classify(n: int, *, method: str, budget: Budget) -> Classification: ...
def verify_composite_witness(witness: CompositeWitness) -> Verification: ...
def validate_corpus(path: str) -> CorpusValidation: ...
def run_benchmark(config: BenchmarkConfig) -> BenchmarkRun: ...
def build_report(run_paths: list[str], output_dir: str) -> ReportManifest: ...
```

Example shape, with genuine small values:

```json
{
  "schema_version": "1.0",
  "n": "17",
  "verdict": "prime",
  "execution_status": "completed",
  "method": "miller_bounded_reference",
  "bound_policy": "two_bit_length_squared_capped",
  "base_upper_inclusive": "15",
  "bases_completed": 14,
  "assurance": {
    "route": "complete_base_range",
    "implementation_formally_verified": false
  },
  "runtime_ns": null
}
```

This illustrates the schema; runtime is null because the handoff does not supply a benchmark measurement. Real results add the code revision, dependency profile, backend versions, budget, and any witness. Each benchmark row references the immutable classification settings and input id. A report export must preserve raw measurements in JSONL or CSV alongside the HTML.

Proposed CLI:

```bash
primality-bench classify 2047 --method miller-bounded --json
primality-bench classify 2047 --method fixed-bases --bases 2 --json
primality-bench corpus validate corpora/small.jsonl
primality-bench run configs/smoke.toml --output results/smoke
primality-bench report results/smoke --output reports/smoke
```

CLI exit codes distinguish invalid input, interrupted execution, and internal error. A mathematical composite result is a successful execution and must not be confused with a program crash. Define this behavior in the README before consumers depend on it.

## 12 Resource limits and process model

Suggested MVP defaults are product choices: maximum input 4096 bits for the CLI parser, a five-second per-case deadline, 100000 completed-base cap for ordinary runs, and a ten-minute total smoke-suite budget. Small correctness tests use much smaller cases. Record the selected defaults and permit explicit operator overrides.

Preflight the exact required base count B-1 before execution. Distinguish parser capacity from default proof-completion capacity: with this conservative cutoff, a 224-bit prime requires 100351 bases, so primes of 224 bits or more cannot complete under the default 100000-base cap. Time limits may constrain smaller inputs as well. The CLI and report must show this predicted budget shortfall; they may run the bounded prefix for compositeness discovery only if it is clearly labeled as unable to establish prime under those limits.

A deadline checked between rounds cannot interrupt one expensive modular exponentiation. Run each bounded task in a worker process so the supervisor can terminate it at the wall-time limit. Cap memory where the platform supports it and clearly report when only wall-time/input-size control is available. Do not present a cooperative deadline as a guaranteed hard limit.

Inputs above a method's supported size use `verdict="inconclusive"`, `execution_status="not_started"`, and `reason="unsupported_size"`. An unavailable optional adapter is a benchmark execution status with no mathematical verdict. A budget shorter than the required work returns `inconclusive`. A zero base budget must never yield a prime verdict for a nontrivial input. Do not persist a resumed partial run as complete without validating its input, algorithm revision, bound policy, tested-prefix coverage, and evidence profile; resumption can be deferred from MVP.

## 13 Architecture and repository layout

Use Python 3.12 or a newer version supported by all selected dependencies, a locked environment, Typer or argparse for the CLI, typed data models, and an HTML report generator. Choose a simple chart library with an export path and a license compatible with the report. Vendor no third-party source without preserving its license. Runtime model calls are unnecessary.

| Path | Responsibility |
| --- | --- |
| `src/primality_bench/reference.py` | Readable round and bounded driver |
| `src/primality_bench/witnesses.py` | Independent witness verification |
| `src/primality_bench/models.py` | Results, budgets, adapter manifests |
| `src/primality_bench/adapters/` | Existing algorithm integrations |
| `src/primality_bench/corpus.py` | Fixture generation and validation |
| `src/primality_bench/runner.py` | Isolated timed execution and budgets |
| `src/primality_bench/report.py` | Offline report and raw-data export |
| `src/primality_bench/cli.py` | User commands and exit semantics |
| `corpora/` | Small licensed/derived fixtures and manifests |
| `configs/` | Reproducible workload configurations |
| `evidence/` | Dated theorem and algorithm source profiles |
| `tests/` | Correctness, witness, timeout, schema, and reporting checks |
| `docs/` | Methods, source ledger, guarantees, and decision log |

Keep the reference algorithm readable. If an optimized implementation is added, maintain both paths and cross-check them; do not hide all arithmetic behind a backend and call the wrapper a new deterministic algorithm.

## 14 Report design

The report begins with the exact question, corpus description, assurance categories, and environment. Show completion/timeout counts before speed comparisons. Users can filter by bit length, input class, backend, and guarantee. Include a table of raw cases and a downloadable machine-readable dataset.

Use log-scale timing charts only with a clear axis label and accurate handling of censored cases. Display error and unsupported cases explicitly. Include measured setup and certificate costs where relevant. Every claimed speedup must identify the comparison method, corpus subset, and assurance category. The report title must not announce a faster prime algorithm unless measurements actually support that claim for a stated workload.

## 15 Milestones

| Milestone | Deliverable | Exit criterion |
| --- | --- | --- |
| M0 Source and contracts | Dated evidence profile, verdict model, selected backend semantics | Bound, scope, and backend guarantees are documented |
| M1 Reference core | Exact round, bounded driver, independent witness checker | Small oracle and adversarial cases pass |
| M2 Corpus and adapters | Versioned fixtures and at least one exact comparison backend | Labels independently validated; unavailable modes visible |
| M3 Budgeted runner | Worker isolation, raw result records, interruption accounting | Deliberate time/base/memory failures preserve correct statuses |
| M4 Report and CLI | Reproducible smoke run and offline HTML report | Report can be rebuilt entirely from stored raw results |
| M5 Release preparation | README, lockfiles, licenses, test/benchmark evidence | Clean environment reproduces correctness and documented workflow |

Finish this sequence before expanding to novel optimized cutoffs, certificate ecosystems, or distributed infrastructure. No other handoff or repository is required to implement this specification.

## 16 Acceptance tests

1. Compare the bounded algorithm against an independent sieve for all n through 100000. Keep the workload bounded and record runtime; a smaller default CI subset plus a release suite is acceptable if documented.
2. Independently verify all authored fixture labels. Confirm 2047 passes base 2 but is classified composite by the completed deterministic mode.
3. Test the full inclusive base range with an instrumented driver. Ensure no skipped base or off-by-one termination can yield `prime`.
4. Mutate serialized witnesses, factors, n, and residues. The verifier must reject invalid evidence and recompute essential arithmetic.
5. Exhaust a base budget on a prime that needs more rounds. Expect `inconclusive`, never `prime` or `composite`.
6. Kill a worker during modular exponentiation and confirm the supervisor records a timeout with incomplete coverage.
7. Mark the small-witness hypothesis profile unresolved. The completed full-base run for 17 still returns `prime` with `route="complete_base_range"`; the small-bound run for 257, with B=162 less than 255, returns `inconclusive`. A valid composite witness must still verify.
8. Confirm fixed-base passes remain `probable_prime`, empty effective base vectors are rejected, and backend status codes map correctly at documented size thresholds.
9. Round-trip integers above 2^53 through JSON, report data, and browser display without changing digits.
10. Produce a report with missing adapters and timeouts. Completion counts and censored cases must remain visible, and no fabricated speedup or zero-duration value may appear.
11. Rebuild an HTML report from saved raw data with no mathematical backends installed; the report renderer must not recompute results.
12. Run a clean-install smoke workflow and capture the actual commands, versions, and outcomes.

## 17 Definition of done and completion report

The reference algorithm and independent checks pass their correctness gates. At least one real exact backend comparison runs. The corpus is reproducible, all labels are justified, budgets are enforced as documented, and the report separates different levels of assurance. No model API, secret, paid service, or live source download is needed for ordinary operation.

The completion report states what was built, exact commands and results, supported input sizes, dependency and source profiles, numerical and timing observations, limitations, and the most valuable next experiment. If the benchmark does not show an advantage, say so plainly. Documenting that outcome is part of the project's value.

## 18 Sources

Primary references reviewed for this brief on 21 November 2026; resolve and record actual revisions during implementation.

- **S1** Lean and Mathlib communities, [Lean verification of the Riemann Hypothesis](https://leanprover-community.github.io/blog/posts/riemann-hypothesis/). Earlier retrieved endorsement dated 21 November 2026; one subsequent retrieval was temporarily unavailable.
- **S2** Andrew Sutherland, [Verification files for the Lean proof of the Riemann Hypothesis](https://math.mit.edu/~drew/grh/). Exact checked statements and scope.
- **S3** Victor Shoup, [A Computational Introduction to Number Theory and Algebra](https://www.shoup.net/ntb/ntb-v2_1.pdf), chapter 10 and especially section 10.5, printed pages 322–323. Miller/Rabin/Bach witness and probability foundations. Write original explanations and code; do not copy the textbook's licensed exposition.
- **S4** Lean community, [Riemann hypothesis verification repository](https://github.com/leanprover-community/riemann-hypothesis). Source provenance and licensing note.
- **S5** FLINT, [Primality proving with APRCL and ECPP](https://flintlib.org/doc/algorithms/number-theory/primality-proving.html). Existing exact-method capabilities and algorithms; verify the chosen installed release separately from development documentation.
- **S6** [Python-FLINT documentation](https://python-flint.readthedocs.io/en/stable/) and [repository](https://github.com/flintlib/python-flint). Candidate Python integration; inspect precise primality API semantics before use.
- **S7** SymPy, [Number theory documentation](https://docs.sympy.org/latest/modules/ntheory.html). Candidate comparison backend; preserve its documented deterministic-size boundaries and probable-prime behavior.
