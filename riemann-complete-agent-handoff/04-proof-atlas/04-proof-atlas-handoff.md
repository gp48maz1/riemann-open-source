# Proof Atlas — Coding Agent Handoff

**Project:** `proof-atlas`  
**Prepared:** November 21, 2026  
**Deliverable:** A local application for exploring formal proofs, beginning with a small licensed fixture and an external-source profile for the Riemann Hypothesis proof.

## Copy-paste assignment

Build the local open source application specified below. Deliver working source, a documented development setup, a small demonstrably extracted Lean fixture, an independently labelled synthetic stress fixture, the bounded read-only API, an accessible browser interface, and executable acceptance tests. Implement and exercise the fixture path completely. Add the external RH source profile and a resource-limited extraction adapter, but do not download, redistribute, compile, or fully verify the large RH submission by default. Do not publish a repository, deploy a website, buy services, or use paid model APIs. Use the project’s own code and fixtures under MIT; retain upstream notices for any reused dependencies. Clearly report any unsupported extractor/toolchain combination instead of substituting an invented graph. Finish with local run commands, verification results, and any remaining limitations.

This document is the complete assignment. No previous conversation or other handoff is required.

## 1. Mission and audience

Proof Atlas helps a mathematician, Lean developer, or technical reader answer: **What does this theorem depend on, where is that dependence recorded, and which explanations have actually been reviewed?**

Its primary artifact is a small navigable neighborhood around a selected declaration. Each displayed relationship must trace to an extracted formal expression, a module import, or an explicitly authored explanatory relationship. The application must preserve these distinctions. A module being imported does not establish that a particular theorem uses everything in it.

The project uses the newly released proof as a demanding navigation and explanation use case. It does not promise faster prime generation, improved cryptography, or new number-theory algorithms. Its contribution is understandable access to proof structure and evidence.

The first useful journey is: choose the licensed demonstration dataset; search for a theorem; inspect its statement; expand immediate prerequisites; distinguish statement and proof references; open a source pointer; and compare a draft explanation with its review record.

## 2. Mathematical and source baseline

