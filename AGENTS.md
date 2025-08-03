# AGENTS.md — Operating Guide for OpenAI Codex CLI on the LLM‑Optimized Linux Laptop

**Audience:** OpenAI Codex CLI (the “Agent”) and any cooperating automation scripts.  
**Owner:** Christopher Woods (project lead).  
**Purpose:** Provide clear guardrails and step‑by‑step operating instructions so the Agent can (1) ingest and analyze prior LLM recommendations, (2) synthesize a consensus installation plan, and (3) provision a Linux laptop optimized for LLM‑centric workflows, software development, and gaming (Steam/Proton/WINE).

---

## 0) Ground Rules

1. **Safety first.**  
   - Never execute destructive actions without an explicit confirmation comment in the task file (see `@confirm:` tags below).  
   - Prefer **dry‑runs** (`--dry-run`, `--check`) and **idempotent** scripts.  
   - For any command requiring `sudo`, explain why in a log note and request confirmation.  
   - Never edit bootloader, disk partition tables, or firmware settings without an `@confirm:critical` tag.

2. **Determinism & repeatability.**  
   - All configuration must be declarative where feasible and committed to a Git repo (`infra/` directory).  
   - Capture hardware and system facts as JSON.  
   - Every change produces a machine‑readable diff and a rollback note.

3. **LLM‑friendly surface area.**  
   - Prefer **TOML/YAML/JSON/Nix** declarative formats over shell one‑liners.  
   - Avoid interactive wizards; supply explicit flags and config files.  
   - Heavily comment files with consistent, parseable patterns.

4. **Short feedback loops.**  
   - Target a working **Minimum System** quickly, then iterate.  
   - Install the **OpenAI Codex CLI** early and continue through the CLI.

5. **Observability.**  
   - All steps log to `logs/` with timestamps and command transcripts.  
   - Keep a `FACTS.json` (hardware + distro facts) and `STATE.json` (phase checkpoints).

---

## 1) Inputs the Agent Should Expect

- `survey/` — a folder of model outputs (Markdown, text, JSON) collected from various LLMs.
- `constraints/` — any constraints (e.g., corporate VPN, CUDA required, offline package mirrors).
- `preferences/` — user preferences & priorities (kept in `preferences.yaml`).
- `secrets/` — credentials and tokens (never commit to Git; use `.env` + system keyring).

### Normalization Schema (for parsed survey content)

Create `infra/sources/survey-normalized.jsonl`, one JSON per document:

```json
{
  "model": "provider/model@version",
  "date_utc": "2025-08-03T12:00:00Z",
  "topic_tags": ["distro", "gaming", "devtools"],
  "strength_of_claim": 1-5,
  "citations": [],
  "verbatim": "…raw snippet…",
  "extracted_recommendations": [
    {"area":"OS","option":"NixOS","rationale":"…","risks":["…"]},
    {"area":"GPU","option":"NVIDIA proprietary + CUDA","rationale":"…"}
  ]
}
```

---

## 2) Outputs the Agent Must Produce

1. **`plan/CONSENSUS.md`** — synthesized, ranked plan with rationale and risks.  
2. **`plan/DECISION_RECORDS/ADR-XXXX.md`** — Architecture Decision Records for each major choice.  
3. **`infra/`** — reproducible configs for the chosen OS track (A or B below).  
4. **`scripts/`** — idempotent shell/python tasks with `--dry-run` and `--apply`.  
5. **`reports/`** — verification results: GPU/Steam/Proton, compiler toolchains, CUDA/Vulkan, etc.  
6. **`logs/`** — step‑by‑step transcripts.  
7. **`STATE.json`** — checkpointed phase + timestamps.

---

## 3) Roles (Single CLI can assume these personas sequentially)

- **Research Agent:** Ingest & normalize `survey/`, cluster ideas, detect agreement and conflict.  
- **Synthesis Agent:** Generate `plan/CONSENSUS.md` and draft ADRs.  
- **Systems Architect:** Choose Track A or B (see §5) using explicit decision rules.  
- **Provisioner:** Render configs, execute bootstrap steps with confirmations.  
- **Verifier:** Run post‑install checks and benchmarks, produce `reports/`.  
- **Tuner:** Apply targeted optimizations (compiler flags, I/O, power, kernel params).  
- **Archivist:** Commit, tag, and emit rollback recipes.

---

## 4) Phase Workflow

### Phase 0 — Bootstrap (Laptop already has a basic Linux shell)
**Goal:** Minimal environment and Codex CLI installed.

Tasks:
- Create project workspace: `llm-laptop/` with directories above.  
- Detect hardware facts to `FACTS.json`:
  ```
  lspci -nnk; lsusb; uname -a; cat /etc/os-release; df -h; free -h; 
  nproc; lscpu; lsblk -o NAME,SIZE,TYPE,MOUNTPOINT; 
  vulkaninfo || true; nvidia-smi || true; glxinfo -B || true
  ```
