## Why

Resident users querying legitimate financial data for their own apartment unit (e.g. asking for charges in "junio de 2026 y octubre de 2026") are experiencing false-positive RBAC security blocks (`rbac_unauthorized_access`). The current text-level regex heuristic in `_detect_unauthorized_unit_in_text` mistakenly interprets Spanish conjunctions and calendar years like `"2026 y"` as unauthorized apartment units (e.g., `Y-2026`), triggering security strikes that can lock residents out for 24 hours. Maintaining an ever-expanding blacklist of Spanish stopwords and regex edge cases is fragile and unsustainable. This change shifts tenant unit validation directly to the LLM Supervisor via structured outputs, providing scalable, semantic understanding of resident query scope.

## What Changes

- **Add structured RBAC fields to `SupervisorDecision`**: Extend the supervisor output contract in `backend/app/agent/state.py` with `is_unauthorized_unit_access` and `unauthorized_target_unit`.
- **Update Supervisor system prompt**: Update `backend/app/agent/prompts/supervisor_system.md` with explicit RBAC evaluation rules so Gemini identifies when a resident explicitly asks about another neighbor's unit versus temporal expressions or general building spend.
- **Remove heuristic regex unit scanner**: Deprecate and remove `_detect_unauthorized_unit_in_text` from `backend/app/agent/tools.py`.
- **Enforce deterministic Python RBAC enforcement**: Validate resident access using the LLM's structured decision and explicit tenant unit comparison (`decision.unit_code != user_context.unit_code`).
- **Comprehensive test coverage**: Add regression test cases covering multi-period Spanish resident queries, years with conjunctions, and legitimate unauthorized access attempts.

## Capabilities

### New Capabilities
- `resident-unit-rbac`: Semantic validation of tenant unit boundaries and deterministic RBAC enforcement for resident users, replacing text-level regex scanning with structured LLM classification.

### Modified Capabilities
<!-- None: openspec/specs/ is currently empty -->

## Impact

- **Backend Agent (`backend/app/agent/`)**:
  - `state.py`: Modified `SupervisorDecision` schema.
  - `prompts/supervisor_system.md`: Enhanced RBAC classification instructions.
  - `tools.py`: Removal of `_detect_unauthorized_unit_in_text` and streamlined RBAC checks in `lookup_unit_bill`, `lookup_spend_evolution`, and `run_deep_financial_analysis`.
  - `supervisor.py`: Propagation of structured RBAC decision to tool execution and audit logging.
- **Security & Reliability**:
  - Eliminates false-positive Layer B security strikes and 24-hour lockout risk for legitimate resident questions.
  - Zero cross-submodule impact (frontend schema remains decoupled via REST contract).
