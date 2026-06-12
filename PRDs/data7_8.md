<img src="https://r2cdn.perplexity.ai/pplx-full-logo-primary-dark%402x.png" style="height:64px;margin-right:32px"/>

# now please generate for M7_8

For M7_8, the right design is a **finance-ledger-centered schema** where fee setup, student fee assignment, invoice snapshots, payment allocation, receipt issuance, refund handling, and reconciliation are modeled separately, because the PRD requires auditable billing, partial payments, immutable receipts, and reconciliation without rewriting source transactions.[^1]
I would also keep student, enrollment, class, section, academic year, and guardian references outside finance and consume them from SIS and configuration modules, because the PRD explicitly says finance must not recreate student master data and must use platform master modules as the source of truth.[^1]

## 1. Entity list

These are the production entities I would use for Module 7.8, aligned to fee setup, billing, collections, refunds, reconciliation, reporting, and auditability.[^1]

- `fee_categories` — tenant and school scoped charge heads such as tuition, transport, books, exam, and lab, because schools need a master list of charge types and optional services must be modeled explicitly.[^1]
- `fee_structures` — versioned billing templates by academic year, grade, section, and billing context, because fee structures must be versioned by academic year and may be defined by grade, section, or student group.[^1]
- `fee_structure_lines` — charge rows inside a structure, including recurrence and due behavior, because invoices must be generated from structured fee definitions rather than ad hoc free text.[^1]
- `late_fee_rules` — penalty configuration by school context, because overdue handling must be configurable and late fee policies are explicitly required.[^1]
- `discounts` — configurable discount definitions for fee setup and invoice generation, because discounts may be fixed amount or percentage and must be reported separately from scholarships.[^1]
- `scholarships` — named student aid records with effective dates and status, because scholarships must be modeled separately from standard discounts for reporting clarity.[^1]
- `student_fee_assignments` — one active fee structure per student per billing context, because each student must be assigned one active fee structure per billing context and assignment history must be preserved.[^1]
- `student_fee_assignment_addons` — optional services and student-specific add-on billing rows, because transport, hostel, and activity charges should be add-on items rather than forced into every student invoice.[^1]
- `invoices` — invoice headers with immutable financial snapshots, because invoice totals must be stored as calculated snapshots and not recomputed later from mutable setup rules.[^1]
- `invoice_line_items` — detailed line-level charge, discount, scholarship, tax, penalty, and adjustment snapshots, because invoice details must remain historically trustworthy after issuance.[^1]
- `payment_transactions` — every payment attempt or confirmed collection, online or offline, because failed transactions must remain visible and payment state is independent from invoice state.[^1]
- `payment_allocations` — mappings from one payment to one or more invoices, because partial payments are allowed and split allocation may be enabled by school policy.[^1]
- `receipts` — immutable proof-of-payment records, because receipts are generated only for successful or confirmed payments and cannot be edited after issuance.[^1]
- `refunds` — partial or full refund workflow records, because refunds require reason, original payment reference, refund mode, and auditable approval handling.[^1]
- `adjustments` — controlled manual balance corrections, reversals, waivers, and write-offs, because manual adjustments require mandatory reason capture and actor audit.[^1]
- `credit_balances` — unapplied credits for future use or refund, because overpayments must become credit or enter refund workflow and must never disappear silently.[^1]
- `ledger_entries` — immutable accounting or operational ledger rows, because refunds, reversals, collections, and adjustments need auditable financial movement history.[^1]
- `reconciliation_batches` — imported or manual reconciliation sessions, because reconciliation compares internal payments to external settlement evidence such as gateway reports or bank imports.[^1]
- `reconciliation_items` — match results for each external settlement row, because a transaction may be unmatched, mismatched, or manually resolved without mutating the original payment record.[^1]
- `finance_audit_logs` — finance-specific audit history, because invoice issue, payment record, receipt issue, refund, reversal, and reconciliation actions must all be fully auditable.[^1]


## 2. Table definitions

The DDL below is designed for strict tenant and school isolation, versioned fee setup, immutable billing snapshots, split payment support, reversal-only receipt correction, and reconciliation-safe transaction history.[^1]
It assumes `tenants`, `schools`, `users`, `students`, `student_enrollments`, `academic_years`, `grade_levels`, and `sections` already exist in the platform core and SIS layers, because the PRD says finance must reference those master modules instead of duplicating them.[^1]

