<img src="https://r2cdn.perplexity.ai/pplx-full-logo-primary-dark%402x.png" style="height:64px;margin-right:32px"/>

# please give the same data architecture for Module M7_6

For M7_6, the right design is an **attendance-session-centered schema** where attendance is always tied to academic year, enrollment, class-section, and optionally timetable slot, because the PRD explicitly says attendance must never be stored independently of enrollment and period-wise attendance requires published timetable data.[^1]
I would also make this module workflow-driven with draft, submit, lock, correction, notification, and audit layers, because the PRD requires secure, auditable attendance capture with approval-based corrections, automatic notifications, and reporting.[^1]

## 1. Entity list

These are the production entities I would use for Module 7.6, aligned to attendance capture, correction workflow, notification workflow, reporting, and auditability.[^1]

- `attendance_settings` — tenant-level attendance configuration, including primary mode, lock policy, and notification behavior, because attendance mode and notification preferences are configurable by school and only one primary attendance mode is allowed at a time.[^1]
- `attendance_statuses` — tenant-scoped status catalog for Present, Absent, Late, Excused, and Half Day, because these are the supported attendance statuses and schools need configurable status behavior.[^1]
- `attendance_calendar_days` — school day calendar used to block attendance on non-instructional days and support percentage calculations that exclude non-school days.[^1]
- `attendance_sessions` — the header record for one attendance-taking event by date, academic year, grade, section, and optionally timetable slot.[^1]
- `attendance_records` — one row per student per attendance session, because each student can have only one attendance status per session and only active enrolled students should appear.[^1]
- `attendance_corrections` — approval workflow for post-submission or post-lock corrections, because all corrections require a reason and corrections after lock require School Admin approval.[^1]
- `attendance_notifications` — notification trail for absent and late alerts sent to guardians, because absent and late notifications may be triggered automatically and their delivery history must be tracked.[^1]
- `attendance_audit_logs` — immutable audit trail for attendance modifications, corrections, and lock or unlock actions, because all modifications and lock events must be audited.[^1]
- `attendance_summary_daily` — materialized view for school-wide daily reporting and dashboards, because the PRD explicitly calls for daily summaries and analytics and recommends materialized reporting views at scale.[^1]


## 2. Table definitions

The DDL below implements M7_6 as a production-ready schema with tenant isolation, attendance-mode control, historical enrollment linkage, correction workflow, notification history, and audit support.[^1]

