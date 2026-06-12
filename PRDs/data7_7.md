<img src="https://r2cdn.perplexity.ai/pplx-full-logo-primary-dark%402x.png" style="height:64px;margin-right:32px"/>

# now please generate for M7_7

Yes — for M7_7, the correct design is an assessment-to-marks-to-verification-to-publication model with immutable published report cards, versioned revisions, automatic grade conversion, and full auditability, because the PRD defines Gradebook and Report Cards as the official academic record system and requires published results to be reliable and tamper-resistant.[^1]
I am also treating this module as downstream of SIS, academic structure, and teacher assignments, because the dependency chain in the PRD explicitly flows from student information and academic structure into assessments, marks, grade calculation, report cards, and finally parent/student visibility.[^1]

## 1. Entity list

These are the entities I would use for M7_7 in a production multi-tenant PostgreSQL design.[^1]

1. `grading_schemes` — tenant-scoped grading configuration, because each school can configure percentage-based, grade-based, or GPA-based grading.[^1]
2. `grading_scheme_ranges` — score-to-grade mapping rows, because grade conversion must be automatic and driven by school-configured rules.[^1]
3. `assessments` — academic evaluation header by academic year, grade, section, and subject.[^1]
4. `assessment_components` — theory, practical, project, assignment, and similar weighted parts, because an assessment may have multiple components and total weight must equal 100.[^1]
5. `assessment_student_rosters` — frozen assessment roster per student, because late-joining students may be added and mid-term section changes must preserve historical academic context.[^1]
6. `marks_entries` — per-student, per-component marks record storing raw marks, weighted marks, and final grade, because the PRD explicitly recommends storing all three values.[^1]
7. `marks_entry_import_batches` and `marks_entry_import_errors` — bulk upload control tables, because bulk uploads must be validated before import.[^1]
8. `assessment_verification_runs` — assessment-level verification gate, because results cannot move to verification until all mandatory marks are entered and authorized staff verifies them.[^1]
9. `report_card_templates` and `report_card_template_versions` — approved template model, because report cards must be generated from approved templates and template changes should not rewrite past report cards.[^1]
10. `report_cards` — immutable versioned report card headers, because published report cards become immutable and any correction must create a new version.[^1]
11. `report_card_subject_results` — subject-wise published snapshot rows for each report card.[^1]
12. `report_card_component_results` — component-wise published snapshot rows for each subject result, so published academic detail is preserved even if live grading rules later change.[^1]
13. `result_publications` — scheduled or immediate publication batches, because publication may be scheduled and only published results are visible to parents and students.[^1]
14. `result_publication_items` — per-student publication membership rows, because publication scope and history must remain auditable.[^1]
15. `mark_change_audit_logs` — immutable log of mark edits, because every mark change must be logged.[^1]
16. `grade_recalculation_audit_logs` — recalculation log, because every grade recalculation must be logged.[^1]
17. `report_card_revision_logs` — revision audit trail, because every report card revision must be logged and previous versions must remain available to administrators.[^1]

## 2. Table definitions

This schema keeps all tenant-owned tables on `tenant_id`, uses explicit foreign keys, stores published snapshots, and avoids polymorphic modeling by separating marks audit, recalculation audit, and report-card revision audit into dedicated tables.[^1]
It also preserves parent and student access only through published records, which matches the PRD’s visibility rules for own-child and self-only result access.[^2][^1]

