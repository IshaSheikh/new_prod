<img src="https://r2cdn.perplexity.ai/pplx-full-logo-primary-dark%402x.png" style="height:64px;margin-right:32px"/>

# please go forward with the next module M7_4

For M7_4, the schema should make the SIS the **authoritative source of truth** for student identity, enrollment, guardians, documents, health alerts, and status history, because the PRD says all downstream modules should consume student data from SIS rather than maintain their own copies.[^1]
I am also designing this module so `student_id` is the master reference for attendance, gradebook, exams, timetable, fees, parent portal, transport, and communication, because the PRD explicitly recommends that future modules reference SIS `student_id` instead of duplicating student records.[^1]

## Entity list

These are the production-ready entities for Module 7.4, aligned to admissions, enrollment lifecycle, guardian relationships, configurable profile fields, versioned documents, and protected health information.[^1]

- `sis_settings` — tenant-level SIS controls such as dual-enrollment policy and duplicate-detection behavior, because the PRD says dual enrollment is disallowed by default and schools need configurable validation behavior.[^1]
- `students` — tenant-owned student master with immutable student number, admission number, lifecycle status, optional linked user account, and justified soft delete.[^1]
- `student_profiles` — demographic and address data for the student, separated from enrollment history so SIS remains the single source of truth for profile data.[^1]
- `student_custom_field_definitions` — configurable mandatory and optional profile fields per school, because the PRD says mandatory profile fields are configurable.[^1]
- `student_custom_field_options` — allowed values for select-type custom fields such as category or nationality lists.[^1]
- `student_custom_field_values` — actual student values for configured custom fields, with typed storage rather than loose polymorphic JSON.[^1]
- `student_enrollments` — academic-year, grade, and section assignment history, because enrollments must reference academic year, grade, and section and history must never be deleted.[^1]
- `guardians` — reusable guardian contact master, optionally linked to a platform `users` record when parent portal access exists.[^1]
- `guardian_student_links` — many-to-many guardian relationship table with one primary guardian per student and support for one guardian linked to multiple students.[^1]
- `student_emergency_contacts` — emergency contact records for fast access by teachers and admins during incidents.[^1]
- `student_document_categories` — configurable document categories such as Birth Certificate, Transfer Certificate, Passport, Aadhaar, and Medical Records.[^1]
- `student_documents` — logical student document header used to group versions of the same document.[^1]
- `student_document_versions` — immutable uploaded file versions stored by object key rather than in PostgreSQL, because the PRD recommends S3 storage and says uploaded documents cannot be modified after upload.[^1]
- `student_health_notes` — role-protected health alerts and confidential medical notes, including critical alerts visible to authorized staff.[^1]
- `student_status_history` — mandatory audit trail for every student status transition, because the PRD says status transitions must be auditable and never occur without history.[^1]


## Table definitions

The DDL below implements M7_4 with tenant isolation on every tenant-owned table, immutable history, configurable profile fields, versioned documents, and soft delete only on `students`, because the PRD explicitly allows soft delete there after enrollment and requires non-deletable enrollment history and document versioning.[^1]

