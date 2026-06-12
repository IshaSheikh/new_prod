<img src="https://r2cdn.perplexity.ai/pplx-full-logo-primary-dark%402x.png" style="height:64px;margin-right:32px"/>

# continue with M7_10

For M7_10, the right design is a **message orchestration and audit schema** with notice composition, audience snapshotting, approval workflow, outbound channel dispatch, provider callback tracking, recipient preferences, automation triggers, and immutable delivery history, because the PRD defines Communication Hub as a cross-cutting multi-channel service whose core value is searchable, auditable, tenant-safe communication delivery.[^1]
I would also keep identity and audience resolution dependent on upstream relationship data rather than duplicating a parallel recipient model, because the PRD explicitly says this module must consume guardian, student, class, section, and role relationships from source modules.[^1]

## 1. Entity list

These are the production-ready entities I would use for Module M7_10.[^1]

- `communications` — master record for notice, alert, reminder, or operational message, because one communication can be authored, approved, scheduled, sent, tracked, and searched centrally.[^1]
- `communication_audiences` — declarative targeting rows such as school-wide, grade, section, role, or individual.[^1]
- `communication_audience_snapshots` — persisted resolved-recipient snapshot at send time, because the PRD requires audience snapshots for auditability.[^1]
- `communication_templates` — reusable templates with variable schema, because routine messaging should be templated and placeholders validated before dispatch.[^1]
- `communication_template_versions` — immutable template versions, because scheduled messages must preserve the rendered or versioned snapshot even if templates later change.[^1]
- `communication_attachments` — attachments or source-linked files, because notices may include files or links with platform restrictions.[^1]
- `communication_approvals` — approval workflow for sensitive or high-impact communication.[^1]
- `communication_channels` — tenant/school channel configuration for in-app, email, SMS, WhatsApp, and push.[^1]
- `communication_channel_policies` — per-category routing, fallback, and override rules, because channel eligibility and fallback may be configured by message type.[^1]
- `notification_preferences` — per-user and per-category preferences, because the PRD allows configurable user and policy-based channel preferences.[^1]
- `outbound_messages` — per-recipient, per-channel dispatch unit, because one communication may fan out across multiple channels and delivery states.[^1]
- `outbound_message_attempts` — retry attempts separated from final delivery state, because retries must be tracked without producing duplicate-success ambiguity.[^1]
- `communication_delivery_logs` — normalized provider callback and status history, because delivery callbacks must be idempotent and searchable.[^1]
- `communication_events` — lifecycle audit stream for create, edit, schedule, approve, send, retry, fail, cancel, archive, and read.[^1]
- `automation_rules` — event-triggered communication rules for attendance submission, fee due, payment success, timetable change, and result publication.[^1]
- `recipient_resolution_jobs` — asynchronous resolution and dispatch preparation, because large schools need queue-based fan-out and auditable resolution.[^1]
- `communication_search_index` — denormalized search table or materialized structure for fast history search by sender, audience, channel, date, and status.[^1]
- `recipient_inbox_items` — per-user in-app inbox visibility/read/acknowledgment record for parents and students, because recipients need portal-visible history and read state.[^1]


## 2. Table definitions

This schema treats communication authoring and delivery as separate concerns: a communication is the authored business object, while outbound messages are the channel-and-recipient execution units.[^1]
That separation is important because one communication may target many recipients, multiple channels, multiple retries, and partially successful outcomes across channels, which the PRD explicitly supports.[^1]