```sql
CREATE EXTENSION IF NOT EXISTS pgcrypto;
CREATE EXTENSION IF NOT EXISTS citext;

CREATE TYPE grading_type AS ENUM ('percentage', 'letter_grade', 'gpa');
CREATE TYPE assessment_type AS ENUM ('exam', 'test', 'quiz', 'project', 'practical', 'assignment', 'oral', 'other');
CREATE TYPE assessment_status AS ENUM ('draft', 'active', 'marks_entered', 'verified', 'published', 'revised', 'archived');
CREATE TYPE component_type AS ENUM ('theory', 'practical', 'project', 'assignment', 'viva', 'internal', 'other');
CREATE TYPE roster_status AS ENUM ('active', 'late_added', 'withdrawn');
CREATE TYPE marks_entry_status AS ENUM ('draft', 'entered', 'submitted', 'verified');
CREATE TYPE mark_state AS ENUM ('present', 'absent', 'exempt', 'malpractice');
CREATE TYPE verification_status AS ENUM ('pending', 'ready', 'verified', 'rejected');
CREATE TYPE template_status AS ENUM ('draft', 'approved', 'archived');
CREATE TYPE report_card_status AS ENUM ('generated', 'verified', 'published', 'revised', 'withdrawn');
CREATE TYPE publication_status AS ENUM ('scheduled', 'published', 'revised', 'withdrawn', 'cancelled');
CREATE TYPE import_batch_status AS ENUM ('uploaded', 'validated', 'imported', 'failed');

-- Assumes upstream master tables already exist:
-- tenants, users, academic_years, grade_levels, sections, subjects,
-- students, student_enrollments, teacher_assignments.

CREATE TABLE grading_schemes (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id),
    scheme_name VARCHAR(120) NOT NULL,
    grading_type grading_type NOT NULL,
    scale_max NUMERIC(6,2) NOT NULL DEFAULT 100,
    is_active BOOLEAN NOT NULL DEFAULT TRUE,
    effective_from DATE NOT NULL,
    effective_to DATE,
    deleted_at TIMESTAMPTZ,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID REFERENCES users(id),
    updated_by UUID REFERENCES users(id),
    CONSTRAINT ck_grading_scheme_dates CHECK (effective_to IS NULL OR effective_to >= effective_from),
    CONSTRAINT uq_grading_scheme UNIQUE (tenant_id, scheme_name, effective_from)
);

CREATE TABLE grading_scheme_ranges (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id),
    grading_scheme_id UUID NOT NULL REFERENCES grading_schemes(id),
    seq_no INTEGER NOT NULL,
    min_score NUMERIC(6,2) NOT NULL,
    max_score NUMERIC(6,2) NOT NULL,
    grade_code VARCHAR(20) NOT NULL,
    grade_label VARCHAR(50),
    grade_point NUMERIC(6,2),
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID REFERENCES users(id),
    updated_by UUID REFERENCES users(id),
    CONSTRAINT ck_grade_range_bounds CHECK (max_score >= min_score),
    CONSTRAINT uq_grading_scheme_range_seq UNIQUE (tenant_id, grading_scheme_id, seq_no),
    CONSTRAINT uq_grading_scheme_grade_code UNIQUE (tenant_id, grading_scheme_id, grade_code)
);

CREATE TABLE assessments (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id),
    academic_year_id UUID NOT NULL REFERENCES academic_years(id),
    grade_level_id UUID NOT NULL REFERENCES grade_levels(id),
    section_id UUID NOT NULL REFERENCES sections(id),
    subject_id UUID NOT NULL REFERENCES subjects(id),
    teacher_assignment_id UUID REFERENCES teacher_assignments(id),
    grading_scheme_id UUID NOT NULL REFERENCES grading_schemes(id),
    assessment_name VARCHAR(150) NOT NULL,
    assessment_type assessment_type NOT NULL,
    total_marks NUMERIC(8,2) NOT NULL,
    marks_entry_deadline_at TIMESTAMPTZ,
    scheduled_publish_at TIMESTAMPTZ,
    status assessment_status NOT NULL DEFAULT 'draft',
    instructions TEXT,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID REFERENCES users(id),
    updated_by UUID REFERENCES users(id),
    CONSTRAINT ck_assessment_total_marks CHECK (total_marks > 0),
    CONSTRAINT uq_assessment_context UNIQUE (
        tenant_id, academic_year_id, grade_level_id, section_id, subject_id, assessment_name
    )
);

CREATE TABLE assessment_components (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id),
    assessment_id UUID NOT NULL REFERENCES assessments(id) ON DELETE CASCADE,
    component_name VARCHAR(100) NOT NULL,
    component_type component_type NOT NULL,
    max_marks NUMERIC(8,2) NOT NULL,
    weight_percentage NUMERIC(5,2) NOT NULL,
    display_order INTEGER NOT NULL DEFAULT 1,
    is_mandatory BOOLEAN NOT NULL DEFAULT TRUE,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID REFERENCES users(id),
    updated_by UUID REFERENCES users(id),
    CONSTRAINT ck_component_max_marks CHECK (max_marks > 0),
    CONSTRAINT ck_component_weight CHECK (weight_percentage > 0 AND weight_percentage <= 100),
    CONSTRAINT uq_component_name UNIQUE (tenant_id, assessment_id, component_name)
);

CREATE TABLE assessment_student_rosters (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id),
    assessment_id UUID NOT NULL REFERENCES assessments(id) ON DELETE CASCADE,
    student_id UUID NOT NULL REFERENCES students(id),
    student_enrollment_id UUID NOT NULL REFERENCES student_enrollments(id),
    roster_status roster_status NOT NULL DEFAULT 'active',
    roll_no_snapshot VARCHAR(50),
    section_id_snapshot UUID REFERENCES sections(id),
    added_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID REFERENCES users(id),
    updated_by UUID REFERENCES users(id),
    CONSTRAINT uq_assessment_student UNIQUE (tenant_id, assessment_id, student_id)
);

CREATE TABLE marks_entries (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id),
    assessment_id UUID NOT NULL REFERENCES assessments(id) ON DELETE CASCADE,
    assessment_component_id UUID NOT NULL REFERENCES assessment_components(id) ON DELETE CASCADE,
    assessment_roster_id UUID NOT NULL REFERENCES assessment_student_rosters(id) ON DELETE CASCADE,
    student_id UUID NOT NULL REFERENCES students(id),
    mark_state mark_state NOT NULL DEFAULT 'present',
    raw_marks NUMERIC(8,2),
    weighted_marks NUMERIC(8,2),
    final_grade VARCHAR(20),
    teacher_comment TEXT,
    status marks_entry_status NOT NULL DEFAULT 'draft',
    entered_by UUID REFERENCES users(id),
    entered_at TIMESTAMPTZ,
    submitted_by UUID REFERENCES users(id),
    submitted_at TIMESTAMPTZ,
    verified_by UUID REFERENCES users(id),
    verified_at TIMESTAMPTZ,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID REFERENCES users(id),
    updated_by UUID REFERENCES users(id),
    CONSTRAINT uq_marks_entry UNIQUE (tenant_id, assessment_component_id, assessment_roster_id),
    CONSTRAINT ck_marks_raw_nonnegative CHECK (raw_marks IS NULL OR raw_marks >= 0),
    CONSTRAINT ck_marks_weighted_nonnegative CHECK (weighted_marks IS NULL OR weighted_marks >= 0),
    CONSTRAINT ck_marks_presence_logic CHECK (
        (mark_state = 'present' AND raw_marks IS NOT NULL)
        OR (mark_state IN ('absent', 'exempt', 'malpractice'))
    )
);

CREATE TABLE marks_entry_import_batches (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id),
    assessment_id UUID NOT NULL REFERENCES assessments(id) ON DELETE CASCADE,
    file_name VARCHAR(255) NOT NULL,
    file_sha256 CHAR(64),
    status import_batch_status NOT NULL DEFAULT 'uploaded',
    total_rows INTEGER NOT NULL DEFAULT 0,
    valid_rows INTEGER NOT NULL DEFAULT 0,
    invalid_rows INTEGER NOT NULL DEFAULT 0,
    uploaded_by UUID NOT NULL REFERENCES users(id),
    validated_at TIMESTAMPTZ,
    imported_at TIMESTAMPTZ,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID REFERENCES users(id),
    updated_by UUID REFERENCES users(id)
);

CREATE TABLE marks_entry_import_errors (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id),
    import_batch_id UUID NOT NULL REFERENCES marks_entry_import_batches(id) ON DELETE CASCADE,
    row_number INTEGER NOT NULL,
    field_name VARCHAR(100),
    error_code VARCHAR(50) NOT NULL,
    error_message TEXT NOT NULL,
    raw_payload JSONB,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID REFERENCES users(id)
);

CREATE TABLE assessment_verification_runs (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id),
    assessment_id UUID NOT NULL REFERENCES assessments(id) ON DELETE CASCADE,
    verification_status verification_status NOT NULL DEFAULT 'pending',
    missing_entries_count INTEGER NOT NULL DEFAULT 0,
    requested_by UUID NOT NULL REFERENCES users(id),
    verified_by UUID REFERENCES users(id),
    rejected_by UUID REFERENCES users(id),
    verification_notes TEXT,
    requested_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    verified_at TIMESTAMPTZ,
    rejected_at TIMESTAMPTZ,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID REFERENCES users(id),
    updated_by UUID REFERENCES users(id)
);

CREATE TABLE report_card_templates (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id),
    template_name VARCHAR(120) NOT NULL,
    status template_status NOT NULL DEFAULT 'draft',
    current_version_no INTEGER NOT NULL DEFAULT 1,
    deleted_at TIMESTAMPTZ,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID REFERENCES users(id),
    updated_by UUID REFERENCES users(id),
    CONSTRAINT uq_template_name UNIQUE (tenant_id, template_name)
);

CREATE TABLE report_card_template_versions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id),
    template_id UUID NOT NULL REFERENCES report_card_templates(id) ON DELETE CASCADE,
    version_no INTEGER NOT NULL,
    template_schema JSONB NOT NULL,
    schema_checksum CHAR(64),
    is_approved BOOLEAN NOT NULL DEFAULT FALSE,
    approved_by UUID REFERENCES users(id),
    approved_at TIMESTAMPTZ,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID REFERENCES users(id),
    updated_by UUID REFERENCES users(id),
    CONSTRAINT uq_template_version UNIQUE (tenant_id, template_id, version_no)
);

CREATE TABLE result_publications (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id),
    academic_year_id UUID NOT NULL REFERENCES academic_years(id),
    grade_level_id UUID REFERENCES grade_levels(id),
    section_id UUID REFERENCES sections(id),
    publication_name VARCHAR(150) NOT NULL,
    publication_date TIMESTAMPTZ NOT NULL,
    status publication_status NOT NULL DEFAULT 'scheduled',
    scheduled_at TIMESTAMPTZ,
    published_at TIMESTAMPTZ,
    withdrawn_at TIMESTAMPTZ,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID REFERENCES users(id),
    updated_by UUID REFERENCES users(id)
);

CREATE TABLE report_cards (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id),
    result_publication_id UUID REFERENCES result_publications(id),
    student_id UUID NOT NULL REFERENCES students(id),
    academic_year_id UUID NOT NULL REFERENCES academic_years(id),
    grade_level_id UUID NOT NULL REFERENCES grade_levels(id),
    section_id UUID NOT NULL REFERENCES sections(id),
    template_id UUID NOT NULL REFERENCES report_card_templates(id),
    template_version_id UUID NOT NULL REFERENCES report_card_template_versions(id),
    version_number INTEGER NOT NULL DEFAULT 1,
    supersedes_report_card_id UUID REFERENCES report_cards(id),
    overall_grade VARCHAR(20),
    overall_grade_point NUMERIC(6,2),
    total_raw_marks NUMERIC(10,2),
    total_weighted_marks NUMERIC(10,2),
    percentage_score NUMERIC(6,2),
    teacher_remarks TEXT,
    principal_remarks TEXT,
    result_snapshot JSONB NOT NULL,
    pdf_storage_key TEXT,
    status report_card_status NOT NULL DEFAULT 'generated',
    generated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    verified_by UUID REFERENCES users(id),
    verified_at TIMESTAMPTZ,
    published_at TIMESTAMPTZ,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID REFERENCES users(id),
    updated_by UUID REFERENCES users(id),
    CONSTRAINT uq_report_card_version UNIQUE (tenant_id, result_publication_id, student_id, version_number),
    CONSTRAINT ck_report_card_version CHECK (version_number > 0)
);

CREATE TABLE report_card_subject_results (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id),
    report_card_id UUID NOT NULL REFERENCES report_cards(id) ON DELETE CASCADE,
    subject_id UUID NOT NULL REFERENCES subjects(id),
    total_raw_marks NUMERIC(10,2),
    total_weighted_marks NUMERIC(10,2),
    final_grade VARCHAR(20),
    grade_point NUMERIC(6,2),
    subject_result_snapshot JSONB NOT NULL,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID REFERENCES users(id),
    updated_by UUID REFERENCES users(id),
    CONSTRAINT uq_report_card_subject UNIQUE (tenant_id, report_card_id, subject_id)
);

CREATE TABLE report_card_component_results (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id),
    report_card_subject_result_id UUID NOT NULL REFERENCES report_card_subject_results(id) ON DELETE CASCADE,
    assessment_id UUID NOT NULL REFERENCES assessments(id),
    assessment_component_id UUID NOT NULL REFERENCES assessment_components(id),
    component_name VARCHAR(100) NOT NULL,
    raw_marks NUMERIC(8,2),
    weighted_marks NUMERIC(8,2),
    final_grade VARCHAR(20),
    mark_state mark_state NOT NULL,
    component_result_snapshot JSONB NOT NULL,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID REFERENCES users(id)
);

CREATE TABLE result_publication_items (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id),
    result_publication_id UUID NOT NULL REFERENCES result_publications(id) ON DELETE CASCADE,
    report_card_id UUID NOT NULL REFERENCES report_cards(id) ON DELETE CASCADE,
    student_id UUID NOT NULL REFERENCES students(id),
    is_current_version BOOLEAN NOT NULL DEFAULT TRUE,
    visible_from TIMESTAMPTZ,
    withdrawn_at TIMESTAMPTZ,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID REFERENCES users(id),
    updated_by UUID REFERENCES users(id),
    CONSTRAINT uq_publication_report_card UNIQUE (tenant_id, result_publication_id, report_card_id)
);

CREATE TABLE mark_change_audit_logs (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id),
    marks_entry_id UUID NOT NULL REFERENCES marks_entries(id) ON DELETE RESTRICT,
    old_value JSONB NOT NULL,
    new_value JSONB NOT NULL,
    change_reason TEXT,
    changed_by UUID NOT NULL REFERENCES users(id),
    changed_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    request_id UUID,
    ip_address INET,
    user_agent TEXT
);

CREATE TABLE grade_recalculation_audit_logs (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id),
    assessment_id UUID REFERENCES assessments(id),
    report_card_id UUID REFERENCES report_cards(id),
    old_value JSONB,
    new_value JSONB,
    recalculation_reason TEXT NOT NULL,
    changed_by UUID NOT NULL REFERENCES users(id),
    changed_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    request_id UUID
);

CREATE TABLE report_card_revision_logs (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id),
    old_report_card_id UUID NOT NULL REFERENCES report_cards(id),
    new_report_card_id UUID NOT NULL REFERENCES report_cards(id),
    revision_reason TEXT NOT NULL,
    revised_by UUID NOT NULL REFERENCES users(id),
    revised_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    request_id UUID
);
```


