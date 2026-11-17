## 1. Supervisor Off-Topic Guard Prompt Update

- [x] 1.1 Update `backend/app/agent/prompts/supervisor_system.md` with explicit instructions and template response strictly rejecting general knowledge trivia, general math (e.g. "2+2"), and off-topic questions under `direct_response`.
- [x] 1.2 Add regression tests in `backend/tests/test_supervisor_and_workflow.py` validating that general knowledge trivia and abstract arithmetic queries are rejected with polite direct responses.

## 2. Frontend Real-Time Milestone Progress

- [x] 2.1 Update `_execute_turn_with_live_step` in `front/app.py` to run an asynchronous milestone ticker that advances progress labels while awaiting backend response.
- [x] 2.2 Add unit tests in `front/tests/test_api_client.py` verifying that `_execute_turn_with_live_step` cleanly transitions progress and finalizes on completion.

## 3. Verification & Test Suite

- [x] 3.1 Run backend test suite (`cd backend && pytest tests/ -v --cov=app --cov-fail-under=65`) to verify zero regressions.
- [x] 3.2 Run frontend test suite (`cd front && pytest tests/ -v`) to verify all frontend tests pass.