```sql
CREATE EXTENSION IF NOT EXISTS pgcrypto;
CREATE EXTENSION IF NOT EXISTS citext;

CREATE TYPE communication_category AS ENUM (
    'general_notice',
    'attendance_alert',
    'fee_reminder',
    'payment_confirmation',
    'timetable_change',
    'result_publication',
    'operational_update',
    'safety_alert',
    'event_notice',
    'other'
);

CREATE TYPE communication_priority AS ENUM ('low', 'normal', 'high', 'urgent');

CREATE TYPE communication_status AS ENUM (
    'draft',
    'pending_approval',
    'approved',
    'rejected',
    'scheduled',
    'queued',
    'sending',
    'sent',
    'partially_delivered',
    'delivered',
    'read',
    'cancelled',
    'archived'
);

CREATE TYPE audience_type AS ENUM (
    'school',
    'grade',
    'section',
    'role',
    'student',
    'parent',
    'teacher',
    'staff',
    'individual_user'
);

CREATE TYPE template_status AS ENUM ('draft', 'active', 'inactive', 'archived');

CREATE TYPE approval_status AS ENUM ('pending', 'approved', 'rejected', 'cancelled');

CREATE TYPE channel_code AS ENUM ('in_app', 'email', 'sms', 'whatsapp', 'push');

CREATE TYPE channel_config_status AS ENUM ('not_configured', 'configured', 'error', 'disabled');

CREATE TYPE preference_override_policy AS ENUM (
    'none',
    'mandatory_override_allowed',
    'mandatory_override_applied'
);

CREATE TYPE outbound_status AS ENUM (
    'queued',
    'sending',
    'sent',
    'delivered',
    'read',
    'failed',
    'retrying',
    'cancelled',
    'skipped'
);

CREATE TYPE delivery_event_type AS ENUM (
    'queued',
    'provider_accepted',
    'sent',
    'delivered',
    'read',
    'failed',
    'bounced',
    'rejected',
    'expired',
    'callback_ignored_duplicate'
);

CREATE TYPE communication_event_type AS ENUM (
    'created',
    'edited',
    'approval_requested',
    'approved',
    'rejected',
    'scheduled',
    'queued',
    'cancelled',
    'send_started',
    'send_completed',
    'retry_triggered',
    'delivery_failed',
    'archived',
    'read'
);

CREATE TYPE automation_trigger_type AS ENUM (
    'attendance_submitted',
    'fee_due',
    'fee_overdue',
    'payment_success',
    'result_published',
    'timetable_changed',
    'manual'
);

CREATE TYPE automation_status AS ENUM ('draft', 'active', 'inactive', 'archived');

CREATE TYPE resolution_job_status AS ENUM (
    'queued',
    'resolving',
    'resolved',
    'failed',
    'dispatching',
    'completed'
);

CREATE TYPE inbox_item_status AS ENUM ('visible', 'read', 'acknowledged', 'archived');

-- Assumes upstream tables exist:
-- tenants, schools, users, students, grade_levels, sections, portal_users

CREATE TABLE communications (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id),
    school_id UUID NOT NULL REFERENCES schools(id),
    template_id UUID NULL,
    template_version_id UUID NULL,
    title VARCHAR(255) NOT NULL,
    subject_text TEXT,
    body_text TEXT NOT NULL,
    normalized_body_text TEXT NOT NULL,
    category communication_category NOT NULL,
    priority communication_priority NOT NULL DEFAULT 'normal',
    status communication_status NOT NULL DEFAULT 'draft',
    requires_approval BOOLEAN NOT NULL DEFAULT FALSE,
    requires_delivery_tracking BOOLEAN NOT NULL DEFAULT FALSE,
    allow_preference_override BOOLEAN NOT NULL DEFAULT FALSE,
    approval_submitted_at TIMESTAMPTZ,
    approved_at TIMESTAMPTZ,
    rejected_at TIMESTAMPTZ,
    scheduled_at TIMESTAMPTZ,
    queued_at TIMESTAMPTZ,
    sent_at TIMESTAMPTZ,
    archived_at TIMESTAMPTZ,
    source_module VARCHAR(80),
    source_ref_id UUID,
    rendered_payload_snapshot JSONB,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID NOT NULL REFERENCES users(id),
    updated_by UUID REFERENCES users(id),
    approved_by UUID REFERENCES users(id),
    rejected_by UUID REFERENCES users(id)
);

CREATE TABLE communication_audiences (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id),
    communication_id UUID NOT NULL REFERENCES communications(id) ON DELETE CASCADE,
    audience_type audience_type NOT NULL,
    audience_ref_id UUID,
    audience_ref_code VARCHAR(100),
    recipient_count INTEGER,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID NOT NULL REFERENCES users(id)
);

CREATE TABLE communication_audience_snapshots (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id),
    communication_id UUID NOT NULL REFERENCES communications(id) ON DELETE CASCADE,
    snapshot_version INTEGER NOT NULL DEFAULT 1,
    resolved_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    recipient_count INTEGER NOT NULL DEFAULT 0,
    duplicate_count INTEGER NOT NULL DEFAULT 0,
    resolution_rules_json JSONB NOT NULL DEFAULT '{}'::jsonb,
    resolved_snapshot_json JSONB NOT NULL,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID NOT NULL REFERENCES users(id),
    CONSTRAINT uq_comm_snapshot_version UNIQUE (tenant_id, communication_id, snapshot_version)
);

CREATE TABLE communication_templates (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id),
    school_id UUID NOT NULL REFERENCES schools(id),
    template_name VARCHAR(150) NOT NULL,
    template_code VARCHAR(80) NOT NULL,
    category communication_category NOT NULL,
    default_priority communication_priority NOT NULL DEFAULT 'normal',
    status template_status NOT NULL DEFAULT 'draft',
    active_version_no INTEGER,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID NOT NULL REFERENCES users(id),
    updated_by UUID REFERENCES users(id),
    CONSTRAINT uq_template_code UNIQUE (tenant_id, school_id, template_code)
);

CREATE TABLE communication_template_versions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id),
    template_id UUID NOT NULL REFERENCES communication_templates(id) ON DELETE CASCADE,
    version_no INTEGER NOT NULL,
    channel_code channel_code NOT NULL,
    subject_template TEXT,
    body_template TEXT NOT NULL,
    variable_schema JSONB NOT NULL DEFAULT '{}'::jsonb,
    render_rules_json JSONB NOT NULL DEFAULT '{}'::jsonb,
    is_active BOOLEAN NOT NULL DEFAULT FALSE,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID NOT NULL REFERENCES users(id),
    CONSTRAINT uq_template_version UNIQUE (tenant_id, template_id, version_no, channel_code)
);

ALTER TABLE communications
    ADD CONSTRAINT fk_communications_template
    FOREIGN KEY (template_id) REFERENCES communication_templates(id);

ALTER TABLE communications
    ADD CONSTRAINT fk_communications_template_version
    FOREIGN KEY (template_version_id) REFERENCES communication_template_versions(id);

CREATE TABLE communication_attachments (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id),
    communication_id UUID NOT NULL REFERENCES communications(id) ON DELETE CASCADE,
    file_name VARCHAR(255) NOT NULL,
    file_storage_key TEXT NOT NULL,
    mime_type VARCHAR(120) NOT NULL,
    file_size_bytes BIGINT NOT NULL,
    source_module VARCHAR(80),
    source_ref_id UUID,
    checksum_sha256 CHAR(64),
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID NOT NULL REFERENCES users(id),
    CONSTRAINT ck_attachment_size CHECK (file_size_bytes > 0)
);

CREATE TABLE communication_approvals (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id),
    communication_id UUID NOT NULL REFERENCES communications(id) ON DELETE CASCADE,
    status approval_status NOT NULL DEFAULT 'pending',
    requested_by UUID NOT NULL REFERENCES users(id),
    reviewed_by UUID REFERENCES users(id),
    review_reason TEXT,
    requested_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    reviewed_at TIMESTAMPTZ,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    CONSTRAINT uq_communication_approval UNIQUE (tenant_id, communication_id)
);

CREATE TABLE communication_channels (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id),
    school_id UUID NOT NULL REFERENCES schools(id),
    channel_code channel_code NOT NULL,
    provider_name VARCHAR(100),
    enabled BOOLEAN NOT NULL DEFAULT FALSE,
    config_status channel_config_status NOT NULL DEFAULT 'not_configured',
    priority_order INTEGER NOT NULL DEFAULT 1,
    config_json JSONB NOT NULL DEFAULT '{}'::jsonb,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID REFERENCES users(id),
    updated_by UUID REFERENCES users(id),
    CONSTRAINT uq_channel_per_school UNIQUE (tenant_id, school_id, channel_code)
);

CREATE TABLE communication_channel_policies (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id),
    school_id UUID NOT NULL REFERENCES schools(id),
    category communication_category NOT NULL,
    primary_channel channel_code NOT NULL,
    fallback_channels channel_code[] NOT NULL DEFAULT '{}',
    require_delivery_tracking BOOLEAN NOT NULL DEFAULT FALSE,
    mandatory_override_allowed BOOLEAN NOT NULL DEFAULT FALSE,
    deduplicate_by_contact BOOLEAN NOT NULL DEFAULT TRUE,
    active BOOLEAN NOT NULL DEFAULT TRUE,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID REFERENCES users(id),
    updated_by UUID REFERENCES users(id),
    CONSTRAINT uq_channel_policy UNIQUE (tenant_id, school_id, category)
);

CREATE TABLE notification_preferences (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id),
    user_id UUID NOT NULL REFERENCES users(id),
    channel channel_code NOT NULL,
    message_category communication_category NOT NULL,
    enabled BOOLEAN NOT NULL DEFAULT TRUE,
    override_policy preference_override_policy NOT NULL DEFAULT 'none',
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_by UUID REFERENCES users(id),
    CONSTRAINT uq_notification_preference UNIQUE (tenant_id, user_id, channel, message_category)
);

CREATE TABLE outbound_messages (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id),
    communication_id UUID REFERENCES communications(id) ON DELETE SET NULL,
    template_id UUID REFERENCES communication_templates(id),
    recipient_user_id UUID REFERENCES users(id),
    recipient_student_id UUID REFERENCES students(id),
    recipient_contact VARCHAR(255),
    channel channel_code NOT NULL,
    priority communication_priority NOT NULL,
    status outbound_status NOT NULL DEFAULT 'queued',
    provider_ref VARCHAR(255),
    failure_reason TEXT,
    eligibility_issue TEXT,
    payload_snapshot JSONB NOT NULL,
    queued_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    sent_at TIMESTAMPTZ,
    delivered_at TIMESTAMPTZ,
    read_at TIMESTAMPTZ,
    retry_count INTEGER NOT NULL DEFAULT 0,
    last_attempt_at TIMESTAMPTZ,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID REFERENCES users(id)
);

CREATE TABLE outbound_message_attempts (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id),
    outbound_message_id UUID NOT NULL REFERENCES outbound_messages(id) ON DELETE CASCADE,
    attempt_no INTEGER NOT NULL,
    provider_name VARCHAR(100),
    request_payload_json JSONB,
    response_payload_json JSONB,
    attempt_status outbound_status NOT NULL,
    failure_reason TEXT,
    attempted_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    CONSTRAINT uq_outbound_attempt UNIQUE (tenant_id, outbound_message_id, attempt_no)
);

CREATE TABLE communication_delivery_logs (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id),
    outbound_message_id UUID NOT NULL REFERENCES outbound_messages(id) ON DELETE CASCADE,
    event_type delivery_event_type NOT NULL,
    event_time TIMESTAMPTZ NOT NULL,
    provider_event_id VARCHAR(255),
    provider_payload JSONB,
    normalized_status outbound_status NOT NULL,
    callback_hash CHAR(64),
    created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE TABLE communication_events (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id),
    communication_id UUID REFERENCES communications(id) ON DELETE CASCADE,
    outbound_message_id UUID REFERENCES outbound_messages(id) ON DELETE CASCADE,
    event_type communication_event_type NOT NULL,
    actor_user_id UUID REFERENCES users(id),
    event_ref VARCHAR(120),
    metadata_json JSONB NOT NULL DEFAULT '{}'::jsonb,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE TABLE automation_rules (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id),
    school_id UUID NOT NULL REFERENCES schools(id),
    rule_name VARCHAR(150) NOT NULL,
    trigger_type automation_trigger_type NOT NULL,
    category communication_category NOT NULL,
    template_id UUID REFERENCES communication_templates(id),
    channel_policy_id UUID REFERENCES communication_channel_policies(id),
    condition_json JSONB NOT NULL DEFAULT '{}'::jsonb,
    active BOOLEAN NOT NULL DEFAULT FALSE,
    status automation_status NOT NULL DEFAULT 'draft',
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID NOT NULL REFERENCES users(id),
    updated_by UUID REFERENCES users(id),
    CONSTRAINT uq_automation_rule UNIQUE (tenant_id, school_id, rule_name)
);

CREATE TABLE recipient_resolution_jobs (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id),
    communication_id UUID NOT NULL REFERENCES communications(id) ON DELETE CASCADE,
    status resolution_job_status NOT NULL DEFAULT 'queued',
    requested_by UUID NOT NULL REFERENCES users(id),
    started_at TIMESTAMPTZ,
    completed_at TIMESTAMPTZ,
    error_text TEXT,
    resolution_summary_json JSONB,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE TABLE recipient_inbox_items (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id),
    communication_id UUID NOT NULL REFERENCES communications(id) ON DELETE CASCADE,
    recipient_user_id UUID NOT NULL REFERENCES users(id),
    recipient_student_id UUID REFERENCES students(id),
    portal_user_id UUID REFERENCES portal_users(id),
    channel channel_code NOT NULL DEFAULT 'in_app',
    inbox_status inbox_item_status NOT NULL DEFAULT 'visible',
    visible_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    read_at TIMESTAMPTZ,
    acknowledged_at TIMESTAMPTZ,
    archived_at TIMESTAMPTZ,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    CONSTRAINT uq_inbox_item UNIQUE (tenant_id, communication_id, recipient_user_id, channel, COALESCE(recipient_student_id, '00000000-0000-0000-0000-000000000000'::uuid))
);

CREATE TABLE communication_search_index (
    communication_id UUID PRIMARY KEY REFERENCES communications(id) ON DELETE CASCADE,
    tenant_id UUID NOT NULL REFERENCES tenants(id),
    school_id UUID NOT NULL REFERENCES schools(id),
    sender_user_id UUID REFERENCES users(id),
    category communication_category NOT NULL,
    priority communication_priority NOT NULL,
    status communication_status NOT NULL,
    channel_codes channel_code[] NOT NULL DEFAULT '{}',
    searchable_text tsvector NOT NULL,
    audience_summary_json JSONB,
    sent_at TIMESTAMPTZ,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);
```