## 3. Constraints and indexes

The core enforcement points for M7_7 are tenant isolation, assessment context integrity, weight totaling, marks bounds, verification gating, immutable publication, and versioned revisions.[^1]
The PRD also requires that only assigned teachers enter marks, missing mandatory marks block verification and publication, and published records never be silently overwritten.[^1]

### 3. Constraints

- Add a deferred trigger on `assessment_components` so `SUM(weight_percentage) = 100` per `assessment_id`, because total component weight must equal 100.[^1]
- Add a trigger on `marks_entries` to block `raw_marks > assessment_components.max_marks`, because marks must not exceed configured maximum marks.[^1]
- Add a trigger on `marks_entries` to validate `entered_by` belongs to the upstream teacher assignment for that assessment, because only assigned teachers may enter marks.[^1]
- Add a trigger on `assessment_verification_runs` and `result_publications` to block progression when mandatory component entries are missing, because verification and publication require complete mandatory marks.[^1]
- Add an immutability trigger on `report_cards` when `status = 'published'`, allowing only revision via a new row linked by `supersedes_report_card_id`, because published report cards must be immutable and corrections create new versions.[^1]
- Allow soft delete only on `grading_schemes` and `report_card_templates` through `deleted_at`, because those are setup records; do not soft delete `marks_entries`, `report_cards`, `result_publications`, or audit tables.[^1]