```sql
CREATE EXTENSION IF NOT EXISTS pgcrypto;
CREATE EXTENSION IF NOT EXISTS citext;

CREATE TYPE fee_category_type AS ENUM (
    'tuition','transport','books','exam','lab','activity','hostel','other'
);

CREATE TYPE billing_frequency AS ENUM (
    'monthly','term','one_time','ad_hoc'
);

CREATE TYPE fee_structure_status AS ENUM (
    'draft','active','inactive','archived'
);

CREATE TYPE fee_assignment_status AS ENUM (
    'draft','active','ended','cancelled'
);

CREATE TYPE discount_type AS ENUM (
    'fixed_amount','percentage'
);

CREATE TYPE scholarship_type AS ENUM (
    'fixed_amount','percentage'
);

CREATE TYPE invoice_state AS ENUM (
    'draft','issued','partially_paid','paid','overdue','cancelled','refunded_partial','refunded_full'
);

CREATE TYPE invoice_line_type AS ENUM (
    'charge','discount','scholarship','tax','penalty','adjustment'
);

CREATE TYPE payment_method AS ENUM (
    'cash','cheque','bank_transfer','pos','card','upi','net_banking','wallet','other'
);

CREATE TYPE payment_channel AS ENUM (
    'offline','online_gateway','bank_import','manual'
);

CREATE TYPE payment_status AS ENUM (
    'initiated','pending','success','failed','cancelled','expired','reconciled',
    'refund_initiated','refunded_partial','refunded_full','refund_failed'
);

CREATE TYPE receipt_status AS ENUM (
    'issued','reversed'
);

CREATE TYPE refund_status AS ENUM (
    'requested','approved','rejected','processing','completed','failed'
);

CREATE TYPE refund_mode AS ENUM (
    'original_method','bank_transfer','cash','cheque','credit_balance'
);

CREATE TYPE adjustment_type AS ENUM (
    'discount_adjustment','penalty_waiver','balance_correction','reversal','write_off'
);

CREATE TYPE credit_balance_status AS ENUM (
    'available','partially_applied','fully_applied','refunded','expired'
);

CREATE TYPE late_fee_mode AS ENUM (
    'fixed_amount','percentage','per_day'
);

CREATE TYPE reconciliation_batch_status AS ENUM (
    'uploaded','processing','completed','failed'
);

CREATE TYPE reconciliation_match_status AS ENUM (
    'unreconciled','matched','mismatched','manually_resolved'
);

CREATE TABLE fee_categories (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
    school_id UUID NOT NULL REFERENCES schools(id) ON DELETE RESTRICT,
    code VARCHAR(50) NOT NULL,
    name VARCHAR(120) NOT NULL,
    category_type fee_category_type NOT NULL,
    optional_flag BOOLEAN NOT NULL DEFAULT FALSE,
    is_active BOOLEAN NOT NULL DEFAULT TRUE,
    deleted_at TIMESTAMPTZ,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID NULL REFERENCES users(id),
    updated_by UUID NULL REFERENCES users(id),
    CONSTRAINT uq_fee_category_code UNIQUE (tenant_id, school_id, code)
);

CREATE TABLE fee_structures (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
    school_id UUID NOT NULL REFERENCES schools(id) ON DELETE RESTRICT,
    academic_year_id UUID NOT NULL REFERENCES academic_years(id) ON DELETE RESTRICT,
    grade_level_id UUID NULL REFERENCES grade_levels(id) ON DELETE RESTRICT,
    section_id UUID NULL REFERENCES sections(id) ON DELETE RESTRICT,
    billing_context_code VARCHAR(50) NOT NULL,
    billing_frequency billing_frequency NOT NULL,
    currency_code CHAR(3) NOT NULL DEFAULT 'INR',
    version_no INTEGER NOT NULL,
    status fee_structure_status NOT NULL DEFAULT 'draft',
    effective_from DATE NOT NULL,
    effective_to DATE NULL,
    notes TEXT,
    deleted_at TIMESTAMPTZ,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID NULL REFERENCES users(id),
    updated_by UUID NULL REFERENCES users(id),
    CONSTRAINT ck_fee_structure_dates CHECK (effective_to IS NULL OR effective_to >= effective_from),
    CONSTRAINT ck_fee_structure_version CHECK (version_no > 0),
    CONSTRAINT uq_fee_structure_version UNIQUE (
        tenant_id, school_id, academic_year_id,
        COALESCE(grade_level_id, '00000000-0000-0000-0000-000000000000'::uuid),
        COALESCE(section_id, '00000000-0000-0000-0000-000000000000'::uuid),
        billing_context_code,
        version_no
    )
);

CREATE TABLE fee_structure_lines (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
    school_id UUID NOT NULL REFERENCES schools(id) ON DELETE RESTRICT,
    fee_structure_id UUID NOT NULL REFERENCES fee_structures(id) ON DELETE CASCADE,
    fee_category_id UUID NOT NULL REFERENCES fee_categories(id) ON DELETE RESTRICT,
    line_description VARCHAR(255) NOT NULL,
    recurrence_type billing_frequency NOT NULL,
    amount NUMERIC(14,2) NOT NULL,
    quantity NUMERIC(12,2) NOT NULL DEFAULT 1,
    due_offset_days INTEGER NOT NULL DEFAULT 0,
    optional_flag BOOLEAN NOT NULL DEFAULT FALSE,
    sort_order INTEGER NOT NULL DEFAULT 1,
    deleted_at TIMESTAMPTZ,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID NULL REFERENCES users(id),
    updated_by UUID NULL REFERENCES users(id),
    CONSTRAINT ck_fee_structure_line_amount CHECK (amount >= 0),
    CONSTRAINT ck_fee_structure_line_qty CHECK (quantity > 0),
    CONSTRAINT ck_fee_structure_line_sort CHECK (sort_order > 0)
);

CREATE TABLE late_fee_rules (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
    school_id UUID NOT NULL REFERENCES schools(id) ON DELETE RESTRICT,
    rule_name VARCHAR(120) NOT NULL,
    trigger_days INTEGER NOT NULL,
    mode late_fee_mode NOT NULL,
    value NUMERIC(14,2) NOT NULL,
    cap_amount NUMERIC(14,2),
    is_active BOOLEAN NOT NULL DEFAULT TRUE,
    deleted_at TIMESTAMPTZ,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID NULL REFERENCES users(id),
    updated_by UUID NULL REFERENCES users(id),
    CONSTRAINT ck_late_fee_trigger CHECK (trigger_days >= 0),
    CONSTRAINT ck_late_fee_value CHECK (value >= 0),
    CONSTRAINT ck_late_fee_cap CHECK (cap_amount IS NULL OR cap_amount >= 0)
);

CREATE TABLE discounts (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
    school_id UUID NOT NULL REFERENCES schools(id) ON DELETE RESTRICT,
    name VARCHAR(120) NOT NULL,
    discount_type discount_type NOT NULL,
    value NUMERIC(14,2) NOT NULL,
    reason VARCHAR(255),
    approval_required BOOLEAN NOT NULL DEFAULT FALSE,
    is_active BOOLEAN NOT NULL DEFAULT TRUE,
    deleted_at TIMESTAMPTZ,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID NULL REFERENCES users(id),
    updated_by UUID NULL REFERENCES users(id),
    CONSTRAINT ck_discount_value CHECK (value >= 0)
);

CREATE TABLE scholarships (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
    school_id UUID NOT NULL REFERENCES schools(id) ON DELETE RESTRICT,
    student_id UUID NOT NULL REFERENCES students(id) ON DELETE RESTRICT,
    scholarship_code VARCHAR(50) NOT NULL,
    scholarship_name VARCHAR(120) NOT NULL,
    scholarship_type scholarship_type NOT NULL,
    value NUMERIC(14,2) NOT NULL,
    sponsor_name VARCHAR(120),
    start_date DATE NOT NULL,
    end_date DATE NULL,
    status VARCHAR(30) NOT NULL,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID NULL REFERENCES users(id),
    updated_by UUID NULL REFERENCES users(id),
    CONSTRAINT ck_scholarship_value CHECK (value >= 0),
    CONSTRAINT ck_scholarship_dates CHECK (end_date IS NULL OR end_date >= start_date),
    CONSTRAINT uq_student_scholarship_code UNIQUE (tenant_id, school_id, student_id, scholarship_code)
);

CREATE TABLE student_fee_assignments (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
    school_id UUID NOT NULL REFERENCES schools(id) ON DELETE RESTRICT,
    student_id UUID NOT NULL REFERENCES students(id) ON DELETE RESTRICT,
    enrollment_id UUID NOT NULL REFERENCES student_enrollments(id) ON DELETE RESTRICT,
    fee_structure_id UUID NOT NULL REFERENCES fee_structures(id) ON DELETE RESTRICT,
    academic_year_id UUID NOT NULL REFERENCES academic_years(id) ON DELETE RESTRICT,
    billing_context_code VARCHAR(50) NOT NULL,
    effective_from DATE NOT NULL,
    effective_to DATE NULL,
    status fee_assignment_status NOT NULL DEFAULT 'active',
    assigned_by UUID NOT NULL REFERENCES users(id),
    notes TEXT,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_by UUID NULL REFERENCES users(id),
    CONSTRAINT ck_assignment_dates CHECK (effective_to IS NULL OR effective_to >= effective_from)
);

CREATE TABLE student_fee_assignment_addons (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
    school_id UUID NOT NULL REFERENCES schools(id) ON DELETE RESTRICT,
    student_fee_assignment_id UUID NOT NULL REFERENCES student_fee_assignments(id) ON DELETE CASCADE,
    fee_category_id UUID NOT NULL REFERENCES fee_categories(id) ON DELETE RESTRICT,
    addon_name VARCHAR(120) NOT NULL,
    billing_frequency billing_frequency NOT NULL,
    amount NUMERIC(14,2) NOT NULL,
    effective_from DATE NOT NULL,
    effective_to DATE NULL,
    is_active BOOLEAN NOT NULL DEFAULT TRUE,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID NULL REFERENCES users(id),
    updated_by UUID NULL REFERENCES users(id),
    CONSTRAINT ck_assignment_addon_amount CHECK (amount >= 0),
    CONSTRAINT ck_assignment_addon_dates CHECK (effective_to IS NULL OR effective_to >= effective_from)
);

CREATE TABLE invoices (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
    school_id UUID NOT NULL REFERENCES schools(id) ON DELETE RESTRICT,
    student_id UUID NOT NULL REFERENCES students(id) ON DELETE RESTRICT,
    enrollment_id UUID NOT NULL REFERENCES student_enrollments(id) ON DELETE RESTRICT,
    academic_year_id UUID NOT NULL REFERENCES academic_years(id) ON DELETE RESTRICT,
    grade_level_id UUID NULL REFERENCES grade_levels(id) ON DELETE RESTRICT,
    section_id UUID NULL REFERENCES sections(id) ON DELETE RESTRICT,
    student_fee_assignment_id UUID NULL REFERENCES student_fee_assignments(id) ON DELETE RESTRICT,
    invoice_no VARCHAR(50) NOT NULL,
    billing_context_code VARCHAR(50) NOT NULL,
    billing_frequency billing_frequency NOT NULL,
    period_start DATE NOT NULL,
    period_end DATE NOT NULL,
    due_date DATE NOT NULL,
    subtotal NUMERIC(14,2) NOT NULL,
    discount_total NUMERIC(14,2) NOT NULL DEFAULT 0,
    scholarship_total NUMERIC(14,2) NOT NULL DEFAULT 0,
    tax_total NUMERIC(14,2) NOT NULL DEFAULT 0,
    penalty_total NUMERIC(14,2) NOT NULL DEFAULT 0,
    adjustment_total NUMERIC(14,2) NOT NULL DEFAULT 0,
    paid_total NUMERIC(14,2) NOT NULL DEFAULT 0,
    refunded_total NUMERIC(14,2) NOT NULL DEFAULT 0,
    balance_due NUMERIC(14,2) NOT NULL,
    state invoice_state NOT NULL DEFAULT 'draft',
    issued_at TIMESTAMPTZ,
    issued_by UUID NULL REFERENCES users(id),
    cancelled_at TIMESTAMPTZ,
    cancelled_by UUID NULL REFERENCES users(id),
    cancel_reason TEXT,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID NOT NULL REFERENCES users(id),
    updated_by UUID NULL REFERENCES users(id),
    CONSTRAINT ck_invoice_period CHECK (period_end >= period_start),
    CONSTRAINT ck_invoice_amounts CHECK (
        subtotal >= 0 AND discount_total >= 0 AND scholarship_total >= 0 AND
        tax_total >= 0 AND penalty_total >= 0 AND adjustment_total >= 0 AND
        paid_total >= 0 AND refunded_total >= 0 AND balance_due >= 0
    ),
    CONSTRAINT uq_invoice_no UNIQUE (tenant_id, school_id, invoice_no)
);

CREATE TABLE invoice_line_items (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
    school_id UUID NOT NULL REFERENCES schools(id) ON DELETE RESTRICT,
    invoice_id UUID NOT NULL REFERENCES invoices(id) ON DELETE CASCADE,
    fee_category_id UUID NULL REFERENCES fee_categories(id) ON DELETE RESTRICT,
    fee_structure_line_id UUID NULL REFERENCES fee_structure_lines(id) ON DELETE SET NULL,
    line_type invoice_line_type NOT NULL,
    description VARCHAR(255) NOT NULL,
    quantity NUMERIC(12,2) NOT NULL DEFAULT 1,
    unit_amount NUMERIC(14,2) NOT NULL DEFAULT 0,
    line_discount NUMERIC(14,2) NOT NULL DEFAULT 0,
    scholarship_amount NUMERIC(14,2) NOT NULL DEFAULT 0,
    tax_amount NUMERIC(14,2) NOT NULL DEFAULT 0,
    penalty_amount NUMERIC(14,2) NOT NULL DEFAULT 0,
    line_total NUMERIC(14,2) NOT NULL,
    calculation_sequence_no INTEGER NOT NULL,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID NULL REFERENCES users(id),
    CONSTRAINT ck_invoice_line_qty CHECK (quantity > 0),
    CONSTRAINT ck_invoice_line_nonnegative CHECK (
        unit_amount >= 0 AND line_discount >= 0 AND scholarship_amount >= 0 AND
        tax_amount >= 0 AND penalty_amount >= 0
    ),
    CONSTRAINT ck_invoice_line_seq CHECK (calculation_sequence_no > 0)
);

CREATE TABLE payment_transactions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
    school_id UUID NOT NULL REFERENCES schools(id) ON DELETE RESTRICT,
    student_id UUID NOT NULL REFERENCES students(id) ON DELETE RESTRICT,
    invoice_id UUID NULL REFERENCES invoices(id) ON DELETE RESTRICT,
    amount NUMERIC(14,2) NOT NULL,
    currency_code CHAR(3) NOT NULL DEFAULT 'INR',
    payment_method payment_method NOT NULL,
    payment_channel payment_channel NOT NULL,
    gateway_ref VARCHAR(120),
    bank_ref VARCHAR(120),
    reference_no VARCHAR(120),
    instrument_no_last4 VARCHAR(4),
    received_at TIMESTAMPTZ,
    received_by UUID NULL REFERENCES users(id),
    status payment_status NOT NULL DEFAULT 'initiated',
    duplicate_check_hash TEXT,
    provider_payload JSONB,
    idempotency_key VARCHAR(120),
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID NULL REFERENCES users(id),
    updated_by UUID NULL REFERENCES users(id),
    CONSTRAINT ck_payment_amount CHECK (amount > 0),
    CONSTRAINT uq_payment_idempotency UNIQUE (tenant_id, school_id, idempotency_key)
);

CREATE TABLE payment_allocations (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
    school_id UUID NOT NULL REFERENCES schools(id) ON DELETE RESTRICT,
    payment_transaction_id UUID NOT NULL REFERENCES payment_transactions(id) ON DELETE CASCADE,
    invoice_id UUID NOT NULL REFERENCES invoices(id) ON DELETE RESTRICT,
    allocated_amount NUMERIC(14,2) NOT NULL,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID NULL REFERENCES users(id),
    CONSTRAINT ck_allocated_amount CHECK (allocated_amount > 0),
    CONSTRAINT uq_payment_invoice_alloc UNIQUE (payment_transaction_id, invoice_id)
);

CREATE TABLE receipts (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
    school_id UUID NOT NULL REFERENCES schools(id) ON DELETE RESTRICT,
    receipt_no VARCHAR(50) NOT NULL,
    payment_transaction_id UUID NOT NULL REFERENCES payment_transactions(id) ON DELETE RESTRICT,
    issued_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    issued_by UUID NOT NULL REFERENCES users(id),
    total_amount NUMERIC(14,2) NOT NULL,
    status receipt_status NOT NULL DEFAULT 'issued',
    reversed_receipt_id UUID NULL REFERENCES receipts(id) ON DELETE SET NULL,
    reversal_reason TEXT,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    CONSTRAINT ck_receipt_amount CHECK (total_amount > 0),
    CONSTRAINT uq_receipt_no UNIQUE (tenant_id, school_id, receipt_no)
);

CREATE TABLE refunds (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
    school_id UUID NOT NULL REFERENCES schools(id) ON DELETE RESTRICT,
    payment_transaction_id UUID NOT NULL REFERENCES payment_transactions(id) ON DELETE RESTRICT,
    refund_no VARCHAR(50) NOT NULL,
    amount NUMERIC(14,2) NOT NULL,
    reason TEXT NOT NULL,
    refund_mode refund_mode NOT NULL,
    requested_by UUID NOT NULL REFERENCES users(id),
    requested_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    approved_by UUID NULL REFERENCES users(id),
    approved_at TIMESTAMPTZ,
    completed_at TIMESTAMPTZ,
    status refund_status NOT NULL DEFAULT 'requested',
    external_ref VARCHAR(120),
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    CONSTRAINT ck_refund_amount CHECK (amount > 0),
    CONSTRAINT uq_refund_no UNIQUE (tenant_id, school_id, refund_no)
);

CREATE TABLE adjustments (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
    school_id UUID NOT NULL REFERENCES schools(id) ON DELETE RESTRICT,
    student_id UUID NOT NULL REFERENCES students(id) ON DELETE RESTRICT,
    invoice_id UUID NULL REFERENCES invoices(id) ON DELETE RESTRICT,
    payment_transaction_id UUID NULL REFERENCES payment_transactions(id) ON DELETE RESTRICT,
    adjustment_type adjustment_type NOT NULL,
    amount NUMERIC(14,2) NOT NULL,
    reason TEXT NOT NULL,
    approved_by UUID NULL REFERENCES users(id),
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID NOT NULL REFERENCES users(id),
    CONSTRAINT ck_adjustment_amount CHECK (amount <> 0)
);

CREATE TABLE credit_balances (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
    school_id UUID NOT NULL REFERENCES schools(id) ON DELETE RESTRICT,
    student_id UUID NOT NULL REFERENCES students(id) ON DELETE RESTRICT,
    source_payment_transaction_id UUID NOT NULL REFERENCES payment_transactions(id) ON DELETE RESTRICT,
    available_amount NUMERIC(14,2) NOT NULL,
    status credit_balance_status NOT NULL DEFAULT 'available',
    expires_at TIMESTAMPTZ,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID NULL REFERENCES users(id),
    updated_by UUID NULL REFERENCES users(id),
    CONSTRAINT ck_credit_amount CHECK (available_amount >= 0)
);

CREATE TABLE ledger_entries (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
    school_id UUID NOT NULL REFERENCES schools(id) ON DELETE RESTRICT,
    source_type VARCHAR(50) NOT NULL,
    source_id UUID NOT NULL,
    student_id UUID NULL REFERENCES students(id) ON DELETE RESTRICT,
    invoice_id UUID NULL REFERENCES invoices(id) ON DELETE RESTRICT,
    payment_transaction_id UUID NULL REFERENCES payment_transactions(id) ON DELETE RESTRICT,
    refund_id UUID NULL REFERENCES refunds(id) ON DELETE RESTRICT,
    adjustment_id UUID NULL REFERENCES adjustments(id) ON DELETE RESTRICT,
    entry_type VARCHAR(50) NOT NULL,
    debit_amount NUMERIC(14,2) NOT NULL DEFAULT 0,
    credit_amount NUMERIC(14,2) NOT NULL DEFAULT 0,
    posted_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    account_code VARCHAR(50),
    immutable_hash TEXT NOT NULL,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID NULL REFERENCES users(id),
    CONSTRAINT ck_ledger_nonnegative CHECK (debit_amount >= 0 AND credit_amount >= 0),
    CONSTRAINT ck_ledger_one_side CHECK (
        (debit_amount = 0 AND credit_amount > 0) OR
        (credit_amount = 0 AND debit_amount > 0)
    )
);

CREATE TABLE reconciliation_batches (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
    school_id UUID NOT NULL REFERENCES schools(id) ON DELETE RESTRICT,
    source_type VARCHAR(50) NOT NULL,
    batch_date DATE NOT NULL,
    uploaded_by UUID NOT NULL REFERENCES users(id),
    status reconciliation_batch_status NOT NULL DEFAULT 'uploaded',
    source_file_name VARCHAR(255),
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE TABLE reconciliation_items (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
    school_id UUID NOT NULL REFERENCES schools(id) ON DELETE RESTRICT,
    reconciliation_batch_id UUID NOT NULL REFERENCES reconciliation_batches(id) ON DELETE CASCADE,
    payment_transaction_id UUID NULL REFERENCES payment_transactions(id) ON DELETE RESTRICT,
    external_ref VARCHAR(120) NOT NULL,
    external_amount NUMERIC(14,2) NOT NULL,
    match_status reconciliation_match_status NOT NULL DEFAULT 'unreconciled',
    resolved_by UUID NULL REFERENCES users(id),
    resolved_at TIMESTAMPTZ,
    notes TEXT,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    CONSTRAINT ck_recon_external_amount CHECK (external_amount > 0)
);

CREATE TABLE finance_audit_logs (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE RESTRICT,
    school_id UUID NOT NULL REFERENCES schools(id) ON DELETE RESTRICT,
    entity_type VARCHAR(50) NOT NULL,
    entity_id UUID NOT NULL,
    action VARCHAR(50) NOT NULL,
    performed_by UUID NULL REFERENCES users(id) ON DELETE SET NULL,
    metadata JSONB,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

Only setup tables such as `fee_categories`, `fee_structures`, `fee_structure_lines`, `late_fee_rules`, and `discounts` get soft delete, because the PRD allows soft deletion only for setup records where no financial transaction has occurred and says financial transactions themselves must never be hard deleted.[^1]

## 3. Constraints and indexes

The most important enforcement points for M7_8 are one active fee structure per student per billing context, immutable invoice snapshots, no payment on draft or cancelled invoices, immutable receipts, auditable failed payments, overpayment control, and reconciliation that adds state without changing source payment records.[^1]

**Constraints**

- `student_fee_assignments` should have a partial unique index for one active assignment per student, academic year, and billing context, because each student must have one active fee structure per billing context.[^1]
- `invoices` should enforce at least one line item through application transaction logic or a deferred trigger, because every invoice must contain one or more line items.[^1]
- `invoices.state` must be advanced by controlled workflows, because invoices may start as draft, then be issued, paid, overdue, cancelled, or refunded under policy.[^1]
- Payment creation should allow `invoice_id` to be null for advance or later-allocation flows, because one payment may be allocated to one or many invoices and overpayment may become unapplied credit.[^1]
- Allocation totals must not exceed confirmed payment amount, because overpayments must be handled through credit or refund and not silently vanish.[^1]
- Receipts should be insert-only and reversal-only, because issued receipts are immutable and corrections require reversal or reissuance rather than edit.[^1]
- Refund totals across one payment must not exceed confirmed collected amount net of prior refunds, because negative invoice totals are not allowed and refund handling must remain controlled.[^1]
- Reconciliation state belongs on reconciliation tables, not on original payment mutation, because reconciliation must not alter original payment records.[^1]

```sql
CREATE UNIQUE INDEX uq_active_student_fee_assignment
ON student_fee_assignments (
    tenant_id, school_id, student_id, academic_year_id, billing_context_code
)
WHERE status = 'active';

