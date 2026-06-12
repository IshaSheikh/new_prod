<img src="https://r2cdn.perplexity.ai/pplx-full-logo-primary-dark%402x.png" style="height:64px;margin-right:32px"/>

# please proceed with M7_9

For M7_9, the right design is a **read-optimized portal access schema** with strict relationship-based authorization, tenant-scoped sessions, portal visibility policies, audited downloads and child-switch events, and minimal write capability limited to payments and support workflows, because the PRD defines the portal as a presentation and action layer rather than a source-of-truth module.[^1]
I would also keep attendance, grades, timetable, finance, notices, and documents as upstream-authoritative modules and only reference their published or approved outputs here, because the PRD explicitly says the portal must not maintain separate copies of core school data.[^1]

## 1. Entity list

These are the production-ready entities I would use for Module M7_9.[^1]

- `portal_users` — portal identity for parent or student account, because the PRD distinguishes portal identity and lifecycle from upstream user records.[^1]
- `portal_profiles` — language, notification, and presentation preferences for each portal user.[^1]
- `portal_sessions` — tenant and school scoped session tracking with shared-device-safe controls, because logins, revocations, and session expiry must be auditable and stricter for school contexts.[^1]
- `portal_child_contexts` — parent child-switch state/history, because child-switch events must be auditable and must never permit access to unlinked students.[^1]
- `parent_student_links` — effective guardian-child visibility map consumed by portal, because parent access is allowed only through platform relationship rules and one parent may have multiple linked children.[^1]
- `portal_dashboards` — role-based dashboard definitions, because parent and student dashboard layouts are configurable and role-aware.[^1]
- `portal_widgets` — dashboard cards such as attendance summary, dues, notices, and report cards.[^1]
- `portal_feature_flags` — school-level enablement for student portal, fee visibility, notice reads, support, and similar features.[^1]
- `portal_access_policies` — fine-grained role visibility rules for sections, fields, retention, and sensitive content exposure.[^1]
- `portal_notices` — notices and circulars that are visible in the portal, because notices need publication status and visibility windows before appearing.[^1]
- `portal_notice_audiences` — grade, section, role, or individual audience scoping, because notices may be audience-scoped.[^1]
- `portal_notice_reads` — read tracking where enabled by policy.[^1]
- `portal_documents` — portal-eligible student or general documents with visibility lifecycle, version, and source references.[^1]
- `portal_download_logs` — audit trail for protected downloads, because sensitive downloads must be permission-checked and auditable.[^1]
- `portal_support_requests` — trackable parent/student support submissions, because support requests must create trackable records.[^1]
- `portal_support_messages` — threaded replies on support requests.[^1]
- `portal_form_submissions` — approved limited self-service forms, because portal-submitted forms must be explicitly approved, timestamped or versioned, and auditable.[^1]
- `portal_audit_logs` — login, child-switch, sensitive page access, payment initiation, unauthorized access attempts, and protected download trail.[^1]


## 2. Table definitions

This design separates access-control metadata from source-module content and keeps all portal-owned tables tenant-scoped, while upstream academic, attendance, timetable, and finance records are referenced but not duplicated.[^1]
I am also modeling parent child-switching explicitly instead of inferring it from sessions alone, because the PRD requires auditing every child-switch event and preventing cross-child leakage on shared devices.[^1]

