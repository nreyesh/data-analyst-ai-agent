## Context

Two user-experience gaps have been identified in production:
1. When asked off-topic questions (such as country capitals or abstract math like "2+2"), the model provides answers instead of stating that it is exclusively a condominium financial assistant.
2. During the 5–15 seconds that backend analytics workflows require to process complex queries, the Chainlit frontend displays a static spinner with a single unchanging label, creating user anxiety regarding whether the request is progressing.

## Goals / Non-Goals

**Goals:**
- Intervene `backend/app/agent/prompts/supervisor_system.md` with an explicit Off-Topic & General Knowledge Guard under `direct_response` that politely refuses non-condominium questions.
- Implement an active milestone progress ticker in `front/app.py` (`_execute_turn_with_live_step`) that smoothly updates the `cl.Step` label during network waits and instantly finalizes when the backend response arrives.

**Non-Goals:**
- Creating a new SSE streaming endpoint or changing the HTTP REST contract between frontend and backend.
- Refusing questions about Chilean coproperty law (Ley 21.442), prorate formulas, or condominium financial administration.

## Decisions

### 1. Front-Door Off-Topic Guard in `supervisor_system.md`
- **Decision**: Define a dedicated guardrail in rule 7 (`direct_response`) of `supervisor_system.md` specifying that general trivia, mathematics puzzles, coding, and general knowledge are out-of-scope, with an explicit Spanish template response.
- **Rationale**: Intercepts off-topic queries immediately at the Supervisor Router stage (~300ms) without touching the database or analytics graph.

### 2. Client-Side Milestone Progression (Option A) in `front/app.py`
- **Decision**: In `_execute_turn_with_live_step`, spawn an asynchronous milestone task that updates `step.name` and invokes `step.update()` at timed intervals:
  - `t=1.5s`: *"Verificando registros financieros..."*
  - `t=3.5s`: *"Consultando la base de datos del condominio..."*
  - `t=6.0s`: *"Procesando variaciones y métricas mensuales..."*
  - `t=8.5s`: *"Redactando la respuesta..."*
  When the backend runner finishes, the milestone task is immediately cancelled and the step is finalized with `turn_result.step_name`.
- **Rationale**: Eliminates user anxiety with zero backend changes and no streaming protocol complexity.

## Risks / Trade-offs

- **[Risk] False positive refusal on administrative or legal condo questions** → *Mitigation*: The prompt explicitly clarifies that Chilean coproperty law, prorate rules, and building expense questions are in-domain.
