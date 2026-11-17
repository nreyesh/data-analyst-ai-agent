## 1. Environment & Database Configuration

- [x] 1.1 Verify or configure Python dependencies for PostgreSQL (`sqlalchemy>=2.0`, `alembic`, `psycopg[binary]`, `uuid6`).
- [x] 1.2 Implement `backend/app/database.py` with SQLAlchemy 2.0 engine, declarative base, and sessionmaker configured for PostgreSQL / Cloud SQL connection pooling.

## 2. SQLAlchemy 2.0 ORM Models

- [x] 2.1 Implement dimension models in `backend/app/models.py`: `UnitModel` (`units`) and `BillingPeriodModel` (`billing_periods`).
- [x] 2.2 Implement 3-tier building expense models: `ExpenseReportModel` (`expense_reports`), `ExpenseSectionModel` (`expense_sections`), and `ExpenseItemModel` (`expense_items`).
- [x] 2.3 Implement unit billing models: `UnitBillModel` (`unit_bills`), `UnitBillConceptModel` (`unit_bill_concepts`), `UnitBillConsumptionModel` (`unit_bill_consumptions`), and `UnitBillChargeModel` (`unit_bill_charges`).
- [x] 2.4 Verify foreign key constraints, `ON DELETE CASCADE` on line items, `ON DELETE RESTRICT` on master units, and unique constraints.

## 3. Alembic Migration Setup

- [x] 3.1 Initialize Alembic configuration (`alembic.ini` and `backend/alembic/env.py`) connected to `backend/app/models.py`.
- [x] 3.2 Generate the initial baseline migration script (`0001_initial_condominium_schema.py`) supporting PostgreSQL.

## 4. Verification & Validation

- [x] 4.1 Create a verification script (`backend/tests/test_schema_integrity.py` or verification script) to test table creation, relationships, and constraints.
- [x] 4.2 Test inserting a billing period, a master unit, a building expense report, and a unit bill with metered consumption.
- [x] 4.3 Validate that `ON DELETE RESTRICT` prevents deletion of units with bills and `ON DELETE CASCADE` cleans up orphaned line items.

## 5. CI/CD Pipeline & Cloud Run Migration Job

- [x] 5.1 Implement `backend/Dockerfile` supporting both the FastAPI web server and Alembic CLI migrations.
- [x] 5.2 Add `google_cloud_run_v2_job.migrate` resource in `infra/main.tf` attached to the private VPC subnet and Cloud SQL environment variables.
- [x] 5.3 Implement `.github/workflows/deploy-backend.yml` executing WIF authentication, Docker build & push, Cloud Run Migration Job with `--wait`, and Cloud Run Service deployment.
