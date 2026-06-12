<img src="https://r2cdn.perplexity.ai/pplx-full-logo-primary-dark%402x.png" style="height:64px;margin-right:32px"/>

# ok please generate for module M7_5

For M7_5, the right design is a **versioned academic scheduling schema** where subjects, teacher assignments, rooms, period structures, and timetable versions are modeled separately, because the PRD defines this module as the school’s scheduling engine and requires conflict-free scheduling with clear downstream consumption by attendance, gradebook, reporting, students, parents, and teachers.[^1]
I would also make M7_5 the authoritative owner of published schedules and academic teaching responsibility, because the PRD explicitly says attendance, gradebook, parent portal, and teacher dashboards must never create their own schedules and must consume them from academic structure, teacher assignments, and timetable data.[^1]

## Entity list

These are the production entities I would use for M7_5, based on the PRD’s required entities plus a small number of support tables needed for versioning, validation, and period-structure reuse.[^1]

- `subjects` — tenant-scoped subject master, reusable across academic years, with unique subject codes per tenant.[^1]
- `subject_groups` — logical subject grouping such as Languages, Sciences, and Arts.[^1]
- `subject_group_members` — many-to-many bridge between subjects and groups.[^1]
- `academic_teacher_assignments` — authoritative teaching-responsibility table for academic year, grade, section, and subject; I am naming it this way to avoid collision with the broader RBAC assignment model while keeping it equivalent to the PRD’s `teacher_assignments`.[^1]
- `rooms` — campus-scoped rooms with unique identifiers, capacity, and room type.[^1]
- `period_schemes` — reusable period structures so each timetable references a defined period structure, which the PRD requires.[^1]
- `periods` — ordered periods inside a scheme, including instructional periods, breaks, and lunch as special periods.[^1]
- `timetables` — versioned timetable headers for one academic year plus one grade-section.[^1]
- `timetable_slots` — day-by-day slot assignments for period, subject, teacher assignment, and room.[^1]
- `timetable_publication_history` — immutable publication trail so published versions can be audited and previous versions archived or restored.[^1]
- `timetable_validation_runs` — validation executions before review or publication.[^1]
- `timetable_validation_issues` — teacher, room, section, or missing-assignment issues discovered during validation.[^1]


## Table definitions

The DDL below is designed to support subject reuse across years, preserved teacher-assignment history, unique room identity per campus, non-overlapping period structures, draft-to-published timetable lifecycle, and publication-time conflict blocking.[^1]