## 3. Constraints

The most important rules are tenant-safe audience resolution, no-dispatch on unresolved placeholders, immutable sent history, idempotent provider callbacks, and prevention of zero-recipient or duplicate accidental sends.[^1]
The PRD is also explicit that corrections after dispatch must happen through cancellation-before-send or follow-up communication rather than deleting sent messages, so this module should strongly enforce historical immutability.[^1]

- Every table above that belongs to a tenant carries `tenant_id`, because the PRD requires every communication record, template, audience, and delivery log to belong to exactly one tenant.[^1]
- Add a trigger on `communications` so transition to `scheduled`, `queued`, or `sending` is blocked if unresolved template placeholders exist in `rendered_payload_snapshot` or if required approval is missing.[^1]
- Add a trigger on `recipient_resolution_jobs` or send orchestration so dispatch is blocked when resolved audience count is zero unless an explicit authorized override flag is present, because zero-recipient sends are not allowed by default.[^1]
- Add a trigger on `communication_delivery_logs` to enforce idempotency on provider callbacks using `(tenant_id, outbound_message_id, provider_event_id)` or a computed `callback_hash`, because duplicate callbacks must be processed idempotently.[^1]
- Add a trigger on `outbound_messages` to prevent setting `status = 'delivered'` or `status = 'read'` unless at least one matching delivery log exists.[^1]
- Prohibit `DELETE` on `communications`, `communication_audience_snapshots`, `outbound_messages`, `outbound_message_attempts`, `communication_delivery_logs`, and `communication_events`, because sent history must be retained and searchable.[^1]
- Soft delete is justified only for configuration/master entities such as templates or automation rules by using status enums like `inactive` or `archived`, because the PRD defines explicit lifecycle states for those entities.[^1]


