<img src="https://r2cdn.perplexity.ai/pplx-full-logo-primary-dark%402x.png" style="height:64px;margin-right:32px"/>

# ohk great now, beaing the excellent databse architecture, please give me the architecture for module 7_2 in the same manner

For M7_2, the schema should focus on **school setup as controlled configuration data**, not transactional school operations, because the PRD defines this module as the onboarding foundation for academic structure, campuses, branding, communication defaults, readiness validation, and go-live control before attendance, grades, fees, and timetable can function.[^1]
I am also keeping one active school profile per tenant, one active academic year at a time, non-overlapping academic years and terms, unique grade names per tenant, and unique grade-section combinations per academic year, because those are explicit business rules in Module 7.2.[^1]

## Entity list

These are the core entities for Module 7.2, based directly on the PRD’s required data entities plus supporting production tables needed for readiness validation, branding governance, and safe go-live.[^1]

- `schools` — tenant-owned school master used to track onboarding lifecycle and go-live state.[^1]
- `school_profiles` — the single active school identity and contact record per tenant.[^1]
- `school_settings` — tenant-level functional configuration such as language, currency, date format, and module enablement flags.[^1]
- `academic_years` — school academic calendar headers with lifecycle state.[^1]
- `terms` — academic sub-periods such as semesters or terms under an academic year.[^1]
- `grade_levels` — tenant-wide grade structure definitions such as Grade 1, Grade 2, Grade 10.[^1]
- `sections` — academic-year-specific sections tied to grade levels, with capacity.[^1]
- `campuses` — one or many physical campuses, including primary campus designation.[^1]
- `section_campuses` — optional mapping when sections are assigned to campuses, which is implied by the section management screen.[^1]
- `branding_assets` — uploaded logos and branding files for school identity.[^1]
- `branding_themes` — normalized theme/color configuration so branding is centrally controlled instead of embedded loosely in files.[^1]
- `communication_settings` — onboarding-time mail, SMS, and notification configuration defaults referenced by the communication settings screen.[^1]
- `onboarding_checklists` — step-by-step onboarding progress tracking.[^1]
- `onboarding_validation_runs` — snapshot of each readiness validation execution.[^1]
- `onboarding_validation_issues` — structured failures and warnings from each readiness run.[^1]


## Table definitions

The DDL below is production-ready for M7_2 and uses tenant-owned configuration tables, explicit enums, auditable updates, justified soft delete only for branding assets, and constraints aligned with the PRD rules around profile completeness, academic structure integrity, campus control, and readiness validation.[^1]

