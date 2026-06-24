# LLM Evidence Framework Seed

This repository is a starting point for developing a framework that helps deterministic evidence packages become usable in LLM-assisted analysis without asking the LLM to invent unsupported facts, infer hidden structure, or overstate conclusions.

This is not yet a finalized standard, schema, or implementation contract. It is a field notebook and discovery scaffold.

## Current thesis

Deterministic systems should produce trustworthy evidence. LLMs should help users discuss what that evidence may suggest.

The framework separates:

- deterministic evidence
- evidence structure
- package-specific interpretation
- conversational reasoning rules
- question routing
- reusable extraction recipes
- eventual deterministic extractors

The goal is not to put all evidence into the chat context. The goal is to make the evidence archive navigable, comparable, and epistemically bounded.

## Starting structure

```text
notes/
  current_thesis.md
  open_questions.md

discovery/
  package_intake_questions.md
  evidence_map_discovery.md
  question_route_discovery.md
  script_recipe_discovery.md

patterns/
  deterministic_to_llm_boundary.md

examples/
  revit_open_capture/
    notes.md
```

## Maturity model

The expected promotion path is:

```text
ad hoc question
â candidate question route
â question route
â script recipe
â deterministic extractor
â rollup field or standard report
```

The first repo phase should preserve uncertainty. Durable schemas, validators, and formal skill files should come later, after the framework has been tested against real evidence packages.

## Terms

### Evidence archive

The full set of deterministic outputs, logs, CSVs, JSON, markdown, and raw artifacts produced by a system.

### Chat evidence package

A compact subset of evidence and context intended for use inside an LLM conversation.

### Evidence map

A deterministic description of what files exist, what each file contains, what grain each file represents, and how files relate.

### Question route

A reusable guide for where to look when a recurring analytical question appears.

### Script recipe

A reusable extraction pattern that tells the LLM or user how to pull a targeted answer from the evidence archive.

### Deterministic extractor

A promoted script or tool that implements a stable script recipe without relying on the LLM to recreate it each time.

## Initial implementation target

The first concrete example is Revit Open Capture evidence, including:

- cold/warm open sequences
- cache and hydration signals
- Desktop Connector activity
- journal events
- process samples
- network samples
- run manifest
- evidence rollup
- evidence flags
- model identity checks
- comparability warnings
