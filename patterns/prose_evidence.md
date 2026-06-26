# Prose Evidence Pattern

This document describes how to treat evidence that exists primarily as prose —
narrative documents, design rationale, meeting records, qualitative assessments,
specifications in natural language.

It is provisional and should be refined through real package use.

## Problem

The framework's deterministic layer handles structured outputs cleanly. Prose evidence
challenges the boundary because the LLM is effectively the extraction layer — there is
no CSV exporter for design intent, qualitative reasoning, or narrative explanation.

The boundary principle still applies: the LLM should not reason about evidence that
lacks explicit epistemic provenance. For prose, provenance must be earned through
structured extraction and review rather than produced by construction.

See `patterns/deterministic_to_llm_boundary.md` for the epistemic provenance framework
that this pattern builds on.

## Two categories of prose evidence

### Extractable prose

Facts expressed in natural language that could in principle be structured. Decisions
made, dates, parties involved, options considered, outcomes stated.

The evidence exists in prose form but the underlying commitment is discrete and
separable from its surrounding language.

Approach: promoted extraction artifact with stored provenance and source span anchor.
The extraction step introduces inference, but that inference can be explicitly labeled,
reviewed, and promoted.

### Semantic prose

Meaning irreducibly embedded in language. Design rationale, qualitative risk assessment,
the reasoning behind a decision, the intent behind a specification.

Forcing extraction into structured fields either loses the evidence or misrepresents it
by implying more precision than exists.

Approach: the interpretation guide does heavier work. Constrain how the LLM reads the
document rather than converting it to structured form. Accept that some questions about
semantic prose can only be answered with lower confidence than equivalent questions
about structured data — and make that explicit rather than leaving the LLM to treat
both as equivalent.

## Semantic commitment types

When extracting from extractable prose, each extracted item should be typed:

```text
decision              — a resolved choice
constraint            — a stated limit or requirement
consideration         — something weighed but not decided
rejected_alternative  — an option explicitly ruled out
assumption            — something treated as true without explicit support
open_question         — something explicitly unresolved
```

Commitment type determines expected authority level and appropriate confidence language.

- A `decision` carries higher authority than a `consideration`.
- A `rejected_alternative` answers "why not" questions, not "what was decided"
  questions — a different kind of authority, not a weaker one.
- An `assumption` that goes unstated in the source document cannot be promoted above
  provisional authority without explicit review.

## Extraction fidelity

Fidelity is distinct from provenance. Provenance tracks origin and review state.
Fidelity tracks how faithfully an extracted item represents its source span.

```text
direct_quote          — verbatim from source
faithful_paraphrase   — rewording that preserves meaning without distortion
cross_span_inference  — drawn across multiple passages; introduces more inference
implied_reading       — inferred from what the document implies but does not state
```

Fidelity must be labeled explicitly on each extracted commitment. High provenance
confidence (human-reviewed) does not compensate for low fidelity (implied reading).
Both dimensions travel with the commitment.

## Promoted extraction artifact

An extracted prose commitment should carry:

```yaml
commitment_type:      decision | constraint | consideration |
                      rejected_alternative | assumption | open_question
content:              ...
source_document:      ...
source_span:          [section, paragraph, or sentence reference]
extraction_fidelity:  direct_quote | faithful_paraphrase |
                      cross_span_inference | implied_reading
provenance_status:    observed | inferred | user_confirmed
authority_level:      authoritative | controlled_interpretation |
                      provisional | llm_generated
extracted_by:         [model, prompt_version, timestamp]
reviewed_by:          [reviewer, date] or null
```

The extraction prompt, model version, and timestamp are stored alongside the output.
This makes the extraction reproducible even when not deterministic in the strict sense.

The source span reference is the critical field. Without it, the extraction is not
anchored — a later reviewer cannot verify it, and the LLM reasoning from it cannot
distinguish a direct quote from an implied reading.

## Promotion path for prose evidence

Prose evidence sits below the epistemic provenance boundary until it has been extracted
and reviewed. The path across the boundary:

```text
raw prose document
  (below the boundary — no explicit provenance)

LLM extraction, unreviewed
  (below the boundary — but explicitly labeled as such)

LLM extraction + source span anchor
  (approaching the boundary — fidelity is now traceable)

human-reviewed extraction artifact
  (above the boundary — provisional authority)

stable extraction with confirmed authority level
  (above the boundary — controlled interpretation)

deterministic extractor if extraction pattern stabilizes
  (above the boundary — authoritative)
```