```sql
CREATE TYPE school_onboarding_status AS ENUM (
    'draft',
    'profile_configured',
    'academic_setup_complete',
    'organizational_setup_complete',
    'ready_for_validation',
    'live',
    'validation_failed',
    'needs_action'
);

CREATE TYPE school_record_status AS ENUM ('draft', 'active', 'inactive');
CREATE TYPE academic_year_status AS ENUM ('draft', 'scheduled', 'active', 'completed', 'archived');
CREATE TYPE term_status AS ENUM ('draft', 'active', 'completed', 'archived');
CREATE TYPE campus_status AS ENUM ('draft', 'active', 'inactive');
CREATE TYPE branding_asset_type AS ENUM ('logo', 'favicon', 'letterhead', 'portal_banner', 'seal');
CREATE TYPE branding_asset_status AS ENUM ('active', 'archived');
CREATE TYPE onboarding_step_status AS ENUM ('not_started', 'in_progress', 'completed', 'blocked');
CREATE TYPE validation_run_status AS ENUM ('passed', 'failed', 'warning_only');
CREATE TYPE validation_issue_severity AS ENUM ('error', 'warning', 'info');

CREATE TABLE schools (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL UNIQUE REFERENCES tenants(id) ON DELETE CASCADE,
    school_code VARCHAR(50) NOT NULL,
    status school_onboarding_status NOT NULL DEFAULT 'draft',
    go_live_at TIMESTAMPTZ,
    last_validation_at TIMESTAMPTZ,
    last_validation_status validation_run_status,
    config_locked BOOLEAN NOT NULL DEFAULT FALSE,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID NULL REFERENCES users(id),
    updated_by UUID NULL REFERENCES users(id),
    CONSTRAINT uq_schools_tenant_code UNIQUE (tenant_id, school_code)
);

CREATE TABLE school_profiles (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL UNIQUE REFERENCES tenants(id) ON DELETE CASCADE,
    school_id UUID NOT NULL UNIQUE REFERENCES schools(id) ON DELETE CASCADE,
    school_name VARCHAR(255) NOT NULL,
    legal_name VARCHAR(255),
    contact_email CITEXT NOT NULL,
    contact_phone VARCHAR(20) NOT NULL,
    website VARCHAR(255),
    timezone VARCHAR(64) NOT NULL,
    address_line_1 VARCHAR(255) NOT NULL,
    address_line_2 VARCHAR(255),
    city VARCHAR(100) NOT NULL,
    state VARCHAR(100) NOT NULL,
    country_code CHAR(2) NOT NULL,
    postal_code VARCHAR(20) NOT NULL,
    status school_record_status NOT NULL DEFAULT 'active',
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID NULL REFERENCES users(id),
    updated_by UUID NULL REFERENCES users(id)
);

CREATE TABLE school_settings (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL UNIQUE REFERENCES tenants(id) ON DELETE CASCADE,
    school_id UUID NOT NULL UNIQUE REFERENCES schools(id) ON DELETE CASCADE,
    language_code VARCHAR(10) NOT NULL DEFAULT 'en',
    currency_code CHAR(3) NOT NULL DEFAULT 'INR',
    date_format VARCHAR(20) NOT NULL DEFAULT 'DD-MM-YYYY',
    time_format_24h BOOLEAN NOT NULL DEFAULT TRUE,
    communication_enabled BOOLEAN NOT NULL DEFAULT TRUE,
    attendance_enabled BOOLEAN NOT NULL DEFAULT TRUE,
    gradebook_enabled BOOLEAN NOT NULL DEFAULT TRUE,
    fees_enabled BOOLEAN NOT NULL DEFAULT TRUE,
    timetable_enabled BOOLEAN NOT NULL DEFAULT TRUE,
    parent_portal_enabled BOOLEAN NOT NULL DEFAULT TRUE,
    student_portal_enabled BOOLEAN NOT NULL DEFAULT TRUE,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID NULL REFERENCES users(id),
    updated_by UUID NULL REFERENCES users(id)
);

CREATE TABLE academic_years (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
    school_id UUID NOT NULL REFERENCES schools(id) ON DELETE CASCADE,
    name VARCHAR(50) NOT NULL,
    start_date DATE NOT NULL,
    end_date DATE NOT NULL,
    status academic_year_status NOT NULL DEFAULT 'draft',
    is_active BOOLEAN NOT NULL DEFAULT FALSE,
    notes TEXT,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID NULL REFERENCES users(id),
    updated_by UUID NULL REFERENCES users(id),
    CONSTRAINT ck_academic_year_dates CHECK (end_date >= start_date),
    CONSTRAINT uq_academic_year_name UNIQUE (tenant_id, name)
);

CREATE TABLE terms (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
    school_id UUID NOT NULL REFERENCES schools(id) ON DELETE CASCADE,
    academic_year_id UUID NOT NULL REFERENCES academic_years(id) ON DELETE RESTRICT,
    name VARCHAR(100) NOT NULL,
    start_date DATE NOT NULL,
    end_date DATE NOT NULL,
    status term_status NOT NULL DEFAULT 'draft',
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID NULL REFERENCES users(id),
    updated_by UUID NULL REFERENCES users(id),
    CONSTRAINT ck_term_dates CHECK (end_date >= start_date),
    CONSTRAINT uq_term_name_per_year UNIQUE (tenant_id, academic_year_id, name)
);

CREATE TABLE grade_levels (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
    school_id UUID NOT NULL REFERENCES schools(id) ON DELETE CASCADE,
    code VARCHAR(30) NOT NULL,
    name VARCHAR(100) NOT NULL,
    sort_order INTEGER NOT NULL,
    is_active BOOLEAN NOT NULL DEFAULT TRUE,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID NULL REFERENCES users(id),
    updated_by UUID NULL REFERENCES users(id),
    CONSTRAINT ck_grade_sort_order CHECK (sort_order > 0),
    CONSTRAINT uq_grade_code UNIQUE (tenant_id, code),
    CONSTRAINT uq_grade_name UNIQUE (tenant_id, name)
);

CREATE TABLE sections (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
    school_id UUID NOT NULL REFERENCES schools(id) ON DELETE CASCADE,
    academic_year_id UUID NOT NULL REFERENCES academic_years(id) ON DELETE RESTRICT,
    grade_level_id UUID NOT NULL REFERENCES grade_levels(id) ON DELETE RESTRICT,
    name VARCHAR(50) NOT NULL,
    capacity INTEGER,
    is_active BOOLEAN NOT NULL DEFAULT TRUE,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID NULL REFERENCES users(id),
    updated_by UUID NULL REFERENCES users(id),
    CONSTRAINT ck_section_capacity CHECK (capacity IS NULL OR capacity > 0),
    CONSTRAINT uq_section_grade_year UNIQUE (tenant_id, academic_year_id, grade_level_id, name)
);

CREATE TABLE campuses (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
    school_id UUID NOT NULL REFERENCES schools(id) ON DELETE CASCADE,
    campus_name VARCHAR(150) NOT NULL,
    address_line_1 VARCHAR(255) NOT NULL,
    address_line_2 VARCHAR(255),
    city VARCHAR(100) NOT NULL,
    state VARCHAR(100) NOT NULL,
    postal_code VARCHAR(20),
    country_code CHAR(2) NOT NULL,
    contact_phone VARCHAR(20),
    contact_email CITEXT,
    is_primary BOOLEAN NOT NULL DEFAULT FALSE,
    status campus_status NOT NULL DEFAULT 'draft',
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID NULL REFERENCES users(id),
    updated_by UUID NULL REFERENCES users(id),
    CONSTRAINT uq_campus_name UNIQUE (tenant_id, campus_name)
);

CREATE TABLE section_campuses (
    tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
    section_id UUID NOT NULL REFERENCES sections(id) ON DELETE CASCADE,
    campus_id UUID NOT NULL REFERENCES campuses(id) ON DELETE RESTRICT,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID NULL REFERENCES users(id),
    PRIMARY KEY (tenant_id, section_id, campus_id)
);

CREATE TABLE branding_assets (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
    school_id UUID NOT NULL REFERENCES schools(id) ON DELETE CASCADE,
    asset_type branding_asset_type NOT NULL,
    original_file_name VARCHAR(255) NOT NULL,
    mime_type VARCHAR(100) NOT NULL,
    file_size_bytes BIGINT NOT NULL,
    storage_key TEXT NOT NULL,
    public_url TEXT,
    checksum_sha256 CHAR(64),
    status branding_asset_status NOT NULL DEFAULT 'active',
    deleted_at TIMESTAMPTZ,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID NOT NULL REFERENCES users(id),
    updated_by UUID NULL REFERENCES users(id),
    CONSTRAINT ck_branding_asset_size CHECK (file_size_bytes > 0 AND file_size_bytes <= 5242880)
);

CREATE TABLE branding_themes (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL UNIQUE REFERENCES tenants(id) ON DELETE CASCADE,
    school_id UUID NOT NULL UNIQUE REFERENCES schools(id) ON DELETE CASCADE,
    primary_color VARCHAR(20),
    secondary_color VARCHAR(20),
    accent_color VARCHAR(20),
    logo_asset_id UUID NULL REFERENCES branding_assets(id) ON DELETE SET NULL,
    favicon_asset_id UUID NULL REFERENCES branding_assets(id) ON DELETE SET NULL,
    portal_banner_asset_id UUID NULL REFERENCES branding_assets(id) ON DELETE SET NULL,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID NULL REFERENCES users(id),
    updated_by UUID NULL REFERENCES users(id)
);

CREATE TABLE communication_settings (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL UNIQUE REFERENCES tenants(id) ON DELETE CASCADE,
    school_id UUID NOT NULL UNIQUE REFERENCES schools(id) ON DELETE CASCADE,
    sender_email VARCHAR(255),
    sender_name VARCHAR(150),
    reply_to_email VARCHAR(255),
    sms_sender_id VARCHAR(50),
    email_enabled BOOLEAN NOT NULL DEFAULT FALSE,
    sms_enabled BOOLEAN NOT NULL DEFAULT FALSE,
    push_enabled BOOLEAN NOT NULL DEFAULT TRUE,
    default_notification_language VARCHAR(10) NOT NULL DEFAULT 'en',
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID NULL REFERENCES users(id),
    updated_by UUID NULL REFERENCES users(id)
);

CREATE TABLE onboarding_checklists (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
    school_id UUID NOT NULL REFERENCES schools(id) ON DELETE CASCADE,
    step_code VARCHAR(100) NOT NULL,
    step_label VARCHAR(150) NOT NULL,
    step_order INTEGER NOT NULL,
    status onboarding_step_status NOT NULL DEFAULT 'not_started',
    completed_at TIMESTAMPTZ,
    blocked_reason TEXT,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID NULL REFERENCES users(id),
    updated_by UUID NULL REFERENCES users(id),
    CONSTRAINT uq_onboarding_step UNIQUE (tenant_id, school_id, step_code),
    CONSTRAINT ck_onboarding_step_order CHECK (step_order > 0)
);

CREATE TABLE onboarding_validation_runs (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
    school_id UUID NOT NULL REFERENCES schools(id) ON DELETE CASCADE,
    run_status validation_run_status NOT NULL,
    completion_percentage NUMERIC(5,2) NOT NULL DEFAULT 0,
    missing_requirements_count INTEGER NOT NULL DEFAULT 0,
    validated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID NULL REFERENCES users(id),
    CONSTRAINT ck_completion_percentage CHECK (completion_percentage >= 0 AND completion_percentage <= 100)
);

CREATE TABLE onboarding_validation_issues (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
    validation_run_id UUID NOT NULL REFERENCES onboarding_validation_runs(id) ON DELETE CASCADE,
    school_id UUID NOT NULL REFERENCES schools(id) ON DELETE CASCADE,
    issue_code VARCHAR(100) NOT NULL,
    issue_area VARCHAR(100) NOT NULL,
    severity validation_issue_severity NOT NULL,
    issue_message TEXT NOT NULL,
    blocking_go_live BOOLEAN NOT NULL DEFAULT TRUE,
    resolved_at TIMESTAMPTZ,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID NULL REFERENCES users(id)
);
```


