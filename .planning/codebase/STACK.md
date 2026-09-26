---
last_mapped_commit: 6b3ea99b2988d89fc2fac89d60787054d80864b7
last_mapped_at: 2026-09-26
---
# Technology Stack

**Analysis Date:** 2026-09-26

## Languages

**Primary:**

- Python 3.10+ (Recommended: 3.11 or 3.12) - Entire backend codebase across `src/`, `tests/`, and `scripts/`. Type hints with `from __future__ import annotations` utilized throughout.

**Secondary:**

- JSON / JSON Schema (Draft 7) - Schema validation contracts (`schemas/results.schema.json`), test suites, and structured metrics/results outputs (`outputs/*.json`).

## Runtime

**Environment:**

- Python 3.11/3.12 virtual environment (`.venv`) on Windows / POSIX environments.

**Package Manager:**

- `pip` with `requirements.txt`
- Lockfile: missing (direct specification in `requirements.txt`)

## Frameworks

**Core:**

- Google Agent Development Kit (`google-adk>=0.3.0`, installed `google-adk==2.10.0`) - Provides agent abstractions (`llm_agent.LlmAgent`), plugin architecture (`base_plugin.BasePlugin`), context managers (`InvocationContext`), and execution engines (`runners.InMemoryRunner`).
- Google GenAI SDK (`google-genai>=1.0.0`, installed `google-genai==2.25.0`) - Low-level types (`types.Content`, `types.Part`) for Gemini models and ADK plugin intercepts.
- OpenAI Python SDK (`openai>=1.40.0`, installed `openai==3.19.2`) - Used directly for OpenAI API access (`gpt-4o-mini`, `gpt-5.6-luna`) and wrapped runtime for OpenRouter API (`liquid/lfm-2.5-2.6b`).

**Testing:**

- Pytest (`pytest>=8.0.0`) - Test runner configured via `pytest.ini`.
- JSONSchema (`jsonschema>=4.0.0`) - Schema validation for defense results contract.

**Build/Dev:**

- Standard Library (`asyncio`, `argparse`, `re`, `dataclasses`, `pathlib`, `collections`, `time`) - Zero-overhead native concurrency and state management.
- Python-dotenv (`python-dotenv>=1.0.0`) - Loading environment variables from `.env`.

## Key Dependencies

**Critical:**

- `google-adk`: Defines plugin lifecycle hooks (`on_user_message_callback`, `after_model_callback`) used by guardrails and rate limiting.
- `openai`: Client runtime backing both the Blue agent (via OpenRouter OpenAI-compatible endpoint) and Red agents.
- `jsonschema`: Strict verification of `outputs/results.json` ensuring defense suite outputs conform to grading criteria.

**Infrastructure:**

- `python-dotenv`: Automatic discovery and injection of credentials from `.env`.

## Configuration

**Environment:**

- Managed via `src/core/config.py` with automatic fallback and default provider mappings.
- Key configs required:
  - `OPENROUTER_API_KEY`: Required for Blue agent (`liquid/lfm-2.5-2.6b`).
  - `RED_TEAM_PROVIDER`: Selected red team provider (`openai` or `gemini`).
  - `OPENAI_API_KEY` (if using OpenAI provider) with `OPENAI_MODEL` (`gpt-4o-mini` default).
  - `GOOGLE_API_KEY` (if using Gemini provider) with `GEMINI_MODEL` (`gemini-3.5-flash` default).

**Build:**

- `pytest.ini`: Configures test discovery paths (`tests/smoke`), sets `pythonpath = src`, and suppresses deprecation warnings.
- `requirements.txt`: Root dependency declaration file.

## Platform Requirements

**Development:**

- Python 3.10+ (tested on Python 3.12 / 3.11).
- PowerShell / Bash shell with virtual environment activated.
- Internet connectivity to OpenRouter (`https://openrouter.ai/api/v1`) and OpenAI / Google AI Studio endpoints.

**Production:**

- Headless Python runner executing batch evaluation pipelines (`python src/main.py`).

---

*Stack analysis: 2026-09-26*
