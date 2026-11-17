## ADDED Requirements

### Requirement: Master Units Catalog
The system SHALL maintain a master dimension table `units` storing the 420 condominium apartments with natural primary key `unit_code`, `tower`, `unit_number`, and `alicuota` percentage share.

#### Scenario: Apartment record creation and uniqueness
- **WHEN** an apartment with tower 'B' and unit number 504 is registered
- **THEN** the system stores primary key 'B-504' with an alícuota as `NUMERIC(8, 5)` and rejects any duplicate record with the same unit code or (tower, unit_number) combination.

#### Scenario: Restricting deletion of units with historical bills
- **WHEN** a deletion is attempted on unit 'B-504' that has associated records in `unit_bills`
- **THEN** the database enforces `ON DELETE RESTRICT` and blocks the deletion with a foreign key violation error.

### Requirement: Master Billing Periods
The system SHALL maintain a time dimension table `billing_periods` identifying monthly accounting periods using primary key `period_date` in `'YYYY-MM-01'` format, alongside integer `period_year` and `period_month`.

#### Scenario: Period registration and monthly uniqueness
- **WHEN** a period record for '2026-09-01' is registered
- **THEN** the system stores year 2026 and month 9, and enforces uniqueness on `period_date`.

### Requirement: 3-Tier Building Expense Hierarchy
The system SHALL persist condominium-level monthly expense reports across three hierarchically linked tables: `expense_reports` (header), `expense_sections` (categories), and `expense_items` (line items).

#### Scenario: Persisting general expense report
- **WHEN** a building expense report is persisted for period '2026-09-01'
- **THEN** it links via foreign key to `billing_periods.period_date`, calculates integer CLP subtotals and grand totals, and creates cascading child rows in `expense_sections` and `expense_items`.

#### Scenario: Cascading cleanup on report deletion
- **WHEN** an `expense_reports` row is deleted
- **THEN** all associated child rows in `expense_sections` and `expense_items` are automatically deleted via `ON DELETE CASCADE`.

### Requirement: Unit Billing Statements and Sub-tables
The system SHALL persist individual flat monthly bills in `unit_bills` linked to `units(unit_code)` and `billing_periods(period_date)`, itemized across `unit_bill_concepts`, `unit_bill_consumptions`, and `unit_bill_charges`.

#### Scenario: Registering flat monthly billing statement
- **WHEN** a monthly bill for unit 'B-504' and period '2026-09-01' is saved
- **THEN** the system verifies foreign keys against `units` and `billing_periods`, enforces unique constraint on `(unit_code, period_date)`, and saves integer CLP amounts.

#### Scenario: Metered utility consumption recording
- **WHEN** a hot water reading is recorded in `unit_bill_consumptions`
- **THEN** previous reading, current reading, consumption volume, rate, and total amount in integer CLP are stored linked to the unit bill.

### Requirement: Database Migrations with Alembic
The system SHALL provide an Alembic migration environment that translates the SQLAlchemy 2.0 ORM models into PostgreSQL 15 DDL schemas.

#### Scenario: Applying migrations to database
- **WHEN** `alembic upgrade head` is executed against the target PostgreSQL database
- **THEN** all tables, foreign keys, indexes, and constraints are created cleanly without errors.

### Requirement: Synchronous CI/CD Migration Execution via Cloud Run Job
The CI/CD pipeline SHALL execute database migrations using an ephemeral Cloud Run Job inside the private VPC with synchronous monitoring (`--wait`) prior to deploying new backend service revisions.

#### Scenario: Successful migration triggers service deployment
- **WHEN** the Cloud Run migration job executes `alembic upgrade head` and terminates with exit code 0
- **THEN** the CI/CD pipeline marks the migration step as passed and proceeds to deploy the updated backend service revision.

#### Scenario: Failed migration aborts deployment
- **WHEN** the Cloud Run migration job encounters an error and terminates with a non-zero exit code
- **THEN** the CI/CD pipeline immediately halts, aborts the backend service deployment, and leaves the currently running service and database untouched.
