<img src="https://r2cdn.perplexity.ai/pplx-full-logo-primary-dark%402x.png" style="height:64px;margin-right:32px"/>

# proceed with M7_12

For M7_12, I would design the schema around **assignment definition -> audience snapshot -> publication -> versioned student submissions -> teacher feedback -> analytics and audit**, because the PRD defines exactly that lifecycle and requires integration with academic structure, teacher assignment scope, student enrollment, and parent-child visibility rules.[^1]
This module should stay strictly academic-workflow focused and must not create its own rosters or mutate official gradebook results unless a separate integration is enabled.[^1]

## 1. Entities

These are the main production entities for M7_12.[^1]

- `assignment_settings` — tenant and school policy for late submission, parent visibility, file limits, and default submission modes, because those settings are configurable at school level.[^1]
- `assignments` — the assignment master for academic context, teacher ownership, due date, status, and publish lifecycle.[^1]
- `assignment_audiences` — resolved recipient targeting for section, selected students, or other allowed scope, because publication must preserve recipient snapshot for auditability.[^1]
- `assignment_recipients` — individual recipient snapshot rows to make pending, submitted, late, and missing queries efficient and historically stable after student movement.[^1]
- `assignment_attachments` — assignment-side attachments like worksheets or supporting files, because assignments support optional attachments and platform file policies.[^1]
- `assignment_deadline_history` — auditable due-date changes after publication, because deadline changes must be audited and visible.[^1]
- `assignment_submissions` — the student’s current submission header with latest-attempt linkage and current status.[^1]
- `assignment_submission_versions` — immutable submission attempts, because multiple attempts may be allowed and submission history must remain auditable.[^1]
- `submission_attachments` — uploaded files for each submission version.[^1]
- `assignment_feedback` — teacher feedback record with draft and published states, because final published feedback must be distinct from draft notes.[^1]
- `assignment_feedback_revisions` — immutable feedback history when edited after publication.[^1]
- `late_submission_overrides` — teacher override trail where late policy allows exception handling.[^1]
- `assignment_events` — workflow event log for create, edit, publish, unpublish, submit, resubmit, review, and feedback publish actions.[^1]


## 2. Table definitions

I am using explicit recipient and submission-version tables because the PRD requires recipient snapshot retention, eligibility by active enrollment, identifiable latest submission version, and auditable history for both submission and feedback.[^1]
I am also separating assignment attachments from submission attachments to avoid polymorphic file chaos and to keep permission boundaries clear between teacher-owned content and student-owned artifacts.[^1]

