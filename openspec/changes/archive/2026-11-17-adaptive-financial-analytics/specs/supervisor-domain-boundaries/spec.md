## ADDED Requirements

### Requirement: Direct and Honest Out-of-Domain Boundaries
The conversational supervisor SHALL support `direct_response` for inquiries that fall outside the financial/accounting database scope (such as physical repair progress, construction status, building regulations, or administrative matters). It SHALL NOT forcibly route non-financial questions into the heavy analytics workflow.

#### Scenario: Operational status inquiry handled via direct response
- **WHEN** user asks about the physical status or operational execution of building works (e.g., "¿en qué estado de avance está el tema de la impermeabilización?")
- **THEN** the supervisor selects `route = "direct_response"`, explains that it only manages financial and accounting records (funds, bills, expenses), states that physical/operational progress is not in its database, and directs the user to the building administration or committee.

#### Scenario: No forced analytics hijacking of direct responses
- **WHEN** the supervisor router emits a valid `direct_response` with a clear explanation
- **THEN** python runtime execution does NOT rewrite `decision.route` to `run_deep_financial_analysis`, preserving the direct reply for the user.
