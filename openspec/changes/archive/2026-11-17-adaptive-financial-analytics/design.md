## Context

The analytics engine (`run_deep_financial_analysis`) is orchestrated as a 9-node LangGraph state machine. While powerful for complex building financial audits, its current prompt instructions and node implementations enforce rigid, non-adaptive behaviors:
1. `planner_system.md` forces `requires_visualization = true` on every `variance_driver` and restricts unit analysis to a single sub-analysis, dropping secondary referenced months.
2. `trajectory_inspector_node` executes an unbounded SQL query over all historical billing periods (fetching 25+ months of data regardless of query scope).
3. `synthesizer_system.md` forces a mandatory 3-chapter executive template and requires an inline bracketed date citation (`[Mes Año]`) after every monetary amount.
4. `supervisor.py` contains a defensive override (line 724) that intercepts any non-greeting `direct_response` and redirects it to `run_deep_financial_analysis`, forbidding the supervisor from directly answering or declining out-of-domain questions.

## Goals / Non-Goals

**Goals:**
- Enable the Planner to decide `requires_visualization` proportionally based on analytical utility and user intent.
- Support decomposing multiple referenced periods (e.g., June and October) in `planner_system.md` for apartment units.
- Scope `trajectory_inspector_node` SQL queries by `time_scope` (e.g. 2026), preventing unrequested historical data pollution.
- Remove prompt anti-patterns in `synthesizer_system.md` (mandatory 3-section header format, mandatory `[Mes Año]` per number) and instruct natural, proportional reporting.
- Enable `supervisor.py` and `supervisor_system.md` to cleanly return `direct_response` for non-financial inquiries (such as physical repair progress, building rules, or administrative contacts) without execution hijacking.

**Non-Goals:**
- Creating a brittle forest of `if-else` regexes in Python to intercept queries.
- Modifying the underlying SQLite/PostgreSQL 3NF database schema or Alembic migrations.
- Changing `SupervisorRoute` enums or altering `front/models.py` contracts.

## Decisions

### 1. Agentic Adaptiveness via System Prompt Refinement over Hardcoded Heuristics
- **Decision**: Modify `planner_system.md` and `synthesizer_system.md` to establish the Principle of Proportionality rather than hardcoding procedural routing rules in Python.
- **Rationale**: An agentic system reasons dynamically over user intent. Prescribing rigid constraints in prompts caused the failure; removing them and guiding proportional behavior restores agentic flexibility across infinite user phrasing variations.
- **Alternatives Considered**: Creating a dedicated `explain_unit_bill` tool with regex guards. Rejected because it fragments the routing surface, creates maintenance overhead, and fails on subtle phrasing nuances.

### 2. Time-Scoped Trajectory Inspection
- **Decision**: In `backend/app/agent/analytics/nodes.py`, parse the plan's `time_scope` (e.g., `"2026"`, `"2025-2026"`) and inject a `WHERE p.period_year = {year}` or date range into the SQL query when available. Fall back to all records only when `time_scope` is explicitly unbounded or null.
- **Rationale**: Eliminates data bloat in the synthesizer context and stops the model from hallucinating seasonal commentary about periods the user never asked about.

### 3. Leveraging Existing Graph Conditional Edges
- **Decision**: The LangGraph conditional edge `route_after_sql_gateway` already routes directly to `synthesizer` when `plan.requires_visualization` is false.
- **Rationale**: No graph re-wiring is needed. When the Planner sets `requires_visualization = false`, the execution naturally bypasses `visualizer_node` and `sandbox_runner_node`, cutting turn latency from ~15s to ~3s.

### 4. Permitting Informational `direct_response` in Supervisor Router
- **Decision**: Remove the forced rewrite of `direct_response` in `backend/app/agent/supervisor.py` lines 724-735. In `supervisor_system.md`, document that `direct_response` is authorized for:
  1. Pure greetings and help requests.
  2. Out-of-domain inquiries (e.g., physical progress of repairs, concierge contacts, building assembly dates) where the system honestly states its financial scope and redirects to management.
- **Rationale**: Prevents sending non-financial questions into the SQL/LangGraph engine where they fail or produce nonsensical output.

## Risks / Trade-offs

- **[Risk] Under-answering broad questions** → *Mitigation*: The prompt explicitly guides the synthesizer to provide structured multi-section analysis whenever `time_series_trend`, multi-year cohorts, or explicit reports are requested.
- **[Risk] Accidental direct_response for legitimate financial questions** → *Mitigation*: `supervisor_system.md` preserves the strict invariant that any inquiry regarding expenses, bills, payments, or funds MUST be routed to `lookup_unit_bill`, `lookup_spend_evolution`, or `run_deep_financial_analysis`.