```sql
CREATE EXTENSION IF NOT EXISTS pgcrypto;
CREATE EXTENSION IF NOT EXISTS citext;

CREATE TYPE assignment_status AS ENUM (
  'draft', 'published', 'closed', 'archived'
);

CREATE TYPE submission_mode AS ENUM (
  'text_only', 'file_only', 'text_and_file'
);

CREATE TYPE late_submission_policy AS ENUM (
  'blocked', 'accepted_flagged', 'teacher_override'
);

CREATE TYPE audience_type AS ENUM (
  'section', 'student_group', 'individual_student'
);

CREATE TYPE recipient_status AS ENUM (
  'assigned', 'visible', 'submitted', 'late_submitted', 'missing', 'closed'
);

CREATE TYPE submission_status AS ENUM (
  'not_submitted', 'submitted', 'resubmitted', 'late_submitted',
  'under_review', 'reviewed', 'returned', 'closed'
);

CREATE TYPE feedback_status AS ENUM (
  'draft', 'published', 'revised', 'archived'
);

CREATE TABLE assignment_settings (
  tenant_id UUID NOT NULL REFERENCES tenants(id),
  school_id UUID NOT NULL REFERENCES schools(id),
  default_submission_mode submission_mode NOT NULL DEFAULT 'text_and_file',
  default_late_submission_policy late_submission_policy NOT NULL DEFAULT 'accepted_flagged',
  allow_parent_visibility BOOLEAN NOT NULL DEFAULT TRUE,
  allow_feedback_parent_visibility BOOLEAN NOT NULL DEFAULT TRUE,
  allow_individual_targeting BOOLEAN NOT NULL DEFAULT TRUE,
  allow_resubmission_default BOOLEAN NOT NULL DEFAULT TRUE,
  max_resubmissions INTEGER,
  allowed_attachment_mime_types TEXT[] NOT NULL DEFAULT '{}',
  max_attachment_size_bytes BIGINT NOT NULL DEFAULT 10485760,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  created_by UUID REFERENCES users(id),
  updated_by UUID REFERENCES users(id),
  PRIMARY KEY (tenant_id, school_id)
);

CREATE TABLE assignments (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID NOT NULL REFERENCES tenants(id),
  school_id UUID NOT NULL REFERENCES schools(id),
  academic_year_id UUID NOT NULL REFERENCES academic_years(id),
  grade_level_id UUID NOT NULL REFERENCES grade_levels(id),
  section_id UUID NOT NULL REFERENCES sections(id),
  subject_id UUID NOT NULL REFERENCES subjects(id),
  teacher_user_id UUID NOT NULL REFERENCES users(id),
  teacher_assignment_id UUID NULL REFERENCES teacher_subject_assignments(id),
  title VARCHAR(200) NOT NULL,
  instructions TEXT NOT NULL,
  instructions_format VARCHAR(20) NOT NULL DEFAULT 'plain'
    CHECK (instructions_format IN ('plain', 'rich_text')),
  submission_mode submission_mode NOT NULL,
  due_at TIMESTAMPTZ NOT NULL,
  school_timezone VARCHAR(64) NOT NULL,
  late_submission_policy late_submission_policy NOT NULL,
  allow_resubmission BOOLEAN NOT NULL DEFAULT TRUE,
  max_resubmissions INTEGER,
  parent_visibility_enabled BOOLEAN NOT NULL DEFAULT TRUE,
  feedback_parent_visibility_enabled BOOLEAN NOT NULL DEFAULT TRUE,
  published_content_snapshot JSONB,
  status assignment_status NOT NULL DEFAULT 'draft',
  published_at TIMESTAMPTZ,
  closed_at TIMESTAMPTZ,
  archived_at TIMESTAMPTZ,
  deleted_at TIMESTAMPTZ,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  created_by UUID REFERENCES users(id),
  updated_by UUID REFERENCES users(id),
  CONSTRAINT ck_assignment_due_after_create CHECK (due_at >= created_at),
  CONSTRAINT ck_assignment_publish_timestamps CHECK (
    (status <> 'published' OR published_at IS NOT NULL)
  )
);

CREATE TABLE assignment_audiences (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID NOT NULL REFERENCES tenants(id),
  assignment_id UUID NOT NULL REFERENCES assignments(id) ON DELETE CASCADE,
  audience_type audience_type NOT NULL,
  audience_ref_id UUID NOT NULL,
  recipient_count INTEGER NOT NULL DEFAULT 0,
  resolved_snapshot JSONB NOT NULL DEFAULT '[]'::jsonb,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  created_by UUID REFERENCES users(id)
);

CREATE TABLE assignment_recipients (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID NOT NULL REFERENCES tenants(id),
  assignment_id UUID NOT NULL REFERENCES assignments(id) ON DELETE CASCADE,
  student_id UUID NOT NULL REFERENCES students(id),
  student_enrollment_id UUID NOT NULL REFERENCES student_enrollments(id),
  recipient_status recipient_status NOT NULL DEFAULT 'assigned',
  visible_from TIMESTAMPTZ NOT NULL DEFAULT now(),
  visible_until TIMESTAMPTZ,
  assigned_section_id UUID NOT NULL REFERENCES sections(id),
  recipient_snapshot JSONB NOT NULL,
  first_viewed_at TIMESTAMPTZ,
  latest_submission_id UUID,
  submitted_at TIMESTAMPTZ,
  is_late BOOLEAN NOT NULL DEFAULT FALSE,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  created_by UUID REFERENCES users(id),
  updated_by UUID REFERENCES users(id),
  CONSTRAINT uq_assignment_recipient UNIQUE (tenant_id, assignment_id, student_id)
);

CREATE TABLE assignment_attachments (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID NOT NULL REFERENCES tenants(id),
  assignment_id UUID NOT NULL REFERENCES assignments(id) ON DELETE CASCADE,
  file_name VARCHAR(255) NOT NULL,
  storage_key TEXT NOT NULL,
  mime_type VARCHAR(120) NOT NULL,
  file_size_bytes BIGINT NOT NULL,
  uploaded_by UUID NOT NULL REFERENCES users(id),
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  CONSTRAINT ck_assignment_attachment_size CHECK (file_size_bytes > 0)
);

CREATE TABLE assignment_deadline_history (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID NOT NULL REFERENCES tenants(id),
  assignment_id UUID NOT NULL REFERENCES assignments(id) ON DELETE CASCADE,
  old_due_at TIMESTAMPTZ NOT NULL,
  new_due_at TIMESTAMPTZ NOT NULL,
  changed_by UUID NOT NULL REFERENCES users(id),
  change_reason TEXT,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  CONSTRAINT ck_deadline_history_changed CHECK (old_due_at <> new_due_at)
);

CREATE TABLE assignment_submissions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID NOT NULL REFERENCES tenants(id),
  assignment_id UUID NOT NULL REFERENCES assignments(id) ON DELETE RESTRICT,
  student_id UUID NOT NULL REFERENCES students(id),
  assignment_recipient_id UUID NOT NULL REFERENCES assignment_recipients(id) ON DELETE RESTRICT,
  current_version_no INTEGER NOT NULL DEFAULT 0,
  latest_submission_version_id UUID,
  submission_status submission_status NOT NULL DEFAULT 'not_submitted',
  latest_submitted_at TIMESTAMPTZ,
  is_late BOOLEAN NOT NULL DEFAULT FALSE,
  review_started_at TIMESTAMPTZ,
  reviewed_at TIMESTAMPTZ,
  reviewed_by UUID REFERENCES users(id),
  closed_at TIMESTAMPTZ,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  created_by UUID REFERENCES users(id),
  updated_by UUID REFERENCES users(id),
  CONSTRAINT uq_assignment_student_submission UNIQUE (tenant_id, assignment_id, student_id)
);

CREATE TABLE assignment_submission_versions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID NOT NULL REFERENCES tenants(id),
  assignment_submission_id UUID NOT NULL REFERENCES assignment_submissions(id) ON DELETE CASCADE,
  version_no INTEGER NOT NULL,
  submission_text TEXT,
  submitted_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  submission_status submission_status NOT NULL,
  is_late BOOLEAN NOT NULL DEFAULT FALSE,
  override_id UUID,
  idempotency_key UUID,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  created_by UUID REFERENCES users(id),
  CONSTRAINT uq_submission_version UNIQUE (tenant_id, assignment_submission_id, version_no),
  CONSTRAINT uq_submission_idempotency UNIQUE (tenant_id, idempotency_key)
);

CREATE TABLE submission_attachments (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID NOT NULL REFERENCES tenants(id),
  submission_version_id UUID NOT NULL REFERENCES assignment_submission_versions(id) ON DELETE CASCADE,
  file_name VARCHAR(255) NOT NULL,
  storage_key TEXT NOT NULL,
  mime_type VARCHAR(120) NOT NULL,
  file_size_bytes BIGINT NOT NULL,
  uploaded_by UUID NOT NULL REFERENCES users(id),
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  CONSTRAINT ck_submission_attachment_size CHECK (file_size_bytes > 0)
);

CREATE TABLE late_submission_overrides (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID NOT NULL REFERENCES tenants(id),
  assignment_id UUID NOT NULL REFERENCES assignments(id) ON DELETE CASCADE,
  student_id UUID NOT NULL REFERENCES students(id),
  approved_by UUID NOT NULL REFERENCES users(id),
  approved_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  reason TEXT NOT NULL,
  expires_at TIMESTAMPTZ,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE TABLE assignment_feedback (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID NOT NULL REFERENCES tenants(id),
  assignment_submission_id UUID NOT NULL REFERENCES assignment_submissions(id) ON DELETE CASCADE,
  teacher_user_id UUID NOT NULL REFERENCES users(id),
  feedback_text TEXT,
  score NUMERIC(8,2),
  feedback_status feedback_status NOT NULL DEFAULT 'draft',
  draft_internal_notes TEXT,
  published_at TIMESTAMPTZ,
  latest_revision_no INTEGER NOT NULL DEFAULT 1,
  visible_to_student BOOLEAN NOT NULL DEFAULT TRUE,
  visible_to_parent BOOLEAN NOT NULL DEFAULT FALSE,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  created_by UUID REFERENCES users(id),
  updated_by UUID REFERENCES users(id),
  CONSTRAINT uq_feedback_per_submission UNIQUE (tenant_id, assignment_submission_id)
);

CREATE TABLE assignment_feedback_revisions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID NOT NULL REFERENCES tenants(id),
  feedback_id UUID NOT NULL REFERENCES assignment_feedback(id) ON DELETE CASCADE,
  revision_no INTEGER NOT NULL,
  feedback_text TEXT,
  score NUMERIC(8,2),
  feedback_status feedback_status NOT NULL,
  published_at TIMESTAMPTZ,
  revision_reason TEXT,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  created_by UUID REFERENCES users(id),
  CONSTRAINT uq_feedback_revision UNIQUE (tenant_id, feedback_id, revision_no)
);

CREATE TABLE assignment_events (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID NOT NULL REFERENCES tenants(id),
  school_id UUID NOT NULL REFERENCES schools(id),
  assignment_id UUID REFERENCES assignments(id) ON DELETE CASCADE,
  assignment_submission_id UUID REFERENCES assignment_submissions(id) ON DELETE CASCADE,
  feedback_id UUID REFERENCES assignment_feedback(id) ON DELETE CASCADE,
  actor_user_id UUID REFERENCES users(id),
  event_name VARCHAR(80) NOT NULL,
  old_value JSONB,
  new_value JSONB,
  ip_address INET,
  user_agent TEXT,
  request_id UUID,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

ALTER TABLE assignment_recipients
  ADD CONSTRAINT fk_assignment_recipients_latest_submission
  FOREIGN KEY (latest_submission_id) REFERENCES assignment_submissions(id);

ALTER TABLE assignment_submissions
  ADD CONSTRAINT fk_assignment_submissions_latest_version
  FOREIGN KEY (latest_submission_version_id) REFERENCES assignment_submission_versions(id);

ALTER TABLE assignment_submission_versions
  ADD CONSTRAINT fk_submission_version_override
  FOREIGN KEY (override_id) REFERENCES late_submission_overrides(id);
```


