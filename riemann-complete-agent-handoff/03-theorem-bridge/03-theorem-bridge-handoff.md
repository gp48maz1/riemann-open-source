# Theorem Bridge coding agent handoff

Version 1.0 · Prepared 21 November 2026 · Proposed repository `theorem-bridge` · Primary language Python

## 1 Agent assignment

Build an open source application that connects precise mathematical hypotheses to published consequences, executable examples, and software implementations. The first collection concerns the Riemann Hypothesis and the verified Dirichlet L-function version of its generalization. A developer should be able to inspect whether a consequence applies, why it applies, and what remains unverified in an implementation.

Use this document as the complete product and engineering brief. Start with a local repository or the repository supplied by the operator, read its instructions, implement the milestones in order, and produce working software with reproducible validation. Make routine implementation decisions autonomously. Record consequential choices in a short decision log. The assignment is satisfied by a working catalog, deterministic applicability engine, local website, CLI, and two runnable examples; an AI service is optional. Provide local run commands and a completion report. Repository publication, paid infrastructure, credentials, and external deployments require the operator's existing authorization; they are not prerequisites to completion.

The project must remain useful when no language model is connected. Do not replace missing evidence with generated references, invented review identities, or an inferred formal proof. All project names and interfaces below are proposed designs.

## 2 Purpose and intended users

Scientific software contains algorithms whose guarantees depend on named conjectures. A headline saying that a conjecture is solved is insufficient for updating software: the exact mathematical statement, domain restrictions, implication, and implementation all matter. Theorem Bridge makes that chain inspectable.

The initial users are maintainers of number-theory packages, researchers selecting algorithms, educators building reproducible demonstrations, and AI coding agents that need structured evidence. The application's output should answer three separate questions: whether the required mathematics is established, whether the cited implementation faithfully computes the stated operation, and whether any claimed performance benefit has been measured.

Success means a user can find a consequence, see its exact supported scope, follow its primary references, run an included example, and export the same information as JSON. The initial product is a curated collection with explicit review states. General automatic understanding of arbitrary papers is a later research direction.

## 3 Evidence baseline and scope

The source snapshot for this brief is 21 November 2026. The Lean and Mathlib community statement endorses the announced RH proof [S1]. Andrew Sutherland's verification report identifies the checked statements as ordinary RH and the corresponding statements for all Dirichlet L-functions; general Dedekind zeta, Hecke, and automorphic L-functions are outside that checked scope [S2]. These are reports of external verification, not a claim that this application has independently replayed the proof.

Before implementation, open the primary references and record the observed publication or update time, retrieval date, exact mathematical statement, and any source revisions that are actually available. Older articles may still use conditional language because they predate the new result. Preserve their original hypotheses while recording a separate evidence item establishing a hypothesis. A transient unavailable URL does not revoke a theorem; keep the dated evidence and mark freshness separately.

Use stable internal hypothesis identifiers such as `rh.riemann-zeta` and `grh.dirichlet-all-characters`. The latter means every modulus and Dirichlet character, with the appropriate critical-strip and pole conditions. A label such as `grh.unspecified` is unresolved. A request requiring general Hecke L-functions must not match the Dirichlet identifier merely because both descriptions contain GRH.

The verification repository states that the original formal proof source was published without a license [S3]. The MVP stores original metadata and source links. It does not redistribute that source. This application's source and original catalog prose should receive an explicit open source license selected during repository setup; preserve notices for any reused code.

## 4 First release scope

Deliver these components together:

1. A version-controlled catalog containing at least eight substantive records from the seed list below, with accurate status for each.
2. JSON Schema or equivalent exported validation schemas for hypotheses, consequences, evidence, reviews, implementations, and benchmark reports.
3. A deterministic applicability engine with explicit domain checks and traceable decisions.
4. A CLI for validation, search, explanation, and static-site generation.
5. A local website with searchable catalog and individual consequence pages.
6. Two runnable examples with their own mathematical contracts: prime-count enclosures and bounded quadratic-nonresidue search.
7. Tests that exercise the actual scope, arithmetic, and review-state failure modes.
8. A complete README, contributor guide, source ledger, and reproducible lockfiles.

The first release does not need user accounts, a hosted database, a vector database, live ingestion of papers, an unrestricted agent runner, GitHub auto-PRs, or live theorem verification. Optional MCP support follows a stable core API. The application must not silently change other repositories' proof flags or label their implementations verified.

## 5 Seed collection

The entries below are the proposed initial scope. Each requires a source and a recorded status. The coding agent should implement the first two examples; it can catalog the others without executing external software.

