# Building Analytics AI Agent Ecosystem — Root Orchestration Guide

This document defines the overarching architecture, submodule routing protocols, security invariants, and operational boundaries for AI coding agents operating across the **Building Analytics AI Agent Ecosystem**.

All AI coding assistants (Antigravity, Cursor, Claude Code, GitHub Copilot, Codex, etc.) must adhere to these directives.

---

## 1. ⛔ Absolute Agent Guardrails (Never Violate)

1. **NO GIT COMMITS OR PUSHES:**
   * AI agents are **STRICTLY FORBIDDEN** from executing `git commit`, `git push`, `git tag`, or altering git branches/history.
   * Agents only create, edit, or test files in the workspace.
   * All git staging, committing, branching, and pushing **must be performed manually by the human engineer**.
2. **ZERO SECRETS IN REPOSITORIES:**
   * Never write API keys, service account keys, passwords, or tokens into code, configuration files, or test fixtures.
   * Secrets must come exclusively from environment variables or Google Cloud Secret Manager.
3. **ZERO DIRECT CROSS-SUBMODULE IMPORTS:**
   * Never import code across submodule boundaries (e.g. frontend must never import backend or agent modules). Communication occurs strictly via HTTPS REST APIs.
4. **NO BLIND COMPLETIONS:**
   * Never mark a task as "done" or tell the user it is complete without executing the relevant local verification commands (unit tests, type checks, schema diffs).

---

## 2. Repository Topology & Submodule Routing

This project uses a **Modular Git Submodule Multi-Repo Architecture**. The root repository acts solely as a container and coordinator.

```
data-analyst-ai-agent/                     # Root Orchestration Container
├── AGENTS.md                              # This file (Root router & ecosystem invariants)
├── backend/                               # Submodule: nreyesh/backend (FastAPI / Agent / Cloud SQL)
│   └── AGENTS.md                          # Backend engineering & security guide
├── front/                                 # Submodule: nreyesh/front (Chainlit / Firebase Auth)
│   └── AGENTS.md                          # Frontend presentation & auth guide
├── infra/                                 # Submodule: nreyesh/infra (Terraform / Google Cloud)
│   └── AGENTS.md                          # Infrastructure as Code & naming conventions
└── openspec/                              # OpenSpec specification and change tracking
```

### Submodule Handoff Protocol
When working within a specific subsystem, agents **MUST** immediately read and follow the local `AGENTS.md` in that directory:

| Working Scope | Directory | Actionable Guide | Core Focus |
| :--- | :--- | :--- | :--- |
| **Backend & AI Agent** | `backend/` | [`backend/AGENTS.md`](file:///home/nereyes/agentes/data-analyst-ai-agent/backend/AGENTS.md) | FastAPI, SQLAlchemy 2.0, Alembic migrations, LangGraph, extraction CLI, unit tests. |
| **Presentation Tier** | `front/` | [`front/AGENTS.md`](file:///home/nereyes/agentes/data-analyst-ai-agent/front/AGENTS.md) | Chainlit UI, Firebase Auth JWTs, client-side Pydantic models (`models.py`). |
| **Cloud Infrastructure**| `infra/` | [`infra/AGENTS.md`](file:///home/nereyes/agentes/data-analyst-ai-agent/infra/AGENTS.md) | Terraform, Workload Identity Federation, VPC peering, resource naming. |

---

## 3. Cross-Submodule Contract Synchronization

When implementing full-stack features or API modifications, changes must flow through a strict contract order:

```
[ Backend API / Models ]
           │ (1. Define Pydantic schema in backend)
           ▼
[ Frontend Models ]
           │ (2. Mirror lightweight schema into front/models.py)
           ▼
[ Frontend API Client ]
             (3. Update front/api_client.py to consume the endpoint)
```

* **Failsafe Schemas:** Whenever a backend endpoint response changes, update `front/models.py` (`UserContext`, `FocusContext`, `SupervisorTurnResult`, etc.) to match.
* **Decoupled Dependencies:** Never import backend modules into `front/models.py`. Frontend schemas must remain ultra-lightweight (Pydantic only, no database or LangGraph dependencies).

---

## 4. Universal Verification Commands

Before proposing code changes or requesting human review, run the relevant verification commands from the project root:

```bash
# 1. Backend Schema & Integrity Tests (Enforces >=65% test coverage)
PYTHONPATH=backend pytest backend/tests/ -v --cov=backend/app --cov-fail-under=65

# 2. Frontend Test Suite
PYTHONPATH=. pytest front/tests/ -v

# 3. Infrastructure HCL Syntax Validation
terraform -chdir=infra validate
```

---

## 5. Living Document Maintenance Invariant

This `AGENTS.md` and the submodule `AGENTS.md` files are **living architectural contracts**:
* If an agent adds new CLI commands, alters database migration workflows, introduces new environment variables, or changes architectural boundaries, the agent **MUST** update the corresponding `AGENTS.md` file in the same working tree change.
* Keep all `AGENTS.md` files action-first, concise, and focused on executable commands, invariants, and safety rules.