### 4. Indexes

```sql
CREATE INDEX idx_assessments_context
ON assessments (tenant_id, academic_year_id, grade_level_id, section_id, subject_id, status);

CREATE INDEX idx_assessments_teacher
ON assessments (tenant_id, teacher_assignment_id, status);

CREATE INDEX idx_components_assessment
ON assessment_components (tenant_id, assessment_id, display_order);

CREATE INDEX idx_rosters_assessment_student
ON assessment_student_rosters (tenant_id, assessment_id, student_id);

CREATE INDEX idx_marks_component_status
ON marks_entries (tenant_id, assessment_component_id, status, student_id);

CREATE INDEX idx_marks_student_assessment
ON marks_entries (tenant_id, student_id, assessment_id);

CREATE INDEX idx_verification_assessment
ON assessment_verification_runs (tenant_id, assessment_id, verification_status, requested_at DESC);

CREATE INDEX idx_report_cards_student_year
ON report_cards (tenant_id, student_id, academic_year_id, published_at DESC);

CREATE INDEX idx_report_cards_publication
ON report_cards (tenant_id, result_publication_id, status, published_at DESC);

CREATE INDEX idx_publications_scope
ON result_publications (tenant_id, academic_year_id, grade_level_id, section_id, status, publication_date DESC);

CREATE INDEX idx_publication_items_student
ON result_publication_items (tenant_id, student_id, is_current_version);

CREATE INDEX idx_mark_audit_entry_time
ON mark_change_audit_logs (tenant_id, marks_entry_id, changed_at DESC);

CREATE INDEX idx_recalc_audit_report_card
ON grade_recalculation_audit_logs (tenant_id, report_card_id, changed_at DESC);

CREATE INDEX idx_revision_logs_old_new
ON report_card_revision_logs (tenant_id, old_report_card_id, new_report_card_id, revised_at DESC);

CREATE UNIQUE INDEX uq_current_publication_item
ON result_publication_items (tenant_id, result_publication_id, student_id)
WHERE is_current_version = TRUE AND withdrawn_at IS NULL;
```