- Install Python (system or `uv`), Git, curl, jq, yq, ripgrep, fd, make, gcc, clang, pkg-config.
- Install **OpenAI Codex CLI** (or fallback to `openai` CLI) and verify auth.
- Check internet, NTP, and time zone.  
- **Checkpoint:** write `STATE.json.phase = "BOOTSTRAPPED"`.

### Phase 1 — Ingest & Normalize
- Parse all `survey/*` into `survey-normalized.jsonl`.  
- Cluster with lightweight topic model or rule‑based tags.  
- Emit `reports/survey-summary.md` with frequency of each recommendation and conflict list.

### Phase 2 — Consensus Synthesis
- Produce `plan/CONSENSUS.md` with:
  - Primary & secondary OS tracks.
  - GPU driver plan (auto‑detect NVIDIA vs AMD/Intel).  
  - Dev toolchain inventory.  
  - Gaming stack plan (Steam/Proton/WINE/Gamescope).  
  - Risk register + mitigations.  
- Open ADRs for each major decision.

### Phase 3 — Track Selection Gate
Apply these rules:

1. **Prefer Track A (NixOS + flakes + home-manager)** when:
   - Desire maximum declarative, LLM‑parseable config and reproducibility.
   - Willing to adopt the Nix language and flake workflow.

2. **Prefer Track B (Fedora Kinoite/Silverblue + rpm‑ostree + Ansible)** when:
   - Familiar with Fedora; desire immutable base with conventional package ecosystem.
   - Want containerized dev envs (toolboxes/distrobox) without learning Nix.

3. **Fallback Track C (Arch + Ansible)** if A/B blocked by hardware or driver issues.

**Checkpoint:** Write `plan/TRACK.txt` with `A|B|C` and the reason.

### Phase 4 — Provision Minimum System
- Render configs into `infra/` for the chosen track.  
- Apply **Minimum System** only:
  - GPU driver + Vulkan stack verified.
  - Network, power (TLP/auto-cpufreq), filesystem mounts.
  - Terminal + editor (Neovim/VS Code), shell config, fonts.
  - Codex CLI + basic SDKs (Python, Rust, Node, Java, Go).
- **Confirm** before any disk/boot edits: requires `@confirm:critical`.

### Phase 5 — Dev Stack Expansion
- Python: `uv` or `pyenv`, `pipx`, `ruff`, `mypy`, `pytest`.  
- Rust: `rustup` + components `rust-analyzer`, `clippy`.  
- Node: `fnm` + LTS/current; `pnpm` or `npm`.  
- Java: `sdkman` (Temurin JDKs).  
- Containers: Podman or Docker, plus `distrobox`/`toolbox` (Track B) or Nix devShells (Track A).  
- IDEs: VS Code + extensions (Python, Rust, Docker, GitLens), JetBrains optional.

### Phase 6 — Gaming Stack
- Steam (with Proton, protonup‑qt or proton-ge installer), Gamescope, MangoHud, vkBasalt.  
- WINE/Wine‑Staging + `winetricks`, DXVK/VKD3D as appropriate.  
- GameMode + I/O scheduler tuning.  
- Verify Vulkan, ProtonDB sample titles, controller support.

### Phase 7 — Verification & Benchmarks
- Produce `reports/verify.md` with:
  - `nvidia-smi` or `vulkaninfo` OK.  
  - Compiler toolchains working: `gcc --version`, `rustc --version`, `node -v`, `java -version`.  
  - Steam launches; Proton test game completes tutorial; WINE app runs.  
  - LLM dev loop: small example toolchain generated by Codex and executed.  
- Tag Git repo `v0.MIN` and create rollback doc.

### Phase 8 — Tuning & Hardening
- Power profiles, suspend/hibernate reliability, thermals.  
- Sysctl and I/O schedulers; filesystem mount options.  
- Optional: CUDA toolchain (if NVIDIA), ROCm (if AMD).  
- Security baselines: firewall (`nftables`), automatic updates policy, secure keyring.

---

## 5) OS Tracks — Deliverables

### Track A: **NixOS + flakes + home-manager**
**Deliverables (`infra/`):**
- `flake.nix`, `flake.lock`
- `nixos/hosts/<hostname>.nix`
- `home/<user>.nix`
- `overlays/`, `modules/`
- `devshells/` for language toolchains

**Policies:**
- No imperative `nix-env`; use flakes only.  
- Pin inputs; use `deploy-rs` or `nixos-rebuild switch --flake .#<host>`.  
- Steam/Proton configured via Nix modules; ensure `hardware.opengl`, `vulkan` enablement.

### Track B: **Fedora Kinoite/Silverblue + rpm-ostree + Ansible**
**Deliverables:**
- `ansible/site.yaml` (idempotent), `group_vars/host.yaml`  
- `rpm-ostree` layered packages list in `infra/ostree.yaml`  
- `toolbox/` or `distrobox/` definitions for dev containers

**Policies:**
- Prefer containerized dev stacks; keep base OS minimal.  
- Record all `rpm-ostree` changes in YAML; rebase immutable image as needed.

