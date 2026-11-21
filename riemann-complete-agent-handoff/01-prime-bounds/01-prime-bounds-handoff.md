# Prime Bounds — Coding Agent Handoff

**Prepared:** November 21, 2026  
**Deliverable type:** Standalone implementation specification; no application has been implemented or validated by this document.  
**Working project name:** `prime-bounds`

## Agent assignment

> Build an open source Python library and local demonstration application that returns rigorously enclosed bounds for the number of primes at most an integer and in an integer interval. Follow the mathematical contract below. Start by checking repository instructions and the cited primary sources, resolve compatible maintained dependencies, and record their actual versions in a lockfile. Implement the numerical core before the interface. Use rigorous ball arithmetic and exact integer serialization throughout; never substitute ordinary floating point while preserving a rigorous-result label. Complete the MVP, its meaningful acceptance tests, documentation, and a reproducible local demonstration. Report what was implemented, what actually passed, and any unresolved mathematical or engineering review. Do not claim formal verification of this application. This assignment does not itself authorize publishing a repository, deploying a service, using paid infrastructure, or sending messages. Definition of done: the specified API, CLI, local UI, mathematical provenance, bounded resource behavior, and acceptance evidence are complete and reviewable.

## 1. Mission, audience, and value

Make published bounds on prime distribution usable by software without requiring every developer to reconstruct the theorem, handle numerical rounding, or distinguish an estimate from an exact count.

The principal users are computational number theorists, mathematics educators, numerical library developers, and AI agents that need a dependable mathematical tool. The core questions are:

- How many primes are at most a given integer, within explicit inclusive integer limits?
- What can be proved about the number of primes in `(a, b]`?
- Which theorem, arithmetic backend, and evidence support that answer?

The product promise is **a mathematical enclosure with understandable provenance**. An enclosure may contain many possible counts. A narrow relative error does not mean a small absolute error, and a prime count bound does not reveal the individual primes.

The new RH result supplies an all-range mathematical justification for familiar conditional bounds. It does not by itself improve runtime, discover a new prime-counting algorithm, or prove the Python implementation correct. Prefer a small reliable interface over claims of a general computational breakthrough.

## 2. Evidence baseline and exact scope

