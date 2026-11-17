# resident-unit-rbac Specification

## Purpose
TBD - created by archiving change llm-resident-unit-rbac. Update Purpose after archive.
## Requirements
### Requirement: LLM Structured RBAC Classification
The Conversational Supervisor LLM SHALL evaluate the user context and query intent to classify whether an authenticated resident user is attempting to access data for an unauthorized unit or tower, emitting explicit structured fields (`is_unauthorized_unit_access` and `unauthorized_target_unit`) in `SupervisorDecision`.

#### Scenario: Resident queries own unit with dates and conjunctions
- **WHEN** an authenticated user with `role="resident"` and assigned unit `B-504` queries `"que cargo particular se cargo en junio y octubre de 2026"`
- **THEN** the Supervisor LLM classifies `is_unauthorized_unit_access = false`, `unauthorized_target_unit = null`, and resolves `unit_code = "B-504"`

#### Scenario: Resident explicitly queries a different apartment unit
- **WHEN** an authenticated user with `role="resident"` and assigned unit `B-504` queries `"Muéstrame la boleta del departamento A-101"`
- **THEN** the Supervisor LLM classifies `is_unauthorized_unit_access = true`, `unauthorized_target_unit = "A-101"`, and sets `unit_code = "A-101"`

#### Scenario: Resident queries building-wide macro expenses
- **WHEN** an authenticated user with `role="resident"` queries `"¿Cuál fue el total del gasto común del edificio en agosto de 2026?"`
- **THEN** the Supervisor LLM classifies `is_unauthorized_unit_access = false`, `unauthorized_target_unit = null`, and `knowledge_domain = "building"`

### Requirement: Deterministic Python RBAC Enforcement
The backend execution tools (`lookup_unit_bill`, `lookup_spend_evolution`, and `run_deep_financial_analysis`) SHALL enforce deterministic tenant scoping in Python: if a resident's query has `is_unauthorized_unit_access = true` or targets a unit code different from their assigned unit, the system SHALL immediately reject the request with an HTTP 200 payload containing `unauthorized = true` and record a security violation.

#### Scenario: Unauthorized resident query is blocked deterministically
- **WHEN** a resident with assigned unit `B-504` submits a request targeting unit `A-101` (where `is_unauthorized_unit_access = true` or `decision.unit_code = "A-101"`)
- **THEN** the tool returns `unauthorized = true` with a clear explanation: `"Acceso denegado: Tu perfil de residente solo está autorizado para consultar la unidad B-504"`
- **THEN** the turn result tags `security_violation_type = "rbac_unauthorized_access"` to increment the Layer B guardrail strike counter

#### Scenario: Authorized resident query proceeds without false positive
- **WHEN** a resident with assigned unit `B-504` submits a query containing Spanish temporal prepositions, conjunctions, or years (e.g. `"2026 y"`, `"de 2026"`, `"2025 o 2026"`)
- **THEN** no RBAC block is triggered, `unauthorized` is not set to true, and no security audit strike is recorded

### Requirement: Elimination of Heuristic Regex Text Scanning
The system SHALL NOT inspect raw natural language query strings with arbitrary digit-letter regex patterns (such as `\b(\d{1,4})\s*([A-Za-z]{1,2})\b`) for apartment unit detection, delegating semantic entity extraction and intent verification to structured LLM classification.

#### Scenario: Temporal conjunction phrases are not misparsed as units
- **WHEN** a resident query contains phrases matching number-space-letter sequences like `"2026 y"`, `"2026 o"`, `"día 15 a fin de mes"`
- **THEN** the system evaluates these purely as temporal and conversational syntax and does not construct phantom apartment codes (such as `Y-2026` or `O-2026`)