| Entry | Precise content | Intended initial treatment |
| --- | --- | --- |
| Schoenfeld global prime-count bound | Absolute error for pi(x) about unoffset li(x), x at least 2657 | Source-supported consequence and runnable interval-arithmetic example [S4, S5] |
| Finite-range prime-count bound | Same bound through 1.101 × 10^26, established independently of full RH | Separate applicability route with its own range [S4] |
| Least quadratic nonresidue | For prime p at least 5, a nonresidue below (log p)^2 | Source-supported consequence and runnable bounded search [S6] |
| First prime in a residue class | For q greater than 3 and gcd(a,q)=1, bound on the least such prime | Catalog only initially; no arbitrary bit-window guarantee [S6] |
| Deterministic Miller witness bound | Small-base witness bound supporting deterministic primality testing | Catalog with precise formulation from Shoup/Bach [S7] |
| Robin divisor-sum consequence | Identify the arithmetic statement and external formal verification reported by MIT | Link the existing evidence; do not invent a local check [S2] |
| Lagarias divisor-sum consequence | Same evidence discipline for the corresponding divisor-sum criterion | Separately identifiable statement and provenance [S2] |
| Generic number-field class-group computation | A representative GRH-dependent application in PARI | Explicit `needs_scope_review` record; never mark covered just from the acronym [S2, S8] |

The two prime-count entries have different applicability routes to the same mathematical bound. The data model must represent this without pretending they are two newly discovered algorithms. An unsupported example is a useful catalog record only when its unresolved state is prominent and it has a legitimate source.

## 6 Domain and applicability semantics

Represent requirements as data, not natural-language instructions executed by an LLM. Use a small, typed predicate vocabulary: integer bounds, equality, primality evidence, coprimality, membership in a supported theorem family, and evidence-version requirements. No Python `eval`, executable YAML, or arbitrary code stored inside catalog predicates.

For each query return one of:

| Status | Meaning |
| --- | --- |
| `supported` | Every required hypothesis and input condition has a documented supported route |
| `conditional` | The mathematical implication is documented but a necessary hypothesis remains assumed |
| `needs_review` | Evidence or scope mapping is incomplete or disputed |
| `out_of_domain` | A required input condition is known to fail |
| `insufficient_input` | The caller omitted a condition needed for a decision |

These statuses concern applicability, not execution success. A supported computation can still time out. Runtime status is a separate field. No result should treat source freshness, mathematical evidence, human review, formalization, and implementation correctness as one boolean.

The engine should evaluate named implications with an AND set of requirements. Alternative valid routes form an OR. Store each route explicitly, including its own domain predicates. The finite-range upper endpoint belongs to the finite-range route, not to a shared condition that would incorrectly restrict the global route. Any fully supported route makes the consequence supported; an excluded or unresolved alternative cannot override that success. If no route succeeds, return `out_of_domain` only when every route is decisively excluded by known input conditions. Otherwise prioritize `needs_review` for unresolved evidence or scope, `insufficient_input` for missing required inputs, and `conditional` when a documented implication has only an assumed hypothesis outstanding. Preserve every route's state in the explanation trace so this summary does not hide the underlying conditions.

Reject every dependency cycle in the MVP catalog, preserve a deterministic explanation order, and return the full requirement trace. A missing prerequisite must remain missing even when an unrelated theorem has a similar name. A future fixed-point engine would require a separate design and tests preventing circular support.

For a primality precondition, `is_probable_prime=true` is insufficient unless that particular consequence explicitly permits probabilistic input. A caller can supply verified evidence through the documented adapter or ask the example runner to prove primality within its resource limit.

## 7 Data model and canonical serialization

Use Pydantic models or equivalent typed Python structures with exported JSON schemas. Store the authored catalog in JSON or strictly loaded YAML and compile a read-only SQLite index. Stable identifiers are independent of display titles. Semantic revisions are immutable; superseding evidence links to the old record rather than rewriting history.

| Entity | Required fields |
| --- | --- |
| `Hypothesis` | id, revision, precise_statement, family, quantifier_scope, exclusions, evidence_ids |
| `Consequence` | id, revision, title, statement, variables, domain_predicates, implication_routes, primary_reference_ids |
| `Evidence` | id, kind, source_url, observed_at, source_publication_time if known, scope, source_revision if known, local_artifact_hash if present |
| `Review` | id, object_id, object_revision, review_type, status, actor if known, evidence_ids, timestamp |
| `Implementation` | id, consequence_revision, repository_url or local example, revision if known, interface, tests, formal_status |
| `BenchmarkReport` | id, implementation_revision, workload_digest, environment, measured_results, limitations |