## 4. Indexes

The likely hot paths are communication history search, pending approvals, queued outbound dispatch, retry processing, provider callback lookup, inbox listing, and analytics by tenant, category, and channel.[^1]

```sql
CREATE INDEX idx_communications_tenant_status_sched
ON communications (tenant_id, school_id, status, scheduled_at);

CREATE INDEX idx_communications_tenant_category_created
ON communications (tenant_id, school_id, category, created_at DESC);

CREATE INDEX idx_communications_source_ref
ON communications (tenant_id, source_module, source_ref_id);

CREATE INDEX idx_comm_audiences_comm
ON communication_audiences (tenant_id, communication_id, audience_type, audience_ref_id);

CREATE INDEX idx_comm_snapshots_comm
ON communication_audience_snapshots (tenant_id, communication_id, resolved_at DESC);

CREATE INDEX idx_templates_school_status
ON communication_templates (tenant_id, school_id, status, category);

CREATE INDEX idx_template_versions_template
ON communication_template_versions (tenant_id, template_id, channel_code, version_no DESC);

CREATE INDEX idx_approvals_pending
ON communication_approvals (tenant_id, status, requested_at);

CREATE INDEX idx_channels_school_enabled
ON communication_channels (tenant_id, school_id, enabled, priority_order);

CREATE INDEX idx_policies_school_category
ON communication_channel_policies (tenant_id, school_id, category, active);

CREATE INDEX idx_preferences_user_category
ON notification_preferences (tenant_id, user_id, message_category, channel);

CREATE INDEX idx_outbound_queue
ON outbound_messages (tenant_id, status, channel, queued_at);

CREATE INDEX idx_outbound_comm_recipient
ON outbound_messages (tenant_id, communication_id, recipient_user_id, channel);

CREATE INDEX idx_outbound_provider_ref
ON outbound_messages (tenant_id, provider_ref);

CREATE INDEX idx_attempts_outbound
ON outbound_message_attempts (tenant_id, outbound_message_id, attempt_no DESC);

CREATE INDEX idx_delivery_logs_outbound_time
ON communication_delivery_logs (tenant_id, outbound_message_id, event_time DESC);

CREATE UNIQUE INDEX uq_delivery_log_provider_event
ON communication_delivery_logs (tenant_id, outbound_message_id, provider_event_id)
WHERE provider_event_id IS NOT NULL;

CREATE UNIQUE INDEX uq_delivery_log_callback_hash
ON communication_delivery_logs (tenant_id, callback_hash)
WHERE callback_hash IS NOT NULL;

CREATE INDEX idx_comm_events_comm_time
ON communication_events (tenant_id, communication_id, created_at DESC);

CREATE INDEX idx_automation_rules_active
ON automation_rules (tenant_id, school_id, status, active, trigger_type);

CREATE INDEX idx_resolution_jobs_status
ON recipient_resolution_jobs (tenant_id, status, created_at);

CREATE INDEX idx_inbox_items_user
ON recipient_inbox_items (tenant_id, recipient_user_id, inbox_status, visible_at DESC);

CREATE INDEX idx_comm_search_gin
ON communication_search_index
USING GIN (searchable_text);

CREATE INDEX idx_comm_search_filters
ON communication_search_index (tenant_id, school_id, category, status, sent_at DESC);
```


