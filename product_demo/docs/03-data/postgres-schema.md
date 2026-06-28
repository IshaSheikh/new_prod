<!-- Source: PRDs | Last Verified: 2026-06-28 | Owner: Engineering -->
# PostgreSQL Schema

Core production-ready schema definitions organized by module. All tables follow platform conventions: `tenant_id NOT NULL`, `created_at/updated_at`, `created_by/updated_by`, and RLS policies.

---

## Global Extensions and Enums

```sql
CREATE EXTENSION IF NOT EXISTS pgcrypto;
CREATE EXTENSION IF NOT EXISTS citext;
CREATE EXTENSION IF NOT EXISTS btree_gist;

-- Platform enums
CREATE TYPE tenant_status AS ENUM ('draft', 'provisioning', 'active', 'suspended', 'cancelled');
CREATE TYPE subscription_status AS ENUM ('trial', 'active', 'grace_period', 'expired', 'cancelled', 'suspended');
CREATE TYPE user_account_status AS ENUM ('pending_acceptance', 'active', 'suspended', 'disabled', 'removed');
CREATE TYPE membership_status AS ENUM ('invited', 'pending', 'active', 'inactive', 'disabled', 'removed');
CREATE TYPE role_type AS ENUM (
    'super_admin', 'school_admin', 'principal', 'teacher', 'accountant',
    'parent', 'student', 'transport_manager', 'librarian'
);
CREATE TYPE invitation_status AS ENUM ('pending', 'accepted', 'expired', 'revoked');
CREATE TYPE session_status AS ENUM ('active', 'revoked', 'expired');
```

---

## M7.1 — Platform Core

