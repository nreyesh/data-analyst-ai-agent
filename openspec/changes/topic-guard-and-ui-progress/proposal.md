## Why

The conversational assistant currently answers off-topic general knowledge questions (e.g., country capitals, abstract arithmetic like "2+2") instead of strictly declining them as outside the condominium financial domain. Furthermore, during conversational turns, the frontend Chainlit interface displays a static spinner without communicating intermediate processing steps, causing user anxiety regarding whether their request is progressing or frozen.

## What Changes

- **Strict Off-Topic & General Knowledge Guard**: Update `backend/app/agent/prompts/supervisor_system.md` to explicitly forbid general trivia, country capitals, mathematics puzzles, coding, and non-condominium queries, enforcing immediate polite refusal via `direct_response`.
- **Active Real-Time Step Progress in Frontend**: Update `front/app.py` (`_execute_turn_with_live_step`) to asynchronously advance live progress milestones (*"Analizando tu consulta..."*, *"Verificando registros financieros..."*, *"Consultando la base de datos..."*, *"Procesando variaciones..."*, *"Redactando la respuesta..."*) while awaiting backend HTTP execution, immediately concluding with the final verified step upon response arrival.

## Capabilities

### New Capabilities
- `ui-step-progress-feedback`: Real-time milestone progress feedback in Chainlit UI during turn execution to reduce user latency anxiety.

### Modified Capabilities
- `supervisor-domain-boundaries`: Broaden domain boundary rules to strictly prohibit answering general knowledge trivia, general mathematics, and off-topic inquiries.

## Impact

- **Backend**:
  - `backend/app/agent/prompts/supervisor_system.md` (supervisor prompt off-topic refusal guidelines)
  - `backend/tests/test_supervisor_and_workflow.py` (regression test for off-topic query rejection)
- **Frontend**:
  - `front/app.py` (`_execute_turn_with_live_step` milestone progression task)
- **APIs**:
  - Zero breaking changes to REST interfaces or Pydantic schemas.
