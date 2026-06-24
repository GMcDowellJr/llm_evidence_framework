# Current Thesis

This document is a provisional statement of the framework thesis. It should evolve as evidence packages are tested.

## Core idea

A deterministic evidence package should not ask the LLM to discover the structure, semantics, or limits of the data from raw files alone.

Instead, the package should provide enough structure for the LLM to help the user reason from the evidence while keeping facts, interpretations, uncertainties, and unsupported claims separate.

## Division of responsibility

### Deterministic layer

The deterministic layer should produce:

- observed facts
- computed metrics
- stable identifiers
- schema/version metadata
- package health checks
- artifact inventories
- comparison fields
- flags and warnings
- drill-down files

It should avoid pretending to answer every possible future analytical question.

### Interpretation layer

The interpretation layer should explain:

- what each metric means
- how the data was collected
- what each signal can suggest
- what each signal cannot prove
- when comparisons are valid
- how missing values should be treated
- which artifacts are authoritative

### LLM conversation layer

The LLM should:

- help the user discuss what the evidence may suggest
- separate observed facts from plausible interpretations
- avoid unsupported assumptions
- identify contradictions
- state missing evidence
- propose targeted drill-downs
- generate extraction scripts when useful

The LLM should not:

- treat missing evidence as zero
- infer causation from overlap alone
- assume a user-declared scenario is verified
- treat summaries as more authoritative than deterministic artifacts
- silently compare non-comparable runs or packages

## Context-window strategy

The evidence archive can grow over time. The chat context should not grow proportionally.

The LLM should usually receive:

- reusable reasoning guidance
- package-specific interpretation guidance
- package manifest
- package health summary
- evidence map
- rollup/flags
- user question

Drill-down files should be retrieved or extracted only when needed.

## Evidence map as navigation layer

The evidence map is the key scalability mechanism. It allows the LLM to know where evidence lives without loading all evidence into context.

The evidence map should describe:

- file paths
- artifact purpose
- row/object grain
- key columns
- join keys
- question types the artifact can answer
- question types the artifact cannot answer
- known limitations
- authority level

## Question routes and script recipes

Question routes and script recipes are expected to grow through use.

A route or recipe should not be codified just because it was imagined. It should be promoted when it has been useful across repeated or clearly reusable questions.

## Promotion path

```text
One-off question
→ candidate route
→ reusable question route
→ script recipe
→ deterministic extractor
→ rollup field / report output
```

The best repeated LLM workflows should eventually become deterministic outputs.
