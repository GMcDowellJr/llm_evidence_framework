# Question Route Discovery

This document helps capture recurring analytical questions and the evidence needed to answer them.

A question route answers: Where should we look?

It does not necessarily define the extraction procedure in detail. That belongs in a script recipe.

## Purpose

LLM evidence packages become more useful when repeated user questions are captured and promoted.

A question route should help the LLM:

- classify the question type
- identify relevant artifacts
- avoid irrelevant artifacts
- know which fields matter
- understand whether the answer is direct or inferential
- recognize when more evidence is needed

## Candidate route scaffold

```markdown
## Question route: <route name>

Status:
- candidate / active / deprecated / promoted-to-recipe / promoted-to-extractor

Question forms:
- <example user phrasing>
- <example user phrasing>

Intent:
- <what the user is trying to understand>

Primary artifacts:
1. <artifact path> — <why it matters>
2. <artifact path> — <why it matters>

Secondary artifacts:
1. <artifact path> — <when needed>
2. <artifact path> — <when needed>

Relevant fields:
- <artifact>: <field>, <field>, <field>

Suggested first check:
- <where to start>

Evidence type:
- direct / indirect / mixed / unknown

Supported conclusion types:
- <what the evidence can support>

Unsupported conclusion types:
- <what the evidence cannot prove>

Comparability requirements:
- <conditions required before comparing>

Common traps:
- <bad inference>
- <bad inference>

Escalation:
- If <condition>, use script recipe <name>
- If <condition>, request/inspect <artifact>
```

## Questions to ask when capturing a route

- What did the user ask?
- Was this a one-off question or likely recurring?
- Which artifact was inspected first?
- Which artifact actually answered the question?
- Which fields mattered?
- Was the answer direct, indirect, or inconclusive?
- Did the route require joining files?
- Did the route require filtering by time, run ID, model identity, user, or scenario?
- Did the route reveal a missing field?
- Did the route reveal a package health issue?
- Did the route require a script?
- Should this route become a script recipe?

## Route maturity levels

```text
Level 0 — Ad hoc question
A question was answered once.

Level 1 — Candidate route
The question seems likely to recur, and the relevant artifacts are known.

Level 2 — Active question route
The route has been useful more than once or is clearly central to the package.

Level 3 — Recipe-backed route
The route requires a repeatable extraction procedure.

Level 4 — Extractor-backed route
The route is handled by a deterministic script/tool.
```

## Route categories

Potential route categories include:

- identity verification
- run comparability
- cold/warm validation
- event overlap
- performance bottleneck exploration
- missing evidence
- anomaly explanation
- data quality
- governance alignment
- activity/churn interpretation
- user/project attribution
- trend comparison
- before/after comparison

## Example route: verify same model

```markdown
## Question route: Verify same model across runs

Status:
- candidate

Question forms:
- Did I open the wrong model?
- Are these runs actually the same file?
- Can we compare these runs?

Intent:
- Confirm whether runs refer to the same model identity before interpreting performance differences.

Primary artifacts:
1. evidence_rollup.csv — high-level model labels and run timing
2. runs_manifest.json — authoritative run metadata
3. runs/{run_id}/journal.events.csv — observed journal open target if captured

Relevant fields:
- run_id
- scenario_label
- model_path
- model_name
- central_model_path
- document_title
- journal_open_target

Evidence type:
- mixed

Supported conclusion types:
- same model confirmed
- model mismatch observed
- model identity unknown

Unsupported conclusion types:
- performance cause
- user intent

Common traps:
- Do not rely only on scenario_label.
- Do not assume same display name means same central model.
```

## Example route: Desktop Connector involvement

```markdown
## Question route: Desktop Connector involvement

Status:
- candidate

Question forms:
- Was Desktop Connector involved?
- Did Desktop Connector slow this down?
- Did cloud/cache activity overlap with the open?

Intent:
- Determine whether Desktop Connector activity was observed during the capture window and whether it overlaps with slow runs.

Primary artifacts:
1. evidence_rollup.csv — event counts and high-level timing
2. runs_manifest.json — capture window and run identity
3. runs/{run_id}/desktop_connector.events.csv — event details

Secondary artifacts:
1. network.samples.csv — optional timing support
2. cache_delta artifacts — optional hydration/cache support

Evidence type:
- indirect

Supported conclusion types:
- Desktop Connector activity observed
- activity overlapped with run window
- evidence is consistent with cloud/cache involvement

Unsupported conclusion types:
- Desktop Connector caused the delay

Common traps:
- Event overlap is not causation.
- Missing events may mean no events or logging limitation.
```
