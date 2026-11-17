## ADDED Requirements

### Requirement: Active Real-Time Step Progress in Frontend
The frontend interface SHALL display an active progression of milestone status messages during conversational turn processing while waiting for the backend response, preventing a frozen or static UI spinner.

#### Scenario: Long-running query advances through milestones
- **WHEN** user sends a query requiring multi-second backend processing
- **THEN** the frontend updates the live `cl.Step` label through sequential milestone labels (e.g. *"Analizando tu consulta..."*, *"Verificando registros financieros..."*, *"Consultando la base de datos..."*, *"Procesando variaciones..."*, *"Redactando la respuesta..."*) at periodic intervals.

#### Scenario: Fast query concludes with final verified step
- **WHEN** the backend response arrives
- **THEN** the milestone ticker immediately halts and the step is finalized with the backend's verified `step_name`.