CREATE INDEX idx_fee_structures_lookup
ON fee_structures (
    tenant_id, school_id, academic_year_id, billing_context_code, status, grade_level_id, section_id
);

CREATE INDEX idx_fee_structure_lines_structure
ON fee_structure_lines (fee_structure_id, sort_order)
WHERE deleted_at IS NULL;

CREATE INDEX idx_invoices_student_state_due
ON invoices (tenant_id, school_id, student_id, state, due_date);

CREATE INDEX idx_invoices_section_due
ON invoices (tenant_id, school_id, academic_year_id, grade_level_id, section_id, state, due_date);

CREATE INDEX idx_invoices_overdue
ON invoices (tenant_id, school_id, due_date, balance_due)
WHERE state IN ('issued', 'partially_paid', 'overdue');

CREATE INDEX idx_payment_transactions_student_status
ON payment_transactions (tenant_id, school_id, student_id, status, received_at DESC);

CREATE INDEX idx_payment_transactions_refs
ON payment_transactions (tenant_id, school_id, gateway_ref, bank_ref, reference_no);

CREATE INDEX idx_payment_allocations_invoice
ON payment_allocations (tenant_id, school_id, invoice_id, payment_transaction_id);

CREATE UNIQUE INDEX uq_receipts_payment
ON receipts (payment_transaction_id)
WHERE status = 'issued';

