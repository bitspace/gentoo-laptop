# Phase 2 — Consensus Synthesis Plan

**Goal:** Produce a ranked, rationale‑backed consensus plan (`plan/CONSENSUS.md`) plus detailed ADRs for each major decision area.

## Inputs
- `infra/sources/survey-normalized.jsonl`
- `preferences/preferences.yaml`
- `llm-laptop/FACTS.json`

## Outputs
- `plan/CONSENSUS.md`
- `plan/DECISION_RECORDS/ADR-XXXX.md` (one ADR per major choice)

## Confirmation Requirements
- No destructive actions; user‑level file creation only.

## Tasks

### 2.1 Collect Required Inputs
- Verify presence and validity of `survey-normalized.jsonl`, `preferences.yaml`, and `FACTS.json`.

### 2.2 Synthesize Consensus Plan
- Draft `plan/CONSENSUS.md` to include:
  - Primary & secondary OS tracks
  - GPU driver strategy
  - Dev toolchain inventory
  - Gaming stack overview
  - Risk register with mitigations

### 2.3 Write ADRs
- For each major decision (OS track, GPU driver, toolchain, gaming), create an ADR in `plan/DECISION_RECORDS/`:
  ```markdown
  # ADR-0001: Title of Decision

  ## Context
  (Summary of inputs & constraints)

  ## Decision
  (Chosen option & rationale)

  ## Consequences
  (Benefits, risks, next steps)
  ```

### 2.4 Checkpoint State
- Update `STATE.json` phase to `CONSENSUS_SYNTHESIZED`:
  ```bash
  jq '.phase="CONSENSUS_SYNTHESIZED" | .timestamp=now' STATE.json >STATE.tmp && mv STATE.tmp STATE.json
  ```

---

*Next:* Phase 3 — Track Selection Gate