```sql
CREATE EXTENSION IF NOT EXISTS pgcrypto;
CREATE EXTENSION IF NOT EXISTS citext;

CREATE TYPE portal_role_type AS ENUM ('parent', 'student');
CREATE TYPE portal_account_status AS ENUM ('invited', 'pending_activation', 'provisioned', 'active', 'suspended', 'reactivated', 'disabled');
CREATE TYPE portal_profile_status AS ENUM ('active', 'disabled');
CREATE TYPE portal_session_status AS ENUM ('active', 'revoked', 'expired');
CREATE TYPE portal_context_event_type AS ENUM ('selected', 'cleared', 'auto_selected');
CREATE TYPE portal_notice_status AS ENUM ('draft', 'scheduled', 'published', 'expired', 'archived');
CREATE TYPE audience_type AS ENUM ('role', 'grade', 'section', 'student', 'parent_user', 'portal_user', 'all_parents', 'all_students');
CREATE TYPE portal_document_status AS ENUM ('uploaded', 'verified', 'visible', 'replaced', 'archived', 'revoked');
CREATE TYPE support_request_status AS ENUM ('submitted', 'acknowledged', 'in_progress', 'waiting_for_user', 'reopened', 'resolved', 'closed');
CREATE TYPE support_priority AS ENUM ('low', 'medium', 'high', 'urgent');
CREATE TYPE support_sender_type AS ENUM ('portal_user', 'staff', 'system');
CREATE TYPE form_submission_status AS ENUM ('submitted', 'under_review', 'approved', 'rejected', 'withdrawn');
CREATE TYPE audit_event_type AS ENUM (
    'login_success', 'login_failed', 'logout', 'session_revoked',
    'child_context_selected', 'child_context_denied',
    'sensitive_page_viewed', 'payment_initiated',
    'notice_read', 'document_downloaded',
    'support_request_created', 'support_message_created',
    'form_submitted', 'unauthorized_access_attempt'
);

-- Assumes upstream tables exist:
-- tenants, schools, users, students, grade_levels, sections.
-- Parent/student relationship truth comes from SIS/RBAC; portal keeps an effective link table.

CREATE TABLE portal_users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id),
    school_id UUID NOT NULL REFERENCES schools(id),
    user_id UUID NOT NULL REFERENCES users(id),
    role_type portal_role_type NOT NULL,
    status portal_account_status NOT NULL,
    last_login_at TIMESTAMPTZ,
    disabled_at TIMESTAMPTZ,
    invited_at TIMESTAMPTZ,
    activated_at TIMESTAMPTZ,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID REFERENCES users(id),
    updated_by UUID REFERENCES users(id),
    CONSTRAINT uq_portal_user UNIQUE (tenant_id, school_id, user_id, role_type)
);

CREATE TABLE portal_profiles (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id),
    portal_user_id UUID NOT NULL UNIQUE REFERENCES portal_users(id) ON DELETE CASCADE,
    preferred_language VARCHAR(16) NOT NULL DEFAULT 'en',
    notification_preferences_json JSONB NOT NULL DEFAULT '{}'::jsonb,
    timezone VARCHAR(64),
    profile_status portal_profile_status NOT NULL DEFAULT 'active',
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID REFERENCES users(id),
    updated_by UUID REFERENCES users(id)
);

CREATE TABLE portal_sessions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id),
    school_id UUID NOT NULL REFERENCES schools(id),
    portal_user_id UUID NOT NULL REFERENCES portal_users(id) ON DELETE CASCADE,
    refresh_token_hash TEXT NOT NULL,
    device_info JSONB,
    ip_address INET,
    user_agent TEXT,
    started_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    expires_at TIMESTAMPTZ NOT NULL,
    last_seen_at TIMESTAMPTZ,
    revoked_at TIMESTAMPTZ,
    revocation_reason TEXT,
    status portal_session_status NOT NULL DEFAULT 'active',
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID REFERENCES users(id),
    updated_by UUID REFERENCES users(id)
);

CREATE TABLE parent_student_links (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id),
    school_id UUID NOT NULL REFERENCES schools(id),
    parent_user_id UUID NOT NULL REFERENCES users(id),
    student_id UUID NOT NULL REFERENCES students(id),
    relationship_type VARCHAR(50) NOT NULL,
    is_primary BOOLEAN NOT NULL DEFAULT FALSE,
    active BOOLEAN NOT NULL DEFAULT TRUE,
    effective_from DATE NOT NULL DEFAULT CURRENT_DATE,
    effective_to DATE,
    visibility_policy_json JSONB NOT NULL DEFAULT '{}'::jsonb,
    source_ref JSONB,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID REFERENCES users(id),
    updated_by UUID REFERENCES users(id),
    CONSTRAINT ck_parent_student_link_dates CHECK (effective_to IS NULL OR effective_to >= effective_from),
    CONSTRAINT uq_parent_student_link UNIQUE (tenant_id, school_id, parent_user_id, student_id, effective_from)
);

CREATE TABLE portal_child_contexts (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id),
    school_id UUID NOT NULL REFERENCES schools(id),
    portal_session_id UUID NOT NULL REFERENCES portal_sessions(id) ON DELETE CASCADE,
    portal_user_id UUID NOT NULL REFERENCES portal_users(id) ON DELETE CASCADE,
    student_id UUID NOT NULL REFERENCES students(id),
    event_type portal_context_event_type NOT NULL,
    was_authorized BOOLEAN NOT NULL DEFAULT TRUE,
    selected_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    ip_address INET,
    user_agent TEXT,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID REFERENCES users(id)
);

CREATE TABLE portal_dashboards (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id),
    school_id UUID NOT NULL REFERENCES schools(id),
    role_type portal_role_type NOT NULL,
    dashboard_code VARCHAR(60) NOT NULL,
    dashboard_name VARCHAR(120) NOT NULL,
    layout_json JSONB NOT NULL,
    active BOOLEAN NOT NULL DEFAULT TRUE,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID REFERENCES users(id),
    updated_by UUID REFERENCES users(id),
    CONSTRAINT uq_dashboard_code UNIQUE (tenant_id, school_id, role_type, dashboard_code)
);

CREATE TABLE portal_widgets (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id),
    dashboard_id UUID NOT NULL REFERENCES portal_dashboards(id) ON DELETE CASCADE,
    widget_code VARCHAR(60) NOT NULL,
    title VARCHAR(120) NOT NULL,
    display_order INTEGER NOT NULL,
    widget_config_json JSONB NOT NULL DEFAULT '{}'::jsonb,
    active BOOLEAN NOT NULL DEFAULT TRUE,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID REFERENCES users(id),
    updated_by UUID REFERENCES users(id),
    CONSTRAINT uq_widget_code UNIQUE (tenant_id, dashboard_id, widget_code)
);

CREATE TABLE portal_feature_flags (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id),
    school_id UUID NOT NULL REFERENCES schools(id),
    feature_code VARCHAR(80) NOT NULL,
    enabled BOOLEAN NOT NULL DEFAULT FALSE,
    config_json JSONB NOT NULL DEFAULT '{}'::jsonb,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID REFERENCES users(id),
    updated_by UUID REFERENCES users(id),
    CONSTRAINT uq_feature_flag UNIQUE (tenant_id, school_id, feature_code)
);

CREATE TABLE portal_access_policies (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id),
    school_id UUID NOT NULL REFERENCES schools(id),
    policy_code VARCHAR(80) NOT NULL,
    role_type portal_role_type NOT NULL,
    config_json JSONB NOT NULL,
    active BOOLEAN NOT NULL DEFAULT TRUE,
    effective_from TIMESTAMPTZ NOT NULL DEFAULT now(),
    effective_to TIMESTAMPTZ,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID REFERENCES users(id),
    updated_by UUID REFERENCES users(id),
    CONSTRAINT ck_policy_dates CHECK (effective_to IS NULL OR effective_to >= effective_from),
    CONSTRAINT uq_access_policy UNIQUE (tenant_id, school_id, policy_code, role_type, effective_from)
);

CREATE TABLE portal_notices (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id),
    school_id UUID NOT NULL REFERENCES schools(id),
    title VARCHAR(255) NOT NULL,
    body TEXT NOT NULL,
    source_module VARCHAR(80),
    source_ref_id UUID,
    published_at TIMESTAMPTZ,
    expires_at TIMESTAMPTZ,
    status portal_notice_status NOT NULL DEFAULT 'draft',
    read_tracking_enabled BOOLEAN NOT NULL DEFAULT FALSE,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID REFERENCES users(id),
    updated_by UUID REFERENCES users(id),
    CONSTRAINT ck_notice_publish_window CHECK (expires_at IS NULL OR published_at IS NULL OR expires_at >= published_at)
);

CREATE TABLE portal_notice_audiences (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id),
    notice_id UUID NOT NULL REFERENCES portal_notices(id) ON DELETE CASCADE,
    audience_type audience_type NOT NULL,
    audience_ref_id UUID,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID REFERENCES users(id)
);

CREATE TABLE portal_notice_reads (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id),
    notice_id UUID NOT NULL REFERENCES portal_notices(id) ON DELETE CASCADE,
    portal_user_id UUID NOT NULL REFERENCES portal_users(id) ON DELETE CASCADE,
    student_context_id UUID REFERENCES students(id),
    read_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    ip_address INET,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID REFERENCES users(id),
    CONSTRAINT uq_notice_read UNIQUE (tenant_id, notice_id, portal_user_id, COALESCE(student_context_id, '00000000-0000-0000-0000-000000000000'::uuid))
);

CREATE TABLE portal_documents (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id),
    school_id UUID NOT NULL REFERENCES schools(id),
    student_id UUID REFERENCES students(id),
    document_type VARCHAR(80) NOT NULL,
    title VARCHAR(255) NOT NULL,
    source_module VARCHAR(80) NOT NULL,
    source_ref_id UUID NOT NULL,
    file_storage_key TEXT NOT NULL,
    mime_type VARCHAR(120),
    file_size_bytes BIGINT,
    version_no INTEGER NOT NULL DEFAULT 1,
    visibility_status portal_document_status NOT NULL DEFAULT 'uploaded',
    visible_from TIMESTAMPTZ,
    visible_until TIMESTAMPTZ,
    sensitivity_level VARCHAR(30) NOT NULL DEFAULT 'standard',
    metadata_json JSONB NOT NULL DEFAULT '{}'::jsonb,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID REFERENCES users(id),
    updated_by UUID REFERENCES users(id),
    CONSTRAINT ck_document_version CHECK (version_no > 0),
    CONSTRAINT ck_document_window CHECK (visible_until IS NULL OR visible_from IS NULL OR visible_until >= visible_from)
);

CREATE TABLE portal_download_logs (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id),
    portal_user_id UUID NOT NULL REFERENCES portal_users(id),
    document_id UUID NOT NULL REFERENCES portal_documents(id),
    student_context_id UUID REFERENCES students(id),
    downloaded_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    ip_address INET,
    user_agent TEXT,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID REFERENCES users(id)
);

CREATE TABLE portal_support_requests (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id),
    school_id UUID NOT NULL REFERENCES schools(id),
    portal_user_id UUID NOT NULL REFERENCES portal_users(id),
    student_id UUID REFERENCES students(id),
    category VARCHAR(80) NOT NULL,
    subject VARCHAR(255) NOT NULL,
    description TEXT NOT NULL,
    status support_request_status NOT NULL DEFAULT 'submitted',
    priority support_priority NOT NULL DEFAULT 'medium',
    assigned_to UUID REFERENCES users(id),
    acknowledged_at TIMESTAMPTZ,
    resolved_at TIMESTAMPTZ,
    closed_at TIMESTAMPTZ,
    sla_due_at TIMESTAMPTZ,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID REFERENCES users(id),
    updated_by UUID REFERENCES users(id)
);

CREATE TABLE portal_support_messages (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id),
    support_request_id UUID NOT NULL REFERENCES portal_support_requests(id) ON DELETE CASCADE,
    sender_type support_sender_type NOT NULL,
    sender_user_id UUID REFERENCES users(id),
    portal_user_id UUID REFERENCES portal_users(id),
    message_body TEXT NOT NULL,
    attachment_meta JSONB,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID REFERENCES users(id)
);

CREATE TABLE portal_form_submissions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id),
    school_id UUID NOT NULL REFERENCES schools(id),
    portal_user_id UUID NOT NULL REFERENCES portal_users(id),
    student_id UUID REFERENCES students(id),
    form_code VARCHAR(80) NOT NULL,
    version_no INTEGER NOT NULL DEFAULT 1,
    payload_json JSONB NOT NULL,
    status form_submission_status NOT NULL DEFAULT 'submitted',
    submitted_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    reviewed_by UUID REFERENCES users(id),
    reviewed_at TIMESTAMPTZ,
    decision_notes TEXT,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID REFERENCES users(id),
    updated_by UUID REFERENCES users(id),
    CONSTRAINT ck_form_version CHECK (version_no > 0)
);

CREATE TABLE portal_audit_logs (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id),
    school_id UUID NOT NULL REFERENCES schools(id),
    portal_user_id UUID REFERENCES portal_users(id),
    portal_session_id UUID REFERENCES portal_sessions(id),
    event_type audit_event_type NOT NULL,
    event_ref VARCHAR(120),
    student_context_id UUID REFERENCES students(id),
    was_authorized BOOLEAN,
    ip_address INET,
    user_agent TEXT,
    metadata_json JSONB NOT NULL DEFAULT '{}'::jsonb,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);
```