```sql
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email CITEXT NOT NULL UNIQUE,
    mobile_e164 VARCHAR(20),
    first_name VARCHAR(100) NOT NULL,
    last_name VARCHAR(100),
    password_hash TEXT NOT NULL,
    status user_account_status NOT NULL DEFAULT 'pending_acceptance',
    is_platform_super_admin BOOLEAN NOT NULL DEFAULT FALSE,
    last_login_at TIMESTAMPTZ,
    deleted_at TIMESTAMPTZ,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID REFERENCES users(id),
    updated_by UUID REFERENCES users(id)
);

CREATE TABLE tenants (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    code VARCHAR(50) NOT NULL UNIQUE,
    name VARCHAR(255) NOT NULL,
    status tenant_status NOT NULL DEFAULT 'draft',
    timezone VARCHAR(64) NOT NULL DEFAULT 'Asia/Kolkata',
    address_line1 VARCHAR(255),
    city VARCHAR(100),
    state VARCHAR(100),
    postal_code VARCHAR(20),
    country_code CHAR(2) NOT NULL DEFAULT 'IN',
    onboarding_completed_at TIMESTAMPTZ,
    activated_at TIMESTAMPTZ,
    suspended_at TIMESTAMPTZ,
    cancelled_at TIMESTAMPTZ,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID REFERENCES users(id),
    updated_by UUID REFERENCES users(id)
);

CREATE TABLE tenant_domains (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
    domain_type VARCHAR(20) NOT NULL CHECK (domain_type IN ('subdomain', 'custom')),
    domain VARCHAR(255) NOT NULL,
    is_primary BOOLEAN NOT NULL DEFAULT FALSE,
    verification_status VARCHAR(20) NOT NULL DEFAULT 'pending',
    verification_token_hash TEXT,
    verified_at TIMESTAMPTZ,
    removed_at TIMESTAMPTZ,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID REFERENCES users(id),
    CONSTRAINT uq_tenant_domain UNIQUE (domain)
);

CREATE UNIQUE INDEX uq_tenant_primary_domain
ON tenant_domains (tenant_id)
WHERE is_primary = TRUE AND removed_at IS NULL;

CREATE TABLE subscription_plans (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    code VARCHAR(50) NOT NULL UNIQUE,
    name VARCHAR(100) NOT NULL,
    max_students INTEGER,
    max_staff INTEGER,
    monthly_price NUMERIC(12,2) NOT NULL,
    annual_price NUMERIC(12,2) NOT NULL,
    is_active BOOLEAN NOT NULL DEFAULT TRUE,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE TABLE subscriptions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
    plan_id UUID NOT NULL REFERENCES subscription_plans(id),
    billing_cycle VARCHAR(10) NOT NULL CHECK (billing_cycle IN ('monthly', 'annual')),
    status subscription_status NOT NULL DEFAULT 'trial',
    start_date DATE NOT NULL,
    end_date DATE NOT NULL,
    grace_until DATE,
    cancelled_at TIMESTAMPTZ,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID REFERENCES users(id)
);

CREATE UNIQUE INDEX uq_tenant_current_subscription
ON subscriptions (tenant_id)
WHERE status IN ('trial', 'active', 'grace_period');

CREATE TABLE permissions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    code VARCHAR(120) NOT NULL UNIQUE,
    module_name VARCHAR(100) NOT NULL,
    action_name VARCHAR(100) NOT NULL,
    description TEXT,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE TABLE roles (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID REFERENCES tenants(id) ON DELETE CASCADE,
    name VARCHAR(100) NOT NULL,
    role_type role_type NOT NULL,
    is_system_role BOOLEAN NOT NULL DEFAULT TRUE,
    description TEXT,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID REFERENCES users(id)
);

CREATE TABLE role_permissions (
    role_id UUID NOT NULL REFERENCES roles(id) ON DELETE CASCADE,
    permission_id UUID NOT NULL REFERENCES permissions(id) ON DELETE CASCADE,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    PRIMARY KEY (role_id, permission_id)
);

CREATE TABLE user_tenant_memberships (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
    user_id UUID NOT NULL REFERENCES users(id) ON DELETE RESTRICT,
    status membership_status NOT NULL DEFAULT 'invited',
    joined_at TIMESTAMPTZ,
    disabled_at TIMESTAMPTZ,
    removed_at TIMESTAMPTZ,
    invited_by UUID REFERENCES users(id),
    accepted_at TIMESTAMPTZ,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_by UUID REFERENCES users(id),
    CONSTRAINT uq_membership_user_tenant UNIQUE (tenant_id, user_id),
    CONSTRAINT uq_membership_id_tenant UNIQUE (id, tenant_id)
);

CREATE TABLE membership_roles (
    tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
    membership_id UUID NOT NULL,
    role_id UUID NOT NULL REFERENCES roles(id) ON DELETE RESTRICT,
    assigned_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    revoked_at TIMESTAMPTZ,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID REFERENCES users(id),
    PRIMARY KEY (tenant_id, membership_id, role_id),
    FOREIGN KEY (membership_id, tenant_id)
        REFERENCES user_tenant_memberships(id, tenant_id) ON DELETE CASCADE
);

CREATE TABLE tenant_invitations (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
    email CITEXT NOT NULL,
    invite_token_hash TEXT NOT NULL,
    status invitation_status NOT NULL DEFAULT 'pending',
    expires_at TIMESTAMPTZ NOT NULL,
    accepted_at TIMESTAMPTZ,
    accepted_membership_id UUID REFERENCES user_tenant_memberships(id),
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID REFERENCES users(id)
);

CREATE TABLE auth_sessions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID REFERENCES tenants(id) ON DELETE CASCADE,
    user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    membership_id UUID REFERENCES user_tenant_memberships(id) ON DELETE CASCADE,
    refresh_token_hash TEXT NOT NULL,
    status session_status NOT NULL DEFAULT 'active',
    ip_address INET,
    user_agent TEXT,
    last_seen_at TIMESTAMPTZ,
    expires_at TIMESTAMPTZ NOT NULL,
    revoked_at TIMESTAMPTZ,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE TABLE audit_logs (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID REFERENCES tenants(id) ON DELETE RESTRICT,
    actor_user_id UUID REFERENCES users(id) ON DELETE SET NULL,
    action VARCHAR(100) NOT NULL,
    entity_table VARCHAR(100) NOT NULL,
    entity_id UUID NOT NULL,
    old_value JSONB,
    new_value JSONB,
    ip_address INET,
    user_agent TEXT,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

---

## M7.2 — School Onboarding

```sql
CREATE TABLE schools (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL UNIQUE REFERENCES tenants(id) ON DELETE CASCADE,
    school_code VARCHAR(50) NOT NULL,
    status VARCHAR(50) NOT NULL DEFAULT 'draft',
    go_live_at TIMESTAMPTZ,
    config_locked BOOLEAN NOT NULL DEFAULT FALSE,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID REFERENCES users(id),
    updated_by UUID REFERENCES users(id)
);

CREATE TABLE school_profiles (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL UNIQUE REFERENCES tenants(id),
    school_id UUID NOT NULL UNIQUE REFERENCES schools(id),
    school_name VARCHAR(255) NOT NULL,
    legal_name VARCHAR(255),
    contact_email CITEXT NOT NULL,
    contact_phone VARCHAR(20) NOT NULL,
    website VARCHAR(255),
    timezone VARCHAR(64) NOT NULL,
    address_line_1 VARCHAR(255) NOT NULL,
    city VARCHAR(100) NOT NULL,
    state VARCHAR(100) NOT NULL,
    country_code CHAR(2) NOT NULL DEFAULT 'IN',
    postal_code VARCHAR(20) NOT NULL,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID REFERENCES users(id),
    updated_by UUID REFERENCES users(id)
);

