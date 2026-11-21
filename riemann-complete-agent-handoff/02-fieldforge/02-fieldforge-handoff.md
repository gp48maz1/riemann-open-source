# FieldForge — coding-agent handoff

**Planning date:** November 21, 2026. **Deliverable:** a local, reviewable open source application and repository, with no deployment or publication implied. **Project status:** proposed; the APIs and filenames below are implementation requirements, not existing software.

## Copy-paste agent assignment

Build FieldForge: a Python application that deterministically constructs and independently verifies public prime-field parameters for transforms of length `N = 2**k`. Deliver a library, CLI, local HTTP API, small web preview, JSON records, and Python/Rust/C++ constant exports. Implement the bounded 64-bit MVP described in this document before attempting larger integers. Every successful result must pass the separate verifier; every interrupted search must retain an honest incomplete status. Keep the mathematical theorem, executable verification, source provenance, and measured performance separate. Use the named sources to establish exact conditions and record the dependency versions actually installed. No LLM, paid service, account, hosted infrastructure, or external repository publication is required. Work autonomously through milestones M0–M5, resolve ordinary implementation decisions, and finish with runnable commands, meaningful verification results, and explicit remaining limitations.

## 1. Mission, audience, and reason to build

FieldForge helps developers obtain reproducible arithmetic constants they can inspect and reuse. A user chooses a power-of-two transform length and an allowed modulus range; the application finds a suitable prime, constructs a root of unity, verifies the result, and exports the constants with their supporting evidence.

The initial audience is people implementing number-theoretic transforms, experimenting with finite-field algorithms, teaching computational algebra, or evaluating arithmetic parameters for larger systems. The deliverable is a convenient construction and checking tool. It does not implement a production transform engine or make a security judgment about a protocol that uses the parameters.

The useful differentiator is the complete workflow: a deterministic search, bounded computation, independent checking, readable mathematical assumptions, and correctly typed exports. Established libraries already provide primality and modular arithmetic. Reuse their strengths while making this workflow easier to audit. Do not promise that the recently verified theorem makes searches faster; performance must come from measurements.

These are **public arithmetic parameters**. FieldForge does not generate private keys, unpredictable secret primes, or a cryptographic security certification. The default search is deliberately reproducible. A valid field and a valid root do not establish the security of a proof system, encryption scheme, commitment scheme, or application built around them.

## 2. Mathematical background and verified scope

As checked on November 21, 2026, the Lean and Mathlib communities endorsed the correctness of the announced RH proof [S1]. Andrew Sutherland's verification page lists RH and the corresponding zero statements for all Dirichlet L-functions. It explicitly excludes general Dedekind zeta, Hecke, and automorphic L-functions from the statements checked [S2].

Use the identifier `dirichlet_grh` for the relevant mathematical premise. Do not treat every occurrence of “GRH” or “ERH” in unrelated software as this same statement. FieldForge does not remove assumptions from general number-field class-group computations.

Lamzouri–Li–Soundararajan provide the useful published implications [S3]:

- Corollary 1.1: assuming GRH, for prime `p >= 5`, the least quadratic nonresidue modulo `p` is less than `(ln p)**2`.
- Corollary 1.2: assuming GRH, for integer `q > 3` and `gcd(a, q) = 1`, the least prime congruent to `a mod q` is at most `(phi(q) * ln(q))**2`.

The new proof resolves the mathematical premise in this Dirichlet setting. FieldForge's reduction and application code still require ordinary mathematical and software review; reading a source URL does not formally verify the implementation.

The second bound concerns the **first prime in a progression from the start**. It does not guarantee a prime inside an arbitrary requested bit window, nor within the application's runtime budget. This distinction must appear in the API model and the user interface.

### Exact conservative bounds for the MVP

Avoid floating-point logarithms in operational stopping rules. For `q = 2**k`, `k >= 2`, use `phi(q) = 2**(k-1)` and `ln(q) < k` to derive the integer upper bound:

```python
first_prime_upper_bound = ((1 << (k - 1)) * k) ** 2
```

Label it a conservative consequence, not the sharp value of the cited formula. For a verified prime `p >= 5`, use `p.bit_length()**2` as an integer upper limit for nonresidue search. Handle `p = 3` directly with `z = 2`; the nonresidue corollary does not cover that case. For `k = 1`, handle the progression separately because `q = 2` does not meet the cited corollary's condition.

These conservative arithmetic bounds need no additional numerical dependency. A future display of sharper real-valued bounds must use outward-rounded interval arithmetic and name its implementation.

## 3. Exact success contract

For an accepted request and successful artifact, verify all of the following:

1. `1 <= k <= 63`, `N = 2**k`, and request bounds satisfy `3 <= lower <= upper < 2**64`.
2. `3 <= p < 2**64`, and `p` is prime.
3. The user's inclusive range contains `p`, and `(p - 1) % N == 0`.
4. `1 < z < p` and `pow(z, (p - 1)//2, p) == p - 1`.
5. `omega == pow(z, (p - 1)//N, p)` and `0 < omega < p`.
6. `pow(omega, N, p) == 1` and `pow(omega, N//2, p) == p - 1`.
7. `1 <= omega_inverse < p`, `1 <= N_inverse < p`, `omega * omega_inverse % p == 1`, and `N * N_inverse % p == 1`.
8. `construction.m == (p - 1)//N >= 1`.

Conditions 2, 3, and 6 establish that the root has exact order `N`. For a power of two, the half-order check rules out every smaller divisor. Condition 4 gives the construction its full power-of-two component: raising a quadratic nonresidue to `(p-1)/N` produces the required root. This argument does not require a complete factorization of `p-1`.

Check both construction and result identities. If an imported record contains a valid root but a false construction transcript, the result can be reported as an arithmetic tuple that passes the core checks while the complete FieldForge record is rejected. Do not silently rewrite evidence to make it pass.

## 4. MVP scope and computation policy

The MVP supports `uint64-v1`: integers below `2**64`, power-of-two orders, ascending deterministic search, one result per request, and integer arithmetic throughout. The default range is `[3, 4294967295]`; users may explicitly extend the upper endpoint to `18446744073709551615`.

Start with these configurable resource limits:

| Resource | Default | Hard local API maximum |
|---|---:|---:|
| Prime candidates examined | 100,000 | 1,000,000 |
| Worker CPU budget | 5 seconds | 30 seconds |
| Worker wall-clock deadline | 10 seconds | 60 seconds |
| Nonresidue candidates | Derived bound, at most 4,096 | Derived bound |
| Concurrent API workers | 1 | 2 |
| Imported artifact size | 64 KiB | 64 KiB |

These are engineering choices, not theorem constants. Make them easy to adjust after profiling. CPU and wall-clock budgets cover the entire worker operation: prime search, nonresidue search, construction, serialization, and independent verification. Use an operating-system worker supervisor plus cooperative cancellation checks in both search loops; the supervisor must terminate a stuck operation even inside a library call. Emit `found` only after the verifier completes within the operation budget. Partial progress is diagnostic: after forced termination, retain the last observed completed candidate and describe any unobserved tail as unknown.

Checkpoint import and search resumption are deferred. Every MVP generation starts from its requested lower endpoint. Imported progress never establishes previous coverage. A future resumption feature must replay the claimed prefix or authenticate trusted producer state bound to the exact request, algorithm, and completed work; an editable progress record is insufficient.

Generation has no random seed: its order is fixed. Reject a generation `seed` option instead of suggesting it changes results. Seeded randomness is allowed only in explicitly identified test and benchmark corpus generation. Parallel search is deferred; if added later, commit results in ascending candidate order so scheduling cannot change the chosen modulus.

### Search outcomes

Use separate structured states:

- `found`: the full artifact passed independent verification.
- `budget_exhausted`: unresolved within the requested resources; includes a reason, the interrupted phase, and available diagnostic progress.
- `range_exhausted`: every eligible candidate in the finite requested range was checked and none was prime.
- `invalid_request`: inconsistent range, unsupported domain, or malformed values.
- `internal_error`: violated invariant, worker failure, or disagreement between implementations.

Never convert a timeout into `range_exhausted` or “no suitable prime exists.” Use this exact contradiction guard: `k >= 2`, `N = 2**k`, `B = ((N//2)*k)**2`, `lower <= N + 1`, `upper >= B`, and `B < 2**64`. If every candidate with `m = 1..floor((B-1)/N)` was completely checked during this operation and none was prime, return `internal_error` and retain diagnostic evidence. A later-starting range, an interrupted prefix, or a bound outside the supported domain does not satisfy this guard.

## 5. Primality and independent verification

The generator may call SymPy `isprime` for inputs strictly below `2**64`, where its documentation describes deterministic results [S4]. Enforce this range before the call. Do not permit an oversized integer to fall through to a probable-prime path and then inherit the `uint64-v1` label.

Implement the independent verifier in a separate package using Python's standard library integer operations. It must not import the generator's search code, primality wrapper, modular helpers, or serialization-to-object conversion shortcuts. It may share a documented schema specification; it should parse and validate values itself.

For verifier primality, implement strong Miller–Rabin against the first twelve prime bases:

```text
2, 3, 5, 7, 11, 13, 17, 19, 23, 29, 31, 37
```

Handle small values and divisibility by these primes before the rounds. Sorenson–Webster's established bound for this base set exceeds the complete 64-bit domain [S5]. Document that domain explicitly. This verifier's correctness does not depend on the newly announced GRH result.

The verifier recomputes primality, all modular identities, request consistency, and any integer theorem-bound fields. It never trusts `verified: true`, a claimed base list, or a producer's checksum as evidence of primality. A content hash identifies bytes; it does not prove mathematics or identify an author. A successful tuple check also does not prove that this was the earliest eligible prime or that the claimed search history occurred. Validate those stronger claims only by replaying the bounded prefix; otherwise label them producer-reported provenance.

This is implementation independence, not independence from Python or from mathematical dependencies. State that limitation. Use a third implementation, such as FLINT or PARI, for a sampled differential validation corpus when available; keep it outside the verifier's runtime dependencies.

For larger integers, a later profile must carry and verify an actual primality certificate, such as an ECPP certificate. FLINT documents generation and verification facilities [S6]. Larger-number support remains disabled until the certificate format, verifier, resource policy, and export representation are implemented and reviewed.

## 6. Algorithms and data flow

Use these steps as the behavioral specification, not as a reason to duplicate code across modules:

```text
generate(request):
    validate schema, range, k, and resource limits
    N := 2^k
    m := max(1, ceil((lower - 1) / N))
    while candidate := m*N + 1 is within upper:
        check cancellation and budgets
        record this candidate as started
        if deterministic_generator_primality(candidate):
            p := candidate
            if p == 3: z := 2
            else scan z = 2,3,... through min(p-1, bit_length(p)^2)
                 checking cancellation and operation budgets on each step
                 and select the first Euler-criterion nonresidue
            if no such z: fail with mathematical invariant error
            omega := modular_power(z, (p-1)/N, p)
            construct inverse values and provenance record
            run independent verifier under the same operation deadline
            return found only if verifier completes and complete record passes
        record candidate as completed; m := m + 1
    evaluate the exact contradiction guard; fail if it applies
    otherwise return range_exhausted
```

Use integer ceiling division. Count every eligible progression candidate, including those rejected by small-prime filtering. A future sieve may skip expensive primality calls but must preserve the candidate count definition and evidence of skipped composites.

An optional `verify` operation checks a supplied artifact without searching. Verification should be quick for this profile and should not require internet access, a source repository checkout, or the complete RH proof. Preserve successful mathematical payloads separately from timing observations so timestamp changes do not destroy reproducibility.

## 7. Artifact, API, CLI, and local preview

Represent all arithmetic integers as **canonical unsigned decimal strings** in JSON, including `p`, `N`, roots, bounds, and search indices. Small schema controls such as `k` may be bounded JSON integers. Reject signs, whitespace, leading zeros except the value `"0"`, scientific notation, floats, duplicate JSON keys, and oversized digit strings.

Apply digit caps before integer conversion. Request endpoints, arithmetic parameters, and `construction.m` admit at most 20 digits, followed by their exact domain and consistency checks. The derived first-prime bound admits at most 41 digits and must equal its recomputed value. The nonresidue bound admits at most four digits and must equal the value determined by its method. These caps accommodate derived values larger than the modulus without weakening the 64-bit parameter limit.

Bounds are discriminated objects with `method` and `value`. For `k >= 2`, first-prime method `lls_integer_envelope_v1` has value `((N//2)*k)**2`; for `k = 1`, use `{"method":"not_applicable","value":null}`. For `p >= 5`, nonresidue method `bit_length_squared_v1` has value `p.bit_length()**2`; for `p = 3`, use `{"method":"direct_p3","value":"2"}`. Reject mismatched methods, values, and nulls. The metadata must not assert that a cited corollary covers an excluded small case.

Use a versioned schema. The following is the required shape; implementation-specific observations should use actual values, never fabricated hashes or dependency versions:

```json
{
  "schema": "fieldforge.parameters.v1",
  "profile": "uint64-v1",
  "request": {"k": 3, "lower": "3", "upper": "100"},
  "parameters": {
    "p": "17", "N": "8", "z": "3", "omega": "9",
    "omega_inverse": "2", "N_inverse": "15"
  },
  "construction": {"sequence": "ascending_m_v1", "m": "2"},
  "evidence": {
    "primality": {"method": "deterministic_mr12", "domain": "n_lt_2pow64"},
    "root_order": {"method": "power_of_two_order_identity"}
  },
  "theorem_context": {
    "family": "dirichlet_grh",
    "first_prime_upper_bound": {"method": "lls_integer_envelope_v1", "value": "144"},
    "nonresidue_upper_bound": {"method": "bit_length_squared_v1", "value": "25"},
    "verification_snapshot_date": "2026-11-21"
  }
}
```

