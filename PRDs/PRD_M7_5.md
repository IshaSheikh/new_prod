# Product Requirements Document (PRD)

# Module 7.5 – Academic Structure and Timetable

---

# 1. Module Objective

The Academic Structure and Timetable module defines how teaching and learning activities are organized within a school.

It establishes the academic framework that connects:

* Academic years
* Grades and sections
* Subjects
* Teachers
* Classrooms
* Periods
* Timetables

This module acts as the operational scheduling engine for the school and becomes the foundation for:

* Attendance
* Lesson delivery
* Gradebook
* Exams
* Parent communication
* Academic reporting

The module must ensure conflict-free scheduling while providing clear visibility to students, parents, teachers, and administrators.

---

# 2. Why Schools Need It

Schools operate around schedules.

Without a structured timetable system:

* Teachers get double-booked
* Rooms conflict
* Students miss classes
* Attendance becomes inaccurate
* Academic reporting becomes unreliable

Schools need this module to:

### Standardize Academic Operations

Ensure every class has a defined schedule.

### Improve Teacher Utilization

Track and optimize teaching workload.

### Reduce Administrative Effort

Avoid manual timetable creation and updates.

### Improve Parent Transparency

Allow parents to see student schedules.

### Improve Student Experience

Provide accurate, up-to-date timetables.

### Enable Downstream Modules

Attendance and gradebook depend on timetable structure.

---

# 3. Primary Users

## School Admin

Primary timetable owner.

Responsibilities:

* Configure academic structure
* Create subjects
* Create timetable versions
* Publish schedules

---

## Principal

Academic oversight.

Responsibilities:

* Review workload
* Approve timetable versions
* Monitor utilization

---

## Teacher

Instructional user.

Responsibilities:

* View timetable
* Access assigned subjects
* View teaching workload

---

## Student

Learner.

Responsibilities:

* View class schedule

---

## Parent

Guardian.

Responsibilities:

* View child's timetable

---

## Super Admin

Platform oversight.

Responsibilities:

* Audit system usage
* Support schools

---

# 4. User Stories

## Subject Management

### US-001

As a School Admin,

I want to create subjects,

so they can be assigned to classes.

---

### US-002

As a School Admin,

I want to organize subjects into groups,

so reporting and curriculum management become easier.

---

## Teacher Assignment

### US-003

As a School Admin,

I want to assign teachers to subjects,

so teaching responsibilities are defined.

---

### US-004

As a Teacher,

I want to see my assigned subjects,

so I know my responsibilities.

---

## Timetable Creation

### US-005

As a School Admin,

I want to create timetable drafts,

so schedules can be reviewed before publication.

---

### US-006

As a School Admin,

I want conflict validation,

so teachers and rooms are not double-booked.

---

### US-007

As a Principal,

I want to review timetable versions,

so schedules can be approved before publishing.

---

## Timetable Consumption

### US-008

As a Student,

I want to see my timetable,

so I know where to be and when.

---

### US-009

As a Parent,

I want to see my child's timetable,

so I can monitor academic schedules.

---

### US-010

As a Teacher,

I want to view my weekly schedule,

so I can manage my workload.

---

# 5. Business Rules

## Tenant Isolation

BR-001

All academic structure records must belong to a single tenant.

---

BR-002

No academic data may be visible across tenants.

---

## Subject Rules

BR-003

Subject codes must be unique within a tenant.

---

BR-004

Subjects may be reused across academic years.

---

BR-005

Subjects may belong to a subject group.

Examples:

* Languages
* Sciences
* Arts

---

## Teacher Assignment Rules

BR-006

Teachers must exist before assignment.

---

BR-007

Assignments must reference:

* Academic Year
* Grade
* Section
* Subject

---

BR-008

A teacher may teach multiple subjects.

---

BR-009

Multiple teachers may teach the same subject.

---

BR-010

Teacher assignment history must be preserved.

---

## Period Rules