CREATE TABLE school_settings (
    tenant_id UUID PRIMARY KEY REFERENCES tenants(id),
    school_id UUID NOT NULL UNIQUE REFERENCES schools(id),
    language_code VARCHAR(10) NOT NULL DEFAULT 'en',
    currency_code CHAR(3) NOT NULL DEFAULT 'INR',
    date_format VARCHAR(20) NOT NULL DEFAULT 'DD-MM-YYYY',
    time_format_24h BOOLEAN NOT NULL DEFAULT TRUE,
    attendance_enabled BOOLEAN NOT NULL DEFAULT TRUE,
    gradebook_enabled BOOLEAN NOT NULL DEFAULT TRUE,
    fees_enabled BOOLEAN NOT NULL DEFAULT TRUE,
    timetable_enabled BOOLEAN NOT NULL DEFAULT TRUE,
    parent_portal_enabled BOOLEAN NOT NULL DEFAULT TRUE,
    student_portal_enabled BOOLEAN NOT NULL DEFAULT FALSE,
    communication_enabled BOOLEAN NOT NULL DEFAULT TRUE,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID REFERENCES users(id),
    updated_by UUID REFERENCES users(id)
);

CREATE TABLE academic_years (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
    school_id UUID NOT NULL REFERENCES schools(id),
    name VARCHAR(50) NOT NULL,
    start_date DATE NOT NULL,
    end_date DATE NOT NULL,
    status VARCHAR(20) NOT NULL DEFAULT 'draft',
    is_active BOOLEAN NOT NULL DEFAULT FALSE,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID REFERENCES users(id),
    updated_by UUID REFERENCES users(id),
    CONSTRAINT ck_academic_year_dates CHECK (end_date >= start_date),
    CONSTRAINT uq_academic_year_name UNIQUE (tenant_id, name)
);

ALTER TABLE academic_years
ADD CONSTRAINT ex_academic_years_no_overlap
EXCLUDE USING gist (
    tenant_id WITH =,
    daterange(start_date, end_date, '[]') WITH &&
);

CREATE UNIQUE INDEX uq_one_active_academic_year
ON academic_years (tenant_id) WHERE is_active = TRUE;

CREATE TABLE terms (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id),
    school_id UUID NOT NULL REFERENCES schools(id),
    academic_year_id UUID NOT NULL REFERENCES academic_years(id) ON DELETE RESTRICT,
    name VARCHAR(100) NOT NULL,
    start_date DATE NOT NULL,
    end_date DATE NOT NULL,
    status VARCHAR(20) NOT NULL DEFAULT 'draft',
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID REFERENCES users(id),
    CONSTRAINT ck_term_dates CHECK (end_date >= start_date),
    CONSTRAINT uq_term_name_per_year UNIQUE (tenant_id, academic_year_id, name)
);

CREATE TABLE grade_levels (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id),
    school_id UUID NOT NULL REFERENCES schools(id),
    code VARCHAR(30) NOT NULL,
    name VARCHAR(100) NOT NULL,
    sort_order INTEGER NOT NULL,
    is_active BOOLEAN NOT NULL DEFAULT TRUE,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID REFERENCES users(id),
    CONSTRAINT uq_grade_code UNIQUE (tenant_id, code),
    CONSTRAINT uq_grade_name UNIQUE (tenant_id, name)
);

CREATE TABLE sections (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id),
    school_id UUID NOT NULL REFERENCES schools(id),
    academic_year_id UUID NOT NULL REFERENCES academic_years(id),
    grade_level_id UUID NOT NULL REFERENCES grade_levels(id),
    name VARCHAR(50) NOT NULL,
    capacity INTEGER,
    is_active BOOLEAN NOT NULL DEFAULT TRUE,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID REFERENCES users(id),
    CONSTRAINT uq_section_grade_year UNIQUE (tenant_id, academic_year_id, grade_level_id, name)
);

CREATE TABLE campuses (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id),
    school_id UUID NOT NULL REFERENCES schools(id),
    campus_name VARCHAR(150) NOT NULL,
    address_line_1 VARCHAR(255) NOT NULL,
    city VARCHAR(100) NOT NULL,
    state VARCHAR(100) NOT NULL,
    country_code CHAR(2) NOT NULL DEFAULT 'IN',
    is_primary BOOLEAN NOT NULL DEFAULT FALSE,
    status VARCHAR(20) NOT NULL DEFAULT 'draft',
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID REFERENCES users(id),
    CONSTRAINT uq_campus_name UNIQUE (tenant_id, campus_name)
);