```sql
CREATE EXTENSION IF NOT EXISTS pgcrypto;
CREATE EXTENSION IF NOT EXISTS citext;

CREATE TYPE student_status AS ENUM (
    'inquiry',
    'applicant',
    'admitted',
    'enrolled',
    'active',
    'transferred',
    'withdrawn',
    'graduated',
    'deceased'
);

CREATE TYPE enrollment_status AS ENUM (
    'draft',
    'pending_approval',
    'active',
    'completed',
    'transferred',
    'withdrawn'
);

CREATE TYPE gender_type AS ENUM (
    'male',
    'female',
    'non_binary',
    'other',
    'undisclosed'
);

CREATE TYPE guardian_relationship_type AS ENUM (
    'father',
    'mother',
    'guardian',
    'legal_guardian',
    'grandparent',
    'other'
);

CREATE TYPE document_version_status AS ENUM (
    'uploaded',
    'verified',
    'replaced',
    'archived'
);

CREATE TYPE health_severity AS ENUM (
    'low',
    'medium',
    'high',
    'critical'
);

CREATE TYPE custom_field_type AS ENUM (
    'text',
    'number',
    'date',
    'boolean',
    'single_select'
);

CREATE TABLE sis_settings (
    tenant_id UUID PRIMARY KEY REFERENCES tenants(id) ON DELETE CASCADE,
    allow_dual_enrollment BOOLEAN NOT NULL DEFAULT FALSE,
    duplicate_match_on_name_dob BOOLEAN NOT NULL DEFAULT TRUE,
    duplicate_match_on_mobile BOOLEAN NOT NULL DEFAULT TRUE,
    duplicate_match_on_government_id BOOLEAN NOT NULL DEFAULT TRUE,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID NULL REFERENCES users(id),
    updated_by UUID NULL REFERENCES users(id)
);

CREATE TABLE students (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
    school_id UUID NOT NULL REFERENCES schools(id) ON DELETE RESTRICT,
    user_id UUID NULL REFERENCES users(id) ON DELETE SET NULL,
    student_number VARCHAR(50) NOT NULL,
    admission_number VARCHAR(50),
    status student_status NOT NULL DEFAULT 'inquiry',
    admission_date DATE,
    current_campus_id UUID NULL REFERENCES campuses(id) ON DELETE RESTRICT,
    deleted_at TIMESTAMPTZ,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID NULL REFERENCES users(id),
    updated_by UUID NULL REFERENCES users(id),
    CONSTRAINT uq_student_number UNIQUE (tenant_id, student_number)
);

CREATE TABLE student_profiles (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
    student_id UUID NOT NULL UNIQUE REFERENCES students(id) ON DELETE RESTRICT,
    first_name VARCHAR(100) NOT NULL,
    middle_name VARCHAR(100),
    last_name VARCHAR(100),
    preferred_name VARCHAR(100),
    gender gender_type,
    date_of_birth DATE NOT NULL,
    nationality VARCHAR(100),
    blood_group VARCHAR(10),
    photo_storage_key TEXT,
    address_line_1 VARCHAR(255),
    address_line_2 VARCHAR(255),
    city VARCHAR(100),
    state VARCHAR(100),
    postal_code VARCHAR(20),
    country_code CHAR(2),
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID NULL REFERENCES users(id),
    updated_by UUID NULL REFERENCES users(id)
);

CREATE TABLE student_custom_field_definitions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
    field_code VARCHAR(100) NOT NULL,
    field_label VARCHAR(150) NOT NULL,
    field_type custom_field_type NOT NULL,
    is_required BOOLEAN NOT NULL DEFAULT FALSE,
    sort_order INTEGER NOT NULL,
    is_active BOOLEAN NOT NULL DEFAULT TRUE,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID NULL REFERENCES users(id),
    updated_by UUID NULL REFERENCES users(id),
    CONSTRAINT uq_student_custom_field_code UNIQUE (tenant_id, field_code),
    CONSTRAINT ck_student_custom_field_sort CHECK (sort_order > 0)
);

CREATE TABLE student_custom_field_options (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
    field_definition_id UUID NOT NULL REFERENCES student_custom_field_definitions(id) ON DELETE CASCADE,
    option_value VARCHAR(100) NOT NULL,
    option_label VARCHAR(150) NOT NULL,
    sort_order INTEGER NOT NULL DEFAULT 1,
    is_active BOOLEAN NOT NULL DEFAULT TRUE,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID NULL REFERENCES users(id),
    updated_by UUID NULL REFERENCES users(id),
    CONSTRAINT uq_student_custom_field_option UNIQUE (field_definition_id, option_value)
);

CREATE TABLE student_custom_field_values (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
    student_id UUID NOT NULL REFERENCES students(id) ON DELETE RESTRICT,
    field_definition_id UUID NOT NULL REFERENCES student_custom_field_definitions(id) ON DELETE RESTRICT,
    text_value TEXT,
    number_value NUMERIC(18,4),
    date_value DATE,
    boolean_value BOOLEAN,
    option_id UUID NULL REFERENCES student_custom_field_options(id) ON DELETE RESTRICT,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID NULL REFERENCES users(id),
    updated_by UUID NULL REFERENCES users(id),
    CONSTRAINT uq_student_custom_field_value UNIQUE (tenant_id, student_id, field_definition_id)
);

CREATE TABLE student_enrollments (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
    student_id UUID NOT NULL REFERENCES students(id) ON DELETE RESTRICT,
    academic_year_id UUID NOT NULL REFERENCES academic_years(id) ON DELETE RESTRICT,
    grade_level_id UUID NOT NULL REFERENCES grade_levels(id) ON DELETE RESTRICT,
    section_id UUID NOT NULL REFERENCES sections(id) ON DELETE RESTRICT,
    campus_id UUID NULL REFERENCES campuses(id) ON DELETE RESTRICT,
    enrollment_date DATE NOT NULL,
    exit_date DATE,
    is_primary BOOLEAN NOT NULL DEFAULT TRUE,
    status enrollment_status NOT NULL DEFAULT 'draft',
    transfer_from_enrollment_id UUID NULL REFERENCES student_enrollments(id) ON DELETE RESTRICT,
    withdrawal_reason TEXT,
    transfer_destination_school VARCHAR(255),
    transfer_destination_city VARCHAR(100),
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID NULL REFERENCES users(id),
    updated_by UUID NULL REFERENCES users(id),
    CONSTRAINT ck_student_enrollment_dates CHECK (
        exit_date IS NULL OR exit_date >= enrollment_date
    )
);

CREATE TABLE guardians (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
    user_id UUID NULL REFERENCES users(id) ON DELETE SET NULL,
    first_name VARCHAR(100) NOT NULL,
    last_name VARCHAR(100),
    mobile_e164 VARCHAR(20),
    email CITEXT,
    occupation VARCHAR(150),
    address_line_1 VARCHAR(255),
    address_line_2 VARCHAR(255),
    city VARCHAR(100),
    state VARCHAR(100),
    postal_code VARCHAR(20),
    country_code CHAR(2),
    is_active BOOLEAN NOT NULL DEFAULT TRUE,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID NULL REFERENCES users(id),
    updated_by UUID NULL REFERENCES users(id)
);

CREATE TABLE guardian_student_links (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
    guardian_id UUID NOT NULL REFERENCES guardians(id) ON DELETE RESTRICT,
    student_id UUID NOT NULL REFERENCES students(id) ON DELETE RESTRICT,
    relationship_type guardian_relationship_type NOT NULL,
    is_primary BOOLEAN NOT NULL DEFAULT FALSE,
    has_portal_access BOOLEAN NOT NULL DEFAULT FALSE,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID NULL REFERENCES users(id),
    updated_by UUID NULL REFERENCES users(id),
    CONSTRAINT uq_guardian_student UNIQUE (tenant_id, guardian_id, student_id)
);

CREATE TABLE student_emergency_contacts (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
    student_id UUID NOT NULL REFERENCES students(id) ON DELETE RESTRICT,
    guardian_id UUID NULL REFERENCES guardians(id) ON DELETE SET NULL,
    contact_name VARCHAR(200) NOT NULL,
    relationship_label VARCHAR(100) NOT NULL,
    mobile_e164 VARCHAR(20) NOT NULL,
    alternate_phone VARCHAR(20),
    is_primary BOOLEAN NOT NULL DEFAULT FALSE,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID NULL REFERENCES users(id),
    updated_by UUID NULL REFERENCES users(id)
);

CREATE TABLE student_document_categories (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
    code VARCHAR(100) NOT NULL,
    name VARCHAR(150) NOT NULL,
    requires_verification BOOLEAN NOT NULL DEFAULT FALSE,
    is_active BOOLEAN NOT NULL DEFAULT TRUE,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID NULL REFERENCES users(id),
    updated_by UUID NULL REFERENCES users(id),
    CONSTRAINT uq_student_document_category UNIQUE (tenant_id, code)
);

CREATE TABLE student_documents (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
    student_id UUID NOT NULL REFERENCES students(id) ON DELETE RESTRICT,
    category_id UUID NOT NULL REFERENCES student_document_categories(id) ON DELETE RESTRICT,
    document_label VARCHAR(150) NOT NULL,
    current_version_no INTEGER NOT NULL DEFAULT 1,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID NULL REFERENCES users(id),
    updated_by UUID NULL REFERENCES users(id)
);

CREATE TABLE student_document_versions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
    student_document_id UUID NOT NULL REFERENCES student_documents(id) ON DELETE RESTRICT,
    version_no INTEGER NOT NULL,
    storage_key TEXT NOT NULL,
    original_file_name VARCHAR(255) NOT NULL,
    mime_type VARCHAR(100) NOT NULL,
    file_size_bytes BIGINT NOT NULL,
    checksum_sha256 CHAR(64),
    status document_version_status NOT NULL DEFAULT 'uploaded',
    verified_at TIMESTAMPTZ,
    replaced_at TIMESTAMPTZ,
    archived_at TIMESTAMPTZ,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID NULL REFERENCES users(id),
    CONSTRAINT uq_student_document_version UNIQUE (student_document_id, version_no),
    CONSTRAINT ck_student_document_file_size CHECK (file_size_bytes > 0)
);

CREATE TABLE student_health_notes (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
    student_id UUID NOT NULL REFERENCES students(id) ON DELETE RESTRICT,
    severity health_severity NOT NULL,
    medical_condition_label VARCHAR(150) NOT NULL,
    note_ciphertext BYTEA NOT NULL,
    effective_date DATE NOT NULL,
    expires_at DATE,
    is_active BOOLEAN NOT NULL DEFAULT TRUE,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID NULL REFERENCES users(id),
    updated_by UUID NULL REFERENCES users(id),
    CONSTRAINT ck_student_health_dates CHECK (
        expires_at IS NULL OR expires_at >= effective_date
    )
);

CREATE TABLE student_status_history (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
    student_id UUID NOT NULL REFERENCES students(id) ON DELETE RESTRICT,
    old_status student_status,
    new_status student_status NOT NULL,
    reason TEXT,
    destination_school VARCHAR(255),
    destination_city VARCHAR(100),
    changed_by UUID NULL REFERENCES users(id),
    changed_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

This structure keeps the SIS authoritative for student identity and history while still linking cleanly to `users`, `academic_years`, `grade_levels`, `sections`, and `campuses` from earlier modules.[^1]

## Constraints and indexes

The most important M7_4 enforcement points are unique student numbers, soft delete instead of permanent deletion, one primary enrollment per academic year, no simultaneous dual active enrollments by default, one primary guardian, immutable document versions, and auditable status transitions with reason rules.[^1]

**Constraints**

- `students.student_number` must be unique within a tenant and never reused, because the PRD requires a unique student number per tenant and prohibits reuse.[^1]
- `students.deleted_at` is the justified soft-delete mechanism, because student profiles cannot be permanently deleted after enrollment.[^1]
- `student_enrollments` should have a partial unique index on one primary active enrollment per student per academic year, because the PRD allows only one primary enrollment in an academic year.[^1]
- A trigger should block multiple simultaneous active section enrollments unless `sis_settings.allow_dual_enrollment = TRUE`, because dual enrollment is disallowed by default.[^1]
- A trigger should validate that `section_id` belongs to the provided `academic_year_id` and `grade_level_id`, because enrollments must reference academic year, grade, and section consistently.[^1]
- A deferred trigger should enforce exactly one primary guardian for active students, because the PRD requires one guardian to be marked as primary while still allowing multiple guardians.[^1]
- `student_document_versions` should be insert-only with `UPDATE` and `DELETE` blocked by trigger, because uploaded documents cannot be modified and replacement must create a new version.[^1]
- A trigger on `students.status` should block direct `inquiry -> enrolled`, require withdrawal reason on `withdrawn`, require destination information on `transferred`, and always insert into `student_status_history`, because those are explicit business rules in the PRD.[^1]
- A trigger on `student_enrollments` should block enrollment when required custom fields are missing, because the PRD says configurable mandatory profile fields must be collected and missing mandatory data blocks enrollment.[^1]

```sql
CREATE UNIQUE INDEX uq_students_admission_number
ON students (tenant_id, admission_number)
WHERE admission_number IS NOT NULL;

