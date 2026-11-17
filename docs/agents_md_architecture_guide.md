# The Definitive Guide to `AGENTS.md` for AI Coding Agents

An `AGENTS.md` file is an **action-oriented, operational contract** between a human software engineer and autonomous AI coding assistants (such as Antigravity, Claude Code, Cursor, GitHub Copilot, Codex, etc.). 

While a traditional `README.md` is written for humans (explaining *why* a project exists and high-level marketing/conceptual overviews), an `AGENTS.md` is written for **deterministic machine execution**: it establishes non-negotiable boundaries, executable CLI verification commands, architectural invariants, and security constraints.

---

## 1. Core Anatomy: The 6 Essential Sections of `AGENTS.md`

Every high-performing `AGENTS.md` file should be structured around these six components:

```
┌────────────────────────────────────────────────────────┐
│ 1. ABSOLUTE GUARDRAILS ("NEVER DO" LIST)              │
│    Red lines: No git commits, no secrets, no raw SQL   │
├────────────────────────────────────────────────────────┤
│ 2. REPOSITORY TOPOLOGY & BOUNDARIES                    │
│    Directory layout, submodules, cross-package rules   │
├────────────────────────────────────────────────────────┤
│ 3. ACTIONABLE VERIFICATION COMMANDS                   │
│    Reproducible shell commands for test, lint, build   │
├────────────────────────────────────────────────────────┤
│ 4. ARCHITECTURAL & DOMAIN INVARIANTS                   │
│    Data types (integers for money), 3NF schemas, etc.  │
├────────────────────────────────────────────────────────┤
│ 5. SECURE CODING & VULNERABILITY MATRIX                │
│    Injection defense, BOLA/IDOR scoping, safe typing   │
├────────────────────────────────────────────────────────┤
│ 6. LIVING DOCUMENT MAINTENANCE                         │
│    Rules forcing agents to update AGENTS.md on changes │
└────────────────────────────────────────────────────────┘
```

### Section 1: ⛔ Absolute Guardrails ("The Never-Do List")
Agents follow negative constraints most reliably when they are placed **at the very top** of the document:
* **No Git Commits or Pushing:** AI agents must not commit or alter history; code modification is autonomous, but commit staging and review remain human.
* **Zero Hardcoded Secrets:** Strictly ban hardcoding tokens, passwords, or keys.
* **Network / Sandbox Red Lines:** Forbid direct connections to production databases or internal VPCs without authorized proxies.

### Section 2: 🗺️ Repository Topology & Submodule Routing
In monorepos or multi-repo submodule setups:
* Explain the role of each directory.
* Direct the agent to submodule-specific guides (e.g. "When editing `backend/`, read `backend/AGENTS.md`").
* Define contract synchronization rules (e.g. backend FastAPI models must be mirrored in frontend Pydantic models).

### Section 3: 🧪 Actionable Verification Commands
Agents suffer from "premature completion syndrome"—they write code and immediately tell the user they are done without testing.
* Provide the exact shell commands needed to verify work.
* Include coverage thresholds (e.g. `--cov-fail-under=65`).
* Specify the exact working directory or environment variables required (`PYTHONPATH=backend pytest ...`).

### Section 4: 🏛️ Architectural & Domain Invariants
These are domain rules that cannot be inferred just by reading a few files:
* Chilean Pesos (CLP) must be integer `BIGINT` (no floating-point rounding errors).
* Database normalization rules (3NF hierarchies, foreign key delete cascades).
* Cloud Run statelessness rules (never save uploaded files to local container disk).

### Section 5: 🛡️ Secure Coding & Vulnerability Prevention Matrix
A tabular or rule-based matrix explicitly pairing **threats**, **mandatory safe patterns**, and **banned anti-patterns**:
* **SQL Injection:** Mandate SQLAlchemy 2.0 ORM expressions (`select().where()`); explicitly ban `f"SELECT ... {input}"`.
* **BOLA / IDOR:** Mandate unit/tenant filtering on all endpoints.
* **Safe Language Idioms:** Enforce static type hints, context managers, and ban `eval`, `exec`, and `pickle`.
* **LLM Output Sanitization:** Force all LLM outputs through Pydantic validators before database persistence.

### Section 6: 🔄 Living Document Maintenance Invariant
Explicitly require that whenever an agent creates a new migration, alters an API schema, or introduces a new tool, it must edit `AGENTS.md` in that same working session.

---

## 2. Best Practices & Optimization Tips

1. **Keep it Action-First, Not Descriptive:**
   * ❌ *Bad:* "This backend is responsible for receiving condominium receipts and parsing them using artificial intelligence."
   * ✅ *Good:* "Parse PDFs using `PYTHONPATH=. python -m backend.app.extractor.cli --target units --unit B-504 --dry-run`."
2. **Token Efficiency & Density:**
   * Keep submodule `AGENTS.md` files under 1,000–1,500 tokens.
   * Use bullet points, concise code blocks, and markdown tables.
   * Avoid dumping raw source code; link to files or provide minimal 3-line snippets.
3. **Use Distributed Hierarchy for Modular Repos:**
   * A root `AGENTS.md` acts as the master orchestrator.
   * Child directories (`backend/AGENTS.md`, `front/AGENTS.md`, `infra/AGENTS.md`) contain deep, language-specific instructions.

---

## 3. Anti-Patterns to Avoid

| Anti-Pattern | Why It Breaks Agents | What to Do Instead |
| :--- | :--- | :--- |
| **Passive Documentation** | Agents treat it as suggestions rather than strict rules. | Use imperative verbs: "MUST", "NEVER", "STRICTLY FORBIDDEN". |
| **Monolithic Bloat** | Dumping 5,000 lines of frontend + backend + terraform into one file causes context overflow. | Use distributed `AGENTS.md` per package. |
| **Ambiguous Test Commands** | Telling the agent "run your tests" without specifying flags or paths. | Specify the exact string: `PYTHONPATH=backend pytest backend/tests/ -v --cov=backend/app --cov-fail-under=65`. |
| **Allowing Autonomous Commits** | Agents generate dirty git trees, commit half-finished experiments, or clobber submodules. | Restrict agents to workspace changes only; let humans review and commit. |

---

## 4. Ecosystem Configuration for This Repository

In this repository (`data-analyst-ai-agent`), the following architecture has been implemented:

* [`/AGENTS.md`](file:///home/nereyes/agentes/data-analyst-ai-agent/AGENTS.md): Root ecosystem orchestrator, submodule handoff rules, cross-boundary contract synchronization, and absolute guardrail banning autonomous git commits.
* [`/backend/AGENTS.md`](file:///home/nereyes/agentes/data-analyst-ai-agent/backend/AGENTS.md): Python/FastAPI/SQLAlchemy 2.0 guide, offline SQLite testing sandbox, 65% coverage verification gate, and explicit Vulnerability Defense Matrix.
* [`/front/AGENTS.md`](file:///home/nereyes/agentes/data-analyst-ai-agent/front/AGENTS.md): Chainlit UI, Firebase Auth JWT validation, zero backend imports, and stateless Cloud Run rules.
* [`/infra/AGENTS.md`](file:///home/nereyes/agentes/data-analyst-ai-agent/infra/AGENTS.md): Terraform IaC, Workload Identity Federation, zero public IP database architecture, and unified resource naming.