## 3. Constraints

The critical business constraints are teacher-scope validation, published-content immutability by snapshot, student visibility only through recipient resolution, submission-mode enforcement, late-policy enforcement, and auditable revision history for both submissions and feedback.[^1]

- Every tenant-owned record includes `tenant_id`, because all assignment, submission, attachment, and feedback data must be tenant-scoped and inaccessible across tenants.[^1]
- `assignments` must reference academic year, grade, section, subject, and teacher context, because assignment creation must align to the official academic structure and teacher assignment scope.[^1]
- A trigger should block publish when the teacher is not authorized for the academic context, because assigning work outside the teacher’s scope is an explicit edge case that must be blocked.[^1]
- A trigger should resolve and persist `assignment_recipients` at publish time, because recipient snapshot must be preserved for auditability and historical visibility even if student enrollment later changes.[^1]
- A trigger should enforce submission mode: `text_only` requires non-empty text and no files, `file_only` requires at least one file and optional empty text, and `text_and_file` allows either or both.[^1]
- A trigger should block submission if the assignment is not published, not visible to that student, closed, or the student is not eligible by active enrollment policy.[^1]
- A trigger should evaluate lateness using canonical `submitted_at` against `due_at` interpreted with school timezone rules, because late handling must be timezone-aware.[^1]
- A trigger should block resubmission when `allow_resubmission = false` or when attempt count exceeds `max_resubmissions`, because multiple submission attempts are configurable rather than always open-ended.[^1]
- Feedback draft notes should never be exposed in student or parent views, because final published feedback must be clearly distinct from draft teacher notes.[^1]
- Hard delete should be disabled for published assignments, submissions, feedback revisions, and events, because history must remain searchable and auditable.[^1]