CREATE UNIQUE INDEX uq_primary_enrollment_per_year
ON student_enrollments (tenant_id, student_id, academic_year_id)
WHERE is_primary = TRUE
  AND status IN ('pending_approval', 'active');

CREATE UNIQUE INDEX uq_primary_guardian_per_student
ON guardian_student_links (tenant_id, student_id)
WHERE is_primary = TRUE;

CREATE UNIQUE INDEX uq_guardian_user_per_tenant
ON guardians (tenant_id, user_id)
WHERE user_id IS NOT NULL;

CREATE UNIQUE INDEX uq_primary_emergency_contact_per_student
ON student_emergency_contacts (tenant_id, student_id)
WHERE is_primary = TRUE;
```

**Indexes**

- `students (tenant_id, status, student_number)` supports student list screens and fast student-number lookup.[^1]
- `student_profiles` should support name and DOB search, because the PRD calls out search-heavy student administration and duplicate checks.[^1]
- `student_enrollments (tenant_id, academic_year_id, grade_level_id, section_id, status)` supports current roster, transfer, and enrollment-history queries.[^1]
- `guardian_student_links` needs lookup indexes by `student_id` and `guardian_id`, because one guardian may be linked to multiple students and one student may have multiple guardians.[^1]
- `student_document_versions` needs indexes by student, category, and created time, because documents are versioned and viewed through history screens.[^1]
- `student_health_notes` needs a critical-alert lookup path, because critical health alerts must be visible to authorized staff.[^1]
- `student_status_history (tenant_id, student_id, changed_at desc)` supports lifecycle audit and retention analysis.[^1]

```sql
CREATE INDEX idx_students_tenant_status_number
ON students (tenant_id, status, student_number);

CREATE INDEX idx_student_profiles_tenant_name_dob
ON student_profiles (tenant_id, last_name, first_name, date_of_birth);

CREATE INDEX idx_student_enrollments_current
ON student_enrollments (tenant_id, academic_year_id, grade_level_id, section_id, status);

CREATE INDEX idx_student_enrollments_student_history
ON student_enrollments (tenant_id, student_id, enrollment_date DESC);

CREATE INDEX idx_guardian_links_student
ON guardian_student_links (tenant_id, student_id, is_primary);

CREATE INDEX idx_guardian_links_guardian
ON guardian_student_links (tenant_id, guardian_id);

CREATE INDEX idx_student_documents_student_category
ON student_documents (tenant_id, student_id, category_id);

CREATE INDEX idx_student_document_versions_doc_created
ON student_document_versions (tenant_id, student_document_id, created_at DESC);

CREATE INDEX idx_student_health_notes_critical
ON student_health_notes (tenant_id, student_id, severity, is_active);

CREATE INDEX idx_student_status_history_student_changed
ON student_status_history (tenant_id, student_id, changed_at DESC);
```


## Sample records and query patterns

These samples reflect the normal SIS flow: create the student, capture profile, enroll in class-section for an academic year, link guardians, and upload versioned documents.[^1]

```sql
INSERT INTO students (
    id, tenant_id, school_id, student_number, admission_number, status, admission_date, created_by
) VALUES (
    '71000000-0000-0000-0000-000000000001',
    '10000000-0000-0000-0000-000000000001',
    '51000000-0000-0000-0000-000000000001',
    'GFS-2026-0001',
    'ADM-2026-0142',
    'admitted',
    '2026-06-10',
    '00000000-0000-0000-0000-000000000002'
);

INSERT INTO student_profiles (
    id, tenant_id, student_id, first_name, last_name, gender, date_of_birth, nationality, city, state, country_code, created_by
) VALUES (
    '72000000-0000-0000-0000-000000000001',
    '10000000-0000-0000-0000-000000000001',
    '71000000-0000-0000-0000-000000000001',
    'Arjun',
    'Menon',
    'male',
    '2019-08-15',
    'Indian',
    'Chennai',
    'Tamil Nadu',
    'IN',
    '00000000-0000-0000-0000-000000000002'
);

INSERT INTO student_enrollments (
    id, tenant_id, student_id, academic_year_id, grade_level_id, section_id, campus_id,
    enrollment_date, is_primary, status, created_by
) VALUES (
    '73000000-0000-0000-0000-000000000001',
    '10000000-0000-0000-0000-000000000001',
    '71000000-0000-0000-0000-000000000001',
    '53000000-0000-0000-0000-000000000001',
    '54000000-0000-0000-0000-000000000001',
    '55000000-0000-0000-0000-000000000001',
    NULL,
    '2026-06-12',
    TRUE,
    'active',
    '00000000-0000-0000-0000-000000000002'
);

INSERT INTO guardians (
    id, tenant_id, first_name, last_name, mobile_e164, email, occupation, created_by
) VALUES (
    '74000000-0000-0000-0000-000000000001',
    '10000000-0000-0000-0000-000000000001',
    'Lakshmi',
    'Menon',
    '+919900111222',
    'lakshmi.menon@example.com',
    'Architect',
    '00000000-0000-0000-0000-000000000002'
);