CREATE INDEX idx_refunds_status_date
ON refunds (tenant_id, school_id, status, requested_at DESC);

CREATE INDEX idx_credit_balances_student
ON credit_balances (tenant_id, school_id, student_id, status, available_amount);

CREATE INDEX idx_ledger_entries_student_posted
ON ledger_entries (tenant_id, school_id, student_id, posted_at DESC);

CREATE INDEX idx_ledger_entries_source
ON ledger_entries (tenant_id, school_id, source_type, source_id);

CREATE INDEX idx_reconciliation_items_match
ON reconciliation_items (tenant_id, school_id, match_status, external_ref);

CREATE INDEX idx_finance_audit_logs_entity
ON finance_audit_logs (tenant_id, school_id, entity_type, entity_id, created_at DESC);
```

I would also add deferred triggers for allocation-sum validation, refund ceiling validation, invoice-line existence on issue, and overdue-state refresh, because those rules are transactional and depend on aggregate context rather than a single row.[^1]
For row-level security, every finance table should be scoped by `tenant_id` and `school_id`, because the PRD requires every finance record to belong to exactly one tenant and one school and forbids cross-tenant references for invoices, payments, receipts, refunds, and ledger entries.[^1]

## 4. Sample records and query patterns

These examples cover the core M7_8 flows: fee setup, student assignment, invoice issue, payment recording, receipt issuance, and ledger or defaulter reporting.[^1]

```sql
INSERT INTO fee_categories (
    id, tenant_id, school_id, code, name, category_type, optional_flag, created_by
) VALUES (
    'a1000000-0000-0000-0000-000000000001',
    '10000000-0000-0000-0000-000000000001',
    '51000000-0000-0000-0000-000000000001',
    'TUITION',
    'Tuition Fee',
    'tuition',
    FALSE,
    '00000000-0000-0000-0000-000000000002'
);