Use evidence kinds such as `published_mathematical_result`, `external_verification_report`, `local_verifier_run`, and `implementation_test_report`. Use review statuses `draft`, `source_checked`, `expert_reviewed`, and `superseded` without automatically promoting between them. A model-assisted draft has a separate `generated_by` field and cannot write an expert approval.

All arbitrary-size integers use canonical decimal strings over JSON. Machine-sized counters, page sizes, and schema versions may be JSON numbers. Do not convert scientific inputs through JavaScript Number or Python float. Hash stable canonical JSON after excluding timestamps and nonsemantic machine-local fields; document the serialization exactly. Use `null` for an unknown revision or hash, not a plausible placeholder.

Example of an authored record, with shortened prose for illustration:

```json
{
  "schema_version": "1.0",
  "id": "prime_count.schoenfeld",
  "revision": 1,
  "title": "Global prime count error bound",
  "variables": {"x": "nonnegative_integer"},
  "domain_predicates": [
    {"kind": "integer_gte", "variable": "x", "value": "2657"}
  ],
  "implication_routes": [
    {"requires": ["rh.riemann-zeta"], "reference_ids": ["schoenfeld_1976"]}
  ],
  "implementation_ids": ["examples.prime_count_bounds"],
  "formal_implication_status": "not_formalized_in_this_project"
}
```

The real record must contain the complete formula, definition of li, and primary-reference details. This example is a schema sketch, not a claim that a published package exists.

## 8 Executable examples

### Prime-count enclosure

Accept nonnegative integer x. For x below 2657 use a small exact sieve. For larger x the mathematical contract is

```text
E(x) = sqrt(x) * ln(x) / (8*pi)
abs(pi_count(x) - li(x)) < E(x), for x >= 2657
li(x) = Ei(ln(x)), the unoffset logarithmic integral
```

Evaluate both li(x) and E(x) using Arb real balls [S4, S5]. If L is a rigorous lower bound on li(x) minus the error term and U is a rigorous upper bound on li(x) plus the error term, return conservative integer endpoints floor(L) and ceil(U), clipped to the trivial interval from 0 through x. This is an enclosure, not an exact prime count. Outward rounding is mandatory. Do not use ordinary floating-point results as certified endpoints.

Increase working precision until the total interval-evaluation uncertainty in each endpoint expression is at most a rigorously obtained lower bound on E(x) divided by 100. Set a configurable precision ceiling and retry limit; return `precision_limit` if the target cannot be achieved within them. Record the achieved precision and numerical uncertainty separately from the analytic error. A rigorous but excessively wide ball is not a successful useful computation under this contract.

Certified execution requires applicability `supported`. For `conditional`, `needs_review`, or `insufficient_input`, the MVP runner returns the applicability trace without a certified prime-count interval. An optional future expression-evaluation mode must label its output conditional and preserve all unmet hypotheses. Rigorous numerical evaluation alone does not establish that a number-theoretic bound applies.

The initial public example accepts at most 1000 decimal digits and runs in a budgeted worker. Input and resource limits are product defaults, not mathematical limits. Return a structured unavailable or resource-limit result when the backend or budget cannot satisfy the contract. Keep the predicate checks and the computation implementation separate.

### Bounded nonresidue search

Accept a prime p at least 5, with deterministic primality evidence from the example's selected backend. Search integers z beginning at 2 in increasing order, testing `pow(z, (p-1)//2, p) == p-1`. A conservative integer upper limit is `p.bit_length()**2`; it exceeds `(ln p)**2` and avoids rounded logarithms. Search only z less than p. Record the exact candidate sequence and stopping policy. If computation is interrupted, report incomplete execution, not failure of the theorem.

The small default demonstration can be limited to p below 2^32, with deterministic trial division or a documented exact backend. This avoids requiring a large certificate ecosystem to ship the catalog. The general mathematical consequence remains separately stated without this software cap.

## 9 Architecture and repository layout

Prefer a Python package, Typer CLI, SQLite FTS5 search, and a static HTML site built from the catalog. Use FastAPI only if a live local API materially simplifies the example runner. A lightweight TypeScript frontend is acceptable, but a large application framework is unnecessary for the first collection. Resolve and pin compatible dependency versions during implementation; do not infer current APIs from this brief.

