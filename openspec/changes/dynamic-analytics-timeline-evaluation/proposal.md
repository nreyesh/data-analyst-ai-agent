# Change: Dynamic Analytics Timeline Evaluation

## Why

The Deep Analytics Engine's Trajectory Evaluator prompt (`evaluator_system.md`) currently contains hardcoded dates (`"2026-05-01"` and `"2026-07-01"`) in Section 2, Rule B, mandating exact baseline and peak period values. Additionally, downstream SQL recipes in `sql_coder_system.md` and supervisor routing heuristics in `supervisor_system.md` and `supervisor.py` contain literal 2025/2026 anchors and fallbacks. 

This hardcoded conditioning deprives the agent of autonomy to inspect chronological trajectories dynamically, and causes severe cognitive conflict and invalid SQL/analysis generation whenever a user asks about alternative years, specific quarters, or custom date ranges (e.g., 2024, 2025, or multi-year historical queries). Removing hardcoded date bias restores full temporal neutrality and enables autonomous, data-driven timeline inference.

## What Changes

- **Trajectory Evaluator Prompt Redesign**:
  - Remove hardcoded dates (`"2026-05-01"`, `"2026-07-01"`, `"2026-04-01"`) from Section 2, Rule B in `evaluator_system.md`.
  - Rewrite Rule B to instruct the evaluator to dynamically determine the primary surge (`baseline_period` and `peak_period`) by analyzing month-over-month deltas directly from the `macro_trajectory` data within the requested query scope.
  - Make Section 1 year-agnostic (remove the restrictive phrase `"(including both 2025 and 2026 when present)"`).
- **SQL Coder Prompt Recipe Generalization**:
  - Generalize few-shot SQL examples in `sql_coder_system.md` (Recipes 1, 2, and 3) to use parameterized/dynamic conventions (`<target_year>`, `<baseline_period>`, `<peak_period>`) rather than hardcoding static 2025/2026 dates and aliases (`p26`, `p25`).
- **Supervisor Routing & Fallback De-biasing**:
  - Update `supervisor_system.md` so that relative expressions like `"este año"` do not hardcode `"2026"`, instructing the supervisor to keep the scope relative or resolve it against the context's latest period.
  - Update the standalone month fallback in `supervisor.py` to resolve missing years from the latest period or current year rather than a hardcoded static integer `2026`.
- **Planner Prompt Explanatory Phrasing**:
  - Neutralize the explanatory comment in `planner_system.md` to reference "the requested period's numbers" instead of "the 2026 numbers".

## Capabilities

### New Capabilities
- `dynamic-trajectory-evaluation`: Autonomous, data-driven detection of chronological expense trajectories, inflection points, and variance baselines without hardcoded date conditioning.

### Modified Capabilities
*(None - existing storage and RBAC specifications remain unchanged).*

## Impact

- **Affected Prompt Templates**:
  - `backend/app/agent/analytics/prompts/evaluator_system.md`
  - `backend/app/agent/analytics/prompts/sql_coder_system.md`
  - `backend/app/agent/analytics/prompts/planner_system.md`
  - `backend/app/agent/prompts/supervisor_system.md`
- **Affected Python Code**:
  - `backend/app/agent/supervisor.py` (`_extract_period_from_text` default year resolution)
- **Downstream Analytics Flow**:
  - Hop 1 (`trajectory_evaluator_node`) autonomously extracts dynamic `'YYYY-MM-01'` strings matching the actual data and user time scope.
  - Hop 2 (`sql_coder_node`) receives dynamic baseline/peak periods and builds variance CTEs accurately for any timeframe.