INSERT INTO fee_structures (
    id, tenant_id, school_id, academic_year_id, grade_level_id, section_id,
    billing_context_code, billing_frequency, version_no, status,
    effective_from, created_by
) VALUES (
    'a2000000-0000-0000-0000-000000000001',
    '10000000-0000-0000-0000-000000000001',
    '51000000-0000-0000-0000-000000000001',
    '53000000-0000-0000-0000-000000000001',
    '54000000-0000-0000-0000-000000000001',
    NULL,
    'STANDARD_TUITION',
    'monthly',
    1,
    'active',
    '2026-04-01',
    '00000000-0000-0000-0000-000000000002'
);

INSERT INTO fee_structure_lines (
    id, tenant_id, school_id, fee_structure_id, fee_category_id, line_description,
    recurrence_type, amount, quantity, due_offset_days, sort_order, created_by
) VALUES (
    'a3000000-0000-0000-0000-000000000001',
    '10000000-0000-0000-0000-000000000001',
    '51000000-0000-0000-0000-000000000001',
    'a2000000-0000-0000-0000-000000000001',
    'a1000000-0000-0000-0000-000000000001',
    'Monthly tuition',
    'monthly',
    5000.00,
    1,
    0,
    1,
    '00000000-0000-0000-0000-000000000002'
);

