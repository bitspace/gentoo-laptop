# Phase 1 — Ingest & Normalize Plan

**Goal:** Parse all raw LLM survey responses in `llm-responses/`, normalize their content into structured JSON Lines, and produce a summary report highlighting recommendation frequencies and conflicts.

## Inputs
- `llm-laptop/llm-responses/` (Markdown, text, JSON outputs from various LLMs)

## Outputs
- `llm-laptop/infra/sources/survey-normalized.jsonl` (one JSON object per document)
- `llm-laptop/reports/survey-summary.md`

## Confirmation Requirements
- No destructive actions; standard user privileges suffice.

## Tasks

### 1.1 Ensure Destination Directory Exists
```bash
mkdir -p llm-laptop/infra/sources
``` 

### 1.2 Normalize Each LLM Response
- For each file in `llm-laptop/llm-responses/`, extract metadata and recommendations according to the schema:
  ```jsonl
  {
    "model": "provider/model@version",
    "date_utc": "YYYY-MM-DDTHH:MM:SSZ",
    "topic_tags": ["distro","gaming","devtools"],
    "strength_of_claim": 1,
    "citations": [],
    "verbatim": "…raw snippet…",
    "extracted_recommendations": [
      {"area":"OS","option":"NixOS","rationale":"…","risks":["…"]},
      {"area":"GPU","option":"NVIDIA proprietary + CUDA","rationale":"…"}
    ]
  }
  ```
  Use a Python or shell script under `scripts/` with `--dry-run`/`--apply` flags to implement this step.

### 1.3 Cluster & Tag Topics
- Apply rule-based or lightweight topic clustering to assign `topic_tags` to each entry, e.g. by keyword matching in `verbatim`.

### 1.4 Emit Normalized JSONL
- Append each normalized JSON object to `infra/sources/survey-normalized.jsonl`, ensuring idempotency (e.g. overwrite or re-create file).

### 1.5 Generate Survey Summary Report
- Count frequencies of each recommendation-option pairing and detect conflicting recommendations.
- Produce `reports/survey-summary.md` outlining:
  - Top N most frequent recommendations
  - Areas of high consensus vs conflict
  - Summary tables and simple bar charts (ASCII or Markdown)

### 1.6 Checkpoint State
- Update `STATE.json` phase to `INGESTED` with timestamp:
  ```bash
  jq '.phase="INGESTED" | .timestamp=now' STATE.json >STATE.tmp && mv STATE.tmp STATE.json
  ```

---

*Next:* Phase 2 — Consensus Synthesis
