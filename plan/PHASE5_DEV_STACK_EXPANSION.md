# Phase 5 — Dev Stack Expansion Plan

**Goal:** Install and configure language runtimes, linters, formatters, and container tooling per consensus.

## Inputs
- `plan/CONSENSUS.md`
- `infra/` configs (devshells or Ansible roles)

## Outputs
- Installed and validated dev toolchains for Python, Rust, Node, Java, and containers
- `reports/verify-devstack.md`

## Confirmation Requirements
- Standard user/system operations; no destructive actions.

## Tasks

### 5.1 Python Toolchain
- Install `uv` or `pyenv`, `pipx`, `ruff`, `mypy`, `pytest`.

### 5.2 Rust Toolchain
- Install `rustup`, `rust-analyzer`, `clippy`.

### 5.3 Node Toolchain
- Install `fnm`, set LTS & current, install `pnpm` or `npm`.

### 5.4 Java Toolchain
- Install `sdkman` and Temurin JDK(s).

### 5.5 Container Tooling
- Install Podman/Docker and setup `distrobox` or devShells per track.

### 5.6 IDE & Editors
- Install VS Code and recommended extensions; configure Neovim if needed.

### 5.7 Generate Dev Stack Verification Report
- Run sample builds/tests for each language and record outputs in `reports/verify-devstack.md`.

### 5.8 Checkpoint State
- Update `STATE.json` phase to `DEVSTACK_READY`:
  ```bash
  jq '.phase="DEVSTACK_READY" | .timestamp=now' STATE.json >STATE.tmp && mv STATE.tmp STATE.json
  ```

---

*Next:* Phase 6 — Gaming Stack