## 4. Indexes

The expected hot paths are teacher dashboards, student pending lists, late-submission review queues, parent child-assignment views, and analytics by class, subject, teacher, and date range.[^1]

```sql
CREATE INDEX idx_assignments_teacher_status_due
ON assignments (tenant_id, teacher_user_id, status, due_at DESC);

CREATE INDEX idx_assignments_section_subject_status
ON assignments (tenant_id, academic_year_id, grade_level_id, section_id, subject_id, status, published_at DESC);

CREATE INDEX idx_assignment_recipients_student_status
ON assignment_recipients (tenant_id, student_id, recipient_status, visible_from DESC);

CREATE INDEX idx_assignment_recipients_assignment
ON assignment_recipients (tenant_id, assignment_id, recipient_status, is_late);

CREATE INDEX idx_assignment_submissions_assignment_student
ON assignment_submissions (tenant_id, assignment_id, student_id, submission_status, latest_submitted_at DESC);

CREATE INDEX idx_assignment_submissions_review_queue
ON assignment_submissions (tenant_id, submission_status, is_late, reviewed_by, updated_at DESC);

CREATE INDEX idx_submission_versions_submission
ON assignment_submission_versions (tenant_id, assignment_submission_id, version_no DESC);

CREATE INDEX idx_assignment_feedback_status
ON assignment_feedback (tenant_id, teacher_user_id, feedback_status, updated_at DESC);

CREATE INDEX idx_assignment_events_assignment
ON assignment_events (tenant_id, assignment_id, created_at DESC);

CREATE INDEX idx_assignment_events_submission
ON assignment_events (tenant_id, assignment_submission_id, created_at DESC);

CREATE INDEX idx_deadline_history_assignment
ON assignment_deadline_history (tenant_id, assignment_id, created_at DESC);
```