INSERT INTO guardian_student_links (
    id, tenant_id, guardian_id, student_id, relationship_type, is_primary, has_portal_access, created_by
) VALUES (
    '75000000-0000-0000-0000-000000000001',
    '10000000-0000-0000-0000-000000000001',
    '74000000-0000-0000-0000-000000000001',
    '71000000-0000-0000-0000-000000000001',
    'mother',
    TRUE,
    TRUE,
    '00000000-0000-0000-0000-000000000002'
);
```

**Query patterns**

1. **Student list with current enrollment filters**[^1]
```sql
SELECT s.id, s.student_number, s.status,
       sp.first_name, sp.last_name,
       se.academic_year_id, se.grade_level_id, se.section_id, se.campus_id
FROM students s
JOIN student_profiles sp
  ON sp.student_id = s.id AND sp.tenant_id = s.tenant_id
LEFT JOIN student_enrollments se
  ON se.student_id = s.id
 AND se.tenant_id = s.tenant_id
 AND se.status = 'active'
WHERE s.tenant_id = $1
  AND s.deleted_at IS NULL
  AND ($2::uuid IS NULL OR se.grade_level_id = $2)
  AND ($3::uuid IS NULL OR se.section_id = $3)
  AND ($4::student_status IS NULL OR s.status = $4)
ORDER BY sp.first_name, sp.last_name;
```

2. **Student profile with primary guardian and critical alerts**[^1]
```sql
SELECT s.student_number, s.status,
       sp.first_name, sp.last_name, sp.date_of_birth,
       g.first_name AS guardian_first_name, g.last_name AS guardian_last_name, g.mobile_e164,
       shn.medical_condition_label, shn.severity
FROM students s
JOIN student_profiles sp
  ON sp.student_id = s.id AND sp.tenant_id = s.tenant_id
LEFT JOIN guardian_student_links gsl
  ON gsl.student_id = s.id AND gsl.tenant_id = s.tenant_id AND gsl.is_primary = TRUE
LEFT JOIN guardians g
  ON g.id = gsl.guardian_id