BR-011

Period timings must not overlap.

---

BR-012

Each timetable must use a defined period structure.

---

BR-013

Breaks and lunch periods are represented as special periods.

---

## Room Rules

BR-014

Rooms must be uniquely identifiable within a campus.

---

BR-015

Room capacity must be configurable.

---

BR-016

A room cannot be booked for overlapping timetable slots.

---

## Timetable Rules

BR-017

Timetables are versioned.

---

BR-018

Only one timetable version can be active per grade-section at a time.

---

BR-019

Students only see active timetable versions.

---

BR-020

Draft timetables are visible only to authorized staff.

---

BR-021

Publishing a timetable automatically archives the previously active version.

---

## Conflict Rules

BR-022

A teacher cannot be assigned to overlapping slots.

---

BR-023

A room cannot be assigned to overlapping slots.

---

BR-024

A section cannot have overlapping periods.

---

BR-025

Validation must occur before publication.

---

## Academic Calendar Rules

BR-026

Timetables must belong to an academic year.

---

BR-027

Archived academic years cannot receive timetable changes.

---

# 6. State Machine

## Timetable Lifecycle

```text
Draft
  |
  v
Under Review
  |
  v
Approved
  |
  v
Published
  |
  +------> Archived
```

---

## Teacher Assignment Lifecycle

```text
Draft
  |
  v
Assigned
  |
  +------> Updated
  |
  +------> Removed
```

---

## Subject Lifecycle

```text
Active
  |
  +------> Inactive
  |
  +------> Archived
```

---

# 7. Required Data Entities

## subjects

| Field        | Type    |
| ------------ | ------- |
| id           | UUID    |
| tenant_id    | UUID    |
| subject_code | VARCHAR |
| subject_name | VARCHAR |
| description  | TEXT    |
| status       | ENUM    |

---

## subject_groups

| Field       | Type    |
| ----------- | ------- |
| id          | UUID    |
| tenant_id   | UUID    |
| group_name  | VARCHAR |
| description | TEXT    |

---

## subject_group_members

| Field            | Type |
| ---------------- | ---- |
| id               | UUID |
| tenant_id        | UUID |
| subject_group_id | UUID |
| subject_id       | UUID |

---

## teacher_assignments

| Field            | Type |
| ---------------- | ---- |
| id               | UUID |
| tenant_id        | UUID |
| teacher_user_id  | UUID |
| academic_year_id | UUID |
| grade_level_id   | UUID |
| section_id       | UUID |
| subject_id       | UUID |
| status           | ENUM |

---

## rooms

| Field     | Type    |
| --------- | ------- |
| id        | UUID    |
| tenant_id | UUID    |
| campus_id | UUID    |
| room_code | VARCHAR |
| room_name | VARCHAR |
| capacity  | INTEGER |
| room_type | ENUM    |

---

## periods

| Field       | Type    |
| ----------- | ------- |
| id          | UUID    |
| tenant_id   | UUID    |
| name        | VARCHAR |
| start_time  | TIME    |
| end_time    | TIME    |
| period_type | ENUM    |

---

## timetables

| Field            | Type      |
| ---------------- | --------- |
| id               | UUID      |
| tenant_id        | UUID      |
| academic_year_id | UUID      |
| grade_level_id   | UUID      |
| section_id       | UUID      |
| version_number   | INTEGER   |
| status           | ENUM      |
| published_at     | TIMESTAMP |

---

## timetable_slots

| Field                 | Type |
| --------------------- | ---- |
| id                    | UUID |
| tenant_id             | UUID |
| timetable_id          | UUID |
| day_of_week           | ENUM |
| period_id             | UUID |
| subject_id            | UUID |
| teacher_assignment_id | UUID |
| room_id               | UUID |

---

## timetable_publication_history

| Field          | Type      |
| -------------- | --------- |
| id             | UUID      |
| tenant_id      | UUID      |
| timetable_id   | UUID      |
| version_number | INTEGER   |
| published_by   | UUID      |
| published_at   | TIMESTAMP |