## 5. Sample records

These examples show one teacher publishing an assignment, one recipient snapshot, one student submission, and one feedback record.[^1]

```sql
INSERT INTO assignments (
  id, tenant_id, school_id, academic_year_id, grade_level_id, section_id, subject_id,
  teacher_user_id, title, instructions, submission_mode, due_at, school_timezone,
  late_submission_policy, allow_resubmission, parent_visibility_enabled,
  feedback_parent_visibility_enabled, status, published_at, created_at, updated_at
) VALUES (
  'c1000000-0000-0000-0000-000000000001',
  '10000000-0000-0000-0000-000000000001',
  '20000000-0000-0000-0000-000000000001',
  '30000000-0000-0000-0000-000000000001',
  '40000000-0000-0000-0000-000000000007',
  '50000000-0000-0000-0000-000000000012',
  '60000000-0000-0000-0000-000000000003',
  '00000000-0000-0000-0000-000000000021',
  'Math Homework - Fractions',
  'Solve questions 1 to 10 and upload your worksheet.',
  'file_only',
  '2026-06-15 12:30:00+00',
  'Asia/Kolkata',
  'accepted_flagged',
  TRUE,
  TRUE,
  TRUE,
  'published',
  now(), now(), now()
);

INSERT INTO assignment_recipients (
  id, tenant_id, assignment_id, student_id, student_enrollment_id, recipient_status,
  assigned_section_id, recipient_snapshot, created_at, updated_at
) VALUES (
  'c1000000-0000-0000-0000-000000000002',
  '10000000-0000-0000-0000-000000000001',
  'c1000000-0000-0000-0000-000000000001',
  '70000000-0000-0000-0000-000000000101',
  '80000000-0000-0000-0000-000000000101',
  'visible',
  '50000000-0000-0000-0000-000000000012',
  '{"student_name":"Riya Nair","section_name":"7-A"}'::jsonb,
  now(), now()
);

INSERT INTO assignment_submissions (
  id, tenant_id, assignment_id, student_id, assignment_recipient_id, current_version_no,
  submission_status, latest_submitted_at, is_late, created_at, updated_at
) VALUES (
  'c1000000-0000-0000-0000-000000000003',
  '10000000-0000-0000-0000-000000000001',
  'c1000000-0000-0000-0000-000000000001',
  '70000000-0000-0000-0000-000000000101',
  'c1000000-0000-0000-0000-000000000002',
  1,
  'submitted',
  now(),
  FALSE,
  now(), now()
);

INSERT INTO assignment_submission_versions (
  id, tenant_id, assignment_submission_id, version_no, submission_text, submitted_at,
  submission_status, is_late, created_at
) VALUES (
  'c1000000-0000-0000-0000-000000000004',
  '10000000-0000-0000-0000-000000000001',
  'c1000000-0000-0000-0000-000000000003',
  1,
  NULL,
  now(),
  'submitted',
  FALSE,
  now()
);

INSERT INTO assignment_feedback (
  id, tenant_id, assignment_submission_id, teacher_user_id, feedback_text, score,
  feedback_status, published_at, visible_to_student, visible_to_parent,
  created_at, updated_at
) VALUES (
  'c1000000-0000-0000-0000-000000000005',
  '10000000-0000-0000-0000-000000000001',
  'c1000000-0000-0000-0000-000000000003',
  '00000000-0000-0000-0000-000000000021',
  'Good work. Please simplify the last two answers more clearly.',
  8.50,
  'published',
  now(),
  TRUE,
  TRUE,
  now(), now()
);
```


