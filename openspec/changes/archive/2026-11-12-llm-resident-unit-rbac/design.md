## Context

The conversational agent in `backend/app/agent/` features a two-tier routing and execution model:
1. **Conversational Supervisor**: Inspects conversation history and user context using Gemini with a structured Pydantic schema (`SupervisorDecision`) and system instructions (`prompts/supervisor_system.md`).
2. **Tool Execution Layer**: Fast-path snapshot tools (`lookup_unit_bill`, `lookup_spend_evolution`) and a 9-node deep analytics LangGraph workflow (`run_deep_financial_analysis`).

Currently, `backend/app/agent/tools.py` implements a helper function `_detect_unauthorized_unit_in_text(query, assigned_unit)` that inspects raw query strings using regular expressions:
`r"\b(\d{1,4})\s*([A-Za-z]{1,2})\b"`
Because Spanish queries frequently combine 4-digit years with prepositions or conjunctions (e.g. `"2026 y"`, `"2026 o"`, `"de 2026"`), this regex matches `"2026"` as an apartment number and `"y"` as a tower identifier (`Y-2026`). Since resident `B-504` is in Tower B, `"Y" != "B"` triggers a false-positive `rbac_unauthorized_access` violation, recording a security strike in `security_audit_logs`.

Attempting to patch this with regex stopwords (e.g. blacklisting `"Y"`, `"DE"`, `"A"`, etc.) is unsustainable and fragile. Instead, we transition unit boundary validation directly to the LLM's structured output contract.

## Goals / Non-Goals

**Goals:**
- Eliminate false-positive RBAC security violations and unwarranted guardrail strikes for legitimate resident queries.
- Utilize the LLM's native semantic comprehension to classify whether a resident is attempting unauthorized access to another tenant's unit.
- Retain strict deterministic Python enforcement: Python verifies tenant boundaries before any database query or workflow execution.
- Completely remove fragile regex text scanning for unit codes from `backend/app/agent/tools.py`.

**Non-Goals:**
- Altering the permissions or capabilities of the `admin` role (admins remain unrestricted).
- Changing frontend schemas or REST endpoint signatures (`/api/chat/turn` contract remains intact).
- Modifying the 3-strike / 24-hour lockout security policy (the policy remains active, but protected against false-positive triggers).

## Decisions

### Decision 1: Extend `SupervisorDecision` with Explicit RBAC Fields
Extend `SupervisorDecision` in `backend/app/agent/state.py`:
```python
is_unauthorized_unit_access: bool = Field(
    default=False,
    description=(
        "True ONLY if user_context.role is 'resident' and the user explicitly requests data, "
        "bills, or comparisons for an apartment unit or tower OTHER than their assigned unit_code. "
        "MUST be False if querying their own unit, building-wide expenses, or general concepts."
    ),
)
unauthorized_target_unit: Optional[str] = Field(
    default=None,
    description="The unauthorized unit or tower code requested by the resident (e.g. 'A-101', 'Torre A'). None if authorized.",
)
```
*Rationale*: Gemini processes the full conversational context, role, and natural language semantics with high accuracy. It naturally understands that `"junio de 2026 y octubre de 2026"` refers to dates, not apartment units.

*Alternatives considered*:
- *Expanding regex exclusion lists*: Rejected as brittle, hard to maintain, and prone to endless language edge cases.
- *LLM pre-sanitization node*: Rejected as redundant latency overhead; `supervisor_router_node` already runs on every conversational turn.

### Decision 2: Deterministic Defense-in-Depth in Python
Even with LLM classification, Python remains the ultimate authority for security enforcement:
1. If `decision.is_unauthorized_unit_access` is `True` or `decision.unauthorized_target_unit` is set, block the request immediately.
2. If `user_context.role == "resident"` and the requested normalized unit differs from `user_context.unit_code` (e.g., `decision.unit_code != user_context.unit_code`), block the request immediately.
3. If `_is_cross_unit_cohort_query(query)` detects multi-unit comparison phrases ("ranking de departamentos", "vecinos"), block the request.

*Rationale*: Zero trust in raw LLM outputs alone. A resident cannot bypass access control even via prompt injection because the deterministic Python check enforces `decision.unit_code == assigned_unit`.

### Decision 3: Complete Removal of `_detect_unauthorized_unit_in_text`
Remove `_detect_unauthorized_unit_in_text` from `backend/app/agent/tools.py` and replace all calls in `lookup_unit_bill`, `lookup_spend_evolution`, and `run_deep_financial_analysis` with:
- Checking `is_unauthorized_unit_access` / `unauthorized_target_unit` passed from the Supervisor decision.
- The existing deterministic `norm_requested != assigned_unit` check.

## Risks / Trade-offs

- **[Risk] LLM hallucinating unauthorized access on unusual Spanish phrasing**
  $\rightarrow$ *Mitigation*: Update `backend/app/agent/prompts/supervisor_system.md` with explicit instructions and contrast examples distinguishing year/date conjunctions (`"2026 y 2025"`) from unauthorized unit references (`"depto 101"`, `"torre A"`).
- **[Risk] Existing unit test suite compatibility**
  $\rightarrow$ *Mitigation*: Update mock decisions in `test_security_policy.py` and related supervisor tests to ensure they assert on the new structured contract and maintain $\ge 65\%$ test coverage.

## Migration Plan

1. Update `backend/app/agent/state.py` (add `is_unauthorized_unit_access` and `unauthorized_target_unit`).
2. Update `backend/app/agent/prompts/supervisor_system.md` (RBAC rules).
3. Update `backend/app/agent/supervisor.py` (pass RBAC flags into tool execution and turn results).
4. Update `backend/app/agent/tools.py` (remove `_detect_unauthorized_unit_in_text` and enforce via structured flags).
5. Run pytest test suite: `PYTHONPATH=backend backend/.venv/bin/pytest backend/tests/ -v`.