### Track C: **Arch + Ansible**
**Deliverables:**
- `ansible/` playbooks for base system, drivers, dev tools, Steam/Proton.  
- Pacman config in `infra/pacman.yaml`, AUR via `paru` role (noninteractive).

---

## 6) Command Execution Policy

- All scripts start with:
  ```bash
  set -Eeuo pipefail
  trap 'echo "[ERROR] ${BASH_SOURCE[0]}:${LINENO}" >&2' ERR
  ```
- Accept `--dry-run` and `--apply`.  
- For `sudo` actions, require a preceding comment line in the task file:
  ```
  # @confirm:reason="install GPU driver", scope="system"
  ```
- After actions, append a record to `logs/changes.log` and update `STATE.json`.

---

## 7) File & Directory Layout (created at bootstrap)

```
llm-laptop/
  plan/
    CONSENSUS.md
    DECISION_RECORDS/
  infra/
  scripts/
  reports/
  logs/
  survey/
  constraints/
  preferences/
  secrets/           (gitignored)
  FACTS.json
  STATE.json
```

---

## 8) Prompt & Task Templates for the Agent

### 8.1 Ingestion Task
```
TASK: Normalize all files in survey/ to infra/sources/survey-normalized.jsonl
GOAL: Extract structured recommendations and rationales.
OUTPUTS: survey-summary.md, survey-normalized.jsonl
CONSTRAINTS: Lossless capture of original claims; annotate strength_of_claim.
```

### 8.2 Consensus Plan Task
```
TASK: Synthesize CONSENSUS.md with ranked recommendations.
GOAL: Select primary and secondary OS track with clear tradeoffs.
REQUIRES: FACTS.json, preferences.yaml, survey-normalized.jsonl
OUTPUTS: plan/CONSENSUS.md, ADRs
```

### 8.3 Provision Minimum System
```
TASK: Render infra/ for chosen track and provision the Minimum System.
GOAL: Reach a state where Codex CLI and core toolchains are usable.
CONFIRM: @confirm:system-change
OUTPUTS: STATE=MINIMUM_READY, logs/, reports/verify.md
```

### 8.4 Verification
```
TASK: Validate GPU stack, Steam/Proton, and dev toolchains.
GOAL: All checks pass; produce verification report.
OUTPUTS: reports/verify.md, tag v0.MIN
```

---

## 9) Hardware Detection & Driver Rules

- If NVIDIA detected (`lspci | grep -i nvidia`):  
  - Track A: use proprietary driver module via NixOS options; enable `nvidia-persistenced`, CUDA optional.  
  - Track B/C: install `akmods`/`dkms` path or distro‑appropriate proprietary driver; verify `nvidia-smi`.  
- If AMD: ensure Mesa, `vulkan-radeon`, RADV; optional ROCm if supported.  
- If Intel: Mesa, `vulkan-intel`.  
- Always validate with `vulkaninfo` and sample `glxinfo -B`.

---

## 10) Gaming Checklist

- Steam installed and launched at least once (creates compatdata).  
- Proton version pinned (GE builds allowed).  
- Gamescope optional; MangoHud overlay toggled and logged.  
- WINE prefix test app (Notepad++) runs; DXVK / VKD3D present where needed.  
- Controller mapping verified.

---

## 11) Dev Environment Checklist

- Python: `uv`/`pipx` tooling; `ruff`, `black`, `mypy`, `pytest`.  
- Rust: `rustup`, `clippy`, `rust-analyzer`.  
- Node: `fnm`, LTS & current; `pnpm`/`npm`.  
- Java: `sdkman` + Temurin.  
- Containers: Podman/Docker; `distrobox`/`toolbox` (B) or Nix devShells (A).  
- Editors: VS Code (+ extensions), optional JetBrains.  
- Git: signing configured; LFS if needed.

---

## 12) Logging & Artifacts

- Each phase emits `reports/*.md` and appends to `logs/commands-YYYYMMDD-HHMMSS.log`.  
- `reports/metrics.json` captures basic benchmarks (compile hello‑world in C/Rust, run `uname -a`, `fio` quick test).  
- Snapshot: tag Git with `v0.MIN`, archive `infra/` + `plan/`.

---

## 13) Rollback & Recovery

- Track A: `nix profile history` / `nixos-rebuild --rollback`; keep previous generations.  
- Track B: `rpm-ostree rollback` to previous deployment.  
- Track C: Pacman log + Ansible re‑apply.  
- Always retain the previous kernel one bootable entry.

---

## 14) Non‑Goals (for now)

- Multi‑boot with Windows.  
- Full disk re‑partitioning or LUKS conversion (unless explicitly confirmed).  
- Kernel custom compilation beyond distro defaults (unless performance requires it).

---

## 15) Success Criteria

- **Minimum System** with Codex CLI operational within 2 hours of Phase 0 start.  
- Verified GPU stack, Steam/Proton, and at least one WINE app.  
- Dev toolchains compile and run a sample project.  
- All state captured as declarative config and reproducible from repo.

---

### End of AGENTS.md