<img src="https://r2cdn.perplexity.ai/pplx-full-logo-primary-dark%402x.png" style="height:64px;margin-right:32px"/>

# continue with M7_13

For M7_13, I would model the module as **exam-term setup -> schedule drafting and conflict validation -> marksheet generation and verification -> publication batch activation -> controlled parent/student visibility**, because the PRD defines that formal workflow and requires publication discipline, schedule conflict prevention, marksheet readiness checks, and auditable release history.[^1]
This schema is designed to stay aligned with academic structure, teacher assignments, timetable logic, student enrollment, and gradebook publication controls rather than creating a disconnected exams subsystem.[^1]

## 1. Entity list

These are the core entities I recommend for M7_13.[^1]

- `exam_terms` — formal exam cycles linked to one academic year with lifecycle control, because exam terms belong to a single academic year and archived years cannot receive new changes.[^1]
- `exam_term_scopes` — optional scope rows for grade, section, or school-wide targeting, because terms may be grade-specific, section-specific, or school-wide by policy.[^1]
- `exam_schedules` — the exam schedule header per exam term, grade, section, subject, date, time, room, and version state.[^1]
- `exam_schedule_invigilators` — optional invigilation assignment rows, because teacher or invigilator overlap may need conflict prevention where enabled.[^1]
- `exam_schedule_conflicts` — validation findings for section, room, and staff overlaps before publication.[^1]
- `exam_schedule_versions` — preserved published schedule versions, because published exam schedules must preserve version history.[^1]
- `exam_subjects` — valid subject mappings for an exam term and academic context, because exam subjects must belong to valid subject and academic context records.[^1]
- `marksheets` — one marksheet per student per exam term and academic context with lifecycle status and versioning.[^1]
- `marksheet_subject_entries` — subject-level marks and readiness rows inside a marksheet, because marksheets cannot be ready unless required subject entries are complete.[^1]
- `marksheet_versions` — immutable readiness and publication snapshots, because published marksheets must not be silently overwritten and post-publication correction needs tracked revision.[^1]
- `publication_batches` — class-wise or school-wide result-release units with scheduling and activation state.[^1]
- `publication_batch_items` — the marksheets included in a publication batch and their item-level publication state.[^1]
- `exam_publication_history` — batch activation, withdrawal, and republication history, because publication scope and activation time must be auditable.[^1]
- `exam_audit_logs` — immutable workflow and support audit trail across create, edit, validate, publish, withdraw, and republish actions.[^1]


## 2. Table definitions

I am separating exam scheduling from marksheet publication so that schedule control, marks readiness, and result visibility each have explicit states and constraints, which matches the PRD’s governance-heavy workflow.[^1]
I am also using explicit batch and version tables because published schedules and published marksheets must preserve history and cannot be silently overwritten.[^1]