```sql
CREATE EXTENSION IF NOT EXISTS pgcrypto;
CREATE EXTENSION IF NOT EXISTS citext;

CREATE TYPE subject_status AS ENUM ('active', 'inactive', 'archived');
CREATE TYPE teacher_assignment_status AS ENUM ('draft', 'assigned', 'updated', 'removed');
CREATE TYPE room_type AS ENUM ('classroom', 'lab', 'library', 'hall', 'music_room', 'art_room', 'sports_area', 'other');
CREATE TYPE room_status AS ENUM ('active', 'inactive', 'unavailable');
CREATE TYPE period_type AS ENUM ('instruction', 'break', 'lunch', 'assembly', 'activity');
CREATE TYPE period_scheme_status AS ENUM ('active', 'inactive', 'archived');
CREATE TYPE timetable_status AS ENUM ('draft', 'under_review', 'approved', 'published', 'archived');
CREATE TYPE validation_run_status AS ENUM ('passed', 'failed', 'warnings');
CREATE TYPE day_of_week_enum AS ENUM ('monday', 'tuesday', 'wednesday', 'thursday', 'friday', 'saturday', 'sunday');

CREATE TABLE subjects (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
    subject_code VARCHAR(50) NOT NULL,
    subject_name VARCHAR(150) NOT NULL,
    description TEXT,
    status subject_status NOT NULL DEFAULT 'active',
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID NULL REFERENCES users(id),
    updated_by UUID NULL REFERENCES users(id),
    CONSTRAINT uq_subject_code UNIQUE (tenant_id, subject_code)
);

CREATE TABLE subject_groups (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
    group_name VARCHAR(150) NOT NULL,
    description TEXT,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID NULL REFERENCES users(id),
    updated_by UUID NULL REFERENCES users(id),
    CONSTRAINT uq_subject_group_name UNIQUE (tenant_id, group_name)
);

CREATE TABLE subject_group_members (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
    subject_group_id UUID NOT NULL REFERENCES subject_groups(id) ON DELETE CASCADE,
    subject_id UUID NOT NULL REFERENCES subjects(id) ON DELETE RESTRICT,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID NULL REFERENCES users(id),
    CONSTRAINT uq_subject_group_member UNIQUE (tenant_id, subject_group_id, subject_id)
);

CREATE TABLE academic_teacher_assignments (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
    teacher_membership_id UUID NOT NULL,
    teacher_user_id UUID NOT NULL REFERENCES users(id) ON DELETE RESTRICT,
    academic_year_id UUID NOT NULL REFERENCES academic_years(id) ON DELETE RESTRICT,
    grade_level_id UUID NOT NULL REFERENCES grade_levels(id) ON DELETE RESTRICT,
    section_id UUID NOT NULL REFERENCES sections(id) ON DELETE RESTRICT,
    subject_id UUID NOT NULL REFERENCES subjects(id) ON DELETE RESTRICT,
    campus_id UUID NULL REFERENCES campuses(id) ON DELETE RESTRICT,
    status teacher_assignment_status NOT NULL DEFAULT 'draft',
    starts_on DATE NOT NULL DEFAULT CURRENT_DATE,
    ends_on DATE,
    notes TEXT,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID NULL REFERENCES users(id),
    updated_by UUID NULL REFERENCES users(id),
    FOREIGN KEY (teacher_membership_id, tenant_id)
        REFERENCES user_tenant_memberships(id, tenant_id)
        ON DELETE CASCADE,
    CONSTRAINT ck_teacher_assignment_dates CHECK (ends_on IS NULL OR ends_on >= starts_on)
);

CREATE TABLE rooms (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
    campus_id UUID NOT NULL REFERENCES campuses(id) ON DELETE RESTRICT,
    room_code VARCHAR(50) NOT NULL,
    room_name VARCHAR(150) NOT NULL,
    capacity INTEGER,
    room_type room_type NOT NULL,
    status room_status NOT NULL DEFAULT 'active',
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID NULL REFERENCES users(id),
    updated_by UUID NULL REFERENCES users(id),
    CONSTRAINT uq_room_code_per_campus UNIQUE (tenant_id, campus_id, room_code),
    CONSTRAINT ck_room_capacity CHECK (capacity IS NULL OR capacity > 0)
);

CREATE TABLE period_schemes (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
    academic_year_id UUID NOT NULL REFERENCES academic_years(id) ON DELETE RESTRICT,
    campus_id UUID NULL REFERENCES campuses(id) ON DELETE RESTRICT,
    scheme_name VARCHAR(150) NOT NULL,
    status period_scheme_status NOT NULL DEFAULT 'active',
    is_default BOOLEAN NOT NULL DEFAULT FALSE,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID NULL REFERENCES users(id),
    updated_by UUID NULL REFERENCES users(id),
    CONSTRAINT uq_period_scheme_name UNIQUE (tenant_id, academic_year_id, scheme_name)
);

CREATE TABLE periods (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
    period_scheme_id UUID NOT NULL REFERENCES period_schemes(id) ON DELETE CASCADE,
    name VARCHAR(100) NOT NULL,
    sequence_no INTEGER NOT NULL,
    start_time TIME NOT NULL,
    end_time TIME NOT NULL,
    period_type period_type NOT NULL DEFAULT 'instruction',
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID NULL REFERENCES users(id),
    updated_by UUID NULL REFERENCES users(id),
    CONSTRAINT ck_period_times CHECK (end_time > start_time),
    CONSTRAINT uq_period_sequence UNIQUE (period_scheme_id, sequence_no),
    CONSTRAINT uq_period_name UNIQUE (period_scheme_id, name)
);

CREATE TABLE timetables (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
    academic_year_id UUID NOT NULL REFERENCES academic_years(id) ON DELETE RESTRICT,
    grade_level_id UUID NOT NULL REFERENCES grade_levels(id) ON DELETE RESTRICT,
    section_id UUID NOT NULL REFERENCES sections(id) ON DELETE RESTRICT,
    period_scheme_id UUID NOT NULL REFERENCES period_schemes(id) ON DELETE RESTRICT,
    version_number INTEGER NOT NULL,
    status timetable_status NOT NULL DEFAULT 'draft',
    submitted_for_review_at TIMESTAMPTZ,
    approved_at TIMESTAMPTZ,
    approved_by UUID NULL REFERENCES users(id),
    published_at TIMESTAMPTZ,
    published_by UUID NULL REFERENCES users(id),
    archived_at TIMESTAMPTZ,
    archived_by UUID NULL REFERENCES users(id),
    notes TEXT,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID NULL REFERENCES users(id),
    updated_by UUID NULL REFERENCES users(id),
    CONSTRAINT ck_timetable_version CHECK (version_number > 0),
    CONSTRAINT uq_timetable_version UNIQUE (tenant_id, academic_year_id, grade_level_id, section_id, version_number)
);

CREATE TABLE timetable_slots (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
    timetable_id UUID NOT NULL REFERENCES timetables(id) ON DELETE CASCADE,
    day_of_week day_of_week_enum NOT NULL,
    period_id UUID NOT NULL REFERENCES periods(id) ON DELETE RESTRICT,
    subject_id UUID NULL REFERENCES subjects(id) ON DELETE RESTRICT,
    teacher_assignment_id UUID NULL REFERENCES academic_teacher_assignments(id) ON DELETE RESTRICT,
    room_id UUID NULL REFERENCES rooms(id) ON DELETE RESTRICT,
    slot_note VARCHAR(255),
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID NULL REFERENCES users(id),
    updated_by UUID NULL REFERENCES users(id),
    CONSTRAINT uq_timetable_slot UNIQUE (timetable_id, day_of_week, period_id)
);

CREATE TABLE timetable_publication_history (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
    timetable_id UUID NOT NULL REFERENCES timetables(id) ON DELETE RESTRICT,
    version_number INTEGER NOT NULL,
    published_by UUID NOT NULL REFERENCES users(id),
    published_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    archive_previous_timetable_id UUID NULL REFERENCES timetables(id) ON DELETE SET NULL,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE TABLE timetable_validation_runs (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
    timetable_id UUID NOT NULL REFERENCES timetables(id) ON DELETE CASCADE,
    run_status validation_run_status NOT NULL,
    total_issues INTEGER NOT NULL DEFAULT 0,
    blocking_issues INTEGER NOT NULL DEFAULT 0,
    executed_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID NULL REFERENCES users(id)
);

CREATE TABLE timetable_validation_issues (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
    validation_run_id UUID NOT NULL REFERENCES timetable_validation_runs(id) ON DELETE CASCADE,
    timetable_id UUID NOT NULL REFERENCES timetables(id) ON DELETE CASCADE,
    issue_code VARCHAR(100) NOT NULL,
    issue_type VARCHAR(100) NOT NULL,
    is_blocking BOOLEAN NOT NULL DEFAULT TRUE,
    day_of_week day_of_week_enum,
    period_id UUID NULL REFERENCES periods(id) ON DELETE SET NULL,
    teacher_assignment_id UUID NULL REFERENCES academic_teacher_assignments(id) ON DELETE SET NULL,
    room_id UUID NULL REFERENCES rooms(id) ON DELETE SET NULL,
    slot_id UUID NULL REFERENCES timetable_slots(id) ON DELETE SET NULL,
    issue_message TEXT NOT NULL,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);
```


## Constraints and indexes

The critical rules for M7_5 are unique subject codes, preserved teacher-assignment history, non-overlapping periods inside a period structure, unique room identity within campus, one published timetable version per grade-section, and blocked publication when teacher, room, or section conflicts exist.[^1]
Because the PRD allows draft creation and review before publication, I would enforce hard uniqueness for section-slot occupancy inside one timetable version, but enforce teacher, room, and cross-version conflicts through validation runs plus a publish transaction instead of blocking drafts too early.[^1]

**Constraints**

- Use a partial unique index so only one timetable version is `published` for a tenant plus academic year plus grade plus section at a time.[^1]
- Add a trigger on `periods` to reject overlapping times within the same `period_scheme_id`, because period timings must not overlap.[^1]
- Add a publish-time trigger or service transaction that blocks publication when the latest validation run contains blocking issues, because validation must occur before publication.[^1]
- Add a trigger that archives the previously published version when a new version is published, because publishing must automatically archive the previously active version.[^1]
- Add a trigger that blocks changes to timetables for archived academic years, because archived academic years cannot receive timetable changes.[^1]
- Add a trigger that blocks archiving or inactivating a subject when active published timetable slots still reference it, because the PRD explicitly calls out that removal should be blocked while an active timetable exists.[^1]

```sql
CREATE UNIQUE INDEX uq_one_published_timetable
ON timetables (tenant_id, academic_year_id, grade_level_id, section_id)
WHERE status = 'published';

CREATE INDEX idx_subjects_tenant_status
ON subjects (tenant_id, status, subject_name);

CREATE INDEX idx_subject_group_members_subject
ON subject_group_members (tenant_id, subject_id, subject_group_id);

CREATE INDEX idx_teacher_assignments_scope
ON academic_teacher_assignments (
    tenant_id, academic_year_id, grade_level_id, section_id, subject_id, teacher_user_id, status
);

CREATE INDEX idx_rooms_campus_status
ON rooms (tenant_id, campus_id, status, room_code);

CREATE INDEX idx_periods_scheme_sequence
ON periods (tenant_id, period_scheme_id, sequence_no);

CREATE INDEX idx_timetables_lookup
ON timetables (tenant_id, academic_year_id, grade_level_id, section_id, status, version_number DESC);

CREATE INDEX idx_timetable_slots_section_view
ON timetable_slots (tenant_id, timetable_id, day_of_week, period_id);

CREATE INDEX idx_timetable_slots_teacher_conflict
ON timetable_slots (tenant_id, teacher_assignment_id, day_of_week, period_id);

CREATE INDEX idx_timetable_slots_room_conflict
ON timetable_slots (tenant_id, room_id, day_of_week, period_id);

CREATE INDEX idx_validation_runs_recent
ON timetable_validation_runs (tenant_id, timetable_id, executed_at DESC);

CREATE INDEX idx_validation_issues_blocking
ON timetable_validation_issues (tenant_id, timetable_id, is_blocking, issue_type);
```