## 3. Constraints

The main control points here are tenant isolation, parent-child relationship enforcement, published-only data visibility, request-time permission checks for downloads, and role-specific behavior for parents versus students.[^1]
The PRD also requires that relationship changes immediately affect visibility, draft data never appears in the portal, and unauthorized access attempts be logged and rate-limited.[^1]

- `portal_users` must be unique by `(tenant_id, school_id, user_id, role_type)`, because one platform user could theoretically participate in different portal roles or school contexts.[^1]
- `parent_student_links.active = true` plus effective-date validity should drive parent visibility, because parent access is allowed only through active guardian linkage.[^1]
- Add a trigger on `portal_child_contexts` to verify the `student_id` belongs to an active `parent_student_links` row for the parent backing the session, because child-switching must never expose an unlinked student.[^1]
- Add a trigger on `portal_notice_reads` so read events are accepted only when the notice is currently published and visible to that user under notice audience rules.[^1]
- Add a trigger on `portal_download_logs` to ensure the corresponding `portal_documents.visibility_status = 'visible'` and the user is permitted at request time, because revoked or archived documents must not be downloadable unless explicitly allowed.[^1]
- Add a trigger on `portal_support_requests` and `portal_form_submissions` to block disallowed student-linked submissions when school policy disables those workflows.[^1]
- Do not soft delete `portal_sessions`, `portal_child_contexts`, `portal_download_logs`, `portal_support_messages`, or `portal_audit_logs`, because these are audit/security records.[^1]
- Soft delete is justified only for setup/config entities if you later add `deleted_at` to `portal_dashboards`, `portal_widgets`, `portal_feature_flags`, or `portal_access_policies`, since those are configuration artifacts rather than transactional history.[^1]