## Constraints

The most important enforcement points in M7_2 are single-school-profile-per-tenant, non-overlapping academic structures, one active academic year, section uniqueness inside grade plus academic year, one primary campus, and go-live blocking until mandatory setup is complete.[^1]

**Core constraints**

- `schools.tenant_id UNIQUE` ensures one school record per tenant, which matches the PRD’s “exactly one active school profile” rule.[^1]
- `school_profiles.tenant_id UNIQUE` and `school_profiles.school_id UNIQUE` ensure a single active profile per tenant school.[^1]
- `grade_levels.name` is unique per tenant, because grade names must be unique within a tenant.[^1]
- `sections` uses `UNIQUE (tenant_id, academic_year_id, grade_level_id, name)` so Grade 1-A and Grade 2-A are both valid, but duplicate Grade 1-A in the same academic year is blocked.[^1]
- `campuses` must have only one active primary campus per tenant, enforced with a partial unique index.[^1]
- `branding_assets.file_size_bytes <= 5 MB` enforces the PRD logo-size rule.[^1]

**Recommended advanced constraints**

- Use PostgreSQL exclusion constraints for date overlap prevention on `academic_years` and `terms`, because the PRD explicitly forbids overlapping academic years and terms.[^1]
- Add trigger-based validation so `terms.start_date/end_date` must stay inside the parent academic year date range, because that rule depends on another table.[^1]
- Add trigger-based validation to block deleting an academic year after student enrollment exists, block deleting a campus with active students, and block lowering section capacity below enrolled count; those dependencies belong to downstream modules but the PRD explicitly requires those protections.[^1]