---

# 8. Permissions Matrix

| Action                 | Super Admin | School Admin | Principal | Teacher           | Accountant | Parent    | Student |
| ---------------------- | ----------- | ------------ | --------- | ----------------- | ---------- | --------- | ------- |
| View Subjects          | ✓           | ✓            | ✓         | ✓                 | ✗          | ✗         | ✗       |
| Create Subjects        | ✓           | ✓            | ✗         | ✗                 | ✗          | ✗         | ✗       |
| Edit Subjects          | ✓           | ✓            | Limited   | ✗                 | ✗          | ✗         | ✗       |
| Assign Teachers        | ✓           | ✓            | ✓         | ✗                 | ✗          | ✗         | ✗       |
| Manage Rooms           | ✓           | ✓            | ✗         | ✗                 | ✗          | ✗         | ✗       |
| Create Timetable       | ✓           | ✓            | ✗         | ✗                 | ✗          | ✗         | ✗       |
| Review Timetable       | ✓           | ✓            | ✓         | ✗                 | ✗          | ✗         | ✗       |
| Publish Timetable      | ✓           | ✓            | ✓         | ✗                 | ✗          | ✗         | ✗       |
| View Teacher Schedule  | ✓           | ✓            | ✓         | Self              | ✗          | ✗         | ✗       |
| View Student Timetable | ✓           | ✓            | ✓         | Assigned Students | ✗          | Own Child | Self    |

---

# 9. APIs Needed

## Subject APIs

### POST /subjects

Create subject

---

### GET /subjects

List subjects

---

### PATCH /subjects/{id}

Update subject

---

### DELETE /subjects/{id}

Archive subject

---

## Subject Group APIs

### POST /subject-groups

Create group

---

### GET /subject-groups

List groups

---

## Teacher Assignment APIs

### POST /teacher-assignments

Assign teacher

---

### GET /teacher-assignments

List assignments

---

### PATCH /teacher-assignments/{id}

Update assignment

---

## Room APIs

### POST /rooms

Create room

---

### GET /rooms

List rooms

---

### PATCH /rooms/{id}

Update room

---

## Period APIs

### POST /periods

Create period

---

### GET /periods

List periods

---

### PATCH /periods/{id}

Update period

---

## Timetable APIs

### POST /timetables

Create timetable draft

---

### GET /timetables

List timetables

---

### GET /timetables/{id}

Get timetable

---

### PATCH /timetables/{id}

Update timetable

---

### POST /timetables/{id}/validate

Run conflict validation

---

### POST /timetables/{id}/submit-review

Submit for review

---

### POST /timetables/{id}/approve

Approve timetable

---

### POST /timetables/{id}/publish

Publish timetable

---

### POST /timetables/{id}/archive

Archive timetable

---

## Timetable View APIs

### GET /teacher/my-timetable

Teacher schedule

---

### GET /student/my-timetable

Student schedule

---

### GET /parent/child/{studentId}/timetable

Parent view

---

# 10. Screens Needed

## Subject Management Screen

Features:

* Subject creation
* Subject grouping
* Subject search
* Status management

---

## Teacher Assignment Screen

Features:

* Assign teacher
* Workload view
* Conflict detection

---

## Room Management Screen

Features:

* Create rooms
* Manage capacities
* Room utilization reports

---

## Period Configuration Screen

Features:

* Create periods
* Break periods
* Lunch periods

---

## Timetable Builder Screen

Features:

* Drag-and-drop scheduling
* Conflict indicators
* Auto-save drafts

---

## Timetable Validation Screen

Displays:

* Teacher conflicts
* Room conflicts
* Section conflicts
* Missing assignments

---

## Timetable Review Screen

Displays:

* Draft vs Active comparison
* Approval workflow

---

## Student Timetable Screen

Displays:

* Daily view
* Weekly view

---

## Teacher Schedule Screen

Displays:

* Daily schedule
* Weekly schedule
* Workload summary

---

# 11. Edge Cases

### EC-001

Teacher teaches in multiple campuses.

System validates travel and scheduling conflicts.

---

### EC-002

Teacher resigns mid-academic year.

Replacement assignment required.

Historical timetable preserved.

---

### EC-003

Room becomes unavailable.

Affected timetable slots flagged.

---

### EC-004

Academic year changes.

New timetable version required.

---

### EC-005

Period timings modified after publication.

Conflict validation automatically reruns.

---

### EC-006

Teacher assigned to overlapping slots.

Publication blocked.

---

### EC-007

Room double-booked.

Publication blocked.

---

### EC-008

Timetable published accidentally.

Previous version can be restored.

---

### EC-009

Subject removed while active timetable exists.

Removal blocked.

---

### EC-010

Student changes section mid-year.

Timetable automatically follows current enrollment.

Historical timetable references remain intact.

---

# 12. Risks

## Scheduling Risk

Timetable conflicts create operational disruption.

**Mitigation**

* Automated validation engine
* Publication blocking

---

## Data Integrity Risk

Incorrect teacher assignments.

**Mitigation**

* Assignment validation
* Approval workflow

---

## Adoption Risk

Timetable builder becomes too complex.

**Mitigation**

* Guided creation
* Templates
* Bulk scheduling

---

## Performance Risk

Large schools generating thousands of timetable slots.

**Mitigation**

* Indexed queries
* Slot caching
* Read replicas

---

## Change Management Risk

Frequent timetable changes confuse users.

**Mitigation**

* Versioning
* Publication history
* Change notifications

---

# 13. Success Metrics

## Operational Metrics

* Timetable creation time < 4 hours for average school
* Conflict detection accuracy > 99%
* Publication success rate > 98%

---

## Product Metrics

* Teacher timetable load time < 2 seconds
* Student timetable load time < 2 seconds
* Room utilization visibility > 95%

---

## Business Metrics

* Reduction in timetable administration effort > 60%
* Reduction in scheduling conflicts > 90%
* Increased timetable visibility for parents and students

---

## Data Quality Metrics

* Double-booked teachers = 0
* Double-booked rooms = 0
* Invalid timetable publications = 0

---

# 14. Out of Scope

The following are excluded from MVP:

* AI timetable generation
* Constraint-based timetable optimization
* Substitute teacher scheduling
* Exam timetable scheduling
* University-style course registration
* Dynamic room booking outside academics
* Real-time timetable change notifications
* Staff leave-aware scheduling
* Curriculum planning
* Lesson planning
* Resource reservation system
* Calendar synchronization with external systems

---

# Architecture Recommendations for a Solo Founder

## MVP First

Build:

1. Subjects
2. Teacher Assignments
3. Rooms
4. Periods
5. Timetable Drafting
6. Conflict Validation
7. Timetable Publishing
8. Student/Teacher Views

Do **not** build automated timetable generation initially.

---

## Timetable Version Strategy

Never update published timetable records directly.

Use:

```text
Version 1 → Published
Version 2 → Draft
Version 2 → Published
Version 1 → Archived
```

This provides complete auditability and rollback capability.

---

## Database Performance Strategy

For PostgreSQL:

```sql
INDEX (
    tenant_id,
    academic_year_id,
    section_id,
    day_of_week
)
```

on timetable_slots.

Also index:

```sql
teacher_user_id
room_id
period_id
```

to support conflict detection efficiently.

---

## Critical Architectural Principle

Attendance, gradebook, parent portal, and teacher dashboards must **never create their own schedules**.

All scheduling information must come from:

```text
Academic Structure
        ↓
Teacher Assignments
        ↓
Timetable
        ↓
Attendance / Grades / Parent Portal
```

This ensures a single source of truth for all academic operations across the platform.