## 4. Indexes

The most likely query patterns are dashboard load by current portal user, list linked children, list visible notices, list visible documents, list own support requests, active-session management, and audit lookup for security review.[^1]

```sql
CREATE INDEX idx_portal_users_user_role
ON portal_users (tenant_id, school_id, user_id, role_type, status);

CREATE INDEX idx_portal_sessions_user_status
ON portal_sessions (tenant_id, portal_user_id, status, expires_at DESC);

CREATE INDEX idx_parent_student_links_parent
ON parent_student_links (tenant_id, school_id, parent_user_id, active, effective_from DESC);

CREATE INDEX idx_parent_student_links_student
ON parent_student_links (tenant_id, school_id, student_id, active);

CREATE INDEX idx_portal_child_contexts_session_time
ON portal_child_contexts (tenant_id, portal_session_id, selected_at DESC);

CREATE INDEX idx_portal_dashboards_role
ON portal_dashboards (tenant_id, school_id, role_type, active);

CREATE INDEX idx_portal_widgets_dashboard_order
ON portal_widgets (tenant_id, dashboard_id, active, display_order);

CREATE INDEX idx_feature_flags_school
ON portal_feature_flags (tenant_id, school_id, feature_code, enabled);

CREATE INDEX idx_access_policies_role
ON portal_access_policies (tenant_id, school_id, role_type, policy_code, active);

CREATE INDEX idx_portal_notices_status_window
ON portal_notices (tenant_id, school_id, status, published_at DESC, expires_at);

CREATE INDEX idx_portal_notice_audiences_notice
ON portal_notice_audiences (tenant_id, notice_id, audience_type, audience_ref_id);

CREATE INDEX idx_portal_notice_reads_user
ON portal_notice_reads (tenant_id, portal_user_id, read_at DESC);

CREATE INDEX idx_portal_documents_student_status
ON portal_documents (tenant_id, school_id, student_id, visibility_status, visible_from DESC);

CREATE INDEX idx_portal_documents_source
ON portal_documents (tenant_id, source_module, source_ref_id, version_no DESC);

CREATE INDEX idx_download_logs_user_time
ON portal_download_logs (tenant_id, portal_user_id, downloaded_at DESC);

CREATE INDEX idx_support_requests_user_status
ON portal_support_requests (tenant_id, portal_user_id, status, created_at DESC);

CREATE INDEX idx_support_requests_school_queue
ON portal_support_requests (tenant_id, school_id, status, priority, sla_due_at);

CREATE INDEX idx_support_messages_request_time
ON portal_support_messages (tenant_id, support_request_id, created_at);

CREATE INDEX idx_form_submissions_user
ON portal_form_submissions (tenant_id, portal_user_id, form_code, submitted_at DESC);

CREATE INDEX idx_portal_audit_logs_user_time
ON portal_audit_logs (tenant_id, portal_user_id, created_at DESC);

CREATE INDEX idx_portal_audit_logs_event_time
ON portal_audit_logs (tenant_id, school_id, event_type, created_at DESC);
```


