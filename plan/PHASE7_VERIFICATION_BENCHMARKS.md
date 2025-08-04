# Phase 7 — Verification & Benchmarks Plan

**Goal:** Perform comprehensive validation and benchmarking of GPU stack, dev toolchains, gaming, and LLM‑dev loop.

## Inputs
- System state after Phase 6

## Outputs
- `reports/verify.md`
- Git tag `v0.MIN`

## Confirmation Requirements
- No destructive actions.

## Tasks

### 7.1 GPU & Vulkan Validation
- Run `nvidia-smi` or `vulkaninfo`; verify expected outputs.

### 7.2 Compiler Toolchain Checks
- `gcc --version`, `rustc --version`, `node -v`, `java -version`; record in report.

### 7.3 Steam & WINE Tests
- Launch a Proton test (e.g. a tutorial in a simple game) and WINE app, record success.

### 7.4 LLM Dev Loop Test
- Use Codex CLI to generate and execute a small sample toolchain example; verify end‑to‑end.

### 7.5 Benchmark Metrics
- Compile “Hello, world” in C and Rust; run a quick `fio` I/O test; log times in `reports/metrics.json`.

### 7.6 Generate Final Verification Report
- Aggregate all checks into `reports/verify.md` with pass/fail statuses.

### 7.7 Tag & Rollback Recipe
- Tag repo: `git tag v0.MIN`
- Create `plan/ROLLBACK_MIN.md` describing rollbacks for each phase.

### 7.8 Checkpoint State
- Update `STATE.json` phase to `VERIFIED`:
  ```bash
  jq '.phase="VERIFIED" | .timestamp=now' STATE.json >STATE.tmp && mv STATE.tmp STATE.json
  ```

---

*Next:* Phase 8 — Tuning & Hardening