```sql
CREATE EXTENSION IF NOT EXISTS pgcrypto;
CREATE EXTENSION IF NOT EXISTS citext;
CREATE EXTENSION IF NOT EXISTS btree_gist;

CREATE TYPE exam_term_status AS ENUM (
  'draft', 'scheduled', 'active', 'completed', 'archived'
);

CREATE TYPE exam_schedule_status AS ENUM (
  'draft', 'under_validation', 'validation_failed', 'published', 'archived'
);

CREATE TYPE conflict_type AS ENUM (
  'section_overlap', 'room_overlap', 'staff_overlap', 'invalid_dependency'
);

CREATE TYPE conflict_status AS ENUM (
  'open', 'resolved', 'ignored'
);

CREATE TYPE marksheet_status AS ENUM (
  'pending', 'generated', 'verified', 'published', 'revised'
);

CREATE TYPE publication_type AS ENUM (
  'class_wise', 'school_wide'
);

CREATE TYPE publication_scope_level AS ENUM (
  'school', 'grade', 'section'
);

CREATE TYPE publication_batch_status AS ENUM (
  'draft', 'ready', 'scheduled', 'active', 'completed', 'withdrawn'
);

CREATE TYPE publication_item_status AS ENUM (
  'pending', 'eligible', 'published', 'withdrawn', 'revised'
);

-- Assumes upstream tables exist:
-- tenants, schools, users, academic_years, grade_levels, sections, subjects,
-- rooms, students, student_enrollments, teacher_subject_assignments.

CREATE TABLE exam_terms (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID NOT NULL REFERENCES tenants(id),
  school_id UUID NOT NULL REFERENCES schools(id),
  academic_year_id UUID NOT NULL REFERENCES academic_years(id),
  term_name VARCHAR(120) NOT NULL,
  description TEXT,
  start_date DATE NOT NULL,
  end_date DATE NOT NULL,
  status exam_term_status NOT NULL DEFAULT 'draft',
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  created_by UUID REFERENCES users(id),
  updated_by UUID REFERENCES users(id),
  CONSTRAINT ck_exam_term_dates CHECK (end_date >= start_date),
  CONSTRAINT uq_exam_term_name UNIQUE (tenant_id, school_id, academic_year_id, term_name)
);

CREATE TABLE exam_term_scopes (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID NOT NULL REFERENCES tenants(id),
  exam_term_id UUID NOT NULL REFERENCES exam_terms(id) ON DELETE CASCADE,
  grade_level_id UUID REFERENCES grade_levels(id),
  section_id UUID REFERENCES sections(id),
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  created_by UUID REFERENCES users(id),
  CONSTRAINT ck_exam_term_scope CHECK (
    grade_level_id IS NOT NULL OR section_id IS NOT NULL
  )
);

CREATE TABLE exam_subjects (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID NOT NULL REFERENCES tenants(id),
  exam_term_id UUID NOT NULL REFERENCES exam_terms(id) ON DELETE CASCADE,
  academic_year_id UUID NOT NULL REFERENCES academic_years(id),
  grade_level_id UUID NOT NULL REFERENCES grade_levels(id),
  section_id UUID REFERENCES sections(id),
  subject_id UUID NOT NULL REFERENCES subjects(id),
  teacher_assignment_id UUID REFERENCES teacher_subject_assignments(id),
  maximum_marks NUMERIC(8,2),
  status VARCHAR(20) NOT NULL DEFAULT 'active'
    CHECK (status IN ('active', 'archived')),
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  created_by UUID REFERENCES users(id),
  updated_by UUID REFERENCES users(id),
  CONSTRAINT uq_exam_subject UNIQUE (
    tenant_id, exam_term_id, academic_year_id, grade_level_id, section_id, subject_id
  )
);

CREATE TABLE exam_schedules (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID NOT NULL REFERENCES tenants(id),
  exam_term_id UUID NOT NULL REFERENCES exam_terms(id) ON DELETE CASCADE,
  academic_year_id UUID NOT NULL REFERENCES academic_years(id),
  grade_level_id UUID NOT NULL REFERENCES grade_levels(id),
  section_id UUID NOT NULL REFERENCES sections(id),
  subject_id UUID NOT NULL REFERENCES subjects(id),
  exam_subject_id UUID NOT NULL REFERENCES exam_subjects(id),
  exam_date DATE NOT NULL,
  start_time TIME NOT NULL,
  end_time TIME NOT NULL,
  room_id UUID REFERENCES rooms(id),
  status exam_schedule_status NOT NULL DEFAULT 'draft',
  version_no INTEGER NOT NULL DEFAULT 1,
  published_at TIMESTAMPTZ,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  created_by UUID REFERENCES users(id),
  updated_by UUID REFERENCES users(id),
  CONSTRAINT ck_exam_schedule_time CHECK (end_time > start_time),
  CONSTRAINT uq_exam_schedule_subject_slot UNIQUE (
    tenant_id, exam_term_id, grade_level_id, section_id, subject_id, exam_date, start_time
  )
);

CREATE TABLE exam_schedule_invigilators (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID NOT NULL REFERENCES tenants(id),
  exam_schedule_id UUID NOT NULL REFERENCES exam_schedules(id) ON DELETE CASCADE,
  user_id UUID NOT NULL REFERENCES users(id),
  role_code VARCHAR(30) NOT NULL DEFAULT 'invigilator',
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  created_by UUID REFERENCES users(id),
  CONSTRAINT uq_schedule_invigilator UNIQUE (tenant_id, exam_schedule_id, user_id)
);

CREATE TABLE exam_schedule_conflicts (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID NOT NULL REFERENCES tenants(id),
  exam_term_id UUID NOT NULL REFERENCES exam_terms(id) ON DELETE CASCADE,
  exam_schedule_id UUID REFERENCES exam_schedules(id) ON DELETE CASCADE,
  conflicting_schedule_id UUID REFERENCES exam_schedules(id) ON DELETE CASCADE,
  conflict_type conflict_type NOT NULL,
  conflict_status conflict_status NOT NULL DEFAULT 'open',
  conflict_message TEXT NOT NULL,
  detected_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  resolved_at TIMESTAMPTZ,
  resolved_by UUID REFERENCES users(id),
  created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE TABLE exam_schedule_versions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID NOT NULL REFERENCES tenants(id),
  exam_schedule_id UUID NOT NULL REFERENCES exam_schedules(id) ON DELETE CASCADE,
  version_no INTEGER NOT NULL,
  schedule_snapshot JSONB NOT NULL,
  published_at TIMESTAMPTZ,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  created_by UUID REFERENCES users(id),
  CONSTRAINT uq_exam_schedule_version UNIQUE (tenant_id, exam_schedule_id, version_no)
);

CREATE TABLE marksheets (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID NOT NULL REFERENCES tenants(id),
  school_id UUID NOT NULL REFERENCES schools(id),
  exam_term_id UUID NOT NULL REFERENCES exam_terms(id),
  student_id UUID NOT NULL REFERENCES students(id),
  academic_year_id UUID NOT NULL REFERENCES academic_years(id),
  grade_level_id UUID NOT NULL REFERENCES grade_levels(id),
  section_id UUID NOT NULL REFERENCES sections(id),
  student_enrollment_id UUID NOT NULL REFERENCES student_enrollments(id),
  status marksheet_status NOT NULL DEFAULT 'pending',
  generated_at TIMESTAMPTZ,
  verified_at TIMESTAMPTZ,
  published_at TIMESTAMPTZ,
  version_no INTEGER NOT NULL DEFAULT 1,
  current_version_id UUID,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  created_by UUID REFERENCES users(id),
  updated_by UUID REFERENCES users(id),
  CONSTRAINT uq_marksheet UNIQUE (tenant_id, exam_term_id, student_id)
);

CREATE TABLE marksheet_subject_entries (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID NOT NULL REFERENCES tenants(id),
  marksheet_id UUID NOT NULL REFERENCES marksheets(id) ON DELETE CASCADE,
  exam_subject_id UUID NOT NULL REFERENCES exam_subjects(id),
  subject_id UUID NOT NULL REFERENCES subjects(id),
  maximum_marks NUMERIC(8,2) NOT NULL,
  marks_obtained NUMERIC(8,2),
  grade_label VARCHAR(20),
  is_absent BOOLEAN NOT NULL DEFAULT FALSE,
  is_exempt BOOLEAN NOT NULL DEFAULT FALSE,
  verification_status VARCHAR(20) NOT NULL DEFAULT 'pending'
    CHECK (verification_status IN ('pending', 'verified', 'rejected')),
  remarks TEXT,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  created_by UUID REFERENCES users(id),
  updated_by UUID REFERENCES users(id),
  CONSTRAINT uq_marksheet_subject UNIQUE (tenant_id, marksheet_id, exam_subject_id),
  CONSTRAINT ck_marks_obtained_range CHECK (
    marks_obtained IS NULL OR (marks_obtained >= 0 AND marks_obtained <= maximum_marks)
  )
);

CREATE TABLE marksheet_versions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID NOT NULL REFERENCES tenants(id),
  marksheet_id UUID NOT NULL REFERENCES marksheets(id) ON DELETE CASCADE,
  version_no INTEGER NOT NULL,
  version_status marksheet_status NOT NULL,
  snapshot_json JSONB NOT NULL,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  created_by UUID REFERENCES users(id),
  CONSTRAINT uq_marksheet_version UNIQUE (tenant_id, marksheet_id, version_no)
);

CREATE TABLE publication_batches (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID NOT NULL REFERENCES tenants(id),
  school_id UUID NOT NULL REFERENCES schools(id),
  exam_term_id UUID NOT NULL REFERENCES exam_terms(id),
  publication_type publication_type NOT NULL,
  scope_level publication_scope_level NOT NULL,
  scope_value UUID,
  status publication_batch_status NOT NULL DEFAULT 'draft',
  scheduled_at TIMESTAMPTZ,
  activated_at TIMESTAMPTZ,
  withdrawn_at TIMESTAMPTZ,
  approved_by UUID REFERENCES users(id),
  approved_at TIMESTAMPTZ,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  created_by UUID REFERENCES users(id),
  updated_by UUID REFERENCES users(id)
);

CREATE TABLE publication_batch_items (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID NOT NULL REFERENCES tenants(id),
  publication_batch_id UUID NOT NULL REFERENCES publication_batches(id) ON DELETE CASCADE,
  marksheet_id UUID NOT NULL REFERENCES marksheets(id) ON DELETE RESTRICT,
  publication_status publication_item_status NOT NULL DEFAULT 'pending',
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  published_at TIMESTAMPTZ,
  withdrawn_at TIMESTAMPTZ,
  CONSTRAINT uq_publication_batch_item UNIQUE (tenant_id, publication_batch_id, marksheet_id)
);

CREATE TABLE exam_publication_history (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID NOT NULL REFERENCES tenants(id),
  school_id UUID NOT NULL REFERENCES schools(id),
  publication_batch_id UUID NOT NULL REFERENCES publication_batches(id) ON DELETE CASCADE,
  event_name VARCHAR(40) NOT NULL,
  event_snapshot JSONB NOT NULL,
  event_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  actor_user_id UUID REFERENCES users(id),
  created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE TABLE exam_audit_logs (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID NOT NULL REFERENCES tenants(id),
  school_id UUID NOT NULL REFERENCES schools(id),
  actor_user_id UUID REFERENCES users(id),
  entity_name VARCHAR(80) NOT NULL,
  entity_id UUID NOT NULL,
  action_name VARCHAR(80) NOT NULL,
  old_value JSONB,
  new_value JSONB,
  ip_address INET,
  user_agent TEXT,
  request_id UUID,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

ALTER TABLE marksheets
  ADD CONSTRAINT fk_marksheets_current_version
  FOREIGN KEY (current_version_id) REFERENCES marksheet_versions(id);
```


