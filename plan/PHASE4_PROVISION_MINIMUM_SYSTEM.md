# Phase 4 — Provision Minimum System Plan

**Goal:** Render and apply the minimum system configuration for the selected track, installing core drivers, basic tools, and ensuring GPU and system health.

## Inputs
- `plan/TRACK.txt`
- `plan/CONSENSUS.md`
- `infra/` (rendered config files)

## Outputs
- Updated `infra/` directory with Configuration (Track A: Nix flake, hosts, modules)
- `reports/verify-minimum.md`

## Confirmation Requirements
- Any bootloader or disk operations require `@confirm:critical`.

## Tasks

### 4.1 Render Infrastructure Config
- Generate `infra/flake.nix`, host/module files, overlays, and devshells for Track A.

### 4.2 Apply Minimum System
- Execute bootstrap scripts in `scripts/` (with `--dry-run` flags) to:
  - Install GPU driver & Vulkan stack
  - Configure network, power management (TLP), filesystem mounts
  - Set up shell, editor, fonts, terminal
  - Install Codex CLI prerequisites

### 4.3 Verify Minimum System
- Run checks: `nvidia-smi` or `vulkaninfo`, basic compile of Hello World in C/Rust.
- Record results to `reports/verify-minimum.md`.

### 4.4 Checkpoint State
- Update `STATE.json` phase to `MINIMUM_READY`:
  ```bash
  jq '.phase="MINIMUM_READY" | .timestamp=now' STATE.json >STATE.tmp && mv STATE.tmp STATE.json
  ```

---

*Next:* Phase 5 — Dev Stack Expansion