```sql
CREATE EXTENSION IF NOT EXISTS btree_gist;

ALTER TABLE academic_years
ADD CONSTRAINT ex_academic_years_no_overlap
EXCLUDE USING gist (
    tenant_id WITH =,
    daterange(start_date, end_date, '[]') WITH &&
);

CREATE UNIQUE INDEX uq_one_active_academic_year_per_tenant
ON academic_years (tenant_id)
WHERE is_active = TRUE;

CREATE UNIQUE INDEX uq_one_primary_campus_per_tenant
ON campuses (tenant_id)
WHERE is_primary = TRUE AND status = 'active';

CREATE INDEX idx_terms_academic_year
ON terms (tenant_id, academic_year_id, start_date);

CREATE INDEX idx_sections_year_grade
ON sections (tenant_id, academic_year_id, grade_level_id, name);

CREATE INDEX idx_onboarding_checklists_status
ON onboarding_checklists (tenant_id, school_id, status, step_order);

CREATE INDEX idx_validation_runs_recent
ON onboarding_validation_runs (tenant_id, school_id, validated_at DESC);

CREATE INDEX idx_validation_issues_blocking
ON onboarding_validation_issues (tenant_id, school_id, blocking_go_live, severity);
```


## Indexes

These indexes are chosen around the likely M7_2 query patterns: loading the setup wizard, listing academic years and terms, rendering grade-section structures, campus lookup, branding resolution, and showing readiness progress and validation failures.[^1]