```sql
CREATE EXTENSION IF NOT EXISTS pgcrypto;
CREATE EXTENSION IF NOT EXISTS citext;

CREATE TYPE attendance_mode AS ENUM ('daily', 'period_wise');
CREATE TYPE attendance_session_status AS ENUM (
    'draft',
    'submitted',
    'locked',
    'corrected_pending',
    'corrected_approved',
    'corrected_rejected'
);
CREATE TYPE attendance_correction_status AS ENUM (
    'requested',
    'pending_review',
    'approved',
    'rejected'
);
CREATE TYPE attendance_notification_type AS ENUM ('absent', 'late');
CREATE TYPE delivery_channel AS ENUM ('sms', 'email', 'push', 'whatsapp');
CREATE TYPE notification_status AS ENUM ('pending', 'queued', 'sent', 'failed');
CREATE TYPE calendar_day_type AS ENUM ('instructional', 'holiday', 'weekend', 'closure');

CREATE TABLE attendance_settings (
    tenant_id UUID PRIMARY KEY REFERENCES tenants(id) ON DELETE CASCADE,
    school_id UUID NOT NULL REFERENCES schools(id) ON DELETE RESTRICT,
    primary_attendance_mode attendance_mode NOT NULL,
    allow_admin_submit_on_behalf BOOLEAN NOT NULL DEFAULT TRUE,
    auto_mark_present_default BOOLEAN NOT NULL DEFAULT TRUE,
    auto_notify_absent BOOLEAN NOT NULL DEFAULT TRUE,
    auto_notify_late BOOLEAN NOT NULL DEFAULT FALSE,
    lock_after_submission BOOLEAN NOT NULL DEFAULT FALSE,
    lock_cutoff_time TIME,
    require_reason_for_late BOOLEAN NOT NULL DEFAULT FALSE,
    require_reason_for_excused BOOLEAN NOT NULL DEFAULT TRUE,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID NULL REFERENCES users(id),
    updated_by UUID NULL REFERENCES users(id)
);

CREATE TABLE attendance_statuses (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
    code VARCHAR(50) NOT NULL,
    name VARCHAR(100) NOT NULL,
    counts_as_present BOOLEAN NOT NULL DEFAULT FALSE,
    counts_as_absent BOOLEAN NOT NULL DEFAULT FALSE,
    is_default BOOLEAN NOT NULL DEFAULT FALSE,
    sort_order INTEGER NOT NULL DEFAULT 1,
    is_active BOOLEAN NOT NULL DEFAULT TRUE,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID NULL REFERENCES users(id),
    updated_by UUID NULL REFERENCES users(id),
    CONSTRAINT uq_attendance_status_code UNIQUE (tenant_id, code),
    CONSTRAINT ck_attendance_status_sort CHECK (sort_order > 0)
);

CREATE TABLE attendance_calendar_days (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
    academic_year_id UUID NOT NULL REFERENCES academic_years(id) ON DELETE RESTRICT,
    campus_id UUID NULL REFERENCES campuses(id) ON DELETE RESTRICT,
    calendar_date DATE NOT NULL,
    day_type calendar_day_type NOT NULL,
    reason VARCHAR(255),
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID NULL REFERENCES users(id),
    CONSTRAINT uq_attendance_calendar_day UNIQUE (tenant_id, academic_year_id, campus_id, calendar_date)
);

CREATE TABLE attendance_sessions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
    school_id UUID NOT NULL REFERENCES schools(id) ON DELETE RESTRICT,
    academic_year_id UUID NOT NULL REFERENCES academic_years(id) ON DELETE RESTRICT,
    grade_level_id UUID NOT NULL REFERENCES grade_levels(id) ON DELETE RESTRICT,
    section_id UUID NOT NULL REFERENCES sections(id) ON DELETE RESTRICT,
    attendance_mode attendance_mode NOT NULL,
    attendance_date DATE NOT NULL,
    timetable_slot_id UUID NULL REFERENCES timetable_slots(id) ON DELETE RESTRICT,
    subject_id UUID NULL REFERENCES subjects(id) ON DELETE RESTRICT,
    teacher_assignment_id UUID NULL REFERENCES academic_teacher_assignments(id) ON DELETE RESTRICT,
    status attendance_session_status NOT NULL DEFAULT 'draft',
    created_by UUID NOT NULL REFERENCES users(id),
    submitted_by UUID NULL REFERENCES users(id),
    submitted_at TIMESTAMPTZ,
    locked_by UUID NULL REFERENCES users(id),
    locked_at TIMESTAMPTZ,
    last_correction_requested_at TIMESTAMPTZ,
    correction_reviewed_at TIMESTAMPTZ,
    remarks TEXT,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE TABLE attendance_records (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
    attendance_session_id UUID NOT NULL REFERENCES attendance_sessions(id) ON DELETE CASCADE,
    student_id UUID NOT NULL REFERENCES students(id) ON DELETE RESTRICT,
    enrollment_id UUID NOT NULL REFERENCES student_enrollments(id) ON DELETE RESTRICT,
    attendance_status_id UUID NOT NULL REFERENCES attendance_statuses(id) ON DELETE RESTRICT,
    remarks TEXT,
    recorded_by UUID NOT NULL REFERENCES users(id),
    recorded_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    corrected_at TIMESTAMPTZ,
    corrected_by UUID NULL REFERENCES users(id),
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    CONSTRAINT uq_attendance_record_student_session UNIQUE (attendance_session_id, student_id)
);

CREATE TABLE attendance_corrections (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
    attendance_record_id UUID NOT NULL REFERENCES attendance_records(id) ON DELETE RESTRICT,
    attendance_session_id UUID NOT NULL REFERENCES attendance_sessions(id) ON DELETE RESTRICT,
    old_status_id UUID NOT NULL REFERENCES attendance_statuses(id) ON DELETE RESTRICT,
    new_status_id UUID NOT NULL REFERENCES attendance_statuses(id) ON DELETE RESTRICT,
    old_remarks TEXT,
    new_remarks TEXT,
    reason TEXT NOT NULL,
    requested_by UUID NOT NULL REFERENCES users(id),
    requested_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    approved_by UUID NULL REFERENCES users(id),
    approved_at TIMESTAMPTZ,
    rejected_by UUID NULL REFERENCES users(id),
    rejected_at TIMESTAMPTZ,
    status attendance_correction_status NOT NULL DEFAULT 'requested',
    created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE TABLE attendance_notifications (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
    attendance_record_id UUID NOT NULL REFERENCES attendance_records(id) ON DELETE CASCADE,
    guardian_id UUID NOT NULL REFERENCES guardians(id) ON DELETE RESTRICT,
    notification_type attendance_notification_type NOT NULL,
    delivery_channel delivery_channel NOT NULL,
    status notification_status NOT NULL DEFAULT 'pending',
    recipient_address VARCHAR(255) NOT NULL,
    provider_message_id VARCHAR(255),
    provider_response JSONB,
    queued_at TIMESTAMPTZ,
    sent_at TIMESTAMPTZ,
    failed_at TIMESTAMPTZ,
    failure_reason TEXT,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID NULL REFERENCES users(id)
);

CREATE TABLE attendance_audit_logs (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE RESTRICT,
    entity_type VARCHAR(100) NOT NULL,
    entity_id UUID NOT NULL,
    action VARCHAR(100) NOT NULL,
    performed_by UUID NULL REFERENCES users(id) ON DELETE SET NULL,
    metadata JSONB,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

This structure follows the PRD’s required entities while strengthening them with school settings, instructional-day control, explicit correction timestamps, and historical enrollment linkage so attendance is never detached from the student’s academic placement.[^1]

## 3. Constraints and indexes

The most important M7_6 enforcement points are one primary attendance mode per school, one attendance status per student per session, duplicate-session prevention, active-enrollment-only attendance, non-instructional-day blocking, no teacher edits after submission, and correction history that is never deleted.[^1]

**Constraints**

- `attendance_settings.primary_attendance_mode` enforces a single primary mode, because the PRD says a school can enable only one primary attendance mode at a time.[^1]
- `attendance_sessions` should use separate uniqueness for daily and period-wise sessions, because duplicate sessions must be prevented for the same date, section, and timetable context.[^1]
- `attendance_records` uses `UNIQUE (attendance_session_id, student_id)` so each student has only one attendance status per attendance session.[^1]
- A trigger should ensure only active enrolled students for that date and section can be inserted into `attendance_records`, because only active enrolled students should appear in attendance sessions and reports must use historical enrollment records.[^1]
- A trigger should require `timetable_slot_id`, `subject_id`, and `teacher_assignment_id` when `attendance_mode = 'period_wise'`, because period attendance must reference timetable slot, subject, and teacher assignment and requires published timetable data.[^1]
- A trigger should block session creation on non-instructional days in `attendance_calendar_days`, because the PRD explicitly says attendance must be blocked for holidays and non-school days.[^1]
- A trigger or service rule should block teacher edits once a session is submitted and force corrections through `attendance_corrections` after lock, because submitted attendance cannot be edited by teachers and locked attendance can only be modified through correction workflow.[^1]
- `attendance_corrections.reason NOT NULL` enforces the PRD rule that all corrections require a reason.[^1]
- `attendance_audit_logs` and `attendance_corrections` should be append-only by policy or trigger, because all modifications, lock actions, and correction history must be auditable and never deleted.[^1]

```sql
CREATE UNIQUE INDEX uq_attendance_session_daily
ON attendance_sessions (tenant_id, attendance_date, section_id)
WHERE attendance_mode = 'daily';

