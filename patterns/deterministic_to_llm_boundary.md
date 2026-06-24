# Deterministic-to-LLM Boundary Pattern

This document describes the central boundary in the framework.

It is provisional and should be refined through real package use.

## Problem

LLMs are useful for discussing complex evidence, but they are unreliable when asked to infer:

- file structure
- metric semantics
- collection methodology
- comparability rules
- missing-value meaning
- causal relationships
- authority hierarchy
- package health

from raw files alone.

A deterministic evidence package should therefore make the evidence navigable and bounded before the LLM reasons about it.

## Boundary principle

```text
Deterministic systems make evidence trustworthy.
LLMs make evidence discussable.
```

## Deterministic responsibilities

The deterministic layer should own:

- data collection
- file generation
- raw evidence preservation
- identifiers
- timestamps
- metric computation
- rollups
- flags
- package health
- evidence inventory
- schema/version metadata
- known collection failures
- deterministic comparability checks where possible

The deterministic layer may produce summaries, but summaries should remain traceable to source artifacts.

## LLM responsibilities

The LLM should own:

- conversational explanation
- user-specific framing
- hypothesis generation
- identifying plausible interpretations
- identifying missing evidence
- routing the user to relevant drill-down artifacts
- generating targeted extraction scripts from recipes
- translating evidence into decision-support language

The LLM should not own:

- raw fact creation
- hidden assumptions
- silent schema inference
- causal claims without evidence
- treating missing data as zero
- deciding authority when deterministic sources conflict unless rules are provided

## Artifact roles

### Evidence archive

The complete set of deterministic outputs.

This is not normally loaded into chat.

### Evidence map

A deterministic navigation layer describing what exists and where to look.

### Interpretation guide

Package-specific semantics explaining how signals should and should not be understood.

### Question routing

Curated or semi-curated guide to recurring question types and relevant artifacts.

### Script recipes

Reusable extraction procedures for targeted answers.

### Extractors

Promoted deterministic scripts that implement stable recipes.

## Authority pattern

Not all artifacts should have equal authority.

Candidate authority levels:

```text
Authoritative deterministic evidence
Controlled interpretation
Convenience summary
User-provided note
LLM-generated provisional interpretation
```

Example rule:

```text
If a generated summary conflicts with a manifest or deterministic CSV, the deterministic artifact wins.
```

## Missing-value pattern

Missing values should be explicit where possible.

Avoid collapsing:

```text
missing
not collected
not applicable
collected but zero
collected but failed
redacted
unknown
```

into a single blank field.

## Comparability pattern

Comparisons should be gated.

A package should define what makes records comparable, weakly comparable, or not comparable.

The LLM should not be allowed to casually compare runs or files when required identity or collection fields differ.

## Context-budget pattern

The full evidence archive should not be placed in chat by default.

Default chat payload should usually include:

- framework reasoning guidance
- package-specific interpretation guide
- package manifest
- package health summary
- evidence map
- rollup/flags
- user question

Drill-down files should be pulled only when the question requires them.

## Confidence language

The LLM should classify conclusions using language such as:

- observed
- directly supported
- consistent with
- plausible
- weakly supported
- contradicted
- unknown
- not enough evidence

Prefer:

```text
The evidence is consistent with cloud/cache involvement.
```

over:

```text
Desktop Connector caused the slow open.
```

unless causation is directly supported.

## Known bad inference pattern

Each package should eventually include known bad inferences.

Examples:

- Do not infer causation from event overlap.
- Do not treat a declared scenario as verified.
- Do not treat a missing file as evidence of no activity.
- Do not compare runs unless comparability rules pass.
- Do not treat low similarity as governance failure without authority context.
- Do not treat high activity as wasted effort without project context.

## Promotion pattern

Repeated LLM work should be promoted.

```text
Ad hoc LLM answer
→ route
→ recipe
→ extractor
→ rollup metric
```

If the LLM is repeatedly generating the same script, the system likely needs a deterministic extractor.