## 3. Constraints

The most important rules are exam-term immutability for archived years, schedule conflict prevention, marksheet readiness gating, controlled publication activation, and post-publication revision tracking.[^1]

- Every tenant-owned table includes `tenant_id`, because all exam terms, schedules, marksheets, and publication batches must belong to exactly one tenant and cannot be queried across tenants.[^1]
- A trigger should block inserts or updates to `exam_terms` when the linked academic year is archived, because archived academic years cannot receive new exam-term changes.[^1]
- A trigger should validate that `exam_subjects` align to valid academic context and authorized teacher assignments, because exam subjects must belong to official subject and academic records.[^1]
- A trigger should block `exam_schedules.status = 'published'` while open `exam_schedule_conflicts` exist, because timetable conflicts must be prevented before publication.[^1]
- A trigger should create `exam_schedule_versions` snapshots whenever a schedule is published or revised, because published schedule version history must be preserved.[^1]
- A trigger should block `marksheets.status = 'verified'` if any mandatory `marksheet_subject_entries` are incomplete or unverified, because marksheets cannot be ready when mandatory subject entries are incomplete.[^1]
- A trigger should block `publication_batches.status = 'active'` unless every required `publication_batch_item` points to a verified or publication-eligible marksheet, because publication must be blocked if required marksheets are incomplete.[^1]
- A trigger should prevent direct overwrite of already published marksheet data and instead create a new `marksheet_versions` row with controlled republication flow, because published marksheets must not be silently overwritten.[^1]
- Parent and student portal queries must check both active publication state and relationship/self scope, because unpublished or unrelated results must never be visible.[^1]


