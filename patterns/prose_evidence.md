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
source_span_text:     [verbatim text of source span, stored for embedding]
extraction_fidelity:  direct_quote | faithful_paraphrase |    ← LLM self-report
                      cross_span_inference | implied_reading
fidelity_similarity:  0.0–1.0                                 ← vector score
fidelity_review_flag: auto | low_similarity | type_mismatch | reviewed
confidence_band:      auto-provisional | use-confirmed | human-reviewed
use_count:            N   ← LLM sessions that reasoned against this commitment
contradiction_count:  N   ← times flagged as wrong
similarity_mean:      0.0–1.0  ← running average across instances
similarity_variance:  0.0–1.0  ← stability of pattern across instances
extracted_by:         [model, prompt_version, timestamp]
reviewed_by:          [reviewer, date] or null
```

The extraction prompt, model version, and timestamp are stored alongside the output.
This makes the extraction reproducible even when not deterministic in the strict sense.

The source span reference is the critical field. Without it, the extraction is not
anchored — a later reviewer cannot verify it, and the LLM reasoning from it cannot
distinguish a direct quote from an implied reading.

## Promotion path for prose evidence

Prose evidence moves from below the epistemic provenance boundary toward higher
confidence through two mechanisms: use-driven accumulation and targeted human review.
Human review is not a front-end gate before use — it is a background process triggered
by consequence or anomaly.

```text
raw prose document
  (below the boundary — no explicit provenance)

LLM extraction + source span anchor
  (approaching the boundary — fidelity is traceable, use at auto-provisional)

auto-provisional
  (operating state — labeled explicitly, LLM surfaces this in answers)

use-confirmed
  (accumulated through use — earned, not reviewed; see confidence accumulation below)

human-reviewed
  (targeted intervention — triggered by consequence, not pipeline position)

deterministic extractor if extraction pattern stabilizes
  (above the boundary — authoritative; prose evidence becomes equivalent to CSV)
```

The LLM can reason against `auto-provisional` and `use-confirmed` commitments. It
surfaces the confidence band in its answer. The analyst decides whether that confidence
level is sufficient for the question at hand — which is the correct decision point.
A low-stakes exploratory question can run on `auto-provisional`. A conclusion that will
be acted on may warrant triggering human review for the specific commitments it depends
on.

When extraction logic becomes stable — same commitment types, same document structure,
same fields reliably present across instances — it can be promoted to a deterministic
extractor. At that point, prose evidence becomes structurally equivalent to CSV evidence
for the purposes of the boundary.

## Confidence accumulation

`use-confirmed` is earned through accumulated use signals, not through review.
The capture system tracks these signals in the background:

**Signals that raise confidence:**
- Repeated extraction of the same commitment from the same document type with
  stable similarity scores across instances
- LLM sessions that reason against the commitment without flagging anomalies
- Low similarity variance across instances (the extraction pattern is stable)
- Pattern promotion — the extraction template has been confirmed for this document type
- LLM self-report and vector similarity in agreement

**Signals that reset or flag confidence:**
- Contradiction — an analyst or the LLM flags the extraction as wrong
- High similarity variance across instances (the pattern is less stable than it appeared)
- Document version change — the source document has been updated; the extraction
  needs re-anchoring
- High negation or modality risk in commitment type — `constraint` and `rejected_alternative`
  commitments involving negated language should have a lower accumulation ceiling before
  triggering a soft review recommendation

The confidence accumulation model is analogous to OB's trust ladder: promotion through
accumulated use and absence of contradiction, not through deliberate review of every
instance.

## The review ceiling

`use-confirmed` confidence never automatically crosses into `human-reviewed`.

The reason is specific: accumulated use does not fix the failure modes that similarity
scoring misses. Commitment type misclassification — extracting "we decided X" from
"we considered X" — scores high similarity and will not be corrected by repeated use.
Twenty sessions reasoning against a misclassified commitment are twenty instances of
the same error propagating, not twenty confirmations of correctness.

Human review remains the authority boundary for high-stakes claims. It activates when:

- A conclusion the analyst is about to act on depends on a `use-confirmed` commitment
- An anomaly is flagged — contradiction signal, high similarity variance, version change
- A commitment type carries high negation or modality risk and has not been reviewed

Everything else operates at `use-confirmed` level. The review burden is proportional
to consequence, not to extraction volume.

## Honest ceiling for semantic prose

For prose where stabilization is not possible, the promotion path tops out at
`use-confirmed` with explicit authority limits. Human review may be triggered by
consequence but is not automatically required. The interpretation guide carries the
constraining work that the structured package does for structured data.

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

## Relationship to the capture system

The capture system is the confidence accumulation mechanism. It runs in the background,
not as a review queue. For a prose-heavy package it tracks:

- which documents have been through extraction
- use signals per commitment — sessions that reasoned against it, contradiction flags,
  similarity stability
- which commitments have accumulated enough use signals to move from `auto-provisional`
  to `use-confirmed`
- which extraction patterns are stable enough to become recipes or deterministic
  extractors
- which absence-semantics rules were invoked in answering a question
- which questions could not be answered because the relevant document had not been
  extracted yet
- which commitments have active contradiction flags or high similarity variance
  requiring targeted review

The output surfaced to the analyst is not a review queue. It is a confidence state:
which commitments recent analysis depended on, what their current confidence band is,
which have accumulated enough use to recommend pattern promotion, and which have
anomaly flags worth examining.

Repeated inability to answer a question due to unextracted prose is a signal to
prioritize that document for extraction — the same way repeated analysis blockers in
structured packages become package health check candidates.