## Samples and queries

These samples reflect the main PRD workflow: create subjects, assign teachers, define period structure, draft a timetable version, and add slots that can later be validated and published.[^1]

```sql
INSERT INTO subjects (
    id, tenant_id, subject_code, subject_name, status, created_by
) VALUES
(
    '81000000-0000-0000-0000-000000000001',
    '10000000-0000-0000-0000-000000000001',
    'MATH',
    'Mathematics',
    'active',
    '00000000-0000-0000-0000-000000000002'
),
(
    '81000000-0000-0000-0000-000000000002',
    '10000000-0000-0000-0000-000000000001',
    'ENG',
    'English',
    'active',
    '00000000-0000-0000-0000-000000000002'
);

INSERT INTO academic_teacher_assignments (
    id, tenant_id, teacher_membership_id, teacher_user_id, academic_year_id, grade_level_id, section_id, subject_id, status, created_by
) VALUES
(
    '82000000-0000-0000-0000-000000000001',
    '10000000-0000-0000-0000-000000000001',
    '62000000-0000-0000-0000-000000000001',
    '61000000-0000-0000-0000-000000000001',
    '53000000-0000-0000-0000-000000000001',
    '54000000-0000-0000-0000-000000000001',
    '55000000-0000-0000-0000-000000000001',
    '81000000-0000-0000-0000-000000000001',
    'assigned',
    '00000000-0000-0000-0000-000000000002'
);

INSERT INTO rooms (
    id, tenant_id, campus_id, room_code, room_name, capacity, room_type, created_by
) VALUES
(
    '83000000-0000-0000-0000-000000000001',
    '10000000-0000-0000-0000-000000000001',
    '56000000-0000-0000-0000-000000000001',
    'R-101',
    'Grade 1 Room A',
    35,
    'classroom',
    '00000000-0000-0000-0000-000000000002'
);

INSERT INTO period_schemes (
    id, tenant_id, academic_year_id, scheme_name, status, is_default, created_by
) VALUES
(
    '84000000-0000-0000-0000-000000000001',
    '10000000-0000-0000-0000-000000000001',
    '53000000-0000-0000-0000-000000000001',
    'Standard Day',
    'active',
    TRUE,
    '00000000-0000-0000-0000-000000000002'
);

INSERT INTO periods (
    id, tenant_id, period_scheme_id, name, sequence_no, start_time, end_time, period_type, created_by
) VALUES
(
    '85000000-0000-0000-0000-000000000001',
    '10000000-0000-0000-0000-000000000001',
    '84000000-0000-0000-0000-000000000001',
    'Period 1',
    1,
    '08:30',
    '09:15',
    'instruction',
    '00000000-0000-0000-0000-000000000002'
),
(
    '85000000-0000-0000-0000-000000000002',
    '10000000-0000-0000-0000-000000000001',
    '84000000-0000-0000-0000-000000000001',
    'Break',
    2,
    '09:15',
    '09:30',
    'break',
    '00000000-0000-0000-0000-000000000002'
);

INSERT INTO timetables (
    id, tenant_id, academic_year_id, grade_level_id, section_id, period_scheme_id, version_number, status, created_by
) VALUES
(
    '86000000-0000-0000-0000-000000000001',
    '10000000-0000-0000-0000-000000000001',
    '53000000-0000-0000-0000-000000000001',
    '54000000-0000-0000-0000-000000000001',
    '55000000-0000-0000-0000-000000000001',
    '84000000-0000-0000-0000-000000000001',
    1,
    'draft',
    '00000000-0000-0000-0000-000000000002'
);
```