CREATE UNIQUE INDEX uq_one_primary_campus
ON campuses (tenant_id) WHERE is_primary = TRUE AND status = 'active';

CREATE TABLE branding_assets (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id),
    school_id UUID NOT NULL REFERENCES schools(id),
    asset_type VARCHAR(30) NOT NULL,
    original_file_name VARCHAR(255) NOT NULL,
    mime_type VARCHAR(100) NOT NULL,
    file_size_bytes BIGINT NOT NULL,
    storage_key TEXT NOT NULL,
    public_url TEXT,
    status VARCHAR(20) NOT NULL DEFAULT 'active',
    deleted_at TIMESTAMPTZ,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID NOT NULL REFERENCES users(id),
    CONSTRAINT ck_branding_asset_size CHECK (file_size_bytes > 0 AND file_size_bytes <= 5242880)
);

CREATE TABLE onboarding_checklists (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id),
    school_id UUID NOT NULL REFERENCES schools(id),
    step_code VARCHAR(100) NOT NULL,
    step_label VARCHAR(150) NOT NULL,
    step_order INTEGER NOT NULL,
    status VARCHAR(30) NOT NULL DEFAULT 'not_started',
    completed_at TIMESTAMPTZ,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    CONSTRAINT uq_onboarding_step UNIQUE (tenant_id, school_id, step_code)
);
```

---

## M7.4 — Student Information System

```sql
CREATE TABLE students (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id),
    school_id UUID NOT NULL REFERENCES schools(id),
    user_id UUID REFERENCES users(id) ON DELETE SET NULL,
    student_number VARCHAR(50) NOT NULL,
    admission_number VARCHAR(50),
    status VARCHAR(30) NOT NULL DEFAULT 'inquiry',
    admission_date DATE,
    deleted_at TIMESTAMPTZ,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID REFERENCES users(id),
    updated_by UUID REFERENCES users(id),
    CONSTRAINT uq_student_number UNIQUE (tenant_id, student_number)
);

CREATE TABLE student_profiles (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id),
    student_id UUID NOT NULL UNIQUE REFERENCES students(id),
    first_name VARCHAR(100) NOT NULL,
    middle_name VARCHAR(100),
    last_name VARCHAR(100),
    gender VARCHAR(20),
    date_of_birth DATE NOT NULL,
    nationality VARCHAR(100),
    blood_group VARCHAR(10),
    photo_storage_key TEXT,
    address_line_1 VARCHAR(255),
    city VARCHAR(100),
    state VARCHAR(100),
    country_code CHAR(2),
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID REFERENCES users(id),
    updated_by UUID REFERENCES users(id)
);

CREATE TABLE student_enrollments (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id),
    student_id UUID NOT NULL REFERENCES students(id),
    academic_year_id UUID NOT NULL REFERENCES academic_years(id),
    grade_level_id UUID NOT NULL REFERENCES grade_levels(id),
    section_id UUID NOT NULL REFERENCES sections(id),
    campus_id UUID REFERENCES campuses(id),
    enrollment_date DATE NOT NULL,
    exit_date DATE,
    is_primary BOOLEAN NOT NULL DEFAULT TRUE,
    status VARCHAR(30) NOT NULL DEFAULT 'draft',
    withdrawal_reason TEXT,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID REFERENCES users(id)
);

CREATE UNIQUE INDEX uq_primary_enrollment_per_year
ON student_enrollments (tenant_id, student_id, academic_year_id)
WHERE is_primary = TRUE AND status IN ('pending_approval', 'active');

CREATE TABLE guardians (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id),
    user_id UUID REFERENCES users(id) ON DELETE SET NULL,
    first_name VARCHAR(100) NOT NULL,
    last_name VARCHAR(100),
    mobile_e164 VARCHAR(20),
    email CITEXT,
    occupation VARCHAR(150),
    is_active BOOLEAN NOT NULL DEFAULT TRUE,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID REFERENCES users(id)
);

CREATE TABLE guardian_student_links (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id),
    guardian_id UUID NOT NULL REFERENCES guardians(id),
    student_id UUID NOT NULL REFERENCES students(id),
    relationship_type VARCHAR(30) NOT NULL,
    is_primary BOOLEAN NOT NULL DEFAULT FALSE,
    has_portal_access BOOLEAN NOT NULL DEFAULT FALSE,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID REFERENCES users(id),
    CONSTRAINT uq_guardian_student UNIQUE (tenant_id, guardian_id, student_id)
);

