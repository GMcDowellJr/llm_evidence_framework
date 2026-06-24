# Package Intake Questions

This document is a discovery scaffold. It is intended to help inspect a new deterministic evidence package before creating a codified evidence map, question routes, or script recipes.

It is not a finalized package standard.

## 1. Package identity

- What system produced this package?
- What is the package intended to help analyze?
- Who is the expected user?
- What decisions or conversations should this package support?
- What should this package not be used to decide?
- What package version or schema version exists?
- What tool version produced the package?
- When was it generated?
- What source data or run folder was used?

## 2. Evidence inventory

For each artifact:

- What is the file name?
- What is the file path?
- What type of artifact is it?
  - summary
  - rollup
  - manifest
  - raw log
  - drill-down file
  - generated narrative
  - LLM context file
  - configuration
  - health check
- Is the artifact deterministic?
- Is the artifact user-authored?
- Is the artifact LLM-authored?
- Is the artifact required or optional?
- Is the artifact expected to exist in every package?

## 3. Artifact purpose

For each artifact:

- What question was this file created to help answer?
- Is it intended for humans, machines, or both?
- Is it an authoritative source or a convenience summary?
- What is the grain?
  - one row per run
  - one row per event
  - one row per sample
  - one row per file
  - one object per package
  - one object per artifact
- What are the key fields?
- What fields are identifiers?
- What fields are labels?
- What fields are measurements?
- What fields are derived?

## 4. Authority and provenance

- Which files are authoritative when conflicts occur?
- Which files are summaries of other files?
- Which fields are observed directly?
- Which fields are computed deterministically?
- Which fields are inferred by deterministic rules?
- Which fields are manually entered?
- Which fields are LLM-generated?
- Are user notes separated from system observations?
- Are generated summaries traceable to source artifacts?

## 5. Null and missing data

- Which fields may be blank?
- What does blank mean in each field?
- Is there a difference between zero and missing?
- Is there a difference between not collected and collected but zero?
- Are failed captures explicitly represented?
- Are redacted values explicitly marked?
- Are unavailable artifacts listed in package health?

## 6. Comparability

- What makes two records comparable?
- What makes two runs comparable?
- What makes two packages comparable?
- What fields are required for comparison?
- What fields can invalidate comparison?
- Are comparability warnings generated deterministically?
- Are there known weak-comparison scenarios?

## 7. Chat context

- Which files belong in default LLM context?
- Which files should be drill-down-only?
- Which files are too large for chat context?
- Which files are useful only for scripting?
- Does the package include a compact context bundle?
- Does the package include an evidence map?
- Does the package include package health?

## 8. Existing questions

- What questions has the user already asked of this package type?
- Which questions recur?
- Which questions required inspecting drill-down files?
- Which questions required generated scripts?
- Which questions revealed missing evidence?
- Which questions should become candidate routes?

## 9. Inference limits

- What claims does the evidence support directly?
- What claims are plausible but unproven?
- What claims are commonly tempting but unsupported?
- What would constitute stronger evidence?
- What additional data would be needed to support causation?
- What should the LLM explicitly avoid saying?

## 10. Promotion candidates

- Which repeated questions should become question routes?
- Which repeated extraction patterns should become script recipes?
- Which script recipes are stable enough to become deterministic extractors?
- Which derived fields should be added to future rollups?