## 6. Query patterns

These query patterns align to the PRD’s teacher dashboard, student pending list, parent child view, review screen, and analytics needs.[^1]

1. Teacher dashboard by status and due date.[^1]
```sql
SELECT id, title, section_id, subject_id, due_at, status
FROM assignments
WHERE tenant_id = $1
  AND teacher_user_id = $2
  AND status IN ('draft', 'published', 'closed')
ORDER BY due_at ASC;
```

2. Student pending and overdue assignments.[^1]
```sql
SELECT a.id, a.title, a.due_at, ar.recipient_status, s.submission_status, s.is_late
FROM assignment_recipients ar
JOIN assignments a
  ON a.id = ar.assignment_id AND a.tenant_id = ar.tenant_id
LEFT JOIN assignment_submissions s
  ON s.assignment_id = ar.assignment_id
 AND s.student_id = ar.student_id
 AND s.tenant_id = ar.tenant_id
WHERE ar.tenant_id = $1
  AND ar.student_id = $2
  AND a.status = 'published'
ORDER BY a.due_at ASC;
```

3. Parent child assignment visibility, because parents may only see linked children and only where school policy allows visibility.[^1]
```sql
SELECT a.id, a.title, a.due_at, s.submission_status, f.feedback_status
FROM parent_student_links psl
JOIN assignment_recipients ar
  ON ar.student_id = psl.student_id AND ar.tenant_id = psl.tenant_id
JOIN assignments a
  ON a.id = ar.assignment_id AND a.tenant_id = ar.tenant_id
LEFT JOIN assignment_submissions s
  ON s.assignment_id = a.id AND s.student_id = psl.student_id AND s.tenant_id = a.tenant_id
LEFT JOIN assignment_feedback f
  ON f.assignment_submission_id = s.id AND f.tenant_id = s.tenant_id
WHERE psl.tenant_id = $1
  AND psl.parent_user_id = $2
  AND psl.status = 'active'
  AND a.parent_visibility_enabled = TRUE;
```