## 5. Sample records

These records reflect the PRD’s core flows: parent login, linked children, role-based dashboard, visible notice, visible report-card document, and support ticket creation.[^1]

```sql
INSERT INTO portal_users (
    id, tenant_id, school_id, user_id, role_type, status, activated_at, created_at, updated_at
) VALUES (
    '90000000-0000-0000-0000-000000000001',
    '10000000-0000-0000-0000-000000000001',
    '20000000-0000-0000-0000-000000000001',
    '30000000-0000-0000-0000-000000000010',
    'parent',
    'active',
    now(),
    now(),
    now()
);

INSERT INTO parent_student_links (
    id, tenant_id, school_id, parent_user_id, student_id, relationship_type, is_primary, active
) VALUES (
    '90000000-0000-0000-0000-000000000011',
    '10000000-0000-0000-0000-000000000001',
    '20000000-0000-0000-0000-000000000001',
    '30000000-0000-0000-0000-000000000010',
    '40000000-0000-0000-0000-000000000101',
    'mother',
    true,
    true
);

INSERT INTO portal_dashboards (
    id, tenant_id, school_id, role_type, dashboard_code, dashboard_name, layout_json, active
) VALUES (
    '90000000-0000-0000-0000-000000000021',
    '10000000-0000-0000-0000-000000000001',
    '20000000-0000-0000-0000-000000000001',
    'parent',
    'parent_default',
    'Parent Dashboard',
    '{"layout":"2-column","widgets":["attendance_summary","fees_summary","notices","report_cards"]}'::jsonb,
    true
);

INSERT INTO portal_notices (
    id, tenant_id, school_id, title, body, published_at, expires_at, status, read_tracking_enabled
) VALUES (
    '90000000-0000-0000-0000-000000000031',
    '10000000-0000-0000-0000-000000000001',
    '20000000-0000-0000-0000-000000000001',
    'Term 1 PTM Schedule',
    'Parent-teacher meetings begin next Monday.',
    now(),
    now() + interval '14 days',
    'published',
    true
);

INSERT INTO portal_documents (
    id, tenant_id, school_id, student_id, document_type, title, source_module, source_ref_id,
    file_storage_key, mime_type, version_no, visibility_status, visible_from
) VALUES (
    '90000000-0000-0000-0000-000000000041',
    '10000000-0000-0000-0000-000000000001',
    '20000000-0000-0000-0000-000000000001',
    '40000000-0000-0000-0000-000000000101',
    'report_card',
    'Term 1 Report Card',
    'gradebook',
    '50000000-0000-0000-0000-000000000001',
    'portal/report-cards/term1-stu101-v1.pdf',
    'application/pdf',
    1,
    'visible',
    now()
);

INSERT INTO portal_support_requests (
    id, tenant_id, school_id, portal_user_id, student_id, category, subject, description, status, priority
) VALUES (
    '90000000-0000-0000-0000-000000000051',
    '10000000-0000-0000-0000-000000000001',
    '20000000-0000-0000-0000-000000000001',
    '90000000-0000-0000-0000-000000000001',
    '40000000-0000-0000-0000-000000000101',
    'fees',
    'Receipt not visible',
    'Payment succeeded but receipt is not showing in portal.',
    'submitted',
    'medium'
);
```


