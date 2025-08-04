# Phase 8 — Tuning & Hardening Plan

**Goal:** Apply targeted performance optimizations, security hardening, and optional GPU compute stacks.

## Inputs
- System state after Phase 7

## Outputs
- Updated `infra/` with tuning configs (sysctl, power profiles)
- `reports/verify-tuning.md`

## Confirmation Requirements
- No destructive actions; ensure backups exist for sysctl and kernel settings.

## Tasks

### 8.1 Power & Thermal Tuning
- Configure TLP or auto-cpufreq; adjust fan curves or governor settings.

### 8.2 I/O & Filesystem Tweaks
- Adjust mount options (noatime, discard), sysctl for scheduler and cache.

### 8.3 GPU Compute Stack (Optional)
- If NVIDIA: install CUDA; if AMD: install ROCm; document in ADR if applied.

### 8.4 Security Hardening
- Enable `nftables` firewall, configure automatic updates policy, secure keyring access.

### 8.5 Generate Tuning Verification Report
- Re-run benchmarks (Phase 7 tests) and record differences/improvements in `reports/verify-tuning.md`.

### 8.6 Checkpoint State
- Update `STATE.json` phase to `TUNED_HARDENED`:
  ```bash
  jq '.phase="TUNED_HARDENED" | .timestamp=now' STATE.json >STATE.tmp && mv STATE.tmp STATE.json
  ```

---

*Plan complete — all phases documented.*
