---
last_mapped_commit: 6b3ea99b2988d89fc2fac89d60787054d80864b7
last_mapped_at: 2026-09-26
---
# Codebase Concerns

**Analysis Date:** 2026-09-26

## Tech Debt

**Starter Implementation Stubs:**

- Issue: Core defense logic is currently unimplemented (`NotImplementedError` stubs).
- Files:
  - `src/guardrails/input_guardrails.py` (`detect_injection`, `topic_filter`, `InputGuardrailPlugin`)
  - `src/guardrails/output_guardrails.py` (`content_filter`, `OutputGuardrailPlugin`)
  - `src/assignment/pipeline.py` (`is_egress_allowed`, `build_production_plugins`, `build_observability`, `run_assignment_suite`)
  - `src/assignment/rate_limiter.py` (`RateLimitPlugin.on_user_message_callback`)
  - `src/assignment/audit_log.py` (`record_input`, `record_output`, `export_json`)
  - `src/assignment/monitoring.py` (`check_metrics`, `export_json`)
- Impact: Running `python src/main.py` directly fails at Checkpoint 2 and Checkpoint 3. Public contract tests currently fail until stubs are implemented.
- Fix approach: Implement functions according to the specifications in `CHECKPOINTS.md` and verify against `pytest tests/public -q`.

## Security Considerations

**API Key Protection & Git Exposure:**

- Risk: Exposing `OPENROUTER_API_KEY`, `OPENAI_API_KEY`, or `GOOGLE_API_KEY` on public GitHub repositories.
- Files: `.env` (must remain gitignored).
- Current mitigation: `.gitignore` includes `.env`.
- Recommendations: Always verify `.gitignore` before committing; ensure no hardcoded API keys exist in script files.

**Adversarial Prompt Injection & Normalization Bypasses:**

- Risk: Adversarial attackers using Unicode homoglyphs, invisible spacing (e.g. zero-width spaces `\u200b`), or base64 encoding to bypass regex-based input guardrails.
- Files: `src/guardrails/input_guardrails.py`.
- Current mitigation: Starter comments emphasize the need for Unicode normalization prior to regex pattern evaluation.
- Recommendations: Implement canonicalization (`unicodedata.normalize`) and strip zero-width characters before evaluating injection regex patterns.

**Secret Leakage Prevention:**

- Risk: Blue agent disclosing internal credentials from system prompt or context (`data/protected/vinbank_secrets.json`).
- Files: `src/guardrails/output_guardrails.py`, `src/agents/agent.py`.
- Current mitigation: Output content filter redacts matched patterns with `[REDACTED]`.
- Recommendations: Output filter should match regex variants (case-insensitive, normalized) for known sensitive parameters.

## Fragile Areas

**Current Working Directory Dependency:**

- Files: `src/assignment/pipeline.py`, `src/assignment/audit_log.py`, `src/assignment/monitoring.py`, `scripts/grade.py`.
- Why fragile: If scripts construct paths using `Path("outputs/...")` without anchoring to the repository root, executing from `src/` writes files into `src/outputs/` rather than the required `<repo>/outputs/`.
- Safe modification: Consistently anchor paths using `Path(__file__).resolve().parents[...]` to dynamically resolve the repository root.

**Schema Strictness for `outputs/results.json`:**

- Files: `schemas/results.schema.json`, `src/assignment/pipeline.py`.
- Why fragile: `scripts/grade.py` validates `results.json` with `jsonschema.validate`. Missing keys, extra disallowed properties, or wrong data types cause automated grading validation failure (0 points for schema contract).
- Safe modification: Validate generated dictionary against `schemas/results.schema.json` using `pytest tests/public/test_results_contract.py` before finalizing runs.

## Dependencies at Risk

**Pinned Small LLM Model on OpenRouter:**

- Model: `liquid/lfm-2.5-2.6b` (fixed for Blue Team).
- Risk: Because this is a 2.6B parameter model, its instruction-following reliability under jailbreak pressure is lower than frontier models.
- Impact: Prompt engineering alone in `BLUE_INSTRUCTION` is insufficient to prevent leaks; defense relies critically on deterministic pre/post-filters (`InputGuardrailPlugin`, `OutputGuardrailPlugin`, `is_egress_allowed`).

---

*Concerns audit: 2026-09-26*
