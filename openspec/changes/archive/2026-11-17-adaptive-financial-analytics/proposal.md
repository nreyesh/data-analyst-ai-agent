## Why

Online evaluation of the financial assistant for Chilean condominium residents scored a conversational follow-up query (*"¿por qué subió mi gasto común en junio y octubre?"*) at 3/10. The system produced an overengineered, 3-section executive report with excessive repetition of bracketed dates (`[Junio 2026]`), mandatory Matplotlib visualization for a simple follow-up, and hallucinated historical commentary (dating back to 2024), while failing to query October details. Furthermore, the supervisor currently forces all non-greeting inquiries into the heavy analytics pipeline, preventing honest, direct answers when users ask about non-financial or operational matters (such as physical progress of repairs).

This change transforms the analytics pipeline into an adaptive, proportional agentic workflow that matches depth to user intent, scopes queries strictly to the requested time horizon, bypasses unnecessary charts, adopts natural Spanish synthesis, and empowers the supervisor to honestly decline non-financial operational questions.

## What Changes

- **Adaptive Analytical Planning**: Update `planner_system.md` to decouple visualization from `variance_driver` (only generating charts when multi-period trends or user requests justify it) and remove the single-sub-analysis constraint on unit queries so multiple target periods (e.g., June and October) are fully analyzed.
- **Scope-Bounded Trajectory Inspection**: Update `trajectory_inspector_node` in `backend/app/agent/analytics/nodes.py` to filter macro queries by the plan's `time_scope` instead of unbounded `SELECT ... ORDER BY period_date` across the entire database history.
- **Proportional, Natural Financial Synthesis**: Update `synthesizer_system.md` to remove the mandatory 3-chapter executive template and the strict inline bracketed date citation rule (`[Mes Año]` on every amount). Replace them with a proportionality principle: direct, concise answers for targeted follow-ups and fluid, natural date expressions.
- **Honest Out-of-Domain Boundaries in Supervisor**: Update `supervisor_system.md` and remove the hardcoded `direct_response` hijacking in `backend/app/agent/supervisor.py` (line 724), allowing the supervisor to issue direct, helpful responses that clarify its financial scope when asked about physical project progress, building rules, or non-accounting matters.

## Capabilities

### New Capabilities
- `adaptive-financial-analytics`: Context-aware analytical planning, dynamic chart bypass, scope-bounded trajectory inspection, and proportional natural synthesis for resident and administrative queries.
- `supervisor-domain-boundaries`: Clear demarcation of financial vs. non-financial condominium inquiries, allowing direct, honest scope explanations without forced tool execution.

### Modified Capabilities
<!-- None: existing storage schema and resident RBAC invariants remain unchanged -->

## Impact

- **Backend Agent**:
  - `backend/app/agent/analytics/prompts/planner_system.md`
  - `backend/app/agent/analytics/prompts/synthesizer_system.md`
  - `backend/app/agent/analytics/nodes.py` (`trajectory_inspector_node`)
  - `backend/app/agent/prompts/supervisor_system.md`
  - `backend/app/agent/supervisor.py` (`supervisor_router_node`)
- **Frontend / Contracts**:
  - Zero breaking changes to `front/models.py` or existing REST endpoints. All supervisor routes and schemas remain backward-compatible.
- **Performance**:
  - Follow-up causal queries drop from ~15-20s (with sandbox image rendering) to ~3-4s (bypassing visualizer/sandbox and unneeded historical data).
