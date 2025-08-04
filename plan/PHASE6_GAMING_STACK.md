# Phase 6 — Gaming Stack Plan

**Goal:** Install and verify the gaming stack, including Steam, Proton, WINE, and performance overlays.

## Inputs
- `plan/CONSENSUS.md`
- `infra/` configs

## Outputs
- Installed gaming components and sample tests in `reports/verify-gaming.md`

## Confirmation Requirements
- No destructive actions; standard system installs.

## Tasks

### 6.1 Steam & Proton
- Install Steam and Proton packages; run Steam once to initialize compatdata.

### 6.2 Performance Overlays
- Install Gamescope, MangoHud, vkBasalt.

### 6.3 WINE Stack
- Install Wine-Staging, `winetricks`, DXVK, VKD3D; test a simple WINE app (e.g. Notepad++).

### 6.4 GameMode & I/O Tuning
- Install and enable `gamemode`; configure scheduler tweaks in sysctl or mount options.

### 6.5 Controller Support
- Verify controller detection and basic mapping (e.g. Xbox, PlayStation).

### 6.6 Generate Gaming Verification Report
- Run `vulkaninfo`, launch a Proton test game, record logs in `reports/verify-gaming.md`.

### 6.7 Checkpoint State
- Update `STATE.json` phase to `GAMING_READY`:
  ```bash
  jq '.phase="GAMING_READY" | .timestamp=now' STATE.json >STATE.tmp && mv STATE.tmp STATE.json
  ```

---

*Next:* Phase 7 — Verification & Benchmarks
