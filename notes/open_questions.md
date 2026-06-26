# Open Questions

This document tracks unresolved design questions. It is intentionally provisional.

## Framework scope

- Is this framework only for local evidence packages, or also for database-backed evidence stores?
- Should this repo define a general package contract, or only discovery scaffolds?
- What is the minimum viable evidence package?
- What belongs in the reusable framework versus the domain-specific package?

## File formats

- Should evidence maps eventually be markdown, YAML, JSON, or a combination?
- Should question routes be human-readable markdown or machine-readable records?
- Should script recipes be structured enough for automated generation?
- When should schemas be introduced?
- What information belongs in front matter versus body text?

## Authority and provenance

- How should authority levels be represented?
- What artifact wins when summaries and deterministic data disagree?
- How should user-declared, inferred, observed, computed, and LLM-derived fields be labeled?
  *Working answer: as origin + fidelity + authority + limits (the four components of
  epistemic provenance). See `patterns/deterministic_to_llm_boundary.md`.*
- Should every generated package have a package ID and hashes?
- How important are checksums at the first implementation stage?

## Missing and null values

- How many null states are needed?
- How should the package distinguish:
  - missing
  - not collected
  - not applicable
  - collected but zero
  - collected but failed
  - redacted
  - unknown
- Should null semantics be global or package-specific?

## Comparability

- What rules determine whether two runs, files, projects, or packages are comparable?
- Should comparability be computed deterministically?
- Should the LLM be allowed to make weak comparability judgments?
- What minimum metadata is required to compare two runs?

## Question routes

- What makes a question route reusable?
- How many times should a question recur before it is promoted?
- Should routes have confidence or maturity labels?
- Should routes include negative guidance, such as "do not use this file for this question"?
- Should question routes be package-specific only, or can some be generalized?

## Script recipes

- How detailed should a script recipe be before becoming useful?
- Should recipes include example output?
- Should recipes specify Python, PowerShell, SQL, or tool-neutral logic?
- Should recipes prefer pandas outputs, CSV exports, markdown tables, or JSON?
- When should a recipe become a deterministic extractor?

## Context budget

- What is the target maximum size for a default chat evidence package?
- Should the package explicitly mark files as default-context, optional-context, or drill-down-only?
- Should the package include a compiled LLM context file?
- How should historical packages be summarized without overloading context?

## Prose evidence

- When is a prose extraction pattern stable enough to become a deterministic extractor?
- Should extraction fidelity be recorded per-commitment or per-document?
- What is the minimum source span reference needed for traceability?
  (section / paragraph / sentence / character offset?)
- How should the interpretation guide encode absence semantics — as rules, examples, or both?
- Should semantic commitment types be standardized across package types or defined
  per package?
- How should conflicting extractions from the same document be handled?
  (two reviewers reading the same sentence as a decision vs. a consideration)
- Should the package health check flag documents that have not been through extraction
  review?
- Is there a practical threshold at which semantic prose should be declared out of scope
  for extraction and handled only through interpretation guide constraints?
- Can vector similarity between source span and extracted commitment be used as
  an automated fidelity scoring mechanism to prioritize human review?
  - What embedding model and similarity threshold is appropriate?
  - How should cross-span extractions be handled — compare against concatenated
    spans, the first span, or the full document?
  - Should the score be compared against LLM-reported fidelity as a cross-validation
    signal? (LLM reports faithful_paraphrase but similarity is low → harder review flag)
  - Can the OB dedup similarity infrastructure (cosine at 0.85) be reused here
    with different threshold semantics?
  - Is similarity scoring useful as an outlier detector across a corpus of same-type
    documents, independent of absolute score?
- What failure modes does similarity scoring miss?
  - Commitment type misclassification (decision vs. consideration) has high similarity
    and is the highest-stakes failure — how should this be compensated for?
  - Negation and modality are poorly captured by standard sentence embeddings —
    should commitment types with high negation risk be flagged for mandatory review
    regardless of similarity score?
    
## Testing

- What are the smallest useful test packages?
- Should test packages include known-bad examples?
- How do we test whether the LLM avoids overclaiming?
- How do we test context compilation?
- Should prompt behavior be tested manually or with automated regression prompts?