CREATE UNIQUE INDEX uq_attendance_session_period
ON attendance_sessions (tenant_id, attendance_date, section_id, timetable_slot_id)
WHERE attendance_mode = 'period_wise';

CREATE INDEX idx_attendance_sessions_lookup
ON attendance_sessions (
    tenant_id, academic_year_id, attendance_date, grade_level_id, section_id, status
);

CREATE INDEX idx_attendance_sessions_teacher
ON attendance_sessions (
    tenant_id, teacher_assignment_id, attendance_date, status
);

CREATE INDEX idx_attendance_records_student_date
ON attendance_records (tenant_id, student_id, recorded_at DESC);

CREATE INDEX idx_attendance_records_enrollment
ON attendance_records (tenant_id, enrollment_id, attendance_session_id);

CREATE INDEX idx_attendance_records_status
ON attendance_records (tenant_id, attendance_status_id, attendance_session_id);

CREATE INDEX idx_attendance_corrections_status
ON attendance_corrections (tenant_id, status, requested_at DESC);

CREATE INDEX idx_attendance_notifications_guardian
ON attendance_notifications (tenant_id, guardian_id, status, created_at DESC);

CREATE INDEX idx_attendance_notifications_record
ON attendance_notifications (tenant_id, attendance_record_id, status);

