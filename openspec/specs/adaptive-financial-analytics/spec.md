# adaptive-financial-analytics Specification

## Purpose
TBD - created by archiving change adaptive-financial-analytics. Update Purpose after archive.
## Requirements
### Requirement: Decoupled and Proportional Analytical Visualization
The analytical planner SHALL dynamically determine whether visualization is required based on the user's explicit request or the analytical utility of rendering a multi-period chart. It SHALL NOT automatically mandate chart rendering for simple follow-up causality questions or single/dual-month variance explanations.

#### Scenario: Follow-up variance inquiry skips visualization
- **WHEN** user asks why expenses increased in specific months (e.g. "¿por qué subió mi gasto en junio y octubre?") without requesting a chart
- **THEN** `requires_visualization` is set to `false`, and the workflow bypasses visualizer and sandbox nodes directly to the synthesizer.

#### Scenario: Multi-month trend inquiry enables visualization
- **WHEN** user asks for an annual evolution or trend analysis (e.g. "¿por qué ha subido tanto mi gasto este año?") or explicitly asks for a graph
- **THEN** `requires_visualization` is set to `true`, and the workflow routes through visualizer and sandbox nodes to produce a chart.

### Requirement: Multi-Period Unit Decomposition in Planner
When an authorized unit inquiry references multiple distinct periods (such as two specific months), the analytical planner SHALL include all referenced periods in `sub_analyses` instead of arbitrarily truncating to a single month.

#### Scenario: Dual-month inquiry retains both target periods
- **WHEN** user asks for causes of variance across June and October
- **THEN** the planner populates `sub_analyses` with decomposition tasks covering both June and October, allowing downstream SQL Coder to fetch concept and charge details for both periods.

### Requirement: Scope-Bounded Trajectory Inspection
The macro trajectory inspector SHALL respect the analytical plan's `time_scope`. It SHALL NOT query unbounded historical records from prior years unless multi-year comparison (e.g., YoY) was explicitly planned.

#### Scenario: Single-year scope restricts inspection query
- **WHEN** plan has `time_scope = "2026"`
- **THEN** trajectory inspector query limits records to year 2026 (or requested comparison baseline), preventing retrieval of irrelevant historical periods (such as 2024 or early 2025).

### Requirement: Proportional and Natural Synthesis
The synthesizer SHALL tailor its response structure and style proportionally to the user's inquiry. For direct and follow-up questions, it SHALL provide concise, fluid explanations without rigid 3-chapter report headers, without hallucinated unrequested historical periods, and without bracketed date repetition (`[Mes Año]`) on every monetary amount.

#### Scenario: Concise follow-up explanation
- **WHEN** synthesizing an answer for a focused variance question
- **THEN** synthesizer outputs a focused explanation (1–3 paragraphs or structured bullet points) naming the specific drivers, using natural Spanish date expressions, and omitting unrequested historical context.