| Path | Responsibility |
| --- | --- |
| `src/theorem_bridge/models.py` | Typed entities and schema export |
| `src/theorem_bridge/validate.py` | Catalog validation and reference integrity |
| `src/theorem_bridge/applicability.py` | Predicate evaluation and implication routes |
| `src/theorem_bridge/index.py` | Reproducible SQLite index build |
| `src/theorem_bridge/search.py` | Exact filters and full-text ranking |
| `src/theorem_bridge/cli.py` | Stable CLI and JSON output |
| `src/theorem_bridge/site.py` | Static site build |
| `src/theorem_bridge/examples/` | The two reviewed numerical examples |
| `catalog/hypotheses/` | Precise hypothesis records |
| `catalog/consequences/` | Consequence records and domains |
| `catalog/evidence/` | Source metadata and local verification records |
| `catalog/reviews/` | Revision-bound review history |
| `schemas/` | Generated schemas checked into releases |
| `tests/` | Scope, numerical, schema, and integration tests |
| `docs/` | Source ledger, methods, decisions, contributor guide |

The authored files are the source of truth. Rebuild the index and website from them; do not maintain a second editable database. Run example execution in a separate process with a wall-time and input-size limit. The browser must never submit arbitrary Python code for execution.

## 10 CLI and optional API contracts

Proposed CLI commands:

```bash
theorem-bridge validate catalog
theorem-bridge search "nonresidue" --family dirichlet --json
theorem-bridge explain prime_count.schoenfeld --input '{"x":"1000000"}'
theorem-bridge run-example prime_count_bounds --x 1000000 --json
theorem-bridge build-site --output build/site
```

`validate` exits nonzero on broken references, invalid predicates, missing domains, or impossible review states. Search uses deterministic tie-breaking by stable id. `explain` returns applicability and the complete route trace. `run-example` returns applicability, execution status, numerical output, backend version, and evidence identifiers.

If implemented, use read-only routes `GET /v1/consequences`, `GET /v1/consequences/{id}`, `GET /v1/evidence/{id}`, and budgeted `POST /v1/evaluate`. Apply maximum page size 100 and request-body limits. MCP tools should wrap these same functions, with no separate policy engine. Do not include shell execution, arbitrary browsing, or catalog mutation in an initial agent-facing interface.

## 11 User experience

The home page offers search, hypothesis-family filters, runnable examples, and a clear source-snapshot date. Each consequence page shows its statement, applicable inputs, what changes computationally, available implementations, evidence links, and review history. Use distinct indicators for mathematical support, code testing, formalization, and performance.

An unsupported result should explain the missing requirement in ordinary language. For example, the interface can say that a routine requires a general number-field statement outside the verified Dirichlet scope. No green success indicator should be derived from the presence of a matching acronym.

Keep evidence visible without making the main interaction intimidating. Show the result first, with expandable assumptions and reproducible commands. Keyboard navigation, readable mathematical expressions, copyable integer strings, and accessible status text are part of the MVP. External links are reference links, not instructions for the application to execute.

## 12 Milestones and expected outputs

| Milestone | Work | Exit criterion |
| --- | --- | --- |
| M0 Evidence and decisions | Read sources, confirm scope, write short source ledger and architecture decisions | No invented verification or review status; requirements are explicit |
| M1 Catalog foundation | Implement models, schemas, predicates, validation, seed records | Catalog validates and scope-negative fixtures are rejected |
| M2 Applicability engine | Implement AND/OR routes, traces, domains, missing-input behavior | Decisions match independently specified fixtures |
| M3 Numerical examples | Implement rigorous prime bounds and exact bounded nonresidue demonstration | Mathematical contracts and edge cases pass |
| M4 Website and CLI | Search, evidence pages, reproducible example flows | A user completes all main flows from documented commands |
| M5 Release preparation | Lock dependencies, license own work, verify packaging, document limitations | Clean checkout reproduces catalog, site, tests, and examples |

Optional next work includes read-only MCP, import adapters for selected libraries, additional consequences, and a contributor review interface. Add one extension only after the MVP acceptance criteria are met. Do not make the project depend on any other repository described in the originating discussion.

## 13 Acceptance tests