4. Review queue with late flags and latest attempt.[^1]
```sql
SELECT s.id, s.student_id, s.submission_status, s.latest_submitted_at, s.is_late,
       sv.version_no, sv.submission_text
FROM assignment_submissions s
JOIN assignment_submission_versions sv
  ON sv.id = s.latest_submission_version_id
WHERE s.tenant_id = $1
  AND s.assignment_id = $2
ORDER BY s.is_late DESC, s.latest_submitted_at ASC;
```

5. Completion analytics by assignment.[^1]
```sql
SELECT
  a.id,
  a.title,
  count(ar.id) AS recipients,
  count(s.id) FILTER (WHERE s.submission_status IN ('submitted','resubmitted','late_submitted','under_review','reviewed','closed')) AS submitted_count,
  count(s.id) FILTER (WHERE s.is_late = TRUE) AS late_count,
  count(s.id) FILTER (WHERE s.submission_status = 'reviewed') AS reviewed_count
FROM assignments a
JOIN assignment_recipients ar
  ON ar.assignment_id = a.id AND ar.tenant_id = a.tenant_id
LEFT JOIN assignment_submissions s
  ON s.assignment_id = a.id AND s.student_id = ar.student_id AND s.tenant_id = a.tenant_id
WHERE a.tenant_id = $1
GROUP BY a.id, a.title;
```


## 7. Migration sequence

The safest sequence is settings and assignment master first, then audience resolution, then submissions, then feedback, and finally eventing and policy triggers, because publication and student eligibility depend on stable academic-context references.[^1]

1. Create enums and `assignment_settings`.[^1]
2. Create `assignments`, `assignment_audiences`, `assignment_recipients`, `assignment_attachments`, and `assignment_deadline_history`.[^1]
3. Create `assignment_submissions`, `assignment_submission_versions`, `submission_attachments`, and `late_submission_overrides`.[^1]
4. Create `assignment_feedback` and `assignment_feedback_revisions`.[^1]
5. Create `assignment_events`, then add foreign keys for latest submission pointers.[^1]
6. Add publish-validation triggers, recipient-resolution triggers, submission-mode triggers, late-policy triggers, feedback-revision triggers, and tenant RLS policies.[^1]

## 8. Security-sensitive columns

This module contains academic content, student work, and potentially family-visible feedback, so the sensitive fields are more about access scope and file exposure than classic finance-style secrets.[^1]

- `assignment_recipients.recipient_snapshot` contains student identity context and should not leak outside authorized teacher, student-self, or linked-parent views.[^1]
- `assignment_submission_versions.submission_text` may contain student-generated personal content and should be restricted to teacher scope, student self, and eligible parent visibility.[^1]
- `assignment_attachments.storage_key` and `submission_attachments.storage_key` should never be exposed directly and should be served through signed access control, because the PRD flags privacy and submission reliability risks around attachments.[^1]
- `assignment_feedback.draft_internal_notes` is especially sensitive and should never appear in student or parent APIs, because draft teacher notes must remain distinct from published feedback.[^1]
- `assignment_events.old_value`, `assignment_events.new_value`, `ip_address`, and `user_agent` are audit-sensitive operational fields and should be least-privilege only.[^1]

The next clean continuation is M7_13, or I can also refactor M7_11 and M7_12 into one cross-module canonical academic schema map before we proceed.

<div align="center">⁂</div>

[^1]: PRD_M7_12.txt