## 4. Indexes

These indexes support the PRD’s expected workloads around exam dashboards, conflict validation, marksheet readiness, publication batches, and published result visibility.[^1]

```sql
CREATE INDEX idx_exam_terms_year_status
ON exam_terms (tenant_id, school_id, academic_year_id, status, start_date);

CREATE INDEX idx_exam_subjects_context
ON exam_subjects (tenant_id, exam_term_id, academic_year_id, grade_level_id, section_id, subject_id);

CREATE INDEX idx_exam_schedules_term_context
ON exam_schedules (tenant_id, exam_term_id, grade_level_id, section_id, exam_date, start_time);

CREATE INDEX idx_exam_schedules_room_slot
ON exam_schedules (tenant_id, room_id, exam_date, start_time, end_time);

CREATE INDEX idx_exam_schedule_conflicts_status
ON exam_schedule_conflicts (tenant_id, exam_term_id, conflict_status, conflict_type, detected_at DESC);

CREATE INDEX idx_marksheets_term_status
ON marksheets (tenant_id, exam_term_id, grade_level_id, section_id, status, generated_at DESC);

CREATE INDEX idx_marksheets_student
ON marksheets (tenant_id, student_id, exam_term_id, published_at DESC);

CREATE INDEX idx_marksheet_subject_entries_marksheet
ON marksheet_subject_entries (tenant_id, marksheet_id, verification_status);

CREATE INDEX idx_publication_batches_term_status
ON publication_batches (tenant_id, exam_term_id, status, scheduled_at, activated_at);

CREATE INDEX idx_publication_batch_items_batch_status
ON publication_batch_items (tenant_id, publication_batch_id, publication_status);

CREATE INDEX idx_exam_publication_history_batch
ON exam_publication_history (tenant_id, publication_batch_id, event_at DESC);

CREATE INDEX idx_exam_audit_logs_entity
ON exam_audit_logs (tenant_id, school_id, entity_name, entity_id, created_at DESC);
```


