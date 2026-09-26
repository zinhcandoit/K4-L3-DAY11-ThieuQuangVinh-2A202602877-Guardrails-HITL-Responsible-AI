---
last_mapped_commit: 6b3ea99b2988d89fc2fac89d60787054d80864b7
last_mapped_at: 2026-09-26
---
# External Integrations

**Analysis Date:** 2026-09-26

## APIs & External Services

**LLM Inference Providers:**

- **OpenRouter (Blue Team - Locked):**
  - Model: `liquid/lfm-2.5-2.6b`
  - Base URL: `https://openrouter.ai/api/v1`
  - SDK/Client: `openai.OpenAI` configured with custom `base_url` in `src/core/openai_runtime.py` and `src/agents/agent.py`
  - Auth: `OPENROUTER_API_KEY` loaded via `src/core/config.py`

- **OpenAI (Red Team Option 1):**
  - Models: `gpt-4o-mini` (soft default), `gpt-5.6-luna` (hard/bonus)
  - SDK/Client: `openai.OpenAI` in `src/core/openai_runtime.py`
  - Auth: `OPENAI_API_KEY` loaded via `src/core/config.py`

- **Google AI Studio / Gemini (Red Team Option 2):**
  - Models: `gemini-3.5-flash` (soft default), `gemini-3.8-flash` (hard/bonus)
  - SDK/Client: `google-genai` / `google.adk.agents.llm_agent.LlmAgent`
  - Auth: `GOOGLE_API_KEY` loaded via `src/core/config.py`

## Data Storage

**Databases:**

- None (Local JSON mock data only)
- Simulated Database Reference: VinBank internal host fixture defined in `data/protected/vinbank_secrets.json` used strictly as a leak detection sentinel.

**File Storage:**

- Local filesystem only.
- Output artifacts generated into `<repo>/outputs/`:
  - `outputs/results.json`: Checkpoint 3 pipeline evaluation report.
  - `outputs/audit_log.json`: Structured interaction audit log.
  - `outputs/metrics.json`: Performance metrics, block rates, and alerts.
  - `outputs/attack_results.json`: Checkpoint 4 red team attack benchmarks.
  - `outputs/grade_report.json` & `outputs/lab_report.md`: Automated grading summaries.

**Caching:**

- In-memory data structures:
  - `RateLimitPlugin`: Sliding window timestamps managed via `collections.defaultdict(collections.deque)`.
  - `AuditLogPlugin`: Pending request dictionary `self._open: dict[str, float]`.

## Authentication & Identity

**Auth Provider:**

- Custom API Key injection and validation.
- Centralized verification in `src/core/config.py:setup_api_key()` to confirm environment configuration before execution.

## Monitoring & Observability

**Error Tracking:**

- Standard Python exception propagation with graceful degradation messages in `src/main.py`.

**Logs & Metrics:**

- **Audit Logging:** Handled by `AuditLogPlugin` in `src/assignment/audit_log.py`, capturing timestamped inputs, outputs, decisions (`ALLOW`/`BLOCK`), intercepting layers, and latency.
- **Metrics & Alerting:** Handled by `MonitoringAlert` in `src/assignment/monitoring.py`, computing block rates, rate limit hits, judge failures, and firing threshold alerts.

## CI/CD & Deployment

**Hosting:**

- Local execution environment; automated evaluation via `scripts/grade.py`.

**CI Pipeline:**

- GitHub workflows under `.github/workflows/` (if enabled) executing `pytest` contracts and grading checks.

## Environment Configuration

**Required env vars:**

- `OPENROUTER_API_KEY`: API key for OpenRouter Liquid model.
- `RED_TEAM_PROVIDER`: Value `openai` or `gemini`.
- `OPENAI_API_KEY` (if `RED_TEAM_PROVIDER=openai`).
- `GOOGLE_API_KEY` (if `RED_TEAM_PROVIDER=gemini`).

**Secrets location:**

- Stored exclusively in `.env` (gitignored). Template provided in `.env.example`.
- Synthetic demo secrets for red team testing located in `data/protected/vinbank_secrets.json`.

## Webhooks & Callbacks

**Incoming:**

- None (CLI and batch test invocation).

**Outgoing:**

- Egress Policy Gateway: Controlled by `src/assignment/pipeline.py:is_egress_allowed()`, enforcing destination allowlists and sensitive data leak checks before any external sink.

---

*Integration audit: 2026-09-26*
