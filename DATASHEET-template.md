# Datasheet for a SuffBench release

Every data release MUST ship this document completed. Sections follow the structure of
"Datasheets for Datasets" (Gebru et al.), adapted to this format.

## Motivation
- For what purpose was the dataset created? Which gap does it fill relative to SWE-bench,
  Multi-SWE-bench, Agent Retrieval Bench, and ContextBench?
- Who created it, and who funded or hosted it?

## Composition
- Repositories, languages, granularity, number of instances per repository.
- Date range of `created_at`; per-model cutoff flags and how many instances predate each cutoff.
- Task text sources (issue / pr_body_stripped / rewritten_spec) and counts.
- Size distributions: tokens of W, B(W), F(W), coverage slice, mutation slice.
- Which slices are present (coverage MUST be; mutation and stubbing optional) and their parameters.
- Known violations of the closure check, with counts.

## Collection process
- Mining recipe, filters, and attrition at each step (candidates → compiling tests → fail-to-pass → green suite → screened).
- Manual screening protocol and who performed it.
- Flakiness protocol and number of excluded tests.
- Tool versions, platform, runner image.

## Preprocessing
- How task text was stripped or rewritten; what was removed; checks performed.
- How interfaces were extracted for named contexts; serialisation order; position of W.

## Uses
- Verification relevance is labelled; generation sufficiency is not. State this.
- Intended uses: scoring context selectors on coverage and cost; estimating degradation curves at fixed coverage.
- Uses to avoid: ranking editors by pass rate alone; treating slices as what an editor "needs".

## Distribution and maintenance
- Hosting, DOI, licence, upstream licences, how to report errors, versioning policy.
