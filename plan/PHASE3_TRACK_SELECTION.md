# Phase 3 — Track Selection Gate Plan

**Goal:** Choose the primary OS track (A: NixOS, B: Fedora Kinoite/Silverblue, or C: Arch/Ansible) based on decision rules and document the choice.

## Inputs
- `plan/CONSENSUS.md`
- `llm-laptop/FACTS.json`
- `preferences/preferences.yaml`

## Outputs
- `plan/TRACK.txt` (single letter A, B, or C plus rationale)

## Confirmation Requirements
- No destructive actions; file creation only.

## Tasks

### 3.1 Evaluate Decision Rules
- Apply rules:
  1. Prefer Track A if maximum declarative reproducibility is desired.
  2. Prefer Track B if Fedora familiarity and rpm-ostree immutability are priorities.
  3. Fallback Track C if hardware or driver issues block A/B.

### 3.2 Draft TRACK.txt
- Write `plan/TRACK.txt` with chosen track letter and a concise rationale.

### 3.3 Checkpoint State
- Update `STATE.json` phase to `TRACK_SELECTED`:
  ```bash
  jq '.phase="TRACK_SELECTED" | .timestamp=now' STATE.json >STATE.tmp && mv STATE.tmp STATE.json
  ```

---

*Next:* Phase 4 — Provision Minimum System
