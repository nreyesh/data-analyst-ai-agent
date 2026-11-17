## Why

The condominium management system requires a robust, cloud-native database architecture in Google Cloud SQL (PostgreSQL 15+) to reliably store, query, and reconcile financial records for 420 apartments. Migrating from an unnormalized local prototype to an enterprise relational schema eliminates massive data duplication (apartments repeated every month), guarantees exact financial arithmetic with integer CLP, anchors transactions to distinct billing periods, and establishes versioned schema migrations via Alembic.

## What Changes

- **Add Master Dimension Tables**: Introduce `units` (the 420 apartments, their tower, unit number, and official prorate percentage `alicuota`) and `billing_periods` (monthly fiscal periods `YYYY-MM-01`).
- **Preserve Building Expense Hierarchy**: Retain the proven 3-tier building expense hierarchy (`expense_reports` ➔ `expense_sections` ➔ `expense_items`) while anchoring the report to `billing_periods`.
- **Refactor Flat Unit Billing**: Normalize `unit_bills` to reference `units(unit_code)` and `billing_periods(period_date)`, preserving child tables `unit_bill_concepts`, `unit_bill_consumptions`, and `unit_bill_charges`.
- **PostgreSQL 15+ Native Types**: Adopt native 16-byte `UUID` primary keys for transactions, exact `NUMERIC(8, 5)` for alícuotas, `BIGINT` for CLP amounts, and `JSONB` for extraction telemetry.
- **Audit Safeguards**: Enforce `ON DELETE RESTRICT` on master units (blocking accidental deletion of units with billing history) and `ON DELETE CASCADE` for parent-child line items.
- **Database Migration Pipeline**: Set up SQLAlchemy 2.0 ORM models and Alembic migration harness for automated Cloud SQL deployments.
- **CI/CD Automated VPC Migrations**: Implement a dedicated Cloud Run Migration Job (`building-analytics-migrate-${env}`) in Terraform and a GitHub Actions workflow (`.github/workflows/deploy-backend.yml`) that executes migrations synchronously inside the VPC prior to service deployment.

## Capabilities

### New Capabilities
- `condominium-financial-storage`: Relational persistence models, constraints, and migration scripts for Chilean condominium building expenses and apartment billing statements on PostgreSQL.

### Modified Capabilities
<!-- None: No existing specs in openspec/specs/ -->

## Impact

- **Backend Architecture**: Establishes `backend/app/models.py` (SQLAlchemy 2.0 ORM models), `backend/app/database.py` (PostgreSQL engine and session), and `backend/alembic/` (database migrations).
- **Cloud Infrastructure**: Targets the GCP Cloud SQL PostgreSQL 15 instance deployed via Terraform (`infra/main.tf`).
- **Data Ingestion**: Upgrades data extraction pipelines to populate master `units` and `billing_periods` before persisting monthly reports and unit bills.