CREATE UNIQUE INDEX uq_primary_guardian_per_student
ON guardian_student_links (tenant_id, student_id)
WHERE is_primary = TRUE;

CREATE TABLE student_documents (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id),
    student_id UUID NOT NULL REFERENCES students(id),
    document_type VARCHAR(100) NOT NULL,
    document_label VARCHAR(150) NOT NULL,
    current_version_no INTEGER NOT NULL DEFAULT 1,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID REFERENCES users(id)
);

CREATE TABLE student_document_versions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id),
    student_document_id UUID NOT NULL REFERENCES student_documents(id),
    version_no INTEGER NOT NULL,
    storage_key TEXT NOT NULL,
    original_file_name VARCHAR(255) NOT NULL,
    mime_type VARCHAR(100) NOT NULL,
    file_size_bytes BIGINT NOT NULL,
    status VARCHAR(20) NOT NULL DEFAULT 'uploaded',
    verified_at TIMESTAMPTZ,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID REFERENCES users(id),
    CONSTRAINT uq_document_version UNIQUE (student_document_id, version_no)
);

CREATE TABLE student_health_notes (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id),
    student_id UUID NOT NULL REFERENCES students(id),
    severity VARCHAR(20) NOT NULL,
    medical_condition_label VARCHAR(150) NOT NULL,
    note TEXT NOT NULL,
    effective_date DATE NOT NULL,
    is_active BOOLEAN NOT NULL DEFAULT TRUE,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID REFERENCES users(id)
);

CREATE TABLE student_status_history (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id),
    student_id UUID NOT NULL REFERENCES students(id),
    old_status VARCHAR(30),
    new_status VARCHAR(30) NOT NULL,
    reason TEXT,
    changed_by UUID REFERENCES users(id),
    changed_at TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

---

## Key Indexes

```sql
-- Platform Core
CREATE INDEX idx_users_email ON users (email);
CREATE INDEX idx_memberships_tenant ON user_tenant_memberships (tenant_id, status, user_id);
CREATE INDEX idx_memberships_user ON user_tenant_memberships (user_id, status, tenant_id);
CREATE INDEX idx_sessions_user_status ON auth_sessions (user_id, tenant_id, status, expires_at);
CREATE INDEX idx_audit_tenant_created ON audit_logs (tenant_id, created_at DESC);

-- Onboarding
CREATE INDEX idx_academic_years_tenant ON academic_years (tenant_id, status, start_date DESC);
CREATE INDEX idx_grade_levels_tenant ON grade_levels (tenant_id, sort_order);
CREATE INDEX idx_sections_year_grade ON sections (tenant_id, academic_year_id, grade_level_id);

-- Students
CREATE INDEX idx_students_tenant_status ON students (tenant_id, status, student_number);
CREATE INDEX idx_student_profiles_name ON student_profiles (tenant_id, last_name, first_name, date_of_birth);
CREATE INDEX idx_enrollments_current ON student_enrollments (tenant_id, academic_year_id, section_id, status);
CREATE INDEX idx_guardian_links_student ON guardian_student_links (tenant_id, student_id, is_primary);
```

---

## Row-Level Security Policies

```sql
-- Enable RLS on all tenant-owned tables
ALTER TABLE tenants ENABLE ROW LEVEL SECURITY;
ALTER TABLE schools ENABLE ROW LEVEL SECURITY;
ALTER TABLE academic_years ENABLE ROW LEVEL SECURITY;
ALTER TABLE grade_levels ENABLE ROW LEVEL SECURITY;
ALTER TABLE sections ENABLE ROW LEVEL SECURITY;
ALTER TABLE campuses ENABLE ROW LEVEL SECURITY;
ALTER TABLE students ENABLE ROW LEVEL SECURITY;
ALTER TABLE student_profiles ENABLE ROW LEVEL SECURITY;
ALTER TABLE student_enrollments ENABLE ROW LEVEL SECURITY;
ALTER TABLE guardians ENABLE ROW LEVEL SECURITY;
-- ... (applied to ALL tenant-owned tables)

-- Standard RLS policy pattern
CREATE POLICY tenant_isolation ON students
  USING (tenant_id = current_setting('app.tenant_id', true)::uuid);

-- Application sets this before queries:
-- SET LOCAL app.tenant_id = 'tenant-uuid-here';
```

> **Note:** The full schema for modules M7.5–M7.17 follows the same conventions established here. See `data7_1.md` through `data7_16.md` in the `PRDs/` directory for complete DDL for each module.