## 6. Query patterns

The portal should be read-fast and authorization-safe, so most queries should resolve through current session, role type, active child context, and request-time relationship checks.[^1]

1. Parent linked-children list for dashboard switcher.[^1]
```sql
SELECT psl.student_id, s.admission_no, s.first_name, s.last_name, s.current_grade_level_id, s.current_section_id
FROM parent_student_links psl
JOIN students s ON s.id = psl.student_id
WHERE psl.tenant_id = $1
  AND psl.school_id = $2
  AND psl.parent_user_id = $3
  AND psl.active = TRUE
  AND (psl.effective_to IS NULL OR psl.effective_to >= CURRENT_DATE)
ORDER BY psl.is_primary DESC, s.first_name;
```

2. Parent dashboard summary using published-only upstream outputs.[^1]
```sql
SELECT
  att.present_days,
  att.absent_days,
  fin.outstanding_amount,
  grd.latest_report_card_published_at,
  ntc.unread_notice_count
FROM portal_parent_dashboard_vw
WHERE tenant_id = $1
  AND school_id = $2
  AND parent_user_id = $3
  AND student_id = $4;
```

3. Visible notices for current portal user and student context.[^1]
```sql
SELECT DISTINCT n.id, n.title, n.published_at, n.expires_at, nr.read_at
FROM portal_notices n
JOIN portal_notice_audiences a
  ON a.notice_id = n.id AND a.tenant_id = n.tenant_id
LEFT JOIN portal_notice_reads nr
  ON nr.notice_id = n.id
 AND nr.portal_user_id = $3
 AND nr.tenant_id = n.tenant_id
WHERE n.tenant_id = $1
  AND n.school_id = $2
  AND n.status = 'published'
  AND (n.expires_at IS NULL OR n.expires_at > now())
  AND (
       (a.audience_type = 'all_parents' AND $4 = 'parent')
    OR (a.audience_type = 'all_students' AND $4 = 'student')
    OR (a.audience_type = 'student' AND a.audience_ref_id = $5)
    OR (a.audience_type = 'portal_user' AND a.audience_ref_id = $3)
    OR (a.audience_type = 'grade' AND a.audience_ref_id = $6)
    OR (a.audience_type = 'section' AND a.audience_ref_id = $7)
  )
ORDER BY n.published_at DESC;
```