On November 20, 2026, Levent Alpöge released a Lean formalization accompanying an RH announcement. A November 21 statement by the Lean and Mathlib communities reports confidence in the proof and describes independent statement matching and proof checking. Andrew Sutherland's MIT verification page separately records compilation, axiom, and checker results. These are the primary evidence behind this project's RH status; this handoff has not independently compiled that proof. See [the community statement](https://leanprover-community.github.io/blog/posts/riemann-hypothesis/) and [MIT verification records](https://math.mit.edu/~drew/grh/).

This project needs **ordinary RH for the Riemann zeta function only**. Do not infer general statements about arbitrary number-field zeta functions, every kind of L-function, zero simplicity, or other conjectures from that scope.

There is a useful older route for finite inputs. Johnston's 2022 Corollary 3.3, using Theorem 3.2, equation (3.6), establishes the same prime-counting inequality through `1.101 × 10^26` without assuming full RH. Therefore, ordinary-sized demonstrations should not be advertised as newly possible. See [Johnston's paper](https://link.springer.com/article/10.1007/s11139-022-00616-x).

Before implementation, refresh the dated evidence summary and record the sources actually inspected. If a source becomes temporarily inaccessible, preserve the last documented observation and its date; do not invent a new checker result. Runtime queries use a reviewed local theorem registry and require no live web access.

## 3. MVP and later milestones

### MVP

1. Typed Python API for `pi_bounds(x)` and `interval_bounds(a, b)`.
2. Exact sieve-backed results for small inputs.
3. Rigorous analytic bounds using FLINT/Arb through `python-flint`.
4. Automatic theorem selection with explicit provenance.
5. CLI supporting readable output and versioned JSON.
6. Small local web interface for both query types and JSON download.
7. Reproducible tests, resource limits, documentation, and dependency inventory.

### Later, after the core is reviewed

- An optional `primecount` adapter for exact comparisons at larger feasible inputs.
- A theorem registry containing independently reviewed stronger bounds.
- A bounded batch API and an MCP adapter reusing the same core contracts.
- Bounds on the nth prime, with a separate reviewed inversion contract.
- Independently replayable numerical transcripts or formalized derivations.

Do not include nth-prime inversion, generalized RH applications, zeta-zero summation, or a custom primality-testing engine in the MVP. They add different correctness obligations. No external service, database, account system, LLM call, or GPU is needed for the MVP.

## 4. Mathematical contract

### 4.1 Input and counting conventions

`pi(x)` means the number of prime integers `p` satisfying `p <= x`. It is the ordinary integer-valued count, not a midpoint-normalized function.

The Python public API accepts nonnegative Python integers and rejects booleans, floats, decimals, and negative integers. JSON and CLI mathematical inputs are canonical base-10 integer strings: `0` or a nonzero digit followed by digits. Reject exponent notation, signs, embedded separators, whitespace inside numbers, and expressions. The UI may trim surrounding whitespace before validation, but must show the canonical value submitted.

`interval_bounds(a, b)` counts primes in `(a, b]`, requiring `0 <= a <= b`. Equal endpoints produce the exact result zero immediately. A lower endpoint that is prime is excluded; an upper endpoint that is prime is included.

Every successful result contains inclusive integer bounds `lower <= actual_count <= upper`. `exact` is true exactly when the returned interval is a singleton. An exact result obtained from singleton bounds still records the analytic method that established it.

### 4.2 Small inputs

Handle `x = 0` and `x = 1` explicitly as zero. Provide an exact Eratosthenes sieve and prefix counts for the small-input path. A suggested initial exact limit is `1_000_000`; it is a configurable engineering limit, not a mathematical threshold.

The default `auto` method uses exact counting through that limit. The required small-input path must always cover `0 <= x < 2657`. Do not evaluate the logarithmic integral at zero or one, and do not extrapolate the analytic theorem below its stated threshold.

For forced `exact` requests above the configured exact limit, return a resource-limit error. For forced `analytic` requests below `2657`, return a theorem-domain error. Neither request silently changes its requested method.

### 4.3 Analytic theorem

For `x >= 2657`, Schoenfeld's 1976 Corollary 1 gives, under RH,

\[
|\pi(x)-\operatorname{li}(x)| < E(x),\qquad
E(x)=\frac{\sqrt{x}\log x}{8\pi}.
\]

The denominator's `pi` is the circle constant. `log` is the natural logarithm. Use the **unoffset** logarithmic integral, equivalently `Ei(log(x))` for positive `x > 1`. In `python-flint`, the documented call is `arb(x).li(offset=False)`. Do not substitute `li(x) - li(2)`.

Use one numerical kernel and select its mathematical justification separately:

| Domain | Preferred evidence | Internal theorem identifier |
|---|---|---|
| Exact sieve domain | Exhaustive integer computation | `exact-sieve-v1` |
| `2657 <= x <= 110100000000000000000000000` | Johnston 2022, Corollary 3.3 and equation (3.6) | `johnston-2022-cor-3.3-pi` |
| Larger permitted `x` | Schoenfeld 1976, Corollary 1, with ordinary RH | `schoenfeld-1976-cor-1-pi-rh` |

The decimal upper endpoint above is exact. Test it and the next integer. Preserve the logical RH dependency in the registry even when current evidence supports discharging it; distinguish theorem dependencies from the public evidence used to accept them.

### 4.4 Numerical enclosure and outward rounding

Use arbitrary-precision real balls for the exact integer input, square root, logarithm, circle constant, and unoffset logarithmic integral. An `arb` value represents a real enclosure; its midpoint is not an exact answer.

Let `M` enclose `li(x)` and `R` enclose `E(x)`. Compute balls `A = M - R` and `B = M + R`. Obtain exact dyadic endpoints `a_lo = lower(A)` and `b_hi = upper(B)`, rounded outward by the backend. The theorem then implies:

\[
a_{lo} < \pi(x) < b_{hi}.
\]

Because the count is an integer, return:

\[
L=\max(0,\lfloor a_{lo}\rfloor+1),\qquad
U=\min(x,\lceil b_{hi}\rceil-1).
\]

This integer conversion uses the theorem's strict inequalities. Implement floor and ceiling on the exact dyadic endpoints, not on decimal display strings or binary64 approximations. The result can be slightly wider than the ideal mathematical interval if finite precision straddles an integer; that is acceptable and must remain enclosing.

Keep two error concepts distinct. `E(x)` is an analytic bound on the discrepancy between prime count and logarithmic integral. Ball widths describe numerical uncertainty while computing that bound. Increasing precision reduces numerical uncertainty; it does not shrink the theorem's analytic error.

Start with precision `max(128, x.bit_length() + 64)` bits, subject to resource limits. Adapt upward if needed. For the default quality target, require the widths of the computed `A` and `B` balls each to be at most `1/8` before issuing the normal result. This quality policy keeps rounding overhead modest; it is not the theorem itself. If the target cannot be met within the budget, return a clear precision-limit error instead of an unmarked approximation.

Assert finite enclosures, valid domain, positive analytic error, and `0 <= L <= U <= x`. A violation is an internal error and must never produce an apparently successful result. Encapsulate precision state; overlapping requests must not alter each other's arithmetic precision.

### 4.5 Interval queries

For endpoint bounds `[La, Ua]` and `[Lb, Ub]`, derive:

\[
\max(0,L_b-U_a)\le\pi(b)-\pi(a)
\le\min(b-a,U_b-L_a).
\]

Preserve both endpoint provenance records. The endpoints need not use the same method. A narrow interval may yield a lower bound of zero and a large upper bound because subtracting global bounds loses precision. Report that result honestly; zero as a lower bound does not prove that the interval has no primes.

For equal endpoints, use exact zero without two numerical evaluations. Never apply a short-interval prime-existence theorem unless it has been separately added and reviewed.

## 5. Python API, JSON, and evidence

The following is a proposed interface, not a claim about an existing package:

```python
from prime_bounds import Limits, interval_bounds, pi_bounds

answer = pi_bounds(10**100, method="auto", limits=Limits())
assert answer.lower <= answer.upper
print(answer.to_json())

local = interval_bounds(100, 1000)
assert (local.lower, local.upper, local.exact) == (143, 143, True)
```

Use immutable result objects and typed errors. Provide `auto`, `exact`, and `analytic` methods. The mathematical result must be independent of presentation or HTTP code.

All mathematical integers in JSON, including inputs, output counts, and exact dyadic components, are strings. Resource settings such as bounded precision bits may be JSON numbers. The browser must retain input and output counts as strings or `BigInt`; never route them through JavaScript `Number`.

| Required response field | Contract |
|---|---|
| `schema_version` | Stable schema identifier, initially `"1"` |
| `operation` | `"pi_bounds"` or `"interval_bounds"` |
| `input` | Canonical string-valued mathematical inputs |
| `bounds.lower`, `bounds.upper` | Inclusive integer strings |
| `bounds.exact` | Boolean; true exactly for a singleton |
| `method` | Actual executed method or endpoint-method mapping |
| `theorem` | Identifier, source URL, theorem locator, and applicable domain |
| `dependencies` | Required mathematical results and dated supporting evidence |
| `numerics` | Backend, actual precision, quality target, optional enclosure transcript |
| `software` | Actual application, Python, python-flint, and available FLINT version identifiers |
| `claims` | Explicit mathematical, numerical, implementation-review, and formal-verification statuses |

For intervals, `theorem` and `numerics` contain endpoint records plus the subtraction rule. Exact sieve answers have no analytic-error term and no RH dependency.

An illustrative result fragment for a forced analytic query at `2657` is:

```json
{
  "schema_version": "1",
  "operation": "pi_bounds",
  "input": {"x": "2657"},
  "bounds": {"lower": "384", "upper": "415", "exact": false},
  "method": "analytic"
}
```

This fragment omits required provenance fields for readability. Do not fabricate software versions or hashes to fill them. Record real values at build or execution time. An optional replay transcript may serialize dyadic endpoints as signed integer mantissa and exponent strings, representing `mantissa * 2**exponent`; a replayable record is not automatically a formal proof.

## 6. CLI and local interface

Proposed commands:

```text
prime-bounds pi 1000000000000 --method analytic --json
prime-bounds interval 100 1000
prime-bounds theorem schoenfeld-1976-cor-1-pi-rh
prime-bounds serve --host 127.0.0.1
```

Readable output starts with the inclusive bounds and whether the count is exact. Then show the method, theorem source, numerical precision, and limitations relevant to that query. JSON output must contain only JSON on stdout; diagnostics go to stderr. Document stable nonzero exit codes for input, domain, resource, and internal failures.

The local UI needs two forms, a clear `(a, b]` explanation, string-preserving result cards, expandable provenance, and a JSON download. Label a midpoint as an approximation if displayed. Avoid fake confidence percentages: the enclosure is a mathematical guarantee under its recorded basis, not a statistical confidence interval. A simple accessible HTML interface is sufficient; charts are optional later.

## 7. Resource contract

Support arbitrary-size integer mathematics within explicit implementation budgets. Suggested initial defaults are 1,000 decimal input digits, 16,384 precision bits, a 10-second analytic request timeout, and the one-million exact-sieve limit. These are adjustable product settings to validate with measurements, not performance promises.

Check length before integer parsing. Do not globally disable Python's conversion safeguards. Reject oversized JSON bodies early. Run web computations in bounded worker processes so timeouts can actually stop native arithmetic; an async HTTP timeout alone may leave C work running. Limit concurrency and use per-worker precision state. Serve locally by default.

No library or web request may silently allocate a sieve to a huge `x`, launch unbounded exact counting, or fall back to a slower algorithm after exceeding a cap. Cache only bounded small tables or bounded completed results. A mathematical theorem having no upper limit does not imply unlimited runtime support.

## 8. Architecture and repository layout

Use a Python package with optional web dependencies. `python-flint` is the numerical backend; a small framework such as FastAPI can serve the local UI. Keep the exact sieve in ordinary Python for its modest domain. Resolve current compatible releases and lock actual dependencies; this handoff intentionally supplies no guessed versions or commit hashes.

```text
pyproject.toml
dependency lockfile
src/prime_bounds/
  inputs.py
  limits.py
  models.py
  exact.py
  analytic.py
  intervals.py
  theorem_registry.py
  serialization.py
  cli.py
  web.py
  static/
tests/
  test_fixed_counts.py
  test_analytic_enclosures.py
  test_domains_and_limits.py
  test_serialization.py
  test_cli_web.py
docs/
  mathematical-contract.md
  theorem-sources.md
  validation.md
  dependency-notices.md
```

Store theorem facts as reviewed data with IDs, exact thresholds, formulas, convention notes, and source locators. Do not have an LLM rewrite the theorem registry at runtime. The UI, CLI, and future MCP adapter call the same core functions.

## 9. Ordered implementation tasks

1. **Establish the workspace.** Read applicable instructions, inspect existing work, and select an isolated implementation location if necessary. Recheck primary evidence and dependency APIs. Write the current theorem registry before coding arithmetic.
2. **Define contracts.** Implement validated inputs, limits, immutable results, JSON schema, and error taxonomy. Make all mathematical integer serialization lossless.
3. **Build the exact path.** Implement the sieve, prefix counts, boundary handling, and equal-endpoint shortcut. Confirm the fixed counts below.
4. **Implement analytic enclosure.** Encapsulate precision, evaluate unoffset `li`, obtain outward dyadic bounds, and convert strict inequalities to inclusive integer endpoints. Enforce quality and resource limits.
5. **Add theorem routing and interval subtraction.** Test finite-range boundaries, mixed endpoint methods, and provenance propagation. Review the mathematical derivation in this document against the implementation.
6. **Expose CLI and UI.** Keep process boundaries and serialization explicit. Add bounded web workers and useful errors without weakening the numerical contract.
7. **Validate meaningful risks.** Run independent exact-count comparisons, numerical convention tests, precision-state isolation, and large-integer round trips. Record actual environment and results.
8. **Finish reviewable delivery.** Provide README examples, source attributions, dependency notices, reproducible install/run commands, and a limitations section. Report any incomplete independent mathematical review. Prepare publication-ready material locally without assuming authorization to publish.

## 10. Acceptance tests and reference values

These are targets for the future implementation, not claims that an application already passed them.

| Input `x` | Exact `pi(x)` |
|---|---:|
| `0`, `1` | `0` |
| `2` | `1` |
| `3` | `2` |
| `10` | `4` |
| `100` | `25` |
| `1000` | `168` |
| `2656` | `383` |
| `2657`, `2658` | `384` |
| `10000` | `1229` |
| `1000000` | `78498` |

Independently regenerate these small counts using a reference implementation separate from the production prefix-count path. For larger reference data, primecount documents `pi(10^12) = 37607912018`.

With adequate precision and the strict endpoint conversion specified above, analytic fixture targets are `[384, 415]` at `2657`, `[78078, 79177]` at `10^6`, and `[37606850878, 37609049684]` at `10^12`. These endpoint targets were calculated at high precision for this specification; confirm them with the rigorous backend before making them committed fixtures. Exact reference counts must lie inside them.

Additional gates:

- `(2, 3]` returns exact `1`; `(3, 3]` returns exact `0`; `(100, 1000]` returns exact `143`.
- Forced analytic `2656` fails its domain check, while `2657` succeeds. Routing changes only above the exact finite-range limit, not at a rounded floating-point substitute.
- `li(10)` numerically encloses `6.1655995047872979375...`; the offset value near `5.1204357246698051527...` must not appear in the analytic kernel. This specifically catches the most plausible convention error.
- Synthetic dyadic endpoints verify floor-plus-one and ceiling-minus-one, including exact integers and values immediately on either side.
- An independent sieve through at least `100000` verifies inclusion for analytic samples covering primes, composites, threshold neighbors, and prime powers.
- A 100-digit input round-trips exactly through Python, CLI JSON, HTTP, and the UI. No exact prime count is asserted for that huge input.
- Increasing precision preserves valid enclosures. Do not require strict nesting unless the implementation explicitly intersects prior certified intervals.
- Parallel requests at different precisions give the same valid contracts as isolated runs. Timeout, maximum digits, maximum precision, and exact-limit failures terminate predictably.

Do not measure correctness solely by comparing the same formula implemented twice with the same backend. Separate exact ground truth, analytic theorem transcription, numeric convention, endpoint rounding, and serialization checks.

## 11. Review and claim levels

Maintain distinct statuses:

| Layer | What may be claimed after evidence exists |
|---|---|
| Mathematical basis | A named published bound applies, with its recorded dependencies |
| Numerical computation | The selected backend encloses rounding error using ball arithmetic |
| Implementation validation | The recorded tests and reviews passed on stated versions |
| Formal verification | Only claim this if the application's complete relevant derivation/code has separately been formally checked |

The RH proof's formal verification does not transfer automatically to this library. Set the application formal-verification field to false for the MVP. Use “rigorously enclosed numerical bounds” only after the numerical contract is implemented and reviewed. Do not call a JSON result a proof certificate merely because it contains a source URL.

Release notes must state whether mathematical review was internal or independent. Do not imply external reviewers endorsed the application without their actual review. There is no universal speedup claim: measure performance by input size, precision, method, hardware, and dependency versions.

## 12. Dependencies and licenses

Use an MIT license for newly authored project code unless the surrounding repository requires another choice. The inspected `python-flint` repository identifies an MIT license; FLINT's README states LGPL version 3 or later; primecount identifies BSD-2-Clause. Record licenses of the actual resolved versions and bundled native dependencies rather than assuming the wrapper's license covers everything.

Keep upstream numerical libraries as dependencies and retain required notices in redistributed packages. If binaries or containers bundle FLINT, GMP, or MPFR, inspect their actual distribution obligations and include corresponding materials. Do not copy proof archives or journal articles into the repository. Link to source documents and paraphrase the specific mathematical contract.

## 13. Source register

Source status was researched on **November 21, 2026**. Recheck at implementation time; changing pages and documentation are not immutable audit artifacts. No source hash or installed package version is asserted here.

| Source | Locator and intended use |
|---|---|
| Lean and Mathlib communities, RH verification statement, November 21, 2026 | [Community statement](https://leanprover-community.github.io/blog/posts/riemann-hypothesis/). Primary public evidence for the ordinary RH proof's checked status. Observed during research; a later refresh was temporarily inaccessible. |
| Andrew Sutherland, MIT, verification records | [Verification page](https://math.mit.edu/~drew/grh/). Primary checker reports and the precise theorem statements. Preserve its reported update time when documenting a snapshot. |
| Levent Alpöge, announcement | [Author's four-page note](https://alpo.ge/remark.pdf). Author's announcement and formalization context; not a substitute for independent checks. |
| Lowell Schoenfeld, 1976, *Sharper bounds for the Chebyshev functions theta(x) and psi(x). II*, Corollary 1 | [Publisher DOI](https://doi.org/10.1090/S0025-5718-1976-0457374-X). Original bound. Publisher access returned an error during this research; the formula and threshold were checked against Johnston's explicit citation and restatement. |
| Daniel R. Johnston, 2022, *Improving bounds on prime counting functions by partial verification of the Riemann hypothesis* | [Open paper](https://link.springer.com/article/10.1007/s11139-022-00616-x). Introduction equation (1.1), Theorem 3.2 equation (3.6), and Corollary 3.3. Supplies the finite-range route and confirms applicability to ordinary counts. |
| Python-FLINT Arb API | [Arb documentation](https://python-flint.readthedocs.io/en/stable/arb.html). Real-ball semantics, unoffset `li`, exact outward endpoints, and integer conversion APIs. Confirm behavior for the selected version. |
| Python-FLINT repository | [Repository](https://github.com/flintlib/python-flint). Dependency, installation, actual release selection, and license. |
| FLINT repository | [Repository](https://github.com/flintlib/flint). Native backend and LGPL-3-or-later statement. |
| Kim Walisch, primecount | [Repository](https://github.com/kimwalisch/primecount). Optional later exact-count backend, independent reference values, and BSD-2-Clause license. |

The mathematical extension, software design, limits, interfaces, and acceptance strategy in this handoff are implementation recommendations. They should be evaluated on their own merits; none is an assertion that these upstream projects endorse or already implement Prime Bounds.
