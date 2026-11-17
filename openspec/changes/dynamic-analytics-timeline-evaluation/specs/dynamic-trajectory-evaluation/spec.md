# Specification: Dynamic Trajectory Evaluation

## ADDED Requirements

### Requirement: Autonomous Trajectory Premise Evaluation
The Deep Analytics Engine's Trajectory Evaluator SHALL evaluate chronological monthly trajectories dynamically against the user's premise without relying on hardcoded calendar dates or predetermined baseline/peak assignments.

#### Scenario: Dynamic baseline and peak identification for custom timeline
- **WHEN** a user requests an analytical evaluation for a specific timeframe (e.g., year 2025 or a custom quarter range) with compound sub-analyses
- **THEN** the Trajectory Evaluator analyzes chronological month-over-month variations from `macro_trajectory` strictly within the requested scope, dynamically assigns `baseline_period` and `peak_period` to the observed trough and peak dates in `'YYYY-MM-01'` format, sets `needs_clarification = false`, and classifies `premise_status = "validated"`.

#### Scenario: Multi-year chronological trajectory inspection
- **WHEN** `macro_trajectory` contains historical records spanning multiple years (such as 2024, 2025, or 2026)
- **THEN** the Trajectory Evaluator inspects the chronological sequence across all available periods within the query scope without restricting inspection to hardcoded years.

### Requirement: Parameterized Downstream SQL Period Provenance
The SQL Coder SHALL generate variance attribution and comparison queries substituting the dynamic `baseline_period` and `peak_period` emitted by the Trajectory Evaluator, without falling back to static recipe date literals.

#### Scenario: Dynamic CTE period substitution
- **WHEN** the SQL Coder receives evaluated `baseline_period` and `peak_period` from Hop 1
- **THEN** it constructs the variance CTEs filtering `WHERE r.period_date = '<baseline_period>'` and `WHERE r.period_date = '<peak_period>'`, and projects these exact dynamic values in the `baseline_period` and `peak_period` provenance columns.

### Requirement: Temporal-Agnostic Supervisor Resolution
The Supervisor and routing subsystem SHALL resolve relative temporal expressions and missing year references dynamically rather than assigning static year integers.

#### Scenario: Relative expression resolution
- **WHEN** a user asks for 'evolución de mi gasto común este año'
- **THEN** the Supervisor resolves `time_scope` relative to the latest available period year or keeps the scope relative, avoiding static hardcoding of year 2026.

#### Scenario: Unspecified year fallback for standalone months
- **WHEN** a user queries a month name without an explicit year and no previous period exists in `FocusContext`
- **THEN** the period extractor determines the year dynamically from the latest known billing period or system calendar year rather than defaulting to static integer 2026.