## 5. Sample records

These examples show an exam term, one scheduled exam, one generated marksheet, and one publication batch.[^1]

```sql
INSERT INTO exam_terms (
  id, tenant_id, school_id, academic_year_id, term_name, description,
  start_date, end_date, status, created_at, updated_at
) VALUES (
  'd1000000-0000-0000-0000-000000000001',
  '10000000-0000-0000-0000-000000000001',
  '20000000-0000-0000-0000-000000000001',
  '30000000-0000-0000-0000-000000000001',
  'Mid Term 1',
  'First formal exam cycle for Term 1',
  '2026-08-10',
  '2026-08-20',
  'scheduled',
  now(), now()
);

INSERT INTO exam_subjects (
  id, tenant_id, exam_term_id, academic_year_id, grade_level_id, section_id,
  subject_id, maximum_marks, status, created_at, updated_at
) VALUES (
  'd1000000-0000-0000-0000-000000000002',
  '10000000-0000-0000-0000-000000000001',
  'd1000000-0000-0000-0000-000000000001',
  '30000000-0000-0000-0000-000000000001',
  '40000000-0000-0000-0000-000000000007',
  '50000000-0000-0000-0000-000000000012',
  '60000000-0000-0000-0000-000000000003',
  100.00,
  'active',
  now(), now()
);

INSERT INTO exam_schedules (
  id, tenant_id, exam_term_id, academic_year_id, grade_level_id, section_id,
  subject_id, exam_subject_id, exam_date, start_time, end_time, room_id,
  status, version_no, created_at, updated_at
) VALUES (
  'd1000000-0000-0000-0000-000000000003',
  '10000000-0000-0000-0000-000000000001',
  'd1000000-0000-0000-0000-000000000001',
  '30000000-0000-0000-0000-000000000001',
  '40000000-0000-0000-0000-000000000007',
  '50000000-0000-0000-0000-000000000012',
  '60000000-0000-0000-0000-000000000003',
  'd1000000-0000-0000-0000-000000000002',
  '2026-08-12',
  '09:00',
  '12:00',
  '90000000-0000-0000-0000-000000000001',
  'published',
  1,
  now(), now()
);

INSERT INTO marksheets (
  id, tenant_id, school_id, exam_term_id, student_id, academic_year_id,
  grade_level_id, section_id, student_enrollment_id, status, generated_at,
  version_no, created_at, updated_at
) VALUES (
  'd1000000-0000-0000-0000-000000000004',
  '10000000-0000-0000-0000-000000000001',
  '20000000-0000-0000-0000-000000000001',
  'd1000000-0000-0000-0000-000000000001',
  '70000000-0000-0000-0000-000000000101',
  '30000000-0000-0000-0000-000000000001',
  '40000000-0000-0000-0000-000000000007',
  '50000000-0000-0000-0000-000000000012',
  '80000000-0000-0000-0000-000000000101',
  'generated',
  now(),
  1,
  now(), now()
);

INSERT INTO publication_batches (
  id, tenant_id, school_id, exam_term_id, publication_type, scope_level,
  scope_value, status, scheduled_at, created_at, updated_at
) VALUES (
  'd1000000-0000-0000-0000-000000000005',
  '10000000-0000-0000-0000-000000000001',
  '20000000-0000-0000-0000-000000000001',
  'd1000000-0000-0000-0000-000000000001',
  'class_wise',
  'section',
  '50000000-0000-0000-0000-000000000012',
  'scheduled',
  now() + interval '2 hours',
  now(), now()
);
```


