## 1. Schema & Contracts

- [x] 1.1 Add `is_unauthorized_unit_access` and `unauthorized_target_unit` fields to `SupervisorDecision` in `backend/app/agent/state.py`
- [x] 1.2 Update system prompt in `backend/app/agent/prompts/supervisor_system.md` with explicit resident RBAC classification rules and temporal contrast examples

## 2. Tool Execution & Deterministic RBAC Enforcement

- [x] 2.1 Remove `_detect_unauthorized_unit_in_text` from `backend/app/agent/tools.py`
- [x] 2.2 Update `lookup_unit_bill` in `backend/app/agent/tools.py` to enforce RBAC via `norm_requested != assigned_unit` and structured decision flags
- [x] 2.3 Update `lookup_spend_evolution` in `backend/app/agent/tools.py` to enforce RBAC via `norm_requested != assigned_unit` and structured decision flags
- [x] 2.4 Update `run_deep_financial_analysis` in `backend/app/agent/tools.py` to enforce RBAC via `norm_requested != assigned_unit` and structured decision flags
- [x] 2.5 Propagate `is_unauthorized_unit_access` and `unauthorized_target_unit` from `decision` into tool execution and security turn tagging in `backend/app/agent/supervisor.py`

## 3. Verification & Regression Testing

- [x] 3.1 Add regression unit tests in `backend/tests/` asserting that temporal queries with conjunctions (e.g., `"2026 y"`, `"de 2026"`, `"junio de 2026 y octubre de 2026"`) are not flagged as unauthorized
- [x] 3.2 Verify that explicit attempts by a resident to access another unit (e.g. `"A-101"`) still trigger `unauthorized = true` and `rbac_unauthorized_access` strikes
- [x] 3.3 Execute the full test suite (`PYTHONPATH=backend pytest backend/tests/ -v --cov=backend/app --cov-fail-under=65`) to verify coverage and passing status
