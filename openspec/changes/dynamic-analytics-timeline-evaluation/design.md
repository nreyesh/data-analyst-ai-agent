# Design: Dynamic Analytics Timeline Evaluation

## Context

The Deep Analytics Engine operates a two-hop analytical pipeline:
1. **Hop 1 (Macro Inspection & Premise Fact-Checking)**:
   - `trajectory_inspector_node` fetches chronological monthly totals (`macro_trajectory`) for the building or unit across all available historical periods.
   - `trajectory_evaluator_node` executes `evaluator_system.md` to evaluate the user's premise, check whether expenses actually rose or fell, and isolate key inflection points (`baseline_period` and `peak_period`).
2. **Hop 2 (Detailed Drilldown & Attribution)**:
   - `sql_coder_node` executes `sql_coder_system.md`, consuming `eval_res.baseline_period` and `eval_res.peak_period` to construct variance attribution CTEs and SQL queries.
   - Downstream visualizer and synthesizer nodes format and present the findings.

The evaluator prompt previously contained hardcoded dates in Section 2, Rule B:
```markdown
isolate the primary surge (baseline_period = "2026-05-01", peak_period = "2026-07-01" or "2026-04-01" to "2026-07-01")
```
This conditioned the model to output May–July 2026 regardless of the user's query timeline, resulting in severe prompt conflicts when evaluating different years (e.g. 2024 or 2025) or custom quarter ranges. Coupled with static date literals in `sql_coder_system.md` and fallback year `2026` in `supervisor.py`, the system was biased towards a single test timeframe.

## Goals / Non-Goals

**Goals:**
- Eliminate all hardcoded date assignments from `evaluator_system.md` and instruct the model to deduce `baseline_period` and `peak_period` dynamically from `macro_trajectory` based on the query's time scope.
- Generalize few-shot SQL recipes in `sql_coder_system.md` using parameterized patterns (`<baseline_period>`, `<peak_period>`, `<target_year>`) to prevent few-shot date anchoring.
- Make supervisor temporal extraction and month fallback dynamic using `datetime.now().year` and conversational context rather than static integer `2026`.
- Ensure zero regression in existing test suites and maintain full contract compatibility with `TrajectoryEvaluation` and `SQLCoderOutput`.

**Non-Goals:**
- Altering the Pydantic schema contracts (`TrajectoryEvaluation`, `SQLCoderOutput`, `IntentPlan`).
- Modifying the LangGraph state machine edge routing in `workflow.py`.
- Replacing LLM trajectory evaluation with a deterministic Python statistical model; the LLM remains the cognitive interpreter of the user's premise.

## Decisions

### Decision 1: Prompt-Driven Dynamic Inflection Detection
Instead of hardcoding specific date strings in Rule B, the prompt will instruct the evaluator to:
1. Restrict analysis to the subset of `macro_trajectory` matching the user's requested `time_scope` (or the complete history if unspecified).
2. Calculate month-over-month deltas ($\Delta = \text{current} - \text{previous}$) across `operational_total` and `calculated_grand_total`.
3. Set `baseline_period` to the period immediately preceding the most prominent increase (the local trough).
4. Set `peak_period` to the period where that increase peaked (the local high).
5. Default to the earliest and latest periods within the requested scope if the trajectory is monotonic or flat.
*Alternatives considered:* Deterministic Python pre-calculation. Rejected because the evaluator must reconcile subjective human phrasing (e.g., "el alza de invierno", "el último incremento") against multi-phase trajectories.

### Decision 2: Parameterized Recipes in SQL Coder Prompt
Few-shot examples in `sql_coder_system.md` will replace static literals (`2026-05-01`, `2026-07-01`, `WHERE p.period_year = 2026`) with explicit parameter placeholders and explanatory comments showing that dates must match the evaluated baseline/peak periods provided in the context.
*Alternatives considered:* Leaving recipes with 2026 dates and relying solely on textual instructions. Rejected because zero-temperature LLMs frequently anchor to code block literals in few-shot recipes.

### Decision 3: Context-Aware Supervisor Year Fallback
In `supervisor.py` (`_extract_period_from_text`), when a user queries a month name without a year:
- First check for explicit year in the text.
- Second check `focus.last_period` from conversation context.
- Third, fallback to `datetime.now().year` (or current context year) rather than a static literal `2026`.
*Alternatives considered:* Calling the database to query `MAX(period_year)`. Rejected to avoid introducing database I/O latency inside regex parsing functions.

## Risks / Trade-offs

- **[Risk] LLM might generate malformed period dates (e.g. `'2025-5-1'` or `'May 2025'`)**
  → *Mitigation*: The prompt explicitly reinforces the strict `'YYYY-MM-01'` format and mandates selecting dates that actually exist in the provided `macro_trajectory` list.
- **[Risk] Ambiguous trajectory with no identifiable peak in compound queries**
  → *Mitigation*: The revised Rule B provides an explicit fallback directive: set `baseline_period` to the first month and `peak_period` to the last month within the query scope.
- **[Risk] Existing unit test mocks might expect legacy output strings**
  → *Mitigation*: Review and run unit tests in `backend/tests/` to confirm that mocked evaluator outputs align with the dynamic schema.
