# SuffBench specification, version 0.1.0-spec

Normative words: MUST, SHOULD, MAY.

## 1. Units and graphs

A corpus is a repository snapshot at a commit. Its **units** are the elements of one declared
granularity: `package`, `file`, or `symbol`. A release MUST state its granularity. The **dependency
graph** G = (U, E) MUST be produced by a named tool at a named version, and the edge types included
(imports, calls, test-imports) MUST be listed. For Go at package granularity the reference graph is
`go list -json -deps -test ./...` at the base commit.

## 2. Instance

One instance is one real code change. Required fields (see `schema/instance.schema.json`):

| Field | Meaning |
|---|---|
| `instance_id` | `<org>__<repo>-<base_commit[:12]>` |
| `repo`, `base_commit`, `gold_commit` | the parent and the accepted change |
| `created_at` | commit date of `gold_commit`, ISO-8601, used for cutoff filtering |
| `task_text`, `task_text_source` | the natural-language task; source is `issue`, `pr_body_stripped`, or `rewritten_spec` |
| `gold_patch` | unified diff from base to gold, non-test files only |
| `test_patch` | unified diff of test files |
| `write_set` | units changed by `gold_patch` (W) |
| `interface_write_set` | units whose exported interface changes (W_I), by a named API-diff tool |
| `fail_to_pass` | tests that compile at base with `test_patch` applied, fail at base, pass at gold |
| `pass_to_pass` | tests passing at both base (with `test_patch`) and gold, the frame set |
| `excluded_tests` | tests excluded for flakiness, with the clean-run counts |
| `backward_slice`, `forward_slice` | B(W) and F(W) from G, transitive; F includes test-only importers |
| `coverage_slice` | units executed by `fail_to_pass ∪ pass_to_pass` on the gold tree (§3.1) |
| `mutation_slice` | units in which some mutant flips a verdict (§3.2), with the operator list and sample size |
| `closure_check` | whether W ∪ B ∪ F covers the import closure of the executed test binaries (§3.3) |
| `expressibility` | per named context, whether every identifier the gold patch references is present (§4) |
| `new_imports` | packages the gold patch imports that are not in deps(W) at base |
| `tokens` | token counts per unit for a named public tokenizer |
| `provenance` | tool versions, platform, runner image, dates |

## 3. Slices

### 3.1 Coverage slice
Run `fail_to_pass ∪ pass_to_pass` on the gold tree with per-unit coverage enabled
(`go test -coverpkg=./... -coverprofile`). A unit is in the coverage slice iff any statement in it
executes. This is cheap, deterministic up to flakiness, and MUST be present.

### 3.2 Mutation slice
For each unit in the coverage slice, inject k body-only, interface-preserving mutants (operators
MUST be listed) and run the verdict tests. A unit is in the mutation slice iff at least one mutant
changes the verdict of some test in `fail_to_pass ∪ pass_to_pass`. Report k and the number of
compiling mutants per unit. This approximates semantic reliance; coverage approximates execution.

### 3.3 Closure check
Compute the import closure of the test binaries that run the verdict tests. Record whether it is
contained in W ∪ B(W) ∪ F(W). Units in the closure but outside that set MUST be listed; they are
the measured violations of the dependency relation's soundness at this granularity.

### 3.4 Stubbing slice (optional)
If a release computes an observation-based slice by stubbing and delta debugging, it MUST state
the stub semantics (panic, zero-value, or specification-satisfying) and the search domain. Panic
stubbing over the full closure reduces to the coverage slice; releases SHOULD say so.

## 4. Named contexts and expressibility

A release MAY ship named contexts per instance (e.g. `W`, `W+iface(deps)`, `W+bodies(deps)`,
`transitive`). For each, it MUST record `expressibility`: whether every identifier, declaration,
and import the gold patch references appears in the context. Contexts MUST be serialised
deterministically (stated file order, stated position of W).

## 5. Cutoffs and leakage

Each release MUST state, per evaluated model, the vendor-documented training cutoff, and MUST flag
instances created before any such cutoff. Task text from PR bodies MUST have patch-describing
content removed and the source recorded; releases SHOULD prefer issue text or a rewritten
behaviour-only specification with a manual check.

## 6. Flakiness

Every test that fails on any run MUST be rerun at least three times on the clean tree. Tests failing
on the clean tree are moved to `excluded_tests` with counts.

## 7. Relation to other formats

`fail_to_pass`/`pass_to_pass` follow SWE-bench (arXiv:2310.06770) and Multi-SWE-bench
(arXiv:2504.02605). Relevance labels in Agent Retrieval Bench (arXiv:2607.24882) and ContextBench
(arXiv:2602.05892) derive from workflow signals or gold-patch files; SuffBench's slices derive from
test verdicts (§3). Editor-side minimal-context search (arXiv:2603.28119) is complementary and MAY
be recorded as a named context.

## 8. Licensing and attribution

Data: CC-BY-4.0. Schema and tools: Apache-2.0. Each instance MUST carry the upstream repository's
licence identifier and commit URLs. Releases MUST ship a completed `DATASHEET-template.md`.

## 9. Versioning

Specification versions are tagged `vMAJOR.MINOR.PATCH-spec`. Data releases are tagged
`vMAJOR.MINOR.PATCH` and state the spec version they conform to. Breaking changes to required
fields bump MAJOR.