### RLS baseline

Enable PostgreSQL RLS on every tenant-owned table and bind access to `current_setting('app.tenant_id', true)::uuid`, because all academic records belong to exactly one tenant and results cannot cross tenants.[^1]
For portal reads, expose only `result_publication_items` joined to `report_cards` where publication is published and relationship scope is valid for own-child or self-only access.[^2][^1]

## 4. Sample records and query patterns

These examples cover the PRD’s main workflows: assessment creation, marks entry, verification readiness, report-card generation, and published result access for parents and students.[^1]

### 5. Sample records

```sql
INSERT INTO grading_schemes
(id, tenant_id, scheme_name, grading_type, scale_max, effective_from, created_by, updated_by)
VALUES
('10000000-0000-0000-0000-000000000001', 't-001', 'Default Percentage 2026', 'percentage', 100, '2026-04-01', 'u-admin', 'u-admin');

INSERT INTO grading_scheme_ranges
(id, tenant_id, grading_scheme_id, seq_no, min_score, max_score, grade_code, grade_label, grade_point, created_by, updated_by)
VALUES
('11000000-0000-0000-0000-000000000001', 't-001', '10000000-0000-0000-0000-000000000001', 1, 90, 100, 'A1', 'Excellent', 10, 'u-admin', 'u-admin'),
('11000000-0000-0000-0000-000000000002', 't-001', '10000000-0000-0000-0000-000000000001', 2, 80, 89.99, 'A2', 'Very Good', 9, 'u-admin', 'u-admin');

INSERT INTO assessments
(id, tenant_id, academic_year_id, grade_level_id, section_id, subject_id, teacher_assignment_id, grading_scheme_id,
 assessment_name, assessment_type, total_marks, status, created_by, updated_by)
VALUES
('12000000-0000-0000-0000-000000000001', 't-001', 'ay-2026', 'grade-8', 'sec-8a', 'sub-math', 'ta-math-8a',
 '10000000-0000-0000-0000-000000000001', 'Term 1 Mathematics', 'exam', 100, 'active', 'u-admin', 'u-admin');

INSERT INTO assessment_components
(id, tenant_id, assessment_id, component_name, component_type, max_marks, weight_percentage, display_order, created_by, updated_by)
VALUES
('13000000-0000-0000-0000-000000000001', 't-001', '12000000-0000-0000-0000-000000000001', 'Theory', 'theory', 70, 70, 1, 'u-admin', 'u-admin'),
('13000000-0000-0000-0000-000000000002', 't-001', '12000000-0000-0000-0000-000000000001', 'Practical', 'practical', 30, 30, 2, 'u-admin', 'u-admin');

INSERT INTO assessment_student_rosters
(id, tenant_id, assessment_id, student_id, student_enrollment_id, roster_status, created_by, updated_by)
VALUES
('14000000-0000-0000-0000-000000000001', 't-001', '12000000-0000-0000-0000-000000000001', 'stu-001', 'enr-001', 'active', 'u-admin', 'u-admin');

INSERT INTO marks_entries
(id, tenant_id, assessment_id, assessment_component_id, assessment_roster_id, student_id, mark_state, raw_marks, weighted_marks, final_grade,
 status, entered_by, entered_at, created_by, updated_by)
VALUES
('15000000-0000-0000-0000-000000000001', 't-001', '12000000-0000-0000-0000-000000000001', '13000000-0000-0000-0000-000000000001',
 '14000000-0000-0000-0000-000000000001', 'stu-001', 'present', 63, 63, 'A1', 'entered', 'u-teacher-1', now(), 'u-teacher-1', 'u-teacher-1');
```