## 5. Sample records

These sample rows cover a common school workflow: a fee reminder template, a finance-triggered communication, grade/section targeting, resolved audience snapshot, one SMS outbound dispatch, and one delivery callback log.[^1]

```sql
INSERT INTO communication_templates (
    id, tenant_id, school_id, template_name, template_code, category, default_priority, status, created_by
) VALUES (
    'a1000000-0000-0000-0000-000000000001',
    '10000000-0000-0000-0000-000000000001',
    '20000000-0000-0000-0000-000000000001',
    'Fee Due Reminder',
    'FEE_DUE_REMINDER',
    'fee_reminder',
    'high',
    'active',
    '30000000-0000-0000-0000-000000000001'
);

INSERT INTO communication_template_versions (
    id, tenant_id, template_id, version_no, channel_code, subject_template, body_template,
    variable_schema, is_active, created_by
) VALUES (
    'a1000000-0000-0000-0000-000000000002',
    '10000000-0000-0000-0000-000000000001',
    'a1000000-0000-0000-0000-000000000001',
    1,
    'sms',
    NULL,
    'Dear {{parent_name}}, fee of {{due_amount}} for {{student_name}} is due on {{due_date}}.',
    '{"required":["parent_name","due_amount","student_name","due_date"]}'::jsonb,
    true,
    '30000000-0000-0000-0000-000000000001'
);

INSERT INTO communications (
    id, tenant_id, school_id, template_id, template_version_id, title, body_text, normalized_body_text,
    category, priority, status, requires_approval, requires_delivery_tracking, scheduled_at,
    source_module, source_ref_id, created_by
) VALUES (
    'a1000000-0000-0000-0000-000000000010',
    '10000000-0000-0000-0000-000000000001',
    '20000000-0000-0000-0000-000000000001',
    'a1000000-0000-0000-0000-000000000001',
    'a1000000-0000-0000-0000-000000000002',
    'Fee Reminder - June Cycle',
    'Dear Parent, fee payment is due.',
    'dear parent fee payment is due',
    'fee_reminder',
    'high',
    'scheduled',
    false,
    true,
    now() + interval '30 minutes',
    'finance',
    'f1000000-0000-0000-0000-000000000001',
    '30000000-0000-0000-0000-000000000011'
);

INSERT INTO communication_audiences (
    id, tenant_id, communication_id, audience_type, audience_ref_code, recipient_count, created_by
) VALUES (
    'a1000000-0000-0000-0000-000000000011',
    '10000000-0000-0000-0000-000000000001',
    'a1000000-0000-0000-0000-000000000010',
    'grade',
    'GRADE_8',
    124,
    '30000000-0000-0000-0000-000000000011'
);

INSERT INTO communication_audience_snapshots (
    id, tenant_id, communication_id, snapshot_version, recipient_count, duplicate_count,
    resolution_rules_json, resolved_snapshot_json, created_by
) VALUES (
    'a1000000-0000-0000-0000-000000000012',
    '10000000-0000-0000-0000-000000000001',
    'a1000000-0000-0000-0000-000000000010',
    1,
    118,
    6,
    '{"deduplicate_by_contact":true,"skip_missing_contacts":true}'::jsonb,
    '{"recipients":[{"user_id":"u1","contact":"+9199XXXX001"},{"user_id":"u2","contact":"+9199XXXX002"}]}'::jsonb,
    '30000000-0000-0000-0000-000000000011'
);

INSERT INTO outbound_messages (
    id, tenant_id, communication_id, template_id, recipient_user_id, recipient_student_id, recipient_contact,
    channel, priority, status, payload_snapshot, queued_at, created_by
) VALUES (
    'a1000000-0000-0000-0000-000000000013',
    '10000000-0000-0000-0000-000000000001',
    'a1000000-0000-0000-0000-000000000010',
    'a1000000-0000-0000-0000-000000000001',
    '30000000-0000-0000-0000-000000000050',
    '40000000-0000-0000-0000-000000000050',
    '+919900000050',
    'sms',
    'high',
    'queued',
    '{"body":"Dear Meera, fee of 12500 is due on 2026-06-20 for Aarav."}'::jsonb,
    now(),
    '30000000-0000-0000-0000-000000000011'
);

INSERT INTO communication_delivery_logs (
    id, tenant_id, outbound_message_id, event_type, event_time, provider_event_id, provider_payload, normalized_status
) VALUES (
    'a1000000-0000-0000-0000-000000000014',
    '10000000-0000-0000-0000-000000000001',
    'a1000000-0000-0000-0000-000000000013',
    'provider_accepted',
    now(),
    'provider-msg-789',
    '{"provider":"sms_gateway_x","status":"accepted"}'::jsonb,
    'sent'
);
```