- `schools (tenant_id)` unique lookup for tenant school bootstrap.[^1]
- `school_profiles (tenant_id)` unique lookup for profile screens and portal branding resolution.[^1]
- `academic_years (tenant_id, status, start_date desc)` for year list and activation workflows.[^1]
- `terms (tenant_id, academic_year_id, start_date)` for calendar and reporting period setup.[^1]
- `grade_levels (tenant_id, sort_order)` for grade structure screen ordering.[^1]
- `sections (tenant_id, academic_year_id, grade_level_id, name)` for section management and duplicate checking.[^1]
- `campuses (tenant_id, status, is_primary)` for campus management and primary-campus selection.[^1]
- `branding_assets (tenant_id, asset_type, status)` for logo/theme resolution.[^1]
- `onboarding_checklists (tenant_id, school_id, step_order)` for setup wizard progression.[^1]
- `onboarding_validation_issues (tenant_id, school_id, blocking_go_live)` for readiness dashboard blockers.[^1]

```sql
CREATE INDEX idx_academic_years_tenant_status_dates
ON academic_years (tenant_id, status, start_date DESC, end_date DESC);

CREATE INDEX idx_grade_levels_tenant_sort
ON grade_levels (tenant_id, sort_order, name);

CREATE INDEX idx_campuses_tenant_status_primary
ON campuses (tenant_id, status, is_primary);

CREATE INDEX idx_branding_assets_tenant_type_status
ON branding_assets (tenant_id, asset_type, status)
WHERE deleted_at IS NULL;

CREATE INDEX idx_branding_themes_tenant
ON branding_themes (tenant_id);

CREATE INDEX idx_school_profiles_tenant
ON school_profiles (tenant_id);

CREATE INDEX idx_sections_tenant_active
ON sections (tenant_id, is_active, academic_year_id);

CREATE INDEX idx_checklists_tenant_order
ON onboarding_checklists (tenant_id, school_id, step_order);

CREATE INDEX idx_validation_issues_run
ON onboarding_validation_issues (validation_run_id, severity, blocking_go_live);
```


