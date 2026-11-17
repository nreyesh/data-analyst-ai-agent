# Tasks: Dynamic Analytics Timeline Evaluation

## 1. Trajectory Evaluator Prompt De-biasing

- [x] 1.1 Rewrite Section 2, Rule B in `backend/app/agent/analytics/prompts/evaluator_system.md` to eliminate hardcoded dates (`"2026-05-01"`, `"2026-07-01"`) and instruct dynamic baseline/peak period deduction from `macro_trajectory`.
- [x] 1.2 Update Section 1, Point 1 in `evaluator_system.md` to remove restrictive `(including both 2025 and 2026 when present)` phrasing in favor of timeline-agnostic instructions.
- [x] 1.3 Define dynamic fallback behavior in `evaluator_system.md` (e.g., earliest and latest scope periods) when trajectories are monotonic or without distinct surges.

## 2. SQL Coder and Planner Prompt Parameterization

- [x] 2.1 Update Recipe 1 in `backend/app/agent/analytics/prompts/sql_coder_system.md` to replace static `WHERE p.period_year = 2026` with dynamic timeframe filtering matching the query's `time_scope`.
- [x] 2.2 Update Recipe 2 in `sql_coder_system.md` to use generalized year aliases and parameterized join conditions instead of static `r25` / `r26` and `2025` / `2026`.
- [x] 2.3 Update Recipe 3 in `sql_coder_system.md` to use dynamic `<baseline_period>` and `<peak_period>` placeholders in the CTE filters and provenance projections.
- [x] 2.4 Update explanatory comment in Section 4 of `backend/app/agent/analytics/prompts/planner_system.md` to reference the requested period's data instead of "2026 numbers".

## 3. Supervisor Prompt & Python Extraction Updates

- [x] 3.1 Update `backend/app/agent/prompts/supervisor_system.md` line 52 to avoid mapping relative expressions like `"este año"` to static `"2026"`.
- [x] 3.2 Update `_extract_period_from_text` in `backend/app/agent/supervisor.py` to resolve missing years from `datetime.now().year` or conversational context rather than static integer `2026`.

## 4. Verification & Testing

- [x] 4.1 Run backend unit tests (`pytest backend/tests/test_supervisor_and_workflow.py -v`) to confirm routing and prompt integrity.
- [x] 4.2 Run supervisor extraction unit tests (`test_extract_period_dynamic_timeline_resolution` in `test_supervisor_and_workflow.py`) to verify date resolution across various query formats.