1. **Scope separation:** `rh.riemann-zeta` alone does not satisfy a Dirichlet-GRH requirement. Dirichlet GRH does not satisfy a generic Hecke requirement. Unspecified GRH yields `needs_review`.
2. **Alternative routes:** the finite-range prime-count route can support an input in its documented range even when the global-RH route is disabled. Above that finite limit, it cannot.
3. **Domain boundaries:** x=2656 uses the exact example; x=2657 can use the analytic route. A nonresidue request p=9 is rejected; missing primality evidence yields insufficient input until the runner establishes it.
4. **Independent numerical checks:** exact prime counts at 10, 100, 1000, and 10^6 are 4, 25, 168, and 78498; every computed enclosure includes the count. Verify these fixtures with an independent sieve during implementation.
5. **Nonresidue checks:** for p=17 the least returned z is 3; for p=97 it is 5. Euler's criterion must hold; smaller candidates must fail that criterion.
6. **Review integrity:** changing an object's semantic revision invalidates a review of the previous revision for current display. An AI draft cannot satisfy an expert-review filter.
7. **Reference integrity:** nonexistent source ids, unsupported predicates, every dependency cycle, and missing input-domain definitions fail validation.
8. **Execution failures:** numerical-backend failure, worker timeout, and optional dependency absence produce distinct results without invented output.
9. **Serialization:** large integers round-trip byte-for-byte through CLI, JSON, database, and browser; no float conversion occurs.
10. **Reproducibility:** identical semantic catalog content produces the same compiled content digest; timestamps and local build paths do not perturb it.

Test the actual claims and dangerous boundaries. Avoid a large set of superficial snapshot tests that merely repeat field assignments.

## 14 Performance and operational constraints

The normal catalog and static website require no AI calls. Start with SQLite and precomputed pages. Record index build time, search latency, example runtime, and peak memory on the actual test machine; label them as measurements, not service guarantees. Public demo workers should have a short configurable wall-time limit and a bounded concurrency pool. The CLI can expose explicitly larger limits.

Optional AI enrichment must run as an offline authoring task with an explicit token or cost budget. Cache drafts by source and model revision. A generated reference must be resolved to a real primary source before inclusion. Do not crawl all of GitHub or continuously monitor sources in the MVP.

## 15 Definition of done and completion report

The repository installs from a clean environment, validates its authored catalog, builds its local site, and runs both examples without an LLM. All required tests pass. The README accurately distinguishes mathematical support from implementation and formal verification. Every substantive catalog record has primary references and a dated evidence record. No unavailable benchmark, review, source hash, or external verification is fabricated.

The completion report must include the implemented scope, commands actually run and their outcomes, source versions captured, known limitations, any blocked optional capability, measured resource use, and the next most valuable contribution. Do not claim a public deployment unless the operator authorized one and it was verified.

## 16 Sources

These are starting references checked for this handoff on 21 November 2026, including the source review in the preceding project research. Recheck live revisions when implementing; a new retrieval failure must be recorded without erasing an earlier dated observation.

- **S1** Lean and Mathlib communities, [Lean verification of the Riemann Hypothesis](https://leanprover-community.github.io/blog/posts/riemann-hypothesis/), statement dated 21 November 2026. The URL was temporarily unavailable on one later retrieval; its earlier retrieved endorsement is the dated baseline, not a new local verification.
- **S2** Andrew Sutherland, [Verification files for the Lean proof of the Riemann Hypothesis](https://math.mit.edu/~drew/grh/). Exact checked scope, independent verification reports, and Robin/Lagarias consequences.
- **S3** Lean community, [Riemann hypothesis verification repository](https://github.com/leanprover-community/riemann-hypothesis). Source provenance, build procedure, cost, and licensing statement.
- **S4** Daniel R. Johnston, [Improving bounds on prime counting functions by partial verification of the Riemann hypothesis](https://link.springer.com/article/10.1007/s11139-022-00616-x), 2022. Introduction and finite-range result; identifies Schoenfeld's bound and threshold.
- **S5** Python-FLINT, [Arb real-number API](https://python-flint.readthedocs.io/en/stable/arb.html). Interval semantics and logarithmic-integral support.
- **S6** Youness Lamzouri, Xiannan Li, Kannan Soundararajan, [Conditional bounds for the least quadratic non-residue and related problems](https://www.math.ksu.edu/about/personal/xiannan/papers/QuaResL1ver2Sound.pdf). Corollaries 1.1 and 1.2.
- **S7** Victor Shoup, [A Computational Introduction to Number Theory and Algebra](https://www.shoup.net/ntb/ntb-v2_1.pdf), section 10.5. Miller/Bach small-witness result; consult original references when formalizing the reduction.
- **S8** PARI/GP, [General number fields](https://pari.math.u-bordeaux.fr/dochtml/ref/General_number_fields.html). Representative GRH-dependent class-group and unit computations requiring scope review.
- **S9** [LeanExplore repository](https://github.com/lean-explore/lean-explore). Existing search and MCP capabilities; reuse or integrate where useful rather than rebuilding generic theorem search.