When extraction logic becomes stable — same commitment types, same document structure,
same fields reliably present across instances — it can be promoted to a deterministic
extractor. At that point, prose evidence becomes structurally equivalent to CSV evidence
for the purposes of the boundary.

## Honest ceiling for semantic prose

For prose where stabilization is not possible, the promotion path tops out at
human-reviewed extraction with explicit authority limits. The interpretation guide then
carries the constraining work that the structured package does for structured data.

This is not a failure state. It is the correct epistemic position for evidence that
cannot be fully reduced to structured form.

## Interpretation guide responsibilities for prose packages

For extractable prose, the interpretation guide describes how extracted artifacts should
be read and compared — the same role it plays for structured data.

For semantic prose, the interpretation guide must also encode:

**Rhetorical conventions** — what hedging language signals in this document type.
"We considered X" is different from "we decided X" is different from "X was the
preferred option." The LLM needs to be told how to read these signals, not left to
infer them from general training.

**Absence semantics** — what it means that a topic was not mentioned. In a formal
specification, absence may mean out of scope. In meeting notes, it may mean assumed,
tabled, or simply not raised. In a design rationale document, absence may mean the
alternative was too obviously wrong to record or too contested to resolve. These are
different and cannot be collapsed.

**Document authority** — who wrote this, for what audience, under what constraints. A
whiteboard capture and a signed-off requirements document are both prose but carry
entirely different epistemic weight.

**Cannot-answer entries** — what this document cannot tell you regardless of what it
says. A design rationale document can tell you why a decision was made but probably
cannot tell you whether that decision was implemented as described.

## Automated fidelity scoring

Vector similarity between the source span embedding and the extraction embedding is a
candidate mechanism for prioritizing human review of extracted commitments.

The signal measures semantic overlap — whether the same concepts and relationships are
present in both the source and the extraction. This is useful for detecting gross
faithfulness failures and for sorting the review queue, but it is not a substitute for
human review.

### What similarity scoring catches

- Extractions that introduce concepts not present in the source span
- Extractions that lose key content from the source span
- Cross-span inferences that diverge significantly from any individual span
- Systematic drift in a document type (outlier detection across a corpus)

### What similarity scoring misses

**Commitment type misclassification** is the highest-stakes failure and the least
visible to vector similarity. "We considered X" → "We decided X" scores high similarity
because the concepts and entities are identical. The fidelity score will not flag this.
Commitment type review cannot be delegated to similarity scoring.

**Negation and modality** are poorly captured by standard sentence embeddings. "We will
not use X" and "We will use X" have high cosine similarity. Extractions involving
negated constraints or conditional decisions should be flagged for mandatory review
regardless of similarity score.

### Integration with the extraction artifact

The extraction artifact carries two fidelity signals that should be evaluated together:

```yaml
extraction_fidelity:   direct_quote | faithful_paraphrase |    ← LLM self-report
                       cross_span_inference | implied_reading
fidelity_similarity:   0.0–1.0                                 ← vector score
fidelity_review_flag:  auto | low_similarity | type_mismatch | reviewed
```

Candidate thresholds (to be validated against real packages):

```text
similarity > 0.85 AND fidelity ∈ {direct_quote, faithful_paraphrase}
  → auto  (eligible for spot review, no immediate block)

similarity 0.6–0.85 OR fidelity = cross_span_inference
  → low_similarity  (review candidate)

similarity < 0.6
  → low_similarity  (review required before use above provisional authority)

LLM reports faithful_paraphrase AND similarity < 0.70
  → type_mismatch  (LLM self-report and vector signal disagree — higher priority)
```

Divergence between LLM self-report and vector similarity is a stronger flag than either
signal alone. Agreement is weak confirmation; disagreement is a hard review trigger.

### Scope of the mechanism

Similarity scoring makes the human review step cheaper by sorting the queue. It does
not make the review step smaller. Extractions that score well can be reviewed more
quickly; extractions that diverge are prioritized. The human review gate remains the
authority boundary for all extractions above provisional level.

## Relationship to the capture system

The capture system for a prose-heavy package should track:

- which documents have been through extraction review
- which commitment types recur across documents of the same type
- which extraction patterns are stable enough to become recipes
- which absence-semantics rules were invoked in answering a question
- which questions could not be answered because the relevant document
  had not been extracted yet

Repeated inability to answer a question due to unextracted prose is a signal to
prioritize that document for extraction review — the same way repeated analysis
blockers in structured packages become package health check candidates.
