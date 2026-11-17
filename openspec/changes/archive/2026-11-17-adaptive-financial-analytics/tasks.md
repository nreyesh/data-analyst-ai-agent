## 1. Planner System Prompt Updates

- [x] 1.1 Update `backend/app/agent/analytics/prompts/planner_system.md` to decouple `requires_visualization` from `variance_driver`, making visualization conditional on multi-period trends or explicit chart requests.
- [x] 1.2 Update `backend/app/agent/analytics/prompts/planner_system.md` to allow multi-period sub-analyses for apartment units when multiple target months are referenced.

## 2. Scope-Bounded Trajectory Inspection

- [x] 2.1 Update `trajectory_inspector_node` in `backend/app/agent/analytics/nodes.py` to filter SQL queries by `time_scope` (e.g., matching `p.period_year`).
- [x] 2.2 Add unit tests verifying that trajectory inspection restricts date ranges according to `time_scope`.

## 3. Proportional Financial Synthesizer Prompt Updates

- [x] 3.1 Update `backend/app/agent/analytics/prompts/synthesizer_system.md` to remove the mandatory 3-chapter report template and establish the Proportionality Principle for follow-up and direct questions.
- [x] 3.2 Update `backend/app/agent/analytics/prompts/synthesizer_system.md` to replace the strict `[Mes Año]` bracketed citation requirement with natural, fluid date expressions in Spanish.

## 4. Supervisor Domain Boundary & Direct Response Enablement

- [x] 4.1 Update `backend/app/agent/prompts/supervisor_system.md` to authorize `direct_response` for non-financial or operational status queries.
- [x] 4.2 Remove the defensive `direct_response` hijacking in `backend/app/agent/supervisor.py` (lines 724–735).
- [x] 4.3 Add regression tests in `backend/tests/test_supervisor_and_workflow.py` validating direct response for operational inquiries and chart bypass for focused variance inquiries.

## 5. Verification & Coverage Validation

- [x] 5.1 Run backend test suite (`PYTHONPATH=backend pytest backend/tests/ -v --cov=backend/app --cov-fail-under=65`) and verify zero regressions and >=65% coverage.
