# SuffBench

**Status: specification only. No instances are released yet.** Version 0.1.0-spec, October 2026. DOI [10.5281/zenodo.23282863](https://doi.org/10.5281/zenodo.23282863).

SuffBench is a benchmark format for *context sufficiency* in code editing: for each real code-change
task it records not only the task, the accepted fix, and the tests that decide whether a fix works,
but also a **verified oracle slice**, the set of repository units whose contents determine that
verdict, together with a **frame set** of tests that must keep passing. With those labels, any
context-selection method can be scored on coverage and cost offline, without running an editor.

The definitions behind the format are in the note
*ε-Sufficient Context for Bounded Readers: Definitions and a Non-Monotonicity Result for Code Editing*,
concept DOI [10.5281/zenodo.23282195](https://doi.org/10.5281/zenodo.23282195) (version 1.1: 10.5281/zenodo.23282824). This repository is the
normative specification of the instance format, the slice definitions, the datasheet, and the
command-line contract of the planned context compiler.

| Document | Purpose |
|---|---|
| [SPEC.md](SPEC.md) | Instance format, slice definitions, frame sets, provenance, licensing |
| [schema/instance.schema.json](schema/instance.schema.json) | Machine-checkable JSON Schema for one instance |
| [DATASHEET-template.md](DATASHEET-template.md) | Datasheet every release must ship |
| [tools/CONTEXT-COMPILER.md](tools/CONTEXT-COMPILER.md) | Command-line contract of the checker and context compiler (planned) |
| [CITATION.cff](CITATION.cff) | How to cite the specification |

## What SuffBench labels, and what it does not

SuffBench labels **verification relevance**: what determines the test verdict on the gold patch.
It does not label what an editor needs in order to *write* the patch; that is **generation
sufficiency**, which is relative to the editor and is measured by experiments, not by a dataset.
Releases must state this distinction in their datasheet.

## Relation to existing benchmarks

SWE-bench and Multi-SWE-bench provide fail-to-pass and pass-to-pass test sets; SuffBench reuses
that structure and adds verdict-based slices. Agent Retrieval Bench and ContextBench label
context relevance from agent workflow signals or from the files a gold patch touches; SuffBench
labels it from test verdicts under stubbing and mutation. See SPEC.md §7.

## Licence

Specification text and future data: CC-BY-4.0. Schema and tooling: Apache-2.0.

Published by Shockware Limited. Maintainer: Jude Castiel Johansen (ORCID 0009-0002-5644-3073).

## Pilot measurement (v0.1.1-pilot, October 2026)

A 24-instance pilot on cli/cli with three Claude models (2,360 test-verified verdicts) is in [`pilot/`](pilot/): paper, instances, mining log, harness and all verdict records. DOI [10.5281/zenodo.23307204](https://doi.org/10.5281/zenodo.23307204). Exploratory; it fixes the design of the preregistered study and makes no confirmatory claim.

## Reference harness

The pilot harness in [`pilot/harness/`](pilot/harness/) is a working reference implementation of the context compiler for Go modules: `pilot_mine.py` (instance mining with logged rejections), `pilot_contexts.py` (arms W, W+iface, W+bodies, W+iface+F, and token-targeted padding via the vendor token counter), `pilot_run.py` (single-shot editor protocol, hidden-test and dependents-scoped frame verification, verdict cache), `analysis_pilot.py`. The command-line contract in [tools/CONTEXT-COMPILER.md](tools/CONTEXT-COMPILER.md) remains the target interface; the reference scripts do not yet implement it verbatim.

## Preregistered confirmatory study

Stage 1 protocol (Registered Report format, preregistration with fixed margins, tests, instance rules and repository list): DOI [10.5281/zenodo.23307547](https://doi.org/10.5281/zenodo.23307547). No confirmatory data have been collected.

Exploratory ablations run after the pilot (Phases 1 to 3, October 2026): DOI [10.5281/zenodo.23460566](https://doi.org/10.5281/zenodo.23460566). On eleven screened instances from cli/cli and caddy, inert padding lowered Haiku 5.5 pass rates where real dependency interfaces did not, a declaration-level slice of the write-set met the non-inferiority margin only under a post-hoc sample-set choice at about half the tokens, and single-dependency necessity was non-additive on the one dependency-sensitive instance. Exploratory; no confirmatory claim. Harness and records: [suffbench-registered-report](https://github.com/Shockware-Limited/suffbench-registered-report).