### 6. Query patterns

1. Teacher marks-entry roster for one assessment.[^1]
```sql
SELECT r.student_id, s.admission_no, s.first_name, s.last_name,
       c.id AS component_id, c.component_name, m.raw_marks, m.weighted_marks, m.final_grade, m.status
FROM assessment_student_rosters r
JOIN students s ON s.id = r.student_id
JOIN assessment_components c ON c.assessment_id = r.assessment_id
LEFT JOIN marks_entries m
  ON m.assessment_roster_id = r.id
 AND m.assessment_component_id = c.id
 AND m.tenant_id = r.tenant_id
WHERE r.tenant_id = $1
  AND r.assessment_id = $2
ORDER BY s.last_name, s.first_name, c.display_order;
```

2. Missing mandatory marks before verification.[^1]
```sql
SELECT a.id AS assessment_id, a.assessment_name, COUNT(*) AS missing_count
FROM assessments a
JOIN assessment_student_rosters r
  ON r.assessment_id = a.id AND r.tenant_id = a.tenant_id
JOIN assessment_components c
  ON c.assessment_id = a.id AND c.tenant_id = a.tenant_id AND c.is_mandatory = TRUE
LEFT JOIN marks_entries m
  ON m.assessment_component_id = c.id
 AND m.assessment_roster_id = r.id
 AND m.tenant_id = a.tenant_id
WHERE a.tenant_id = $1
  AND a.id = $2
  AND m.id IS NULL
GROUP BY a.id, a.assessment_name;
```