**Query patterns**

1. **Teacher weekly schedule** — the PRD explicitly requires teachers to view their weekly schedule and workload.[^1]
```sql
SELECT ts.day_of_week, p.sequence_no, p.name AS period_name, p.start_time, p.end_time,
       s.subject_name, r.room_name, t.grade_level_id, t.section_id
FROM timetable_slots ts
JOIN timetables t ON t.id = ts.timetable_id
JOIN periods p ON p.id = ts.period_id
LEFT JOIN subjects s ON s.id = ts.subject_id
LEFT JOIN academic_teacher_assignments ata ON ata.id = ts.teacher_assignment_id
LEFT JOIN rooms r ON r.id = ts.room_id
WHERE ts.tenant_id = $1
  AND ata.teacher_user_id = $2
  AND t.status = 'published'
  AND t.academic_year_id = $3
ORDER BY ts.day_of_week, p.sequence_no;
```

2. **Student timetable from current enrollment** — students and parents should only see active timetable versions.[^1]
```sql
SELECT ts.day_of_week, p.sequence_no, p.name, p.start_time, p.end_time,
       s.subject_name, r.room_name
FROM student_enrollments se
JOIN timetables t
  ON t.tenant_id = se.tenant_id
 AND t.academic_year_id = se.academic_year_id
 AND t.grade_level_id = se.grade_level_id
 AND t.section_id = se.section_id
 AND t.status = 'published'
JOIN timetable_slots ts
  ON ts.timetable_id = t.id AND ts.tenant_id = t.tenant_id
JOIN periods p ON p.id = ts.period_id
LEFT JOIN subjects s ON s.id = ts.subject_id
LEFT JOIN rooms r ON r.id = ts.room_id
WHERE se.tenant_id = $1
  AND se.student_id = $2
  AND se.status = 'active'
ORDER BY ts.day_of_week, p.sequence_no;
```

