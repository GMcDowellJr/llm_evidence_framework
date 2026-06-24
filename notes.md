# Revit Open Capture Example Notes

This is a provisional example area for applying the LLM evidence framework to Revit open-capture evidence.

It is not a finalized package contract.

## Package intent

The Revit Open Capture package is intended to help diagnose whether long Revit open times appear more consistent with:

- Revit compute
- ACC/cloud worksharing
- Desktop Connector activity
- cache/hydration state
- VPN/network conditions
- linked-file loading
- disk/security scanning
- memory pressure
- model mismatch or wrong-file selection
- capture/tooling problems

The package should support evidence-based discussion, not definitive causation unless the evidence is strong enough.

## Known artifacts from current harness direction

Candidate default chat artifacts:

- `llm_context_prompt.md`
- `README_LLM_PACKAGE.md`
- `runs_manifest.json`
- `evidence_rollup.csv`
- `evidence_flags.csv`
- `llm_evidence_bundle.md`

Candidate drill-down artifacts:

- `journal.events.csv`
- `process.samples.csv`
- `network.samples.csv`
- `desktop_connector.events.csv`
- cache delta artifacts
- endpoint check artifacts
- raw journals
- full run metadata
- Desktop Connector logs/events
- process/system/network/disk samples

## Key concepts

### Cold run

A cold run is intended to approximate first-open behavior after cache clearing.

It should not be treated as verified cold unless cache-clearing evidence supports it.

### Warm run

A warm run is intended to approximate repeated-open behavior under similar model, machine, network, and cache conditions.

### Declared versus verified state

A run may be declared cold/warm by scenario label, but the evidence should distinguish declaration from verification.

Examples:

```text
declared_cold
verified_cold
declared_warm
verified_warm
cold_intent_cache_clear_failed
unknown_cache_state
```

### Desktop Connector activity

Desktop Connector events during the capture window are evidence of activity overlap.

They do not prove Desktop Connector caused the delay.

### Model identity

Model identity must be checked before performance comparisons.

Possible identity fields:

- model path
- model name
- central model path
- document title
- journal open target
- model identity hash
- project identity hash
- file size or timestamp, if available

### Comparability

Runs are strongest to compare when they share:

- same model identity
- same machine
- same Revit version
- same capture tool version
- same linked-file condition
- same network/VPN condition, if known
- clear cache-state intent and evidence

Runs are weak or invalid comparisons when:

- model identity differs
- model identity cannot be confirmed
- capture failed
- tool versions differ materially
- cache-state evidence is missing
- major environment conditions changed

## Candidate question routes

### Did I open the wrong model?

Primary artifacts:

- `evidence_rollup.csv`
- `runs_manifest.json`
- `journal.events.csv`

Relevant fields:

- run_id
- scenario_label
- model_path
- model_name
- central_model_path
- document_title
- journal_open_target

Supported outputs:

- same model confirmed
- mismatch observed
- identity unknown

### Was Desktop Connector involved?

Primary artifacts:

- `evidence_rollup.csv`
- `runs_manifest.json`
- `desktop_connector.events.csv`

Secondary artifacts:

- network samples
- cache delta artifacts

Supported outputs:

- Desktop Connector activity observed
- activity overlapped with capture window
- evidence consistent with cloud/cache involvement

Unsupported output:

- Desktop Connector caused the delay

### Was Revit computing or waiting?

Primary artifacts:

- `process.samples.csv`
- `journal.events.csv`

Secondary artifacts:

- network samples
- Desktop Connector events
- cache delta artifacts

Relevant signals:

- Revit CPU activity
- Revit memory growth
- journal event gaps
- disk read/write activity
- network activity
- overlapping Desktop Connector events

Supported outputs:

- evidence consistent with Revit compute
- evidence consistent with waiting on external dependency
- insufficient evidence

Unsupported output:

- definitive cause without corroborating signals

### Was the cold/warm order useful?

Primary artifacts:

- `runs_manifest.json`
- `evidence_rollup.csv`
- cache delta artifacts

Relevant concepts:

- super cold
- less cold
- warm
- warmer/repeated warm
- cold/warm/cold/warm sequence
- cache clear success/failure
- model identity consistency

## Candidate script recipes

- Verify same model across runs
- Summarize Desktop Connector activity during slow opens
- Compare cold/warm/cold/warm sequence
- Inspect Revit CPU during journal gaps
- Summarize cache deltas by run
- Extract journal warnings/errors
- Identify non-comparable runs
- Build compact run comparison table

## Candidate package health checks

- Required files exist
- Expected drill-down files exist or are explicitly marked missing
- CSVs are non-empty
- Schema/version fields present
- run_id values align across artifacts
- capture start/end exist
- model identity fields present
- cache state is known
- Desktop Connector capture capability is known
- raw journal redaction status known
- tool version recorded
- package generated timestamp recorded

## Known bad inferences

Do not say:

```text
Desktop Connector caused the delay.
```

Say:

```text
Desktop Connector activity overlapped with the delay and may be a contributing factor.
```

Do not say:

```text
The cold run proves first-time user performance.
```

Say:

```text
The run was intended as cold; whether it was verified cold depends on cache-clear evidence.
```

Do not say:

```text
Run A and Run B prove cache made the difference.
```

Say:

```text
Run A and Run B are consistent with a cache effect if model identity and comparability checks pass.
```

Do not say:

```text
No Desktop Connector activity occurred.
```

unless the package confirms the capture mechanism was active and the event file was successfully collected.

## Open questions for this package type

- Which fields should always appear in `evidence_rollup.csv`?
- Should verified cold/warm state be deterministic?
- How should model identity be hashed or represented?
- What is the minimum evidence required to say cache/hydration likely contributed?
- Should package health produce a single pass/fail status or categorized warnings?
- Which recurring recipes should become built-in extractors first?