On November 21, the Lean and Mathlib communities published an endorsement of Levent Alpöge’s proof, reporting correct statement matching and independent proof checking. This provides the status baseline for the RH profile, rather than an older assumption that the result remains only an unsupported announcement. Record the endorsement’s date and source separately from anything this application has checked locally. [Community statement](https://leanprover-community.github.io/blog/posts/riemann-hypothesis/).

The checked statements include ordinary RH and GRH for **Dirichlet L-functions**, across moduli and characters. Do not label this as verification of arbitrary Dedekind zeta functions, Hecke L-functions, or automorphic L-functions. MIT reports 175,187 compiled modules and 1,468,418 declarations accepted in one full proof export; these are different counting units, not interchangeable project sizes. Its report also identifies the standard axioms and verifier compatibility differences. [MIT verification report](https://math.mit.edu/~drew/grh/).

The source is approximately 17 million lines. Treat that as context for designing bounded queries, not as a performance benchmark completed by this application. The community verification repository describes full checking as hours on a large machine, with roughly 100 GB of disk and substantial memory. It also states that the original submission was published **without a license**. Consequently, the MVP stores links and provenance metadata for that submission; it does not bundle its code, extracted text, generated documentation, or proof export in the distributed application. [Verification repository](https://github.com/leanprover-community/riemann-hypothesis).

A later operator may provide an authorized local checkout for private analysis. Before distributing a derived RH dataset, resolve its reuse terms. A proof being publicly accessible does not establish permission to redistribute its code. The application’s own MIT license applies only to its own material.

## 3. MVP boundaries and success

The MVP includes:

1. An immutable dataset manifest and import pipeline with strict validation.
2. One small original Lean fixture with real extracted relationships.
3. A separate synthetic dataset for layout, pagination, and failure testing.
4. Search, declaration details, bounded dependency exploration, and source/provenance panels.
5. Draft explanations, evidence anchors, review history, and automatic staleness detection.
6. A documented local RH profile whose default status is `metadata_only`.
7. An offline extraction worker and an adapter contract for pinned upstream tooling.

The web application does not accept arbitrary repositories, archive uploads, Lean snippets, build commands, or jobs. There are no accounts, public write endpoints, model subscriptions, live proof repair, full-project graph rendering, or hosted indexing in this release.

Completion requires a real fixture extraction, not only a convincing UI. If the environment cannot run Lean, complete the synthetic UI/API path and document the exact blocked gate; do not describe the full MVP as finished until the real extraction is reproduced.

## 4. Architecture and upstream reuse

Use Python 3.12+, FastAPI, Pydantic v2, SQLite with FTS5, and a React/TypeScript frontend built with Vite. Choose a maintained graph renderer after checking its license; Cytoscape.js is a reasonable implementation choice. Pin dependencies and commit lockfiles. Serve the production frontend from FastAPI so the local production app has one origin.

Keep three separately executable components:

- **Extractor:** offline worker that reads a fixed checkout and writes neutral JSONL plus provenance.
- **Indexer:** validates those files and builds an immutable SQLite database atomically.
- **Reader:** API/UI that opens the database read-only and cannot launch extraction.

Evaluate these existing foundations before writing a new Lean parser:

| Foundation | Intended use |
|---|---|
| [lean-extract](https://github.com/leanprover/leagent/blob/main/lean-extract/README.md) | Declaration/source/axiom extraction and dependency metadata. |
| [result-graph](https://github.com/PierreSenellart/result-graph) | Proof-term dependency extraction and focused graph concepts. |
| [LeanExplore](https://github.com/lean-explore/lean-explore) | Optional future semantic search or agent integration; lexical search is sufficient initially. |
| [verso-blueprint](https://github.com/leanprover/verso-blueprint) | Future export of reviewed exposition and structured dependency views. |

These tools already cover substantial infrastructure. Preserve licenses and attribution, and inspect their actual pinned output formats. Do not assume that `deps` containing the union of type and value references distinguishes statement from proof references. Use upstream separated fields when available; otherwise add a small Lean-side adapter that independently traverses the elaborated type and proof value.

Lean compiled artifacts are version-sensitive. Read the target’s `lean-toolchain`, align the extractor’s compiler, and record the exact extractor commit. Probe compatibility using a small fixture before attempting any large input. If alignment fails, return `unsupported_toolchain` with diagnostics. A metadata-only page and honest unavailable graph are the fallback. A synthetic graph must never appear under a real theorem’s identity.

## 5. Dataset and graph contract

Use content-derived immutable dataset IDs calculated from an explicit `identity_payload`: schema version, dataset kind, canonical source origin, source commit/archive hash, sorted source-file hashes, Lean/Mathlib versions, extractor name/commit/configuration, and hashes of the neutral extractor output files. Those neutral files use symbolic names and contain no dataset or application declaration IDs. The payload excludes `id`, `manifest_sha256`, timestamps, indexing counts, runtime observations, and other derived statistics, preventing a self-referential digest. Set `id` to the SHA-256 of that payload's canonical JSON bytes. Separately compute `manifest_sha256` over the complete final manifest with only its own `manifest_sha256` field omitted; it may change when dated metadata changes while the dataset ID remains stable.

Define canonical JSON as UTF-8 encoded `json.dumps(value, sort_keys=True, separators=(",", ":"), ensure_ascii=False, allow_nan=False)`, with floats forbidden and unordered collections sorted before serialization. Publish this exact convention and golden hash fixtures. Declaration IDs are deterministic hashes of dataset ID, entity kind, module, and fully qualified name; do not derive identity from a line number alone.

Implement these required fields in versioned Pydantic models with `extra="forbid"`:

```typescript
type Dataset = {
  id: string; schema_version: "1"; title: string;
  kind: "real_fixture" | "synthetic_fixture" | "external_proof";
  state: "indexed" | "metadata_only" | "unsupported_toolchain";
  source_url: string | null; source_commit: string | null;
  archive_sha256: string | null; manifest_sha256: string;
  license_id: string | null; redistribution: "allowed" | "not_established";
  lean_version: string | null; mathlib_commit: string | null;
  extractor_name: string | null; extractor_commit: string | null;
  indexed_at: string | null;
  declaration_count: number; edge_count: number;
  coverage: "complete_for_manifest" | "partial" | "none";
  missing_records: number; warnings: string[];
};

type Declaration = {
  entity_type: "declaration";
  id: string; dataset_id: string; name: string; module: string;
  logical_key: { origin_id: string; module: string; name: string };
  kind: "theorem" | "definition" | "axiom" | "other";
  statement: string; statement_sha256: string;
  proof_sha256: string | null;
  provenance: "project" | "dependency" | "synthetic";
  source: {
    relative_path: string; file_sha256: string;
    line_start: number; line_end: number;
    pinned_url: string | null;
  } | null;
  axioms: string[] | null;
  axioms_status: "extracted" | "not_extracted";
};

type UnresolvedStub = {
  entity_type: "unresolved"; id: string; dataset_id: string;
  name: string; module: string; resolved: false;
  origin: "outside_scope" | "missing_record";
};

type ModuleNode = {
  entity_type: "module"; id: string; dataset_id: string;
  name: string; module: string;
  provenance: "project" | "dependency" | "synthetic";
};

type Edge = {
  id: string; dataset_id: string; from_id: string; to_id: string;
  kind: "statement" | "proof" | "import" | "explanation";
  evidence: "elaborated_type" | "elaborated_value" |
            "module_header" | "review_record";
  evidence_sha256: string;
};
```

For dependency edges, `A → B` means **A refers to B**. If B occurs in both A’s type and proof, retain two typed edges. Module import edges connect separate module entities; never silently convert them into theorem dependencies. Explanatory edges belong in a separate overlay, disabled initially. They cannot establish formal dependence.

Preserve external references as explicitly unresolved stubs when their declarations were outside the extraction boundary. A stub has a name, origin, and `resolved=false`; its missing statement must not be generated. Model such stubs separately from `Declaration`, whose statement is required. Missing internal targets fail import unless the manifest explicitly marks partial coverage and lists them. An empty axiom list is meaningful only when `axioms_status="extracted"`; `null` means unexamined.

Record extraction coverage, dropped generated declarations, private-name policy, and unresolved references. No graph view may imply that a selected dependency slice is the complete proof. A valid proof does not establish the accuracy of its informal explanation.

## 6. Reviewable explanations

Store explanations in version-controlled YAML, not as mutable fields silently written by the model. Each record requires an ID, dataset ID, declaration ID, declaration logical key, statement hash, proof hash, text, cited declaration/source anchors, author type (`human` or `ai`), optional model identifier, creation time, and status (`draft`, `reviewed`, `rejected`, `stale`).

A human review record requires reviewer attribution, date, reviewed explanation hash, and a short note describing what was reviewed. The coding agent must not manufacture human approval or mark its own generated prose as reviewed. Ship clearly labelled drafts plus an example review record attached only to an explicitly synthetic fixture.

Dataset-specific declaration IDs cannot identify continuity across snapshots. Match versions by the separate logical key `(origin_id, module, fully qualified name)`, where `origin_id` is the catalog's stable upstream repository/package identity without a commit. Keep an explicit mapping from that key to each snapshot's declaration ID. A rename or origin change requires a reviewed mapping record; never carry reviews through fuzzy name matching automatically.

For a matched declaration in a new snapshot, compare the statement hash, proof hash, explanation text hash, and content hashes of every evidence anchor, including referenced declarations. Reuse the review only when every comparison is available and equal; create an explicit reuse record linking the old review and new snapshot. Snapshot URLs or IDs may change while their pinned content remains identical. Unknown hashes prevent reuse. Any changed statement, proof, explanation, or evidence content marks its carried-forward review `stale`; preserve the original review as valid only for its original snapshot. Reviewing prose does not confer formal verification. A displayed proof-check report must identify its source, version, commands or external URL, and whether the check ran locally. Do not collapse “community reported”, “locally extracted”, “locally compiled”, and “independently verified” into one green badge.

No LLM call is required in the MVP. Include a future adapter interface accepting a bounded declaration neighborhood and returning a draft plus cited IDs. Schema validation verifies those IDs exist; it cannot verify that the prose is mathematically faithful.

## 7. Read-only HTTP API

Generate OpenAPI from the Pydantic contract and commit a deterministic export. All data routes require a dataset ID, preventing accidental cross-version joins.

| Route | Required behavior |
|---|---|
| `GET /health` | Version and readiness; no environment secrets. |
| `GET /api/v1/datasets` | Paginated dataset summaries. |
| `GET /api/v1/datasets/{id}` | Manifest, coverage, source and licensing status. |
| `GET /api/v1/datasets/{id}/declarations` | `q`, `kind`, `limit`, and `cursor`; lexical name/statement search. |
| `GET /api/v1/datasets/{id}/declarations/{decl_id}` | Declaration, explicit missing-reference summaries, explanation references. |
| `GET /api/v1/datasets/{id}/subgraph` | Bounded dependency neighborhood and continuation information. |
| `GET /api/v1/datasets/{id}/explanations/{explanation_id}` | Explanation and immutable review history. |

List responses have `{items, next_cursor, dataset_id}`. Default limit is 25; maximum is 100. Escape FTS query syntax or expose only tokenized plain-text search. Empty search lists declarations in stable name/ID order. Query text is limited to 256 characters.

Subgraph parameters are `root`, `direction=uses|used_by`, repeated `kind=statement|proof|import|explanation`, `depth=0..3`, `node_limit=1..300`, `edge_limit=1..1000`, and optional `cursor`. Defaults: depth 1, 100 nodes, 400 edges, formal statement/proof edges only. Reject requests whose root entity and edge type are incompatible.

```typescript
type SubgraphPage = {
  dataset_id: string; root: string;
  nodes: Array<Declaration | UnresolvedStub | ModuleNode>;
  context_only_node_ids: string[];
  edges: Edge[];
  next_cursor: string | null;
  truncated: boolean;
  reason: "node_limit" | "edge_limit" | "traversal_budget" | null;
  complete_for_requested_depth: boolean;
  visited_nodes: number; scanned_edges: number;
};
```

Implement the entity union using the `entity_type` discriminator. The traversal is deterministic breadth-first with ID tie-breaking; a node can occur only once per page. Across pages, each discovered entity and each result edge is emitted once in the logical traversal. A previously discovered root or edge endpoint may nevertheless recur to make a page understandable; list its ID in `context_only_node_ids`. Such entries provide context and are not new discoveries. Every returned edge must have its endpoints or explicit endpoint stubs on that page. Context entries count against the page's node limit. The client deduplicates entities by ID while preserving new discovery order; pagination progress and counters ignore repeated context.

Bound total work to 10,000 scanned edges and two seconds per request; these are design limits, not measured performance claims. Use indexed adjacency lookups and cancellation. Pagination cursors bind the immutable dataset, query parameters, traversal position, and limits; sign them and reject mismatches. A continuation must make progress without presenting an unbounded closure. When the traversal work budget is exhausted, return an explicit truncated result without an unlimited continuation. The UI requires a narrower root or depth.

Return 404 for unknown IDs, 409 for unavailable datasets or cursor/version mismatch, 422 for malformed/oversized input, and 503 for an unavailable index. Errors use `{error:{code,message,request_id}}` without filesystem paths or stack traces.

## 8. User interface and storage

The landing page displays the dataset’s kind, scope, provenance, and review status before its graph. Make the synthetic label persistent. The RH metadata page links to its source and endorsement and states that no RH dependency graph has been indexed locally.

Use a search/sidebar, bounded graph, and detail panel. Provide keyboard-accessible tabular dependency navigation with the same content as the graph. Differentiate edge types with labels and line patterns as well as color. Include “uses” and “used by” controls, depth, library filtering, a visible expansion budget, and an explanation of truncation. Never make filtered-out nodes disappear without showing the active filter.

Source links must use pinned revisions. Render statements and source as escaped text; sanitize Markdown explanations with raw HTML disabled. Disallow executable URI schemes. Do not fetch arbitrary source URLs from the server. Large proof text is opt-in and bounded; never place a complete proof term in the graph payload.

SQLite tables cover datasets, entities, declarations, typed edges, source files, explanations, reviews, verification reports, and FTS records. Index edges on `(dataset_id,from_id,kind,to_id)` and the reverse. Stream JSONL ingestion in transactions; avoid loading all declarations into Python memory. Build into a temporary database, check referential integrity and hashes, then atomically replace the dataset pointer. The reader holds an immutable snapshot for each request. Corrupt imports leave the prior dataset available.

## 9. Extraction isolation and cost policy

The default fixture worker has no network, no credentials, no home-directory mount, and no host container socket. Run as an unprivileged user with a read-only root filesystem, a read-only source mount, and a dedicated writable output directory. Set CPU, memory, PID, output-size, and wall-time limits. Use a rootless container or an equivalent documented isolation mechanism. A subprocess timeout alone is not a security boundary.

Lean elaboration and project build configuration can execute code. Therefore, never import or compile user-supplied Lean inside the web server, notebook service, or privileged host process. Fetch necessary dependencies in a separate explicit bootstrap step, pin them, and run subsequent compilation offline. Execute fixed argument arrays without a shell; do not interpolate project text into commands.

For optional source archives, hash before extraction, enforce expected size/count limits, and unpack into a fresh private directory. Reject absolute paths, `..` escapes, symlinks, hard links, device files, duplicate normalized names, and entries escaping the target. Enforce total expanded bytes while streaming; do not trust archive headers alone. Use separate explicit budgets for the small fixture and a large-proof profile. Never auto-raise a failed limit.

Initial fixture limits: 2 CPU cores, 4 GiB memory, 128 processes, 2 GiB writable output, and a 15-minute timeout after dependencies are prepared. Record actual measurements and adjust only for a documented reason. Large-proof runs require an explicit CLI flag and an operator-supplied resource budget; no normal test or CI command activates them. Preflight free disk and show the planned limits before starting. Do not claim 100 GB is a safe upper bound merely because an upstream report mentions approximately that amount.

Cache extraction by source/toolchain/extractor hashes. Limit stored logs, remove abandoned temporary directories, and support resumable ingestion without merging different source snapshots. No per-request model inference, full-graph force layout, or automatic cloud provisioning belongs in this MVP.

## 10. Fixtures and acceptance tests

Create a small original Lean package proving elementary arithmetic facts. Include a theorem referencing another fixture theorem, a definition appearing in a statement, an unused module import, and an external standard-library reference. Pin a supported Lean version and record genuine extracted output. Select examples whose expected dependencies can be checked manually. Do not use `sorry` in the positive fixture.

Create a separate generated JSONL fixture with fan-out, long paths, disconnected nodes, duplicate names in different modules, Unicode, unresolved stubs, and deliberately malformed variants. It must declare `kind="synthetic_fixture"` and contain no purported RH proof relationships.

Implement and document these executable gates:

```bash
uv sync --frozen
uv run pytest -q
npm --prefix web ci
npm --prefix web run build
npm --prefix web run test:e2e
uv run proof-atlas fixture extract --offline
uv run proof-atlas fixture verify
uv run proof-atlas serve --host 127.0.0.1
```

The first installation may need dependency access; the extraction/test gate must use preinstalled pinned dependencies. Provide an offline setup description rather than promising an offline package download.

Tests must cover:

- Exact fixture type/proof edges, exclusion of unused-import theorem edges, and preservation of both edge kinds for a shared target.
- Dataset/hash determinism, identity separation across versions, malformed imports, partial coverage, and accurate unresolved stubs.
- Axiom `null` versus empty-list behavior and an isolated negative fixture containing an explicitly declared extra axiom.
- Pagination without omitted or duplicate logical discoveries/result edges, correctly labelled repeated root/endpoint context, tampered cursors, dataset mismatch, high fan-out, depth zero, cancellation, and deterministic truncation.
- Explanation staleness across snapshot IDs after proof or statement changes; unchanged-content review reuse with an explicit record; renamed declarations requiring an explicit mapping; no automatic promotion of AI drafts to reviewed.
- Archive traversal/link/bomb cases, worker timeout cleanup, disabled network, and failure to access an intentionally absent credential file.
- Escaped Lean/Markdown content, rejected unsafe source URLs, keyboard navigation, persistent synthetic labels, and visible unavailable-RH state.
- Server launch and all read APIs succeeding in a runtime image with no Lean executable and no worker credentials.

For performance, generate a documented synthetic load profile and report hardware, record count, p95 query latency, and peak ingestion memory. Target subsecond ordinary fixture queries; treat larger-scale targets as measurements to establish, not acceptance claims invented without running them.

## 11. Milestones and repository layout

Deliver in this order: contracts and fixtures; isolated real extraction; validated indexing; bounded API; UI and review records; external RH metadata profile; final verification and documentation. Each milestone should leave a runnable local state. Stop adding features when the acceptance gates pass.

| Path | Responsibility |
|---|---|
| `src/proof_atlas/models/` | Versioned manifest, graph, review, and API contracts. |
| `src/proof_atlas/adapters/` | Pinned extractor compatibility and normalization. |
| `src/proof_atlas/index/` | Streaming ingestion, FTS, adjacency, snapshot handling. |
| `src/proof_atlas/api/` | Read-only routes and bounded traversal. |
| `src/proof_atlas/cli.py` | Fixture, import, review-record validation, and serve commands. |
| `workers/` | Offline container definition and fixed entrypoint. |
| `fixtures/lean/` | Original licensed Lean package and extraction oracle. |
| `fixtures/synthetic/` | Explicitly synthetic datasets and adversarial variants. |
| `profiles/rh/` | External-source metadata and large-run instructions; no proof source. |
| `web/` | React UI, accessible graph/table views, end-to-end tests. |
| `tests/` | Contract, extraction, traversal, isolation, and review tests. |
| `docs/` | Architecture, provenance, resource measurements, and operator guide. |

Future features may include reviewed mathematical tours, comparisons between proof versions, community annotations with authentication, semantic search, read-only MCP tools, or Verso export. These follow a reliable provenance and extraction foundation. They must not hide uncertainty about coverage, licensing, or explanations.

## 12. Final implementation report

Report what runs locally, the commands used, which extraction was actually performed, the supported toolchain, measured resource usage, and test results. Include screenshots of the real fixture, synthetic label, truncated graph, and RH metadata-only state. Identify any external source whose terms or toolchain prevented indexing. Supply the complete local repository and instructions; publishing and full RH ingestion are separate future actions.
