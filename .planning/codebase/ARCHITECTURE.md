---
last_mapped_commit: 6b3ea99b2988d89fc2fac89d60787054d80864b7
last_mapped_at: 2026-09-26
---
<!-- refreshed: 2026-09-26 -->

# Architecture

**Analysis Date:** 2026-09-26

## System Overview

```text
┌─────────────────────────────────────────────────────────────────────────────┐
│                          Entry Points & Execution                           │
│  `src/main.py` (--part 2, 3, 4)  │  `scripts/grade.py`  │  `pytest tests/`  │
└──────────────────────────────┬──────────────────────────────┬───────────────┘
                               │                              │
                               ▼                              ▼
┌───────────────────────────────────────────┐  ┌──────────────────────────────┐
│       Blue Pipeline (Defense-in-Depth)    │  │    Adversarial Red Team      │
│          `src/assignment/pipeline.py`     │  │    `src/attacks/attacks.py`  │
├───────────────────────────────────────────┤  ├──────────────────────────────┤
│ 1. RateLimiter (`RateLimitPlugin`)        │  │ • Red Default (`agent.py`)   │
│ 2. InputGuardrail (`InputGuardrailPlugin`)│  │   (vulnerable, no guards)     │
│ 3. Core Agent (`create_blue_agent`)       │  │ • Red Advance                │
│    (OpenRouter `liquid/lfm-2.5-2.6b`)     │  │   (`guards_agent.py`)        │
│ 4. OutputGuardrail (`OutputGuardPlugin`)  │  │   (hardened defensive agent) │
│ 5. Egress Gateway (`is_egress_allowed`)   │  │                              │
└─────────────────────┬─────────────────────┘  └──────────────┬───────────────┘
                      │                                       │
                      ▼                                       ▼
┌───────────────────────────────────────────┐  ┌──────────────────────────────┐
│        Observability & Telemetry          │  │       Storage & Artifacts    │
│  • `src/assignment/audit_log.py`          │  │  • `outputs/results.json`    │
│  • `src/assignment/monitoring.py`         │  │  • `outputs/attack_results`  │
│  • `schemas/results.schema.json`          │  │  • `data/protected/`         │
└───────────────────────────────────────────┘  └──────────────────────────────┘
```

## Component Responsibilities

| Component | Responsibility | File |
|-----------|----------------|------|
| Main CLI Runner | Dispatches Checkpoints 2, 3, and 4 executions | `src/main.py` |
| Defense Pipeline | Assembles plugins, executes evaluation suite, enforces egress gates | `src/assignment/pipeline.py` |
| Input Guardrails | Normalizes text, detects prompt injections, enforces banking topics | `src/guardrails/input_guardrails.py` |
| Output Guardrails | Detects and redacts PII, credentials, sensitive leaks | `src/guardrails/output_guardrails.py` |
| Rate Limiter | Sliding-window per-user request throttling | `src/assignment/rate_limiter.py` |
| Audit Logger | Captures chronological request/response traces with latency and layer decisions | `src/assignment/audit_log.py` |
| Monitoring & Alerts | Computes anomaly rates (block rate, judge fail rate) and emits threshold alerts | `src/assignment/monitoring.py` |
| Agent Factories | Instantiates Blue agent (OpenRouter) and Red agent (OpenAI / Gemini) | `src/agents/agent.py` |
| Red Advance Agent | Hardened target agent with reference defenses for red-teaming | `src/agents/guards_agent.py` |
| Attack Suite | Orchestrates adversarial injection payloads against Red targets | `src/attacks/attacks.py` |
| Config & Runtime | Provider switching, API credentials, OpenRouter/OpenAI shims | `src/core/config.py`, `src/core/openai_runtime.py` |
| Evaluation Grader | Grades output schemas and compliance | `scripts/grade.py` |

## Pattern Overview

**Overall:** Multi-layered Defense-in-Depth Pipeline with Plugin Interception Architecture.

**Key Characteristics:**

- **Layered Interception:** Requests flow through discrete, decoupled plugins (`RateLimiter` → `InputGuardrails` → `Agent` → `OutputGuardrails` → `EgressGateway`).
- **Fail-Safe Gateways:** Status conventions use unambiguous `"ALLOW"` and `"BLOCK"` return values instead of potentially ambiguous booleans.
- **Provider Dual-Track:** Strict separation between defense model (OpenRouter Liquid 2.5-2.6b) and attack models (OpenAI/Gemini).
- **Asynchronous Execution:** Async-first request processing supporting concurrent evaluation and simulation.

## Layers

**Entry Layer (`src/main.py`, `scripts/grade.py`):**

