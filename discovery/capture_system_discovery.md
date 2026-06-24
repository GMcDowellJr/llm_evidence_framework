# Capture System Discovery

This document is a discovery scaffold for an OB-like capture and semantic indexing layer around LLM evidence packages.

It is not a finalized schema, implementation contract, or database design.

## Purpose

The capture system is intended to help an evidence workspace learn from use without allowing the LLM to rewrite the evidence or silently promote unsupported conclusions.

The core idea:

```text
The evidence archive stores what was observed.
The evidence map describes where evidence lives.
The interpretation guide constrains what evidence can mean.
The capture system remembers how people interrogate the evidence.
The repo stores what has been reviewed and promoted.
```

The capture system should detect recurring analytical moves, recurring questions, useful routes, repeated scripts, missing evidence patterns, and known inference traps.

It should not treat repeated user interpretations as facts.

## Relationship to the repo

The repo is the durable framework and promoted knowledge base.

The capture system is the learning substrate.

```text
Capture system:
  provisional memory
  semantic clustering
  candidate routes
  candidate recipes
  missing-evidence patterns
  promotion signals

Repo:
  reviewed docs
  accepted patterns
  promoted routes
  promoted script recipes
  deterministic extractors
  tests
  future schemas
```

The capture system should help decide what belongs in the repo, but it should not automatically rewrite repo standards without review.

## Relationship to raw evidence

The capture system should not store or vector the full raw evidence archive by default.

Raw evidence belongs in:

- files
- databases
- package folders
- object storage
- source systems
- deterministic outputs

The capture system should store metadata about how evidence was used.

Example:

```text
Store:
  User asked whether a cold run may have opened the wrong model.
  Package type was revit_open_capture.
  Relevant route was model identity verification.
  Artifacts used were evidence_rollup.csv, runs_manifest.json, journal.events.csv.
  Fields used were run_id, model_path, central_model_path, journal_open_target.
  Answer confidence was moderate.
  Missing evidence was stable model identity hash.

Do not store:
  All rows from journal.events.csv.
  Full raw logs.
  Full process samples.
```

## What the capture system learns

The adaptive layer should learn:

```text
How users interrogate the evidence.
```

It should be cautious about learning:

```text
What the evidence always means.
```

The system should promote methods, routes, caveats, and extraction patterns before promoting conclusions.

## Candidate capture object

Use this scaffold when capturing a single evidence-package interaction.

```markdown
## Capture: <short title>

Capture status:
- raw / reviewed / clustered / promoted / superseded

Package type:
- <package or evidence-domain name>

Package ID / dataset ID:
- <if available>

Question asked:
- <user phrasing>

Question intent:
- <what the user was trying to understand>

Question category:
- identity verification / comparability / anomaly explanation / event overlap / missing evidence / governance interpretation / other

Artifacts consulted:
- <artifact path> — <why it was used>
- <artifact path> — <why it was used>

Fields used:
- <artifact>: <field>, <field>, <field>

Route resemblance:
- <existing route or candidate route>

Script generated:
- yes / no

Script or extraction summary:
- <what the script did, if any>

Answer status:
- answered / partially answered / inconclusive / blocked

Evidence strength:
- strong / moderate / weak / contradicted / unknown

Conclusion type:
- observed / directly supported / consistent with / plausible / unsupported / contradicted

Missing evidence:
- <what would have improved the answer>

Inference limits:
- <what the answer could not prove>

Known bad inference avoided:
- <bad inference>

Candidate promotion:
- none / candidate route / candidate recipe / deterministic extractor / rollup field / package health check / known bad inference

Promotion rationale:
- <why this might be worth promoting>

Related captures:
- <links or IDs if available>
```

## Minimal capture fields

A future database schema may not need every field above. The minimum useful capture may be:

```yaml
capture_id:
created_at:
package_type:
package_id:
question_text:
question_intent:
question_category:
artifacts_consulted:
fields_used:
route_candidate:
recipe_candidate:
script_generated:
answer_status:
evidence_strength:
missing_evidence:
inference_limits:
promotion_candidate:
source_channel:
```

## Semantic clustering

Semantic vectoring is useful because users may ask the same underlying question in many ways.

Examples that may cluster together:

```text
Did I open the wrong file?
Are these actually the same model?
Can we compare these runs?
The latest cold was faster; maybe I opened a different file?
Do we capture the actual file name?
```