CREATE INDEX idx_attendance_audit_logs_entity
ON attendance_audit_logs (tenant_id, entity_type, entity_id, created_at DESC);

CREATE INDEX idx_attendance_audit_logs_user
ON attendance_audit_logs (tenant_id, performed_by, created_at DESC);
```

For larger schools, `attendance_records` should be partitioned by `academic_year_id`, because the PRD recommends partitioning once annual attendance volume becomes high and specifically calls out partitioning by academic year for scale.[^1]

## 4. Sample records and query patterns

These samples represent the normal M7_6 flow: configure statuses, create an attendance session, record attendance for active students, and then use that data for parent history, correction workflows, and reporting.[^1]

```sql
INSERT INTO attendance_statuses (
    id, tenant_id, code, name, counts_as_present, counts_as_absent, is_default, sort_order, created_by
) VALUES
(
    '91000000-0000-0000-0000-000000000001',
    '10000000-0000-0000-0000-000000000001',
    'PRESENT',
    'Present',
    TRUE,
    FALSE,
    TRUE,
    1,
    '00000000-0000-0000-0000-000000000002'
),
(
    '91000000-0000-0000-0000-000000000002',
    '10000000-0000-0000-0000-000000000001',
    'ABSENT',
    'Absent',
    FALSE,
    TRUE,
    FALSE,
    2,
    '00000000-0000-0000-0000-000000000002'
),
(
    '91000000-0000-0000-0000-000000000003',
    '10000000-0000-0000-0000-000000000001',
    'LATE',
    'Late',
    FALSE,
    FALSE,
    FALSE,
    3,
    '00000000-0000-0000-0000-000000000002'
);

INSERT INTO attendance_sessions (
    id, tenant_id, school_id, academic_year_id, grade_level_id, section_id,
    attendance_mode, attendance_date, status, created_by
) VALUES (
    '92000000-0000-0000-0000-000000000001',
    '10000000-0000-0000-0000-000000000001',
    '51000000-0000-0000-0000-000000000001',
    '53000000-0000-0000-0000-000000000001',
    '54000000-0000-0000-0000-000000000001',
    '55000000-0000-0000-0000-000000000001',
    'daily',
    '2026-06-15',
    'draft',
    '61000000-0000-0000-0000-000000000001'
);

INSERT INTO attendance_records (
    id, tenant_id, attendance_session_id, student_id, enrollment_id,
    attendance_status_id, recorded_by
) VALUES (
    '93000000-0000-0000-0000-000000000001',
    '10000000-0000-0000-0000-000000000001',
    '92000000-0000-0000-0000-000000000001',
    '71000000-0000-0000-0000-000000000001',
    '73000000-0000-0000-0000-000000000001',
    '91000000-0000-0000-0000-000000000001',
    '61000000-0000-0000-0000-000000000001'
);
```

**Query patterns**

1. **Load roster for a teacher attendance screen** — this should use only active enrollments for the selected date and section, because attendance sessions may include only active enrolled students.[^1]
```sql
SELECT se.id AS enrollment_id,
       s.id AS student_id,
       s.student_number,
       sp.first_name,
       sp.last_name
FROM student_enrollments se
JOIN students s
  ON s.id = se.student_id AND s.tenant_id = se.tenant_id
JOIN student_profiles sp
  ON sp.student_id = s.id AND sp.tenant_id = s.tenant_id
WHERE se.tenant_id = $1
  AND se.academic_year_id = $2
  AND se.section_id = $3
  AND se.status = 'active'
  AND se.enrollment_date <= $4
  AND (se.exit_date IS NULL OR se.exit_date >= $4)
ORDER BY sp.first_name, sp.last_name;
```

2. **Teacher session detail with recorded statuses** — this supports fast marking and review before submission.[^1]
```sql
SELECT ar.id,
       ar.student_id,
       sp.first_name,
       sp.last_name,
       ast.code AS status_code,
       ast.name AS status_name,
       ar.remarks
FROM attendance_records ar
JOIN students s
  ON s.id = ar.student_id AND s.tenant_id = ar.tenant_id