## 6. Query patterns

These query patterns match the PRD’s dashboards for active exam terms, conflict resolution, marksheet readiness, class-wise publication, and published result visibility.[^1]

1. Active exam schedules for one section.[^1]
```sql
SELECT es.id, es.exam_date, es.start_time, es.end_time, es.room_id, s.name AS subject_name
FROM exam_schedules es
JOIN subjects s ON s.id = es.subject_id
WHERE es.tenant_id = $1
  AND es.exam_term_id = $2
  AND es.section_id = $3
  AND es.status = 'published'
ORDER BY es.exam_date, es.start_time;
```

2. Open conflict dashboard for an exam term.[^1]
```sql
SELECT conflict_type, count(*) AS total_open
FROM exam_schedule_conflicts
WHERE tenant_id = $1
  AND exam_term_id = $2
  AND conflict_status = 'open'
GROUP BY conflict_type
ORDER BY total_open DESC;
```

3. Marksheet readiness by section, because incomplete marksheets must be identified before publication.[^1]
```sql
SELECT grade_level_id, section_id, status, count(*) AS total
FROM marksheets
WHERE tenant_id = $1
  AND exam_term_id = $2
GROUP BY grade_level_id, section_id, status
ORDER BY grade_level_id, section_id, status;
```

4. Publication batch activation readiness.[^1]
```sql
SELECT
  pb.id,
  pb.status,
  count(pbi.id) AS total_items,
  count(*) FILTER (WHERE m.status IN ('verified','published','revised')) AS eligible_items
FROM publication_batches pb
JOIN publication_batch_items pbi
  ON pbi.publication_batch_id = pb.id AND pbi.tenant_id = pb.tenant_id
JOIN marksheets m
  ON m.id = pbi.marksheet_id AND m.tenant_id = pbi.tenant_id
WHERE pb.tenant_id = $1
  AND pb.id = $2
GROUP BY pb.id, pb.status;
```