INSERT INTO student_fee_assignments (
    id, tenant_id, school_id, student_id, enrollment_id, fee_structure_id,
    academic_year_id, billing_context_code, effective_from, status, assigned_by
) VALUES (
    'a4000000-0000-0000-0000-000000000001',
    '10000000-0000-0000-0000-000000000001',
    '51000000-0000-0000-0000-000000000001',
    '71000000-0000-0000-0000-000000000001',
    '73000000-0000-0000-0000-000000000001',
    'a2000000-0000-0000-0000-000000000001',
    '53000000-0000-0000-0000-000000000001',
    'STANDARD_TUITION',
    '2026-04-01',
    'active',
    '00000000-0000-0000-0000-000000000002'
);

INSERT INTO invoices (
    id, tenant_id, school_id, student_id, enrollment_id, academic_year_id,
    invoice_no, billing_context_code, billing_frequency, period_start, period_end,
    due_date, subtotal, discount_total, scholarship_total, tax_total, penalty_total,
    adjustment_total, paid_total, refunded_total, balance_due, state, issued_at,
    issued_by, created_by
) VALUES (
    'a5000000-0000-0000-0000-000000000001',
    '10000000-0000-0000-0000-000000000001',
    '51000000-0000-0000-0000-000000000001',
    '71000000-0000-0000-0000-000000000001',
    '73000000-0000-0000-0000-000000000001',
    '53000000-0000-0000-0000-000000000001',
    'INV-2026-000001',
    'STANDARD_TUITION',
    'monthly',
    '2026-06-01',
    '2026-06-30',
    '2026-06-10',
    5000.00,
    0.00,
    0.00,
    0.00,
    0.00,
    0.00,
    0.00,
    0.00,
    5000.00,
    'issued',
    now(),
    '00000000-0000-0000-0000-000000000002',
    '00000000-0000-0000-0000-000000000002'
);