- Purpose: CLI interface and execution dispatcher for checkpoints and automated grading.
- Depends on: Pipeline, Guardrails, Agents, Attacks.

**Defense & Guardrails Layer (`src/guardrails/`, `src/assignment/`):**

- Purpose: Inspects, sanitizes, and controls inputs and outputs before and after the LLM executes.
- Location: `src/guardrails/input_guardrails.py`, `src/guardrails/output_guardrails.py`, `src/assignment/pipeline.py`.
- Depends on: `google.adk.plugins.base_plugin`, `src/core/config.py`.

**Agent & Model Runtime Layer (`src/agents/`, `src/core/`):**

- Purpose: Manages LLM connections, prompts, system instructions, and mock secret context.
- Location: `src/agents/agent.py`, `src/agents/guards_agent.py`, `src/core/openai_runtime.py`.
- Depends on: `openai`, `google-adk`, `google-genai`.

**Observability Layer (`src/assignment/`):**

- Purpose: Records comprehensive audit trails and triggers alerts on security metric threshold violations.
- Location: `src/assignment/audit_log.py`, `src/assignment/monitoring.py`.
- Output: `outputs/audit_log.json`, `outputs/metrics.json`.

## Data Flow

### Primary Request Path (Checkpoint 3 Defense Pipeline)

1. **User Request Arrival:** User input and `user_id` enters pipeline (`src/assignment/pipeline.py:run_assignment_suite`).
2. **Rate Limit Inspection:** `RateLimitPlugin.on_user_message_callback` checks user timestamp window; blocks if requests exceed `max_requests`.
3. **Input Guardrail Inspection:** `InputGuardrailPlugin.on_user_message_callback` normalizes input, checks for injection patterns (`detect_injection`), and filters topics (`topic_filter`). Blocks on violation.
4. **Audit Input Logging:** `AuditLogPlugin.record_input` stores request timestamp and prompt.
5. **LLM Generation:** Blue Agent (`create_blue_agent`) queries OpenRouter `liquid/lfm-2.5-2.6b`.
6. **Output Guardrail Inspection:** `OutputGuardrailPlugin.after_model_callback` runs `content_filter` to identify and redact PII or sensitive keys.
7. **Egress Check:** `is_egress_allowed` validates destination and payload before external transmission.
8. **Audit Output & Telemetry:** `AuditLogPlugin.record_output` captures execution latency, final text, and intercept layer; `MonitoringAlert` updates counters.

### Secondary Flow (Checkpoint 4 Adversarial Attack Benchmarking)

1. Adversarial prompts loaded from `src/attacks/attacks.py`.
2. Attack runner submits payload to Red Agent (`create_red_agent_default`) and Red Advance (`create_red_agent_advance`).
3. Responses evaluated by `classify_attack_outcome` and `response_leaked_secrets`.
4. Benchmarks recorded in `outputs/attack_results.json`.

**State Management:**

- In-memory deques and dictionaries per user for sliding window rate limiting.
- Volatile accumulator lists for audit records and metrics snapshots prior to JSON export.

## Key Abstractions

**ADK BasePlugin:**

- Purpose: Standardized lifecycle interception hook for agent interactions.
- Examples: `InputGuardrailPlugin`, `OutputGuardrailPlugin`, `RateLimitPlugin`.
- Pattern: Hook / Interceptor pattern.

**Observability Recorders:**

- Purpose: Framework-agnostic telemetric recorders for compliance and forensics.
- Examples: `AuditLogPlugin`, `MonitoringAlert`.
- Pattern: Observer / Aggregator.

## Entry Points

**`src/main.py`:**

- Triggers: CLI invocation (`python src/main.py [--part 2|3|4]`).
- Responsibilities: Executes individual checkpoints or the end-to-end laboratory run.

**`scripts/grade.py`:**

- Triggers: Automated or manual grading (`python scripts/grade.py --submission-dir . --out outputs/grade_report.json`).
- Responsibilities: Validates output artifacts against `schemas/results.schema.json` and runs public pytest contracts.

## Architectural Constraints

- **Single Blue Provider Constraint:** Blue agent must strictly use OpenRouter `liquid/lfm-2.5-2.6b` (locked in `src/core/config.py`).
- **Deterministic Egress:** Egress decisions must be programmatic (`is_egress_allowed`) and never delegated to LLM prompt persuasion.
- **Path Portability:** All output file paths must be resolved relative to repository root (`<repo>/outputs/`) so execution from `src/` does not pollute subdirectories.

## Error Handling

**Strategy:**

- Guardrail failures produce structured canned refusal responses rather than unhandled exceptions.
- Pipeline catches `NotImplementedError` with informative student guidance.

---

*Architecture analysis: 2026-09-26*