5. Parent view of published marksheets for linked children only, because parents may only see linked children after publication is active.[^1]
```sql
SELECT m.id, et.term_name, m.student_id, m.published_at, pb.activated_at
FROM parent_student_links psl
JOIN marksheets m
  ON m.student_id = psl.student_id AND m.tenant_id = psl.tenant_id
JOIN publication_batch_items pbi
  ON pbi.marksheet_id = m.id AND pbi.tenant_id = m.tenant_id
JOIN publication_batches pb
  ON pb.id = pbi.publication_batch_id AND pb.tenant_id = pbi.tenant_id
JOIN exam_terms et
  ON et.id = m.exam_term_id AND et.tenant_id = m.tenant_id
WHERE psl.tenant_id = $1
  AND psl.parent_user_id = $2
  AND psl.status = 'active'
  AND pb.status = 'active'
  AND pbi.publication_status = 'published'
ORDER BY m.published_at DESC;
```


## 7. Migration sequence

The right migration order is term and subject setup first, then schedule and conflict structures, then marksheets and versions, then publication batches and audit history, because publishing depends on both validated scheduling and complete marksheet readiness.[^1]

1. Create enums and `exam_terms`, `exam_term_scopes`, and `exam_subjects`.[^1]
2. Create `exam_schedules`, `exam_schedule_invigilators`, `exam_schedule_conflicts`, and `exam_schedule_versions`.[^1]
3. Create `marksheets`, `marksheet_subject_entries`, and `marksheet_versions`.[^1]
4. Create `publication_batches`, `publication_batch_items`, and `exam_publication_history`.[^1]
5. Create `exam_audit_logs`, then add readiness, validation, snapshot, and publication-state triggers plus tenant RLS policies.[^1]

## 8. Security-sensitive columns

The most sensitive data in this module is student result data, publication timing, version snapshots, and audit history, because unauthorized pre-publication access is explicitly identified as a privacy risk and zero such incidents is a stated success metric.[^1]

- `marksheet_subject_entries.marks_obtained`, `grade_label`, and `remarks` are student academic outcome data and must be visible only to authorized staff, linked parents after publication, and the student self after publication.[^1]
- `marksheet_versions.snapshot_json` contains historical academic records and should be treated as immutable, least-privilege data.[^1]
- `publication_batches.scheduled_at`, `activated_at`, and `withdrawn_at` are sensitive governance timestamps and should be restricted to authorized academic admins.[^1]
- `exam_schedule_versions.schedule_snapshot` may expose prior published timetable states and should be access-controlled for audit and dispute handling only.[^1]
- `exam_audit_logs.old_value`, `new_value`, `ip_address`, and `user_agent` are sensitive audit fields and should be protected accordingly.[^1]

Next in sequence is M7_14, and that one should connect cleanly to both transport billing hooks and parent-child visibility rules.[^2]
<span style="display:none">[^10][^11][^12][^13][^14][^15][^3][^4][^5][^6][^7][^8][^9]</span>

<div align="center">⁂</div>

[^1]: PRD_M7_13.txt

[^2]: PRD_M7_14_15.txt

[^3]: https://www.scribd.com/document/685861239/School-Management-System-Modules-Documentation

[^4]: https://www.edu.gov.mb.ca/k12/assess/report_cards/docs/eng_my.pdf

[^5]: https://www.youtube.com/watch?v=ZPzn3HwPbDQ

[^6]: https://www.markerseven.com/work/college-preparatory-school

[^7]: https://www.educationforallinindia.com/State-report-cards-2007-08-dise-nuepa.pdf

[^8]: https://codecanyon.net/item/ramom-school-subscription-saas/45121444

[^9]: https://www.studocu.com/en-gb/document/london-metropolitan-university/knowledge-in-policing/school-management-system-project-documen/11876052

[^10]: https://www.edu.gov.mb.ca/k12/assess/report_cards/index.html

[^11]: https://www.tcetmumbai.in/IT/Modules/BE/CC_Syllabus_Detailing.pdf

[^12]: https://pt.slideshare.net/slideshow/neweduk8/1041018

[^13]: PRD_M7_12.txt

[^14]: PRD_M7_16.txt

[^15]: PRD_M7_10.txt

