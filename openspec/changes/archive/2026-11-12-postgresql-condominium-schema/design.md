## Context

The system processes financial records for a 420-apartment residential condominium in Chile. The input artifacts consist of:
1. Building-level monthly expense reports (*Informe de Gastos Comunes*): operational expenses, reserve funds, vendor invoices.
2. Flat-level monthly billing statements (*Nota de Cobro por Unidad*): individual flat charges, metered utilities, fines/recargos.

The production deployment environment is Google Cloud Platform (GCP) with PostgreSQL 15 on Cloud SQL, connecting via private VPC peering as defined in `infra/main.tf`.

## Goals / Non-Goals

**Goals:**
- Implement normalized relational schemas in PostgreSQL 15+ using SQLAlchemy 2.0 ORM.
- Model the master `units` dimension table for the 420 apartments once with official `alicuota` shares.
- Model the time dimension `billing_periods` table to anchor monthly building reports and unit bills.
- Preserve the 3-tier building expenses hierarchy (`expense_reports` ➔ `expense_sections` ➔ `expense_items`).
- Preserve the unit bill breakdown (`unit_bills` ➔ `unit_bill_concepts`, `unit_bill_consumptions`, `unit_bill_charges`).
- Use integer Chilean Pesos (`BIGINT` / `INTEGER`) for financial calculations to prevent floating point inaccuracy.
- Store `alicuota` as exact `NUMERIC(8, 5)`.
- Use native PostgreSQL `UUID` primary keys for transaction and child tables.
- Manage schema migrations reproducibly with Alembic.

**Non-Goals:**
- Real-time payment gateway integration (Transbank / Webpay) - out of scope for this data storage layer.
- Tenant identity management / user authentication (handled separately by front-end Chainlit service).
- Document rendering / PDF generation (this layer is the persistence repository).

## Decisions

### 1. Master `units` Table with Natural Key
- **Decision**: Use `unit_code` (e.g. `'B-504'`) as the Primary Key for `units`, with `tower` (VARCHAR(16)), `unit_number` (INTEGER), and `alicuota` (NUMERIC(8, 5)).
- **Rationale**: Keeps foreign key references in `unit_bills` directly readable without requiring joins just to identify the unit.
- **Alternatives Considered**: Surrogate UUID PK with unique constraint. Rejected to avoid extra joins on high-frequency unit lookups.

### 2. Time Dimension `billing_periods` Table with Natural Key
- **Decision**: Use `period_date` (VARCHAR(10), format `'YYYY-MM-01'`) as the Primary Key for `billing_periods`, with `period_year` and `period_month`. Exclude operational metadata like `status` and `due_date`.
- **Rationale**: Unifies the temporal scope for both building reports and unit bills with zero joining overhead.
- **Alternatives Considered**: Surrogate UUID or auto-increment ID. Rejected because `'YYYY-MM-01'` is deterministic and naturally unique per monthly cycle.

### 3. Native PostgreSQL UUID for Transactions and Children
- **Decision**: Use native PostgreSQL `UUID` type (16 bytes) generated via Python UUIDv7 for `expense_reports`, `expense_sections`, `expense_items`, `unit_bills`, `unit_bill_concepts`, `unit_bill_consumptions`, and `unit_bill_charges`.
- **Rationale**: Saves storage, prevents B-tree page fragmentation (due to UUIDv7 time-ordering), and eliminates IDOR enumeration vulnerabilities.
- **Alternatives Considered**: `VARCHAR(36)`. Rejected because native UUIDs are 55% smaller in index size and faster in PostgreSQL joins.

### 4. Integer CLP and Exact Numeric Shares
- **Decision**: All money amounts are stored as integer CLP (`BIGINT` or `INTEGER`). Shares are stored as `NUMERIC(8, 5)`.
- **Rationale**: CLP currency has no fractional cents in circulation. Integer math prevents floating point errors (e.g. `0.1 + 0.2 = 0.30000000000000004`).

### 5. Foreign Key Deletion Behaviors
- **Decision**: 
  - `units` ➔ `unit_bills`: `ON DELETE RESTRICT` (protects audit history).
  - `billing_periods` ➔ `expense_reports` / `unit_bills`: `ON DELETE RESTRICT`.
  - Parent ➔ Child line items (`expense_reports` ➔ `sections` ➔ `items`, `unit_bills` ➔ `concepts`/`consumptions`/`charges`): `ON DELETE CASCADE`.
- **Rationale**: Protects master entities from accidental deletion while ensuring draft or replaced monthly batches leave no orphan line items.

### 6. CI/CD Database Migration Execution via Ephemeral Cloud Run Job
- **Decision**: Define a Cloud Run Job (`google_cloud_run_v2_job.migrate`) attached to the private VPC with access to Cloud SQL (internal IP e.g. `10.135.0.3`). The GitHub Actions CI/CD pipeline triggers this job using `gcloud run jobs execute ... --wait` prior to deploying the backend service revision.
- **Rationale**: Cloud SQL is 100% private with no public IPv4. Running an ephemeral job inside the VPC eliminates the need for VPN tunnels or public database exposure. Using `--wait` guarantees synchronous execution: exit code `0` proceeds to deployment, while any failure (exit code != `0`) aborts the pipeline, preserving the existing running service without downtime.
- **Alternatives Considered**: 
  - Running migrations on container startup: Risk of concurrency conflicts during multi-instance autoscaling.
  - Public IP on Cloud SQL: Violates security requirement of zero public attack surface.

## Risks / Trade-offs

- **[Risk] Alícuotas change over time (e.g. unit remodeling or legal subdivision)** → **Mitigation**: `unit_bills` records the actual billed concepts and amounts applied for that historical month. If alícuotas change, an administrative migration or audit table can be added later without altering historical records.
- **[Risk] Migration execution on Private IP Cloud SQL** → **Mitigation**: Alembic migrations can run in Cloud Run jobs or via Cloud SQL Proxy within the VPC.
- **[Risk] Duplicate ingestion runs** → **Mitigation**: Unique constraints on `(period_date)` for `expense_reports` and `(unit_code, period_date)` for `unit_bills` allow idempotent upserts / atomic replacements.