The example's `144` is the conservative bound for `k=3`; it is not the minimum prime or a bound specialized to `[3,100]`. The actual found result can be correct even when a theorem's general bound lies outside the requested interval.

Keep runtime observations in a separate result envelope: status, candidates completed, elapsed time, CPU time if measurable, dependency versions, platform, and actual code revision when available. The verifier reports arithmetic validity and metadata consistency separately. It cannot independently reconstruct elapsed time or prove that a producer ran a particular executable.

Required CLI:

```bash
fieldforge generate --k 3 --min 3 --max 100 --out params.json
fieldforge verify params.json
fieldforge export params.json --language rust --out params.rs
fieldforge serve --host 127.0.0.1
```

Define documented exit codes for success, invalid input, invalid artifact, incomplete search, exhausted range, and internal error. Write machine-readable data to stdout only when requested; send progress to stderr. File output should be atomic, and existing files should require an explicit overwrite flag.

Expose `POST /v1/generate`, `POST /v1/verify`, and `POST /v1/export` through the same application services as the CLI. A local job endpoint may support cancellation and status polling. Bind the preview locally by default. Do not add accounts or deployment infrastructure to the MVP.

The web preview needs one form, a status area, a verified parameter card, a collapsible mathematical explanation, and export buttons. Show the chosen prime and root, verification results, applicable theorem scope, and search outcome. Explain that a general existence bound may exceed the selected window. Never route arithmetic through JavaScript `Number`; retain strings and use `BigInt` only for display-side checks. The backend verifier remains authoritative.

## 8. Exports and repository architecture

Validate an artifact immediately before exporting. Export public constants and a provenance comment, not a full optimized transform implementation.

- **JSON:** canonical decimal strings and versioned schema.
- **Python:** decimal integer literals; include a small explicit validation function.
- **Rust:** `u64` modulus and root constants, with `u128` intermediates if illustrative multiplication is included.
- **C++:** `std::uint64_t` with `UINT64_C(...)` literals; do not silently depend on signed overflow or nonstandard wide integers. If a helper needs `unsigned __int128`, guard it and state the compiler requirement, or omit that helper.

Do not emit unsafe `u64 * u64 % p` arithmetic for large moduli. Test exported code using compilers that are actually available; record skipped language checks honestly.

Suggested repository structure:

```text
fieldforge/
  README.md
  LICENSE
  pyproject.toml
  dependency-lockfile
  src/fieldforge/
    models.py
    search.py
    construction.py
    bounds.py
    budgets.py
    artifacts.py
    exporters.py
    cli.py
    api.py
    web/
  verifier/fieldforge_verify/
    parser.py
    primality.py
    verify.py
    cli.py
  schemas/parameters-v1.json
  tests/
  fixtures/
  benchmarks/
  docs/mathematics.md
  docs/provenance.md
  docs/limitations.md
```

Use Python integers, SymPy for generator primality, a small HTTP framework, and standard HTML/CSS/JavaScript. Choose and lock compatible versions at implementation time. A compiled arithmetic backend is a later optimization only after profiling identifies a bottleneck. No LLM inference belongs in construction or verification.

Select a permissive project license while preserving the licenses of dependencies and any reused material. Link research papers; do not copy their full text or include the enormous new proof archive in the repository.

## 9. Meaningful validation and adversarial cases

Required positive fixtures, calculated with exact integer arithmetic:

| p | k | N | z | omega | omega inverse | N inverse |
|---:|---:|---:|---:|---:|---:|---:|
| 17 | 3 | 8 | 3 | 9 | 2 | 15 |
| 97 | 5 | 32 | 5 | 28 | 52 | 94 |
| 3 | 1 | 2 | 2 | 2 | 2 | 2 |

For the first two fixtures, generation from lower endpoint 3 must reach the displayed modulus first. Verify every identity independently. Include a small transform example showing an 8-point forward/inverse round trip, with explicit normalization and convention; this demonstrates usefulness without making transform implementation part of the core product.

The `17` fixture has construction `m="2"`, first-prime bound value `"144"`, and nonresidue bound value `"25"`; the `97` fixture has `m="3"`, bound values `"6400"` and `"49"`. Both use the general bound methods. The `3` fixture has `m="1"`, first-prime method `not_applicable` with `null`, and nonresidue method `direct_p3` with `"2"`.