3. Latest published report card for one student in one academic year.[^1]
```sql
SELECT rc.*
FROM report_cards rc
JOIN result_publications rp ON rp.id = rc.result_publication_id
WHERE rc.tenant_id = $1
  AND rc.student_id = $2
  AND rc.academic_year_id = $3
  AND rc.status = 'published'
  AND rp.status = 'published'
ORDER BY rc.version_number DESC, rc.published_at DESC
LIMIT 1;
```

4. Parent view across all linked children using published-only rule.[^2][^1]
```sql
SELECT ps.parent_user_id, rc.student_id, rc.id AS report_card_id, rc.overall_grade, rc.published_at
FROM parent_student_links ps
JOIN result_publication_items rpi ON rpi.student_id = ps.student_id
JOIN report_cards rc ON rc.id = rpi.report_card_id
JOIN result_publications rp ON rp.id = rpi.result_publication_id
WHERE ps.parent_user_id = $1
  AND rc.tenant_id = $2
  AND rp.status = 'published'
  AND rpi.is_current_version = TRUE
ORDER BY rc.published_at DESC;
```

5. Full mark-change audit for dispute handling.[^1]
```sql
SELECT mca.changed_at, mca.changed_by, mca.change_reason, mca.old_value, mca.new_value
FROM mark_change_audit_logs mca
WHERE mca.tenant_id = $1
  AND mca.marks_entry_id = $2
ORDER BY mca.changed_at DESC;
```