## 6. Query patterns

The PRD emphasizes searchability, delivery outcomes, high-priority visibility, and recipient inbox access, so these are the main query paths to optimize.[^1]

1. Pending approvals for principal or school admin.[^1]
```sql
SELECT c.id, c.title, c.category, c.priority, a.requested_at, u.first_name, u.last_name
FROM communication_approvals a
JOIN communications c ON c.id = a.communication_id
JOIN users u ON u.id = a.requested_by
WHERE a.tenant_id = $1
  AND a.status = 'pending'
ORDER BY c.priority DESC, a.requested_at ASC;
```

2. Scheduler query for due communications.[^1]
```sql
SELECT id, tenant_id, school_id, category, priority
FROM communications
WHERE status = 'scheduled'
  AND scheduled_at <= now()
ORDER BY priority DESC, scheduled_at ASC
LIMIT 500;
```

3. Outbound dispatcher query by channel.[^1]
```sql
SELECT id, communication_id, recipient_user_id, recipient_contact, payload_snapshot
FROM outbound_messages
WHERE tenant_id = $1
  AND channel = $2
  AND status IN ('queued', 'retrying')
ORDER BY priority DESC, queued_at ASC
LIMIT 1000;
```

4. Parent or student inbox history.[^1]
```sql
SELECT rii.communication_id, c.title, c.category, c.priority, rii.inbox_status, rii.visible_at, rii.read_at
FROM recipient_inbox_items rii
JOIN communications c ON c.id = rii.communication_id
WHERE rii.tenant_id = $1
  AND rii.recipient_user_id = $2
ORDER BY rii.visible_at DESC
LIMIT 50 OFFSET 0;
```