## Sample records

These samples reflect the normal onboarding flow: profile created, settings defined, one academic year, terms, grades, sections, a primary campus, branding uploaded, and readiness tracked.[^1]

```sql
INSERT INTO schools (
    id, tenant_id, school_code, status, created_by
) VALUES (
    '51000000-0000-0000-0000-000000000001',
    '10000000-0000-0000-0000-000000000001',
    'GFS-CHN',
    'profile_configured',
    '00000000-0000-0000-0000-000000000002'
);

INSERT INTO school_profiles (
    id, tenant_id, school_id, school_name, legal_name, contact_email, contact_phone,
    timezone, address_line_1, city, state, country_code, postal_code, created_by
) VALUES (
    '52000000-0000-0000-0000-000000000001',
    '10000000-0000-0000-0000-000000000001',
    '51000000-0000-0000-0000-000000000001',
    'Greenfield School',
    'Greenfield Educational Trust',
    'info@greenfield.edu',
    '+914400001111',
    'Asia/Kolkata',
    '12 School Road',
    'Chennai',
    'Tamil Nadu',
    'IN',
    '600053',
    '00000000-0000-0000-0000-000000000002'
);

INSERT INTO academic_years (
    id, tenant_id, school_id, name, start_date, end_date, status, is_active, created_by
) VALUES (
    '53000000-0000-0000-0000-000000000001',
    '10000000-0000-0000-0000-000000000001',
    '51000000-0000-0000-0000-000000000001',
    '2026-2027',
    '2026-06-01',
    '2027-03-31',
    'active',
    TRUE,
    '00000000-0000-0000-0000-000000000002'
);

INSERT INTO grade_levels (
    id, tenant_id, school_id, code, name, sort_order, created_by
) VALUES
(
    '54000000-0000-0000-0000-000000000001',
    '10000000-0000-0000-0000-000000000001',
    '51000000-0000-0000-0000-000000000001',
    'G01', 'Grade 1', 1, '00000000-0000-0000-0000-000000000002'
),
(
    '54000000-0000-0000-0000-000000000002',
    '10000000-0000-0000-0000-000000000001',
    '51000000-0000-0000-0000-000000000001',
    'G02', 'Grade 2', 2, '00000000-0000-0000-0000-000000000002'
);

INSERT INTO sections (
    id, tenant_id, school_id, academic_year_id, grade_level_id, name, capacity, created_by
) VALUES
(
    '55000000-0000-0000-0000-000000000001',
    '10000000-0000-0000-0000-000000000001',
    '51000000-0000-0000-0000-000000000001',
    '53000000-0000-0000-0000-000000000001',
    '54000000-0000-0000-0000-000000000001',
    'A', 35, '00000000-0000-0000-0000-000000000002'
);
```


## Query patterns

These are the main operational queries for Module 7.2: bootstrap the setup wizard, fetch active academic configuration, render grade-section hierarchy, resolve current branding, and run readiness checks.[^1]

1. **Load onboarding dashboard for a tenant**[^1]
```sql
SELECT s.status,
       o.step_code,
       o.step_label,
       o.status AS step_status,
       o.step_order
FROM schools s
JOIN onboarding_checklists o
  ON o.school_id = s.id AND o.tenant_id = s.tenant_id
WHERE s.tenant_id = $1
ORDER BY o.step_order;
```

