---
last_mapped_commit: 6b3ea99b2988d89fc2fac89d60787054d80864b7
last_mapped_at: 2026-09-26
---
# Coding Conventions

**Analysis Date:** 2026-09-26

## Naming Patterns

**Files:**

- Snake_case for all Python modules: `input_guardrails.py`, `output_guardrails.py`, `rate_limiter.py`.
- Schema files in lowercase kebab/dot notation: `results.schema.json`.

**Functions:**

- Snake_case for functions and methods: `detect_injection()`, `topic_filter()`, `content_filter()`, `build_production_plugins()`.
- Test functions prefixed with `test_`: `test_detect_injection_basic()`.

**Variables & Constants:**

- Variables: snake_case (`user_input`, `blocked_requests`, `window_seconds`).
- Constants: UPPER_SNAKE_CASE (`BLUE_MODEL`, `ALLOWED_TOPICS`, `BLOCKED_TOPICS`, `DEMO_SECRETS`).
- Status Enums / Literals: String literals `"ALLOW"` and `"BLOCK"` (typed as `Literal["ALLOW", "BLOCK"]` in `src/guardrails/input_guardrails.py`) to avoid inverted boolean ambiguities.

**Types & Classes:**

- PascalCase for all classes: `RateLimitPlugin`, `AuditLogPlugin`, `MonitoringAlert`, `InputGuardrailPlugin`.

## Code Style

**Formatting:**

- PEP 8 standard formatting across all Python source files.
- Modern typing with `from __future__ import annotations` at the top of every module.

**Type Annotations:**

- Full type annotations for public functions: parameter types, optional types (`str | None`), and explicit return types (`dict`, `bool`, `InputStatus`).

## Import Organization

**Order:**

1. Future imports: `from __future__ import annotations`.
2. Standard library: `import os`, `import sys`, `import re`, `import json`, `import time`, `from pathlib import Path`, `from collections import defaultdict, deque`.
3. Third-party packages: `from google.genai import types`, `from google.adk.plugins import base_plugin`, `from openai import OpenAI`.
4. Local project modules: `from core.config import ...`, `from guardrails.input_guardrails import ...`.

**Path Safety Convention:**

- Always resolve filesystem paths relative to the repository root using `Path(__file__).resolve().parents[...]` to prevent accidental writes into `src/outputs/` when invoking commands from subdirectories.
  ```python
  repo_root = Path(__file__).resolve().parents[2]
  output_path = repo_root / "outputs" / "results.json"
  ```

## Error Handling

**Patterns:**

- Starter stubs raise `NotImplementedError("Implement <function_name>")` to guide implementation.
- Guardrails return explicit block content (`_block_response`) or classification dictionaries (`{"safe": False, "issues": [...]}`) rather than raising unhandled runtime exceptions.
- CLI scripts wrap execution in graceful `try/except` blocks informing students of unfinished checkpoints.

## Logging & Observability

**Framework:**

- Structured JSON serialization for all outputs.
- `AuditLogPlugin`: Captures timestamped interaction traces, execution latency, and blocking layers.
- `MonitoringAlert`: Gathers counters (`total_requests`, `blocked_requests`, `rate_limit_hits`) and evaluates thresholds.

## Comments

**When to Comment:**

- Clear docstrings with `Args:` and `Returns:` for API contracts.
- Explicit inline comments marking student task boundaries and rubric criteria (e.g., `# TODO: Implement sliding window`).

---

*Convention analysis: 2026-09-26*