INSERT INTO payment_transactions (
    id, tenant_id, school_id, student_id, invoice_id, amount, currency_code,
    payment_method, payment_channel, reference_no, received_at, received_by,
    status, idempotency_key, created_by
) VALUES (
    'a6000000-0000-0000-0000-000000000001',
    '10000000-0000-0000-0000-000000000001',
    '51000000-0000-0000-0000-000000000001',
    '71000000-0000-0000-0000-000000000001',
    'a5000000-0000-0000-0000-000000000001',
    5000.00,
    'INR',
    'upi',
    'online_gateway',
    'PAYREF12345',
    now(),
    '00000000-0000-0000-0000-000000000002',
    'success',
    'idem-20260611-001',
    '00000000-0000-0000-0000-000000000002'
);

INSERT INTO payment_allocations (
    id, tenant_id, school_id, payment_transaction_id, invoice_id, allocated_amount, created_by
) VALUES (
    'a7000000-0000-0000-0000-000000000001',
    '10000000-0000-0000-0000-000000000001',
    '51000000-0000-0000-0000-000000000001',
    'a6000000-0000-0000-0000-000000000001',
    'a5000000-0000-0000-0000-000000000001',
    5000.00,
    '00000000-0000-0000-0000-000000000002'
);

INSERT INTO receipts (
    id, tenant_id, school_id, receipt_no, payment_transaction_id, issued_by, total_amount
) VALUES (
    'a8000000-0000-0000-0000-000000000001',
    '10000000-0000-0000-0000-000000000001',
    '51000000-0000-0000-0000-000000000001',
    'RCT-2026-000001',
    'a6000000-0000-0000-0000-000000000001',
    '00000000-0000-0000-0000-000000000002',
    5000.00
);
```

**Query patterns**

1. Parent or student outstanding dues by child, because parents need to see unpaid and overdue invoices clearly in one place.[^1]
```sql
SELECT invoice_no, due_date, state, balance_due, period_start, period_end
FROM invoices
WHERE tenant_id = $1
  AND school_id = $2
  AND student_id = $3
  AND state IN ('issued', 'partially_paid', 'overdue')