3. **Conflict review before publication** — the PRD requires teacher, room, and section conflict validation before publish.[^1]
```sql
SELECT issue_type, issue_code, issue_message, day_of_week, period_id, room_id, teacher_assignment_id
FROM timetable_validation_issues
WHERE tenant_id = $1
  AND timetable_id = $2
  AND is_blocking = TRUE
ORDER BY issue_type, created_at;
```

4. **Room utilization** — room utilization is a named screen/report use case in the PRD.[^1]
```sql
SELECT r.id, r.room_name, count(*) AS slot_count
FROM rooms r
LEFT JOIN timetable_slots ts
  ON ts.room_id = r.id AND ts.tenant_id = r.tenant_id
LEFT JOIN timetables t
  ON t.id = ts.timetable_id AND t.status = 'published'
WHERE r.tenant_id = $1
  AND r.campus_id = $2
GROUP BY r.id, r.room_name
ORDER BY slot_count DESC, r.room_name;
```


## Migration and security

M7_5 should be migrated after the onboarding, RBAC, SIS, and academic-structure foundation tables exist, because it depends on tenants, users, memberships, academic years, grades, sections, campuses, and student enrollments for timetable consumption.[^1]
The main security concern in this module is not sensitive health or finance data, but **operational control** over draft schedules, teacher workload visibility, and publication state, because drafts must remain staff-only and published schedules become the authoritative source for downstream modules.[^1]

**Migration sequence**

- Create enums, then `subjects`, `subject_groups`, and `subject_group_members`.[^1]
- Create `academic_teacher_assignments` and `rooms`, because timetable slots depend on both.[^1]
- Create `period_schemes` and `periods`, then add overlap-validation triggers for each scheme.[^1]
- Create `timetables` and `timetable_slots`, then add publish/archive triggers and the partial unique index for one published version per grade-section.[^1]
- Create `timetable_publication_history`, `timetable_validation_runs`, and `timetable_validation_issues`, then wire publish-time blocking to the latest validation result.[^1]
- Add RLS so students and parents can read only published timetable data that matches current enrollment or linked child access, while teachers can read only their own schedules and assigned-student timetable views allowed by policy.[^1]

**Security-sensitive columns**

- `academic_teacher_assignments.teacher_user_id`, `teacher_membership_id`, and assignment scope columns are staff-operational data and should be writable only by authorized academic admins.[^1]
- Draft and review-state data in `timetables.status`, `submitted_for_review_at`, `approved_at`, `published_at`, and publication actors should be protected because draft timetables are staff-only until publication.[^1]
- `timetable_validation_issues.issue_message` and related teacher or room references can reveal staff workload gaps or scheduling problems and should not be exposed to students or parents.[^1]
- `timetable_publication_history.published_by` and archive references are audit fields and should be retained immutably for rollback and investigation.[^1]

This M7_5 design gives you a production-ready, versioned timetable architecture with clean academic ownership, validation before publication, rollback safety, and a single schedule source for attendance, gradebook, parent views, and teacher dashboards.[^1]

<div align="center">⁂</div>

[^1]: PRD_M7_5.txt