Adversarial tests must reject composite moduli, altered roots, roots of smaller order, false or noncanonical inverses, inconsistent `N`/`k` or construction `m`, incorrect `z`, invalid request ranges, oversized decimal fields, mismatched bound methods, duplicate keys, and a forged `verified` field. Include strong pseudoprimes such as `341550071728321` and `3825123056546413051`, both of which must be rejected by the complete twelve-base verifier.

Test boundary values around `2**64`, an empty progression intersection, and `p=3`. Inject exhausted candidate budgets, worker cancellation, and forced timeout separately during prime search, nonresidue search, and verification. No unfinished verifier run may emit `found`. A deterministic candidate budget should report the same completed prefix across runs; a time budget may stop at different points. Progress records cannot be imported to skip work. Exercise every precondition of the exact contradiction guard, including a later-starting interval that must not trigger it.

Check mathematical behavior against a small independent sieve and sample cross-library comparisons. Check that all exports round-trip to the original integers and compile where supported. Add no exhaustive search over the 64-bit space. Publish benchmark methodology and measured samples; never turn a small demonstration into a general speed claim.

## 10. Milestones and completion criteria

**M0 — contracts and source audit.** Create the repository, license, schema, source register, and mathematical specification. Verify the cited conditions against the primary sources and record the actual dependency choices. Completion: a reviewer can distinguish result verification, theorem-dependent search guarantees, and performance claims without reading implementation code.

**M1 — standalone verifier.** Implement strict parsing, deterministic twelve-base primality, root checks, and a verifier CLI with no generator dependency. Completion: all positive fixtures pass and adversarial mutations fail for the correct reasons, including oversized inputs and pseudoprimes.

**M2 — deterministic generator.** Implement ascending progression search, conservative bounds, nonresidue selection, supervised operation budgets, diagnostic progress, and independent verification before success. Completion: the two main fixtures are reproduced; interrupted work reports unknown/incomplete; exhausted finite ranges are distinguished accurately; the exact theorem-contradiction guard is enforced. Resumption is outside this milestone.

**M3 — application surfaces.** Add the CLI, local API, and web preview over shared services. Completion: a fresh local session can generate, verify, cancel, and inspect a result; malformed values are rejected consistently; arithmetic values survive the browser without numeric precision loss.

**M4 — exports and validation.** Implement all four export formats, fixture imports, small transform demonstration, differential tests, and a bounded benchmark command. Completion: exported values match exactly, available compilers accept generated files, and measured results identify hardware and resource budgets.

**M5 — reviewable release candidate.** Finish documentation, reproducible installation, pinned dependencies, CI configuration, and a release checklist. Run the required validation once against the final tree. Completion: deliver the local repository, exact run commands, test evidence, known limitations, and unresolved issues. Do not create an external repository, push code, publish a package, or deploy a service without a separate user instruction.

After M5, consider larger certified primes, tighter interval arithmetic, Rust acceleration, transform-specific parameter constraints, and broader finite-field tooling as separate scoped changes. They are not MVP acceptance requirements.

## 11. Primary sources and review notes

All sources below were checked on **2026-11-21**. Recheck live verification status when implementing; retain this dated snapshot rather than silently rewriting historical claims.

- **[S1] Lean and Mathlib communities, verification statement:** https://leanprover-community.github.io/blog/posts/riemann-hypothesis/ — support for the current proof-verification status.
- **[S2] Andrew Sutherland, MIT, verification files and precise theorem scope:** https://math.mit.edu/~drew/grh/ — the checked Dirichlet statements and exclusions.
- **[S3] Lamzouri, Li, and Soundararajan, Conditional bounds for the least quadratic non-residue and related problems:** https://www.math.ksu.edu/about/personal/xiannan/papers/QuaResL1ver2Sound.pdf ; publication record https://arxiv.org/abs/1309.3595 — use the exact corollaries and conditions, not an asymptotic estimate as an executable bound.
- **[S4] SymPy number theory documentation and source:** https://docs.sympy.org/latest/modules/ntheory.html ; https://github.com/sympy/sympy/blob/master/sympy/ntheory/primetest.py — verify the installed version's domain behavior before relying on it.
- **[S5] Sorenson and Webster, Strong pseudoprimes to twelve prime bases:** https://arxiv.org/abs/1509.00864 ; https://doi.org/10.1090/mcom/3134 — finite-domain basis for the independent primality verifier.
- **[S6] FLINT primality certificates and modular arithmetic:** https://flintlib.org/doc/ecpp/prove.html ; https://flintlib.org/doc/fmpz/modular.html — established numerical foundations and later certificate support.

Research citations support mathematical and dependency facts. The architecture, limits, schemas, milestones, and product decisions in this handoff are proposed engineering requirements.