ORDER BY due_date ASC;
```

2. Student ledger view, because the PRD calls for a chronological ledger of invoices, payments, refunds, adjustments, and credits.[^1]
```sql
SELECT posted_at, source_type, source_id, entry_type, debit_amount, credit_amount, account_code
FROM ledger_entries
WHERE tenant_id = $1
  AND school_id = $2
  AND student_id = $3
ORDER BY posted_at DESC, created_at DESC;
```

3. Defaulters dashboard by class and aging bucket, because principals and accountants need targeted overdue follow-up by class, section, and aging bucket.[^1]
```sql
SELECT i.grade_level_id,
       i.section_id,
       CASE
         WHEN current_date - i.due_date <= 30 THEN '0_30'
         WHEN current_date - i.due_date <= 60 THEN '31_60'
         WHEN current_date - i.due_date <= 90 THEN '61_90'
         ELSE '90_plus'
       END AS aging_bucket,
       COUNT(*) AS invoice_count,
       SUM(i.balance_due) AS outstanding_amount
FROM invoices i
WHERE i.tenant_id = $1
  AND i.school_id = $2
  AND i.state IN ('issued', 'partially_paid', 'overdue')
  AND i.balance_due > 0
GROUP BY i.grade_level_id, i.section_id, aging_bucket
ORDER BY i.grade_level_id, i.section_id, aging_bucket;
```

4. Reconciliation exception queue, because reconciliation must surface unmatched or mismatched items without changing original payment records.[^1]
```sql
SELECT ri.id, ri.external_ref, ri.external_amount, ri.match_status, pt.id AS payment_transaction_id, pt.amount
FROM reconciliation_items ri
LEFT JOIN payment_transactions pt
  ON pt.id = ri.payment_transaction_id
WHERE ri.tenant_id = $1
  AND ri.school_id = $2
  AND ri.reconciliation_batch_id = $3
  AND ri.match_status IN ('unreconciled', 'mismatched')
ORDER BY ri.created_at DESC;
```

5. Duplicate payment warning query, because the PRD requires suspiciously similar amount, student, reference, and timestamp combinations to be flagged.[^1]
```sql
SELECT id, student_id, amount, reference_no, received_at, status
FROM payment_transactions
WHERE tenant_id = $1
  AND school_id = $2
  AND student_id = $3
  AND amount = $4
  AND COALESCE(reference_no, '') = COALESCE($5, '')
  AND received_at >= ($6::timestamptz - interval '10 minutes')
  AND received_at <= ($6::timestamptz + interval '10 minutes')
ORDER BY received_at DESC;
```


## 5. Migration sequence and security-sensitive columns

M7_8 should be migrated after platform core, SIS, academic structure, and parent-student relationship models are stable, because finance depends on student identity, enrollment, academic year, class, section, and guardian visibility from those upstream modules.[^1]

**Migration sequence**

1. Create enums and then setup masters: `fee_categories`, `fee_structures`, `fee_structure_lines`, `late_fee_rules`, `discounts`, and `scholarships`, because fee policy and pricing setup must exist before assignment or invoicing.[^1]
2. Create `student_fee_assignments` and `student_fee_assignment_addons`, then add the partial unique index for one active assignment per billing context.[^1]
3. Create `invoices` and `invoice_line_items`, then add issue-time validation triggers so snapshot billing is locked at issuance.[^1]
4. Create `payment_transactions`, `payment_allocations`, `receipts`, `refunds`, `adjustments`, and `credit_balances`, because collections, receipts, reversals, and overpayment handling form one transactional cluster.[^1]
5. Create `ledger_entries`, `reconciliation_batches`, `reconciliation_items`, and `finance_audit_logs`, then add immutable-write triggers, RLS policies, and background jobs for overdue marking, receipt issuance, and reconciliation processing.[^1]

**Security-sensitive columns**

- `payment_transactions.gateway_ref`, `bank_ref`, `reference_no`, `instrument_no_last4`, and `provider_payload` are sensitive payment-operation fields and should be tightly role-scoped, because payment confirmation, failed attempts, and reconciliation evidence must remain auditable without broad exposure.[^1]
- `receipts.receipt_no`, `refunds.refund_no`, and reversal references are audit-sensitive numbering fields and should be insert-only under controlled services, because receipt numbering must be unique and issued receipts are immutable.[^1]
- `refunds.reason`, `adjustments.reason`, and related approver fields are sensitive operational-justification data and should be visible only to authorized finance roles, because refunds and manual adjustments require mandatory reason capture and actor audit.[^1]
- `finance_audit_logs.metadata` and `ledger_entries.immutable_hash` are compliance-critical integrity fields and should never be editable after insert, because the PRD requires full audit coverage and trustworthy financial history.[^1]
- Student-level dues, balances, and payment history are sensitive financial records and should be exposed to parents only for linked children and to students only where self-service is enabled, because the PRD restricts finance visibility by relationship and role.[^1]

This M7_8 design gives you a production-ready multi-tenant finance architecture with clean setup-to-billing flow, immutable financial records, split payments, refunds, reconciliation, and strong auditability without duplicating SIS ownership.[^1]
<span style="display:none">[^2][^3][^4][^5]</span>

<div align="center">⁂</div>

[^1]: PRD_M7_8.txt

[^2]: PRD_M7_16.txt

[^3]: PRD_M7_12.txt

[^4]: PRD_M7_14_15.txt

[^5]: PRD_M7_11.txt

