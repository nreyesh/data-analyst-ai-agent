## ADDED Requirements

### Requirement: Strict Rejection of General Knowledge and Off-Topic Trivia
The conversational supervisor SHALL strictly decline off-topic general knowledge queries (such as world capitals, history, geography), abstract mathematics (such as "2+2" or arithmetic puzzles), programming, and general trivia that are unrelated to Chilean condominium expenses or building administration. It SHALL NOT provide answers to general knowledge queries.

#### Scenario: General knowledge query rejected
- **WHEN** user asks an off-topic general knowledge question (e.g., "¿cuál es la capital de Francia?")
- **THEN** the supervisor selects `route = "direct_response"`, refuses to answer general knowledge trivia, clarifies that it is an assistant specialized exclusively in condominium expenses, and invites the user to ask about their bills or building expenses.

#### Scenario: Abstract arithmetic query rejected
- **WHEN** user asks an abstract arithmetic question (e.g., "¿cuánto es 2+2?")
- **THEN** the supervisor selects `route = "direct_response"` with a polite refusal stating its dedicated role in condominium financial administration.