2. **Fetch active academic year and its terms**[^1]
```sql
SELECT ay.id, ay.name, ay.start_date, ay.end_date,
       t.id AS term_id, t.name AS term_name, t.start_date AS term_start, t.end_date AS term_end
FROM academic_years ay
LEFT JOIN terms t
  ON t.academic_year_id = ay.id AND t.tenant_id = ay.tenant_id
WHERE ay.tenant_id = $1
  AND ay.is_active = TRUE
ORDER BY t.start_date;
```

3. **Render grade-section structure for one academic year**[^1]
```sql
SELECT g.id AS grade_id, g.name AS grade_name, g.sort_order,
       s.id AS section_id, s.name AS section_name, s.capacity
FROM grade_levels g
LEFT JOIN sections s
  ON s.grade_level_id = g.id
 AND s.tenant_id = g.tenant_id
 AND s.academic_year_id = $2
WHERE g.tenant_id = $1
ORDER BY g.sort_order, s.name;
```

4. **Resolve active school branding for portals and documents**[^1]
```sql
SELECT sp.school_name,
       bt.primary_color,
       bt.secondary_color,
       ba.public_url AS logo_url
FROM school_profiles sp
LEFT JOIN branding_themes bt
  ON bt.tenant_id = sp.tenant_id
LEFT JOIN branding_assets ba
  ON ba.id = bt.logo_asset_id
WHERE sp.tenant_id = $1
  AND sp.status = 'active';
```

5. **Find blockers before go-live**[^1]
```sql
SELECT issue_area, issue_code, severity, issue_message
FROM onboarding_validation_issues
WHERE tenant_id = $1
  AND validation_run_id = $2
  AND blocking_go_live = TRUE
ORDER BY severity DESC, issue_area;
```


## Migration sequence

M7_2 depends on Module 7.1 platform-core tables such as `tenants` and `users`, so the migration order should respect that dependency and then build from school identity to academic structure to readiness tracking.[^1]

1. Create M7_2 enums.[^1]
2. Create `schools`, `school_profiles`, and `school_settings` first, because all other configuration hangs off them.[^1]
3. Create `academic_years` and `terms`, then add non-overlap constraints and term-inside-year validation triggers.[^1]
4. Create `grade_levels`, `sections`, `campuses`, and `section_campuses`.[^1]
5. Create `branding_assets`, `branding_themes`, and `communication_settings`.[^1]
6. Create `onboarding_checklists`, `onboarding_validation_runs`, and `onboarding_validation_issues`.[^1]
7. Add indexes, partial unique indexes, RLS policies, and protection triggers for delete/activation rules.[^1]
8. Seed standard onboarding steps such as `school_profile`, `academic_year`, `terms`, `grades`, `sections`, `campuses`, `branding`, `communication_settings`, `readiness_check`, and `go_live`.[^1]

## Security-sensitive columns

M7_2 contains lower sensitivity than identity/auth modules, but it still holds institutional contact data, branding assets, and operational configuration that should not be freely exposed.[^1]

- `school_profiles.contact_email`, `school_profiles.contact_phone`, and full address columns are institutional contact data and should be exposed only to authorized school or platform admins.[^1]
- `branding_assets.storage_key`, `branding_assets.public_url`, and `checksum_sha256` should be protected from broad writes because they control tenant-visible branding integrity.[^1]
- `communication_settings.sender_email`, `reply_to_email`, and `sms_sender_id` are security-relevant outbound communication identifiers.[^1]
- `schools.config_locked` and go-live fields are control-plane data and should be writable only by authorized admins or super admins.[^1]
- `onboarding_validation_issues` may reveal internal readiness gaps, so parent and student roles should never access them.[^1]

This M7_2 design keeps the onboarding module cleanly separated from downstream transactional modules while still enforcing the exact setup prerequisites the PRD says are needed before a school can go live.[^1]

<div align="center">⁂</div>

[^1]: PRD_M7_2.txt