JOIN student_profiles sp
  ON sp.student_id = s.id AND sp.tenant_id = s.tenant_id
JOIN attendance_statuses ast
  ON ast.id = ar.attendance_status_id
WHERE ar.tenant_id = $1
  AND ar.attendance_session_id = $2
ORDER BY sp.first_name, sp.last_name;
```

3. **Parent attendance history for one child** — parents should be able to view historical attendance for linked children.[^1]
```sql
SELECT asn.attendance_date,
       sts.name AS attendance_status,
       ar.remarks
FROM attendance_records ar
JOIN attendance_sessions asn
  ON asn.id = ar.attendance_session_id
JOIN attendance_statuses sts
  ON sts.id = ar.attendance_status_id
WHERE ar.tenant_id = $1
  AND ar.student_id = $2
ORDER BY asn.attendance_date DESC;
```

4. **Pending correction workflow** — admins and principals need a review list for requested changes.[^1]
```sql
SELECT ac.id,
       asn.attendance_date,
       ar.student_id,
       old_s.name AS old_status,
       new_s.name AS new_status,
       ac.reason,
       ac.requested_at
FROM attendance_corrections ac
JOIN attendance_records ar
  ON ar.id = ac.attendance_record_id
JOIN attendance_sessions asn
  ON asn.id = ac.attendance_session_id
JOIN attendance_statuses old_s
  ON old_s.id = ac.old_status_id
JOIN attendance_statuses new_s
  ON new_s.id = ac.new_status_id
WHERE ac.tenant_id = $1
  AND ac.status IN ('requested', 'pending_review')
ORDER BY ac.requested_at DESC;
```

5. **Daily attendance summary report** — the PRD explicitly defines a daily summary view for operational and compliance reporting.[^1]
```sql
CREATE MATERIALIZED VIEW attendance_summary_daily AS
SELECT
    asn.tenant_id,
    asn.attendance_date,
    count(ar.id) AS total_students,
    count(*) FILTER (WHERE ast.code = 'PRESENT') AS present_count,
    count(*) FILTER (WHERE ast.code = 'ABSENT') AS absent_count,
    count(*) FILTER (WHERE ast.code = 'LATE') AS late_count
FROM attendance_sessions asn
JOIN attendance_records ar
  ON ar.attendance_session_id = asn.id
JOIN attendance_statuses ast
  ON ast.id = ar.attendance_status_id
GROUP BY asn.tenant_id, asn.attendance_date;
```


## 5. Migration sequence and security-sensitive columns

M7_6 should be migrated after SIS, academic structure, timetable, and guardian-linking tables are in place, because attendance depends on academic year, enrollment, section, optional timetable slots, and guardian visibility.[^1]
The main security and compliance risks in this module are unauthorized edits, inaccurate parent notifications, and exposure of student attendance histories, because the PRD treats attendance as a high-frequency operational record with audit, correction, and notification controls.[^1]

**Migration sequence**

1. Create enums and then `attendance_settings`, `attendance_statuses`, and `attendance_calendar_days`, because mode, status catalog, and instructional-day rules must exist before sessions are created.[^1]
2. Create `attendance_sessions` and `attendance_records`, then add duplicate-session constraints and active-enrollment validation triggers.[^1]
3. Create `attendance_corrections` and wire approval logic so post-lock changes flow only through correction workflow.[^1]
4. Create `attendance_notifications` and notification job integration so absent and late alerts are queued only after valid submission.[^1]
5. Create `attendance_audit_logs`, materialized views, indexes, and partitions, then add RLS for teacher, parent, student, and admin scope.[^1]

**Security-sensitive columns**

- `attendance_records.remarks` and `attendance_sessions.remarks` may contain sensitive classroom or incident notes and should be role-restricted.[^1]
- `attendance_corrections.reason`, old and new status references, and reviewer fields are audit-sensitive and should never be editable after decision.[^1]
- `attendance_notifications.recipient_address`, `provider_response`, and failure details contain guardian contact and delivery metadata and should be visible only to authorized staff.[^1]
- `attendance_audit_logs.metadata` may reveal who changed what and when, so it should be append-only and restricted to privileged operational and audit roles.[^1]

This M7_6 design gives you a production-ready attendance architecture with daily or period-wise capture, enrollment-linked accuracy, correction governance, notification history, scalable reporting, and strong auditability.[^1]

<div align="center">⁂</div>

[^1]: PRD_M7_6.txt

