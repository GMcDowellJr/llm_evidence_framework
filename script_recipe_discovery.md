# Script Recipe Discovery

This document helps capture recurring extraction methods.

A script recipe answers: How should we extract the targeted evidence?

It is not necessarily a final script. It is a reusable procedure that can be implemented by an LLM, a human, or later promoted into a deterministic extractor.

## Purpose

Script recipes reduce repeated improvisation.

They help the LLM generate consistent scripts by documenting:

- input artifacts
- required fields
- joins
- filters
- grouping logic
- output shape
- inference limits
- common failure modes

## Difference between route, recipe, and extractor

```text
Question route:
  Look here.

Script recipe:
  Extract it this way.

Deterministic extractor:
  Run this.
```

## Recipe scaffold

```markdown
## Script recipe: <recipe name>

Status:
- candidate / active / deprecated / promoted-to-extractor

Related question routes:
- <route name>

Purpose:
- <what this recipe extracts or verifies>

Inputs:
- <artifact path> — <role>
- <artifact path> — <role>

Required fields:
- <artifact>: <field>, <field>, <field>

Optional fields:
- <artifact>: <field>, <field>, <field>

Parameters:
- <parameter>: <meaning and default>

Procedure:
1. <step>
2. <step>
3. <step>

Joins:
- Join <artifact A> to <artifact B> on <key>

Filters:
- <filter condition>
- <filter condition>

Grouping / aggregation:
- <grouping logic>

Output:
- <field>
- <field>
- <field>

Output format:
- printed table / CSV / JSON / markdown / other

Interpretation:
- <what the output can support>

Inference limits:
- <what the output cannot prove>

Failure modes:
- <missing file>
- <missing field>
- <empty result>
- <schema mismatch>

Promotion criteria:
- <when this should become a deterministic extractor>
```

## Questions to ask when capturing a recipe

- What question triggered the script?
- Which files were read?
- Which fields were required?
- Which fields were missing or unreliable?
- What joins were needed?
- What filters were applied?
- What output answered the question?
- What output was too verbose?
- Did the script need to handle missing files?
- Did the script reveal a package health problem?
- Should this remain ad hoc?
- Should this become a candidate recipe?
- Should this become a deterministic extractor?

## Recipe maturity levels

```text
Level 0 — One-off script
Generated for a single question.

Level 1 — Candidate recipe
The extraction logic seems reusable.

Level 2 — Active recipe
The recipe has a documented procedure and expected output.

Level 3 — Reference implementation
A working script exists but may not yet be part of the package generator.

Level 4 — Deterministic extractor
The script is maintained, tested, and shipped as part of the package or framework.
```

## Example recipe: verify same model across runs

```markdown
## Script recipe: Verify same model across runs

Status:
- candidate

Related question routes:
- Verify same model across runs
- Run comparability

Purpose:
- Identify whether selected runs appear to reference the same model identity.

Inputs:
- evidence_rollup.csv — high-level run fields
- runs_manifest.json — run metadata
- runs/{run_id}/journal.events.csv — observed journal open target when available

Required fields:
- evidence_rollup.csv: run_id, scenario_label, model_path, model_name
- runs_manifest.json: run_id

Optional fields:
- central_model_path
- document_title
- journal_open_target
- model_identity_hash

Procedure:
1. Load evidence_rollup.csv.
2. Load runs_manifest.json if available.
3. Merge run metadata on run_id.
4. For each run, collect available model identity fields.
5. Group by model_path, central_model_path, model_identity_hash, or best available identity field.
6. Flag runs where scenario labels imply comparison but model identity differs.
7. For flagged runs, inspect journal.events.csv if available.
8. Return a compact table.

Output:
- run_id
- scenario_label
- model_name
- model_path
- central_model_path
- journal_open_target
- identity_group
- mismatch_flag
- note

Interpretation:
- Supports confirming or challenging run comparability by model identity.

Inference limits:
- Does not explain performance differences.
- Does not prove user intent.
- Does not guarantee identity if stable identifiers are missing.
```

## Example recipe: Desktop Connector activity during slow opens

```markdown
## Script recipe: Desktop Connector activity during slow opens

Status:
- candidate

Related question routes:
- Desktop Connector involvement
- Cloud/cache activity

Purpose:
- Summarize Desktop Connector events during capture windows for slow or selected runs.

Inputs:
- evidence_rollup.csv — run timing and event counts
- runs_manifest.json — capture start/end
- runs/{run_id}/desktop_connector.events.csv — event details

Required fields:
- evidence_rollup.csv: run_id, elapsed_seconds
- runs_manifest.json: run_id, capture_start, capture_end
- desktop_connector.events.csv: timestamp

Optional fields:
- event_type
- path
- message
- severity

Parameters:
- selected_run_ids: optional list
- slow_threshold: default median elapsed_seconds
- include_top_paths: default true

Procedure:
1. Load evidence_rollup.csv.
2. Select runs by selected_run_ids or elapsed_seconds above slow_threshold.
3. Load runs_manifest.json.
4. For each selected run, load desktop_connector.events.csv if present.
5. Filter events to the capture window when timestamps are available.
6. Count events by run_id and event_type.
7. Capture first and last event time.
8. Return compact run-level summary.

Output:
- run_id
- scenario_label
- elapsed_seconds
- desktop_connector_event_count
- first_event_time
- last_event_time
- top_event_types
- top_paths
- note

Interpretation:
- Supports identifying whether Desktop Connector activity was observed during slow runs.

Inference limits:
- Does not prove Desktop Connector caused the delay.
- Missing event files may indicate no events, logging failure, or tool version limitations.
```