## 5. Migration sequence and security-sensitive columns

The implementation order should follow the PRD’s recommended build path: assessments first, then marks entry, grading schemes, verification workflow, report card generation, result publication, and finally parent/student visibility.[^1]
This order matters because the gradebook is explicitly downstream of student information, academic structure, and teacher assignments, and published records should be generated only after the earlier controls exist.[^1]

### 7. Migration sequence

1. Create enums and core setup tables: `grading_schemes`, `grading_scheme_ranges`, `report_card_templates`, `report_card_template_versions`.[^1]
2. Create academic transaction tables: `assessments`, `assessment_components`, `assessment_student_rosters`.[^1]
3. Create marks capture tables: `marks_entries`, `marks_entry_import_batches`, `marks_entry_import_errors`.[^1]
4. Add calculation and verification controls: grade-calculation functions, `assessment_verification_runs`, validation triggers, and teacher-scope enforcement.[^1]
5. Create publication tables: `result_publications`, `report_cards`, `report_card_subject_results`, `report_card_component_results`, `result_publication_items`.[^1]
6. Create audit tables and immutability triggers: `mark_change_audit_logs`, `grade_recalculation_audit_logs`, `report_card_revision_logs`.[^1]
7. Add RLS policies, indexes, approved template seeds, and grading-scheme seeds; then expose read-only portal queries for parent own-child and student self access.[^2][^1]

### 8. Security-sensitive columns

- `report_cards.result_snapshot`, `report_card_subject_results.subject_result_snapshot`, and `report_card_component_results.component_result_snapshot` are sensitive because they contain official academic history and published snapshots.[^1]
- `report_cards.teacher_remarks` and `report_cards.principal_remarks` are sensitive because they may contain academic commentary and student-related personal information.[^1]
- `marks_entries.teacher_comment` is sensitive because it can expose teacher-only working notes before publication.[^1]
- `mark_change_audit_logs.old_value`, `mark_change_audit_logs.new_value`, `grade_recalculation_audit_logs.old_value`, and `grade_recalculation_audit_logs.new_value` are sensitive because they reveal mark history, corrections, and internal academic operations.[^1]
- `marks_entry_import_errors.raw_payload` is sensitive because uploaded sheets can contain student identifiers and raw academic data.[^1]
- `report_cards.pdf_storage_key` should be protected because report cards are downloadable official records and must not be exposed through guessable storage paths.[^1]

If you want, I can continue next with M7_9 in the same production-ready format.
<span style="display:none">[^3][^4][^5][^6][^7][^8]</span>

<div align="center">⁂</div>

[^1]: PRD_M7_7.txt

[^2]: PRD_M7_9.txt

[^3]: PRD_M7_16.txt

[^4]: PRD_M7_14_15.txt

[^5]: PRD_M7_12.txt

[^6]: PRD_M7_8.txt

[^7]: PRD_M7_13.txt

[^8]: PRD_M7_10.txt