5. Communication history search by keyword, channel, and date range.[^1]
```sql
SELECT csi.communication_id, c.title, c.category, c.priority, c.status, c.sent_at
FROM communication_search_index csi
JOIN communications c ON c.id = csi.communication_id
WHERE csi.tenant_id = $1
  AND csi.school_id = $2
  AND csi.searchable_text @@ plainto_tsquery('simple', $3)
  AND ($4::channel_code IS NULL OR $4 = ANY(csi.channel_codes))
  AND ($5::timestamptz IS NULL OR csi.sent_at >= $5)
  AND ($6::timestamptz IS NULL OR csi.sent_at < $6)
ORDER BY csi.sent_at DESC
LIMIT 100;
```

6. Delivery-failure dashboard by tenant and category.[^1]
```sql
SELECT c.category, om.channel, count(*) AS failed_count
FROM outbound_messages om
JOIN communications c ON c.id = om.communication_id
WHERE om.tenant_id = $1
  AND om.status = 'failed'
  AND om.queued_at >= now() - interval '7 days'
GROUP BY c.category, om.channel
ORDER BY failed_count DESC;
```


## 7. Migration sequence

The safest implementation order is authoring first, then templates and channels, then approval and outbound execution, then automation and search, because the PRD makes composition, governance, delivery tracking, and history the core lifecycle.[^1]