4. Visible documents for current parent child context.[^1]
```sql
SELECT d.id, d.title, d.document_type, d.version_no, d.visible_from
FROM portal_documents d
WHERE d.tenant_id = $1
  AND d.school_id = $2
  AND d.visibility_status = 'visible'
  AND (d.visible_until IS NULL OR d.visible_until > now())
  AND d.student_id = $3
ORDER BY d.document_type, d.version_no DESC, d.created_at DESC;
```

5. Portal audit review for suspicious child-context denials.[^1]
```sql
SELECT created_at, portal_user_id, event_type, event_ref, student_context_id, ip_address, metadata_json
FROM portal_audit_logs
WHERE tenant_id = $1
  AND school_id = $2
  AND event_type IN ('child_context_denied', 'unauthorized_access_attempt')
ORDER BY created_at DESC
LIMIT 200;
```


## 7. Migration sequence

The safest build order is access first, then visibility/config, then content wrappers, then support flows, then audit and RLS, because the PRD makes authorization and relationship-based visibility the non-negotiable foundation.[^1]

1. Create enums and foundational tables: `portal_users`, `portal_profiles`, `portal_sessions`, and `parent_student_links`.[^1]
2. Add `portal_child_contexts` plus authorization triggers for linked-student selection.[^1]
3. Create configuration tables: `portal_dashboards`, `portal_widgets`, `portal_feature_flags`, and `portal_access_policies`.[^1]
4. Create content-facing tables: `portal_notices`, `portal_notice_audiences`, `portal_notice_reads`, `portal_documents`, and `portal_download_logs`.[^1]
5. Create action tables: `portal_support_requests`, `portal_support_messages`, and `portal_form_submissions`.[^1]
6. Create `portal_audit_logs`, then add immutability guards, request-time permission triggers, and RLS policies for all tenant-owned tables.[^1]
7. Create read-only views such as `portal_parent_dashboard_vw` and `portal_student_dashboard_vw` that join only published/approved upstream data from attendance, gradebook, timetable, finance, and communication modules.[^1]

## 8. Security-sensitive columns

The most sensitive data in this module is session material, relationship visibility, download paths, IP/user-agent telemetry, and support payloads that may include family or student details.[^1]

- `portal_sessions.refresh_token_hash` — secret session credential material; store hash only and never expose it.[^1]
- `portal_sessions.ip_address`, `portal_sessions.user_agent`, `portal_child_contexts.ip_address`, and `portal_audit_logs.ip_address` — privacy-sensitive security telemetry that should be tightly role-protected.[^1]
- `parent_student_links.visibility_policy_json` — sensitive relationship-access policy data that controls who can see which child records.[^1]
- `portal_documents.file_storage_key` — protected storage locator for sensitive files such as report cards and student documents; do not expose direct storage paths publicly.[^1]
- `portal_documents.metadata_json` — may include document provenance or internal details and should be filtered before client response.[^1]
- `portal_support_requests.description` and `portal_support_messages.message_body` — may contain PII, fee issues, or student-sensitive descriptions.[^1]
- `portal_form_submissions.payload_json` — potentially high-risk PII depending on approved form type; encrypt or field-protect where necessary.[^1]
- `portal_audit_logs.metadata_json` — may contain denied-route, file, payment-initiation, or relationship context details useful for security but sensitive for general admins.[^1]

Next I can continue with M7_10, or I can compile M7_1, M7_7, M7_8, and M7_9 into one normalized schema pack.

<div align="center">⁂</div>

[^1]: PRD_M7_9.txt

