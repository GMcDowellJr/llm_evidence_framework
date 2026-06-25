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

The technology boundary — deterministic vs LLM — is a proxy for an underlying property:
**explicit epistemic provenance**.

Deterministic outputs carry epistemic provenance by construction. The full production
history of every field is known, traceable, and reproducible. Prose evidence must acquire
epistemic provenance through structured extraction and review.

The boundary restated:

```text
Before the LLM reasons about evidence, every piece of evidence must carry
explicit epistemic provenance. Deterministic outputs have this by default.
Prose evidence must earn it.
```

### Components of epistemic provenance

Every evidence item — whether a CSV field or an extracted prose commitment — should carry
four things before it crosses the boundary:

**Origin** — how was this produced?

```text
deterministic computation
LLM extraction
human observation
model inference
```

**Fidelity** — how faithfully does this item represent its source?

```text
exact              (deterministic data — fidelity is exact by definition)
faithful_paraphrase
cross_span_inference
implied_reading
```

For deterministic data, fidelity is exact by construction. For prose extractions, fidelity
must be labeled explicitly. A human-reviewed extraction (high provenance) with an implied
reading (low fidelity) is still a weak evidence item.

**Authority** — what epistemic weight can this carry in downstream reasoning?

Shaped by commitment type and review outcome. The existing authority levels apply:

```text
Authoritative deterministic evidence
Controlled interpretation
Convenience summary
User-provided note
LLM-generated provisional interpretation
```

**Limits** — what can this item not answer?

Including absence semantics: what does it mean that a topic was not mentioned in this
document or artifact? Limits should be explicit, not left to the LLM to infer.

### Prose evidence and the boundary

For structured data, crossing the boundary is automatic — the deterministic extractor runs
and provenance is established. For prose evidence, crossing the boundary requires a
deliberate path:

```text
raw prose document
  → LLM extraction with source span anchor (approaching boundary)
  → human-reviewed extraction artifact    (above boundary, provisional authority)
  → stable extraction with confirmed authority level (controlled interpretation)
  → deterministic extractor if extraction pattern stabilizes (authoritative)
```

Until that sequence completes, the prose document sits below the boundary. The
interpretation guide constrains how the LLM reads it, but the LLM is operating on
evidence without explicit provenance markers. That is an honest acknowledgment of the
epistemic state, not a framework failure.

See `patterns/prose_evidence.md` for the full prose evidence pattern.

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

The deterministic layer may produce summaries, but summaries should remain traceable to
source artifacts.

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

For prose-heavy packages, the interpretation guide also encodes rhetorical conventions,
absence semantics, and document authority levels before the LLM reasons from the prose.

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
If a generated summary conflicts with a manifest or deterministic CSV, the
deterministic artifact wins.
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

A package should define what makes records comparable, weakly comparable, or not
comparable.

The LLM should not be allowed to casually compare runs or files when required identity or
collection fields differ.

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

If the LLM is repeatedly generating the same script, the system likely needs a
deterministic extractor.