Potential cluster:

```text
model identity verification / run comparability
```

Another cluster:

```text
Was Desktop Connector involved?
Was it hydrating?
Was ACC downloading links?
Was the cache less warm?
Did cloud activity overlap with the open?
```

Potential cluster:

```text
cloud/cache activity during Revit open
```

The capture system should help identify these clusters without requiring the user or repo maintainer to manually notice every recurrence.

## Candidate cluster object

A cluster groups semantically similar captures.

```markdown
## Cluster: <cluster name>

Status:
- emerging / active / promoted / deprecated / split-needed / merge-needed

Package types:
- <package type>
- <package type>

Representative questions:
- <question>
- <question>
- <question>

Shared intent:
- <what these questions are usually trying to understand>

Common artifacts:
- <artifact>
- <artifact>

Common fields:
- <field>
- <field>

Common missing evidence:
- <missing evidence pattern>

Common bad inferences:
- <bad inference>

Likely promotion:
- question route / script recipe / extractor / rollup field / package health check / known bad inference

Promotion readiness:
- low / medium / high

Notes:
- <reviewer notes>
```

## Promotion model

The capture system should support a promotion path like:

```text
raw interaction
→ reviewed capture
→ semantic cluster
→ candidate route
→ active question route
→ script recipe
→ deterministic extractor
→ rollup field or standard report
```

Potential thresholds:

```text
Candidate route:
  2+ semantically similar captures
  same package type or clearly related package types
  same primary artifacts

Active question route:
  3+ uses
  stable artifact path
  stable fields
  at least one successful answer

Script recipe:
  2+ generated scripts with similar logic
  same joins or filters
  similar output shape

Deterministic extractor:
  recipe used repeatedly
  script output is stable
  extraction no longer requires LLM judgment

Rollup field:
  extractor output is repeatedly needed in first-pass analysis
```

These thresholds are placeholders. They should be tuned after real package use.

## What should be promoted

Promote:

- recurring question patterns
- artifact routes
- extraction procedures
- known missing-evidence patterns
- known bad inferences
- comparability checks
- package health checks
- deterministic fields that repeatedly need to be computed

Be cautious about promoting:

- conclusions from a single package
- user opinions
- causal claims
- audience-specific framing
- one-off scripts
- assumptions that were useful only because of a specific dataset

## Guardrails

The capture system should preserve the boundary between evidence and interpretation.

### Do not promote repetition as truth

If users repeatedly conclude the same thing, that does not make it true.

Bad promotion:

```text
Desktop Connector causes slow Revit opens.
```

Better promotion:

```text
Users repeatedly ask whether Desktop Connector activity overlaps with slow opens.
Promote a route and recipe for checking Desktop Connector activity during capture windows.
```

### Do not overwrite deterministic evidence

The capture system may point to evidence, but should not alter raw evidence.

### Preserve source and status

Captured items should indicate whether they are:

- raw
- reviewed
- provisional
- promoted
- superseded
- rejected

### Separate method from conclusion

A route or recipe may be broadly reusable.

A conclusion usually belongs to a specific package, run, or dataset.

## Capture status lifecycle

```text
raw
  Captured from an interaction.

reviewed
  Human or process has cleaned the capture enough to be useful.

clustered
  Linked to similar captures.

candidate
  Suggested for promotion.

promoted
  Moved into repo docs, route files, recipes, extractors, or package generator.

superseded
  Replaced by a newer route, recipe, or extractor.

rejected
  Reviewed and intentionally not promoted.
```

## Suggested metadata fields

```yaml
capture_id:
created_at:
updated_at:
source_channel:
package_type:
package_id:
dataset_id:
tool_version:
schema_version:
question_text:
question_intent:
question_category:
artifacts_consulted:
fields_used:
script_generated:
script_language:
script_summary:
answer_status:
evidence_strength:
confidence_language:
missing_evidence:
inference_limits:
bad_inference_avoided:
route_candidate:
recipe_candidate:
extractor_candidate:
rollup_candidate:
promotion_status:
promotion_rationale:
related_capture_ids:
cluster_id:
review_status:
review_notes:
```

## Relationship to skills

A skill-like instruction layer governs how the LLM should behave during evidence discussion.

The capture system helps the skill improve by surfacing:

- recurring unsupported claims
- repeated missing evidence
- repeated comparability mistakes
- repeated successful routing patterns
- recurring script-generation needs

Potential skill updates may include:

- new known bad inference examples
- stronger confidence language
- new routing rules
- new instruction to check package health first
- new instruction to verify comparability before comparing

## Relationship to evidence maps

The evidence map is deterministic and describes the archive.

The capture system can discover where users actually look.

If users repeatedly need an artifact that is not well described in the evidence map, this is a signal to improve the evidence map.

If users repeatedly need a field that does not exist, this is a signal to improve the deterministic package generator.

## Relationship to script recipes

The capture system should identify repeated generated scripts.

Capture useful details:

- question answered
- files loaded
- fields required
- joins used
- filters used
- output shape
- whether the output answered the question
- errors or missing files
- whether the script should become a recipe

When a script recipe stabilizes, it should be promoted into the repo.

When a recipe is used repeatedly, it should be promoted into a deterministic extractor.

## Relationship to package health

Repeated analysis blockers should become package health checks.

Examples:

```text
Users repeatedly cannot determine whether a run is verified cold.
→ Add cache-clear status to package health.

Users repeatedly cannot confirm model identity.
→ Add model identity hash or journal-open-target check.

Users repeatedly cannot tell whether Desktop Connector logging was active.
→ Add capture capability status.
```

## Possible implementation shapes

This document does not prescribe implementation, but possible shapes include:

### Lightweight repo-only capture

- Markdown capture files
- Manual review
- Manual clustering
- Good for early development
- Low complexity

### Local database + embeddings

- SQLite or Postgres
- embedding vectors
- semantic search
- cluster review
- better for growing usage

### OB-like system

- capture API
- semantic search
- entity extraction
- thought/capture edges
- promotion workflows
- synthesis pages
- health/lint checks

### Workspace per dataset

- Custom GPT / Project / Notebook / repo / database
- raw data pointers
- deterministic rollups
- evidence map
- interpretation guide
- semantic capture layer
- promoted routes and recipes

## Future workspace architecture

A mature evidence workspace may contain:

```text
/data
  Raw or linked evidence

/derived
  Rollups, flags, summaries, health checks

/maps
  Evidence maps and artifact indexes

/guides
  Interpretation guide
  Comparability rules
  Known bad inferences
  Confidence language

/routes
  Promoted question routes

/recipes
  Promoted script recipes

/extractors
  Deterministic scripts

/memory
  Vectored captures of prior questions, answers, gaps, routes, and promotion candidates

/reports
  Human-facing outputs
```

## Discovery questions

Use these questions before implementing the capture system.

### Scope

- Is capture per repo, per evidence package, per package type, or global?
- Should different package types share captures?
- Should captures be linked to package IDs or only package types?
- How long should raw captures be retained?

### Capture timing

- When should a capture be created?
  - after every answer
  - only when explicitly requested
  - when a new route appears
  - when a script is generated
  - when evidence is missing
- Should capture be automatic or user-triggered?

### Review

- Who reviews promotion candidates?
- What makes a capture clean enough to promote?
- Can the LLM suggest promotion but require human approval?
- How are rejected promotions tracked?

### Search

- What text is embedded?
- Are artifacts, fields, and package types embedded separately?
- Should metadata filters be used before vector search?
- How should exact matches and semantic matches combine?

### Clustering

- What similarity threshold indicates recurrence?
- When should clusters be split?
- When should clusters be merged?
- How are contradictions surfaced?

### Promotion

- What promotion targets exist?
  - evidence map update
  - question route
  - script recipe
  - extractor
  - rollup field
  - package health check
  - skill instruction
  - known bad inference
- What evidence is required before each promotion type?

### Safety and trust

- How does the system prevent repeated speculation from becoming accepted truth?
- How does it handle stale captures?
- How does it handle superseded routes or recipes?
- How does it preserve package-specific context?
- How does it distinguish method reuse from conclusion reuse?

## Initial implementation recommendation

Start simple.

For the current repo phase, create a markdown-based capture log and manually review it.

Suggested early files:

```text
capture/
  capture_log.md
  cluster_candidates.md
  promotion_candidates.md
```

Only after repeated use should this become a database-backed semantic layer.

The first implementation goal is not automation. It is to learn what should be captured.