LEFT JOIN student_health_notes shn
  ON shn.student_id = s.id
 AND shn.tenant_id = s.tenant_id
 AND shn.is_active = TRUE
 AND shn.severity = 'critical'
WHERE s.tenant_id = $1
  AND s.id = $2;
```

3. **Enrollment history for transfer and retention analysis**[^1]
```sql
SELECT academic_year_id, grade_level_id, section_id, campus_id,
       enrollment_date, exit_date, status, withdrawal_reason, transfer_destination_school
FROM student_enrollments
WHERE tenant_id = $1
  AND student_id = $2
ORDER BY enrollment_date DESC;
```

4. **Document history with version trail**[^1]
```sql
SELECT dc.name AS category_name,
       sd.document_label,
       sdv.version_no,
       sdv.original_file_name,
       sdv.status,
       sdv.created_at,
       sdv.verified_at
FROM student_documents sd
JOIN student_document_categories dc
  ON dc.id = sd.category_id
JOIN student_document_versions sdv
  ON sdv.student_document_id = sd.id AND sdv.tenant_id = sd.tenant_id
WHERE sd.tenant_id = $1
  AND sd.student_id = $2
ORDER BY dc.name, sd.document_label, sdv.version_no DESC;
```

5. **Student lifecycle dashboard counts**[^1]
```sql
SELECT status, count(*) AS student_count
FROM students
WHERE tenant_id = $1
  AND deleted_at IS NULL
GROUP BY status
ORDER BY status;
```


## Migration sequence and security-sensitive columns

M7_4 should be migrated after M7_2 and M7_3, because it depends on tenant, school, academic-year, grade, section, campus, and optional linked user records, and the PRD says SIS is foundational for downstream modules.[^1]

**Migration sequence**

1. Create extensions and enums, then `sis_settings`, `students`, and `student_profiles`, because student identity is the master layer of the SIS.[^1]
2. Create `student_custom_field_definitions`, `student_custom_field_options`, and `student_custom_field_values`, because schools need configurable mandatory fields before enrollment.[^1]
3. Create `guardians`, `guardian_student_links`, and `student_emergency_contacts`, because guardian relationships and emergency contacts are core operational data.[^1]
4. Create `student_enrollments` and `student_status_history`, then add triggers for status-transition rules, enrollment integrity, and history preservation.[^1]
5. Create `student_document_categories`, `student_documents`, and `student_document_versions`, then add insert-only protection for version rows and S3 object-key conventions.[^1]
6. Create `student_health_notes`, then add RLS policies, field-level access controls, and all indexes.[^1]
7. Seed standard document categories and common custom-field definitions, because the PRD calls out configurable categories such as Birth Certificate, Transfer Certificate, Passport, Aadhaar, and Medical Records.[^1]

**Security-sensitive columns**

- `student_profiles.first_name`, `middle_name`, `last_name`, `date_of_birth`, address fields, nationality, and photo storage fields are direct student PII and should be protected by role-aware RLS.[^1]
- `students.student_number` and `admission_number` are institutional identifiers and should be visible only to roles that legitimately need student lookup or finance linkage.[^1]
- `student_custom_field_values` can hold highly sensitive fields such as religion, nationality, Aadhaar number, or student category, so those values should support masking and encryption where needed.[^1]
- `guardians.mobile_e164`, `guardians.email`, and guardian addresses are guardian PII and should never be exposed outside authorized school staff and linked parent views.[^1]
- `student_health_notes.note_ciphertext` and related health-alert fields are the most sensitive records in this module and should be encrypted, tightly role-protected, and fully audited.[^1]
- `student_document_versions.storage_key`, file metadata, and checksums should be restricted because the documents may contain identity and compliance records and the PRD explicitly recommends object storage rather than database file storage.[^1]

This M7_4 design keeps the SIS clean, auditable, and future-proof while preserving student lifecycle history and giving downstream modules one reliable student master to reference.[^1]

<div align="center">⁂</div>

[^1]: PRD_M7_4.txt