1. Create enums and foundational master tables: `communication_templates`, `communication_template_versions`, `communication_channels`, `communication_channel_policies`, and `notification_preferences`.[^1]
2. Create `communications`, `communication_audiences`, `communication_attachments`, and `communication_approvals`.[^1]
3. Create `recipient_resolution_jobs` and `communication_audience_snapshots`, then implement audience-resolution services using upstream SIS/RBAC data.[^1]
4. Create `outbound_messages`, `outbound_message_attempts`, and `communication_delivery_logs`, then add queue workers and idempotent callback processing.[^1]
5. Create `recipient_inbox_items` so parents and students can see in-app notices and alerts.[^1]
6. Create `communication_events` and `communication_search_index`, then build search refresh and analytics aggregation jobs.[^1]
7. Create `automation_rules`, then add event subscribers for attendance submission, fee due/overdue, payment success, timetable change, and result publication.[^1]
8. Finally enable RLS for every tenant-owned table, revoke `DELETE` on sent-history tables, and add transition triggers for approval, zero-recipient protection, and duplicate callback control.[^1]

## 8. Security-sensitive columns

The most sensitive columns here are recipient contacts, rendered payload snapshots, provider payloads, and audience snapshots, because they can expose PII or sensitive operational data at scale.[^1]

- `outbound_messages.recipient_contact` — direct phone number or email, which is primary PII and should be masked in most admin views.[^1]
- `outbound_messages.payload_snapshot` — may include student name, class, due amount, attendance status, or result-publication context.[^1]
- `communication_audience_snapshots.resolved_snapshot_json` — high-risk bulk recipient map that should be heavily access-controlled and retained carefully.[^1]
- `notification_preferences` rows — indirectly sensitive because they reveal user contact-channel behavior and consent posture.[^1]
- `communication_delivery_logs.provider_payload` and `outbound_message_attempts.request_payload_json` / `response_payload_json` — may contain provider metadata, contact values, or callback details.[^1]
- `communication_attachments.file_storage_key` — protected storage locator for attached files and should never be exposed directly to clients.[^1]
- `communications.rendered_payload_snapshot` — can preserve sensitive business content and should be restricted to authorized operators only.[^1]

One correction to the broader module sequence: I do not have the M7_8 PRD in this conversation, so I should not have implied I could bundle it unless you already shared it elsewhere.[^1]
The next clean step is M7_11, or I can also normalize overlaps across M7_7, M7_9, and M7_10 such as report-card visibility, portal inbox delivery, and event-triggered communications.

<div align="center">⁂</div>

[^1]: PRD_M7_10.txt

