# Evidence Map Discovery

This document helps discover the structure needed for an evidence map.

An evidence map should be deterministic when possible. It should describe the evidence archive without interpreting conclusions from the data.

## Purpose of an evidence map

The evidence map helps an LLM or human navigate a package without loading the full evidence archive into chat context.

It should answer:

- What files exist?
- What does each file contain?
- What is each file's grain?
- Which columns or keys matter?
- Which files join together?
- Which questions can each file help answer?
- Which questions can each file not answer?
- Which files are summaries versus drill-down evidence?
- Which files are authoritative?

## Artifact entry scaffold

Use this structure while discovering an artifact.

```markdown
## Artifact: <path>

Status:
- required / optional / conditional / unknown

Artifact type:
- manifest / rollup / flag file / event log / sample log / raw log / summary / context / configuration / other

Produced by:
- deterministic tool / user / LLM / mixed / unknown

Authority level:
- authoritative / controlled interpretation / convenience summary / provisional / unknown

Purpose:
- <why this artifact exists>

Grain:
- <one row/object per what?>

Key fields:
- <field>: <meaning>
- <field>: <meaning>

Identifiers:
- <field>
- <field>

Join keys:
- Joins to <artifact> on <key>
- Joins to <artifact> on <key>

Can help answer:
- <question type>
- <question type>

Cannot answer by itself:
- <question type>
- <question type>

Known limitations:
- <limitation>
- <limitation>

Null semantics:
- <field>: <blank means what?>
- <field>: <zero means what?>

Context role:
- default chat context / optional chat context / drill-down only / script-only / archive only
```

## Questions to ask per artifact

### Structure

- Is the file tabular, JSON, markdown, raw text, binary, or mixed?
- Is the structure stable?
- Does the file have a schema?
- Are field names consistent across versions?
- Are timestamps timezone-aware?
- Are identifiers stable?

### Grain

- What does one row represent?
- Can multiple rows share the same run ID?
- Can multiple rows share the same timestamp?
- Can one file contain mixed event types?
- Does the file represent state or events?
- Is the file cumulative or scoped to one run/package?

### Authority

- Is this file the source of truth for any field?
- Is it derived from other artifacts?
- Is it safe to use for conclusions?
- Is it only a convenience report?
- What should happen if this file disagrees with another artifact?

### Usefulness

- What first-pass questions does this file answer?
- What drill-down questions does this file answer?
- What fields are most likely to be useful for filtering?
- What fields are likely to be useful for grouping?
- What fields are likely to be useful for joins?

### Context

- Is this file small enough to include in chat?
- Does it provide high information density?
- Can it be summarized deterministically?
- Should only selected rows be retrieved?
- Should a script extract from it instead?

## Evidence map maturity levels

```text
Level 0 — Artifact listed
The file is known to exist.

Level 1 — Artifact described
Purpose, grain, and key fields are known.

Level 2 — Artifact connected
Join keys and related files are known.

Level 3 — Artifact routed
Known question types point to this file.

Level 4 — Artifact operationalized
Script recipes or deterministic extractors use this file reliably.
```

## Candidate evidence-map fields

These may eventually become a structured format.

```yaml
artifact_id:
path:
artifact_type:
required:
producer:
authority_level:
context_role:
grain:
key_fields:
identifiers:
join_keys:
can_answer:
cannot_answer:
known_limitations:
null_semantics:
related_artifacts:
schema_version:
```
