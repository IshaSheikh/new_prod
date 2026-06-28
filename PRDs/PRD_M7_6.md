# Product Requirements Document (PRD)

# Module 7.6 – Attendance Management

---

# 1. Module Objective

The Attendance Management module records, tracks, validates, and reports student attendance across the school.

It enables schools to:

* Capture daily attendance
* Capture period-wise attendance
* Track lateness and absences
* Notify parents automatically
* Monitor attendance trends
* Manage attendance corrections through approval workflows
* Produce operational and compliance reports

Attendance is one of the most frequently used modules in a K-12 platform and serves as a critical data source for:

* Student performance monitoring
* Parent communication
* Government reporting
* Academic eligibility
* Fee and transportation analysis
* School operational KPIs

The module must provide a secure, auditable, and scalable attendance workflow while minimizing teacher effort.

---

# 2. Why Schools Need It

Attendance is a core operational process in every school.

Without a structured attendance system:

* Teachers spend excessive time on manual registers
* Parents receive delayed absence information
* Administrators cannot identify chronic absenteeism
* Attendance reports become unreliable
* Compliance reporting becomes difficult

Schools need this module to:

### Improve Student Safety

Quickly identify absent students.

### Improve Parent Transparency

Notify parents immediately when students are absent or late.

### Reduce Administrative Work

Digitize attendance capture and reporting.

### Improve Academic Monitoring

Identify attendance-related academic risks.

### Support Compliance

Maintain accurate historical attendance records.

### Enable Data-Driven Decisions

Provide attendance analytics and trends.

---

# 3. Primary Users

## Teacher

Primary attendance recorder.

Responsibilities:

* Take attendance
* Submit attendance
* Request corrections

---

## School Admin

Attendance administrator.

Responsibilities:

* Configure attendance settings
* Approve corrections
* Lock attendance
* Monitor compliance

---

## Principal

Academic oversight.

Responsibilities:

* Monitor attendance performance
* Review trends
* Investigate absenteeism

---

## Parent

Guardian user.

Responsibilities:

* View attendance history
* Receive absence notifications

---

## Student

Learner user.

Responsibilities:

* View own attendance records

---

## Super Admin

Platform oversight.

Responsibilities:

* Audit attendance processes
* Investigate issues

---

# 4. User Stories

## Attendance Capture

### US-001

As a Teacher,

I want to mark attendance quickly,

so classroom time is not wasted.

---

### US-002

As a Teacher,

I want to mark attendance by period,

so attendance reflects actual class participation.

---

### US-003

As a Teacher,

I want to mark all students present by default,

so attendance entry becomes faster.

---

## Attendance Management

### US-004

As a School Admin,

I want to lock attendance after validation,

so records cannot be modified accidentally.

---

### US-005

As a School Admin,

I want to approve attendance corrections,

so auditability is maintained.

---

### US-006

As a Principal,

I want attendance summaries,

so I can identify absenteeism trends.

---

## Parent Visibility

### US-007

As a Parent,

I want absence notifications,

so I know when my child misses school.

---

### US-008

As a Parent,

I want attendance history,

so I can monitor attendance patterns.

---

## Reporting

### US-009

As a School Admin,

I want attendance reports,

so I can generate compliance records.

---

### US-010

As a Principal,

I want attendance analytics,

so I can monitor school performance.

---

# 5. Business Rules

## Tenant Isolation

BR-001

All attendance data must belong to exactly one tenant.

---

BR-002

Attendance records cannot be accessed across tenants.

---

## Attendance Mode Rules

BR-003

Attendance mode is configurable per school.

Supported modes:

* Daily
* Period-wise

---

BR-004

A school can enable only one primary attendance mode at a time.

---

BR-005

Period-wise attendance requires published timetable data.

---

## Session Rules

BR-006

Attendance must be recorded against:

* Academic Year
* Grade
* Section
* Date

---

BR-007

For period attendance:

Attendance must also reference:

* Timetable Slot
* Subject
* Teacher Assignment

---

BR-008

Only active enrolled students appear in attendance sessions.

---

## Status Rules

Supported statuses:

BR-009

Present

BR-010

Absent

BR-011

Late

BR-012

Excused

BR-013

Half Day

---

BR-014

Each student may have only one attendance status per attendance session.

---

## Submission Rules

BR-015

Attendance starts in Draft state.

---

BR-016

Submitted attendance cannot be edited by teachers.

---

BR-017

Locked attendance can only be modified through correction workflow.

---

## Correction Rules

BR-018

All corrections require a reason.

---

BR-019

Corrections after lock require School Admin approval.

---

BR-020

Correction history must never be deleted.

---

## Notification Rules

BR-021

Absent notifications may be triggered automatically.

---

BR-022

Late notifications may be triggered automatically.

---

BR-023

Notification preferences are configurable by school.

---

## Reporting Rules

BR-024

Attendance percentage calculations exclude non-school days.

---

BR-025

Attendance reports must use historical enrollment records.

---

BR-026

Transferred or withdrawn students remain in historical reports.

---

## Audit Rules

BR-027

All attendance modifications must be audited.

---

BR-028

Lock and unlock actions must be audited.

---

# 6. State Machine

## Attendance Session Lifecycle

```text
Draft
  |
  v
Submitted
  |
  v
Locked
  |
  +------> Corrected Pending
                    |
                    +------> Corrected Approved
                    |
                    +------> Corrected Rejected
```

---

## Attendance Correction Lifecycle

```text
Requested
   |
   v
Pending Review
   |
   +------> Approved
   |
   +------> Rejected
```

---

## Notification Lifecycle

```text
Pending
   |
   v
Queued
   |
   +------> Sent
   |
   +------> Failed
```

---

# 7. Required Data Entities

## attendance_sessions

| Field             | Type      |
| ----------------- | --------- |
| id                | UUID      |
| tenant_id         | UUID      |
| academic_year_id  | UUID      |
| grade_level_id    | UUID      |
| section_id        | UUID      |
| attendance_mode   | ENUM      |
| attendance_date   | DATE      |
| timetable_slot_id | UUID NULL |
| status            | ENUM      |
| created_by        | UUID      |
| submitted_by      | UUID      |
| locked_by         | UUID      |
| created_at        | TIMESTAMP |

---

## attendance_records

| Field                 | Type      |
| --------------------- | --------- |
| id                    | UUID      |
| tenant_id             | UUID      |
| attendance_session_id | UUID      |
| student_id            | UUID      |
| attendance_status_id  | UUID      |
| remarks               | TEXT      |
| recorded_by           | UUID      |
| recorded_at           | TIMESTAMP |

---

## attendance_statuses

| Field             | Type    |
| ----------------- | ------- |
| id                | UUID    |
| tenant_id         | UUID    |
| code              | VARCHAR |
| name              | VARCHAR |
| counts_as_present | BOOLEAN |

Default values:

* Present
* Absent
* Late
* Excused
* Half Day

---

## attendance_corrections

| Field                | Type      |
| -------------------- | --------- |
| id                   | UUID      |
| tenant_id            | UUID      |
| attendance_record_id | UUID      |
| old_status_id        | UUID      |
| new_status_id        | UUID      |
| reason               | TEXT      |
| requested_by         | UUID      |
| approved_by          | UUID      |
| status               | ENUM      |
| created_at           | TIMESTAMP |

---

## attendance_notifications

| Field                | Type      |
| -------------------- | --------- |
| id                   | UUID      |
| tenant_id            | UUID      |
| attendance_record_id | UUID      |
| guardian_id          | UUID      |
| notification_type    | ENUM      |
| delivery_channel     | ENUM      |
| status               | ENUM      |
| sent_at              | TIMESTAMP |

---

## attendance_summary_daily

(Materialized View)

| Field           | Type    |
| --------------- | ------- |
| tenant_id       | UUID    |
| attendance_date | DATE    |
| total_students  | INTEGER |
| present_count   | INTEGER |
| absent_count    | INTEGER |
| late_count      | INTEGER |

---

## attendance_audit_logs

| Field        | Type      |
| ------------ | --------- |
| id           | UUID      |
| tenant_id    | UUID      |
| entity_type  | VARCHAR   |
| entity_id    | UUID      |
| action       | VARCHAR   |
| performed_by | UUID      |
| metadata     | JSONB     |
| created_at   | TIMESTAMP |

---

# 8. Permissions Matrix

| Action               | Super Admin | School Admin | Principal | Teacher          | Accountant | Parent    | Student |
| -------------------- | ----------- | ------------ | --------- | ---------------- | ---------- | --------- | ------- |
| View Attendance      | ✓           | ✓            | ✓         | Assigned Classes | ✗          | Own Child | Self    |
| Take Attendance      | ✓           | ✓            | Optional  | Assigned Classes | ✗          | ✗         | ✗       |
| Submit Attendance    | ✓           | ✓            | Optional  | Assigned Classes | ✗          | ✗         | ✗       |
| Lock Attendance      | ✓           | ✓            | ✓         | ✗                | ✗          | ✗         | ✗       |
| Request Correction   | ✓           | ✓            | ✓         | ✓                | ✗          | ✗         | ✗       |
| Approve Correction   | ✓           | ✓            | Optional  | ✗                | ✗          | ✗         | ✗       |
| View Reports         | ✓           | ✓            | ✓         | Limited          | ✗          | Own Child | Self    |
| Configure Attendance | ✓           | ✓            | ✗         | ✗                | ✗          | ✗         | ✗       |

---

# 9. APIs Needed

## Attendance Session APIs

### POST /attendance/sessions

Create attendance session

---

### GET /attendance/sessions

List sessions

---

### GET /attendance/sessions/{id}

Get session details

---

### POST /attendance/sessions/{id}/submit

Submit attendance

---

### POST /attendance/sessions/{id}/lock

Lock attendance

---

## Attendance Record APIs

### POST /attendance/records/bulk

Bulk attendance submission

---

### PATCH /attendance/records/{id}

Update draft record

---

### GET /attendance/records

List attendance records

---

## Correction APIs

### POST /attendance/corrections

Request correction

---

### GET /attendance/corrections

List corrections

---

### POST /attendance/corrections/{id}/approve

Approve correction

---

### POST /attendance/corrections/{id}/reject

Reject correction

---

## Notification APIs

### POST /attendance/notifications/send

Trigger notifications

---

### GET /attendance/notifications

Notification history

---

## Reporting APIs

### GET /attendance/reports/daily

Daily report

---

### GET /attendance/reports/student

Student attendance report

---

### GET /attendance/reports/class

Class attendance report

---

### GET /attendance/reports/trends

Attendance trends

---

# 10. Screens Needed

## Teacher Attendance Screen

Features:

* Mark attendance
* One-click present all
* Bulk update
* Submit attendance

---

## Attendance Session Detail Screen

Features:

* Student list
* Status selection
* Remarks
* Session metadata

---

## Admin Attendance Dashboard

Displays:

* Today's attendance
* Missing submissions
* Absence counts
* Attendance rates

---

## Attendance Correction Workflow Screen

Displays:

* Pending corrections
* Approval actions
* Audit trail

---

## Student Attendance Profile

Displays:

* Attendance percentage
* Monthly trends
* Status breakdown

---

## Parent Attendance Timeline

Displays:

* Daily attendance history
* Notifications
* Attendance percentage

---

## Attendance Analytics Dashboard

Displays:

* School-wide attendance
* Class comparisons
* Chronic absenteeism
* Trends over time

---

## Attendance Configuration Screen

Settings:

* Daily vs Period-wise
* Lock schedule
* Notification triggers
* Status configuration

---

# 11. Edge Cases

### EC-001

Student joins after attendance taken.

System allows late enrollment attendance entry.

---

### EC-002

Student transfers section mid-year.

Historical attendance remains linked to original section.

---

### EC-003

Teacher forgets to submit attendance.

Admin can create reminder and submit on behalf if permitted.

---

### EC-004

Student marked absent in morning but arrives later.

Correction workflow required after lock.

---

### EC-005

Period attendance differs from daily attendance.

Reporting uses school-configured attendance mode.

---

### EC-006

Student withdrawn mid-month.

Historical attendance remains visible.

---

### EC-007

Parent linked after attendance recorded.

Historical attendance becomes visible immediately.

---

### EC-008

School holiday accidentally receives attendance entries.

System blocks attendance for non-instructional days.

---

### EC-009

Duplicate attendance session created.

System prevents duplicate sessions for same date/section/period.

---

### EC-010

Teacher reassigned after attendance recorded.

Historical attendance remains linked to original recorder.

---

# 12. Risks

## Data Accuracy Risk

Teachers enter incorrect attendance.

**Mitigation**

* Bulk validation
* Correction workflow
* Audit logs

---

## Operational Risk

Attendance not submitted on time.

**Mitigation**

* Automated reminders
* Missing attendance dashboard

---

## Notification Risk

Parents receive incorrect absence alerts.

**Mitigation**

* Notifications triggered only after submission
* Notification audit logs

---

## Scalability Risk

Large schools generating millions of attendance records.

**Mitigation**

* Partition attendance tables by academic year
* Materialized reporting views

---

## Compliance Risk

Unauthorized attendance edits.

**Mitigation**

* Locking workflow
* Approval process
* Audit logging

---

# 13. Success Metrics

## Operational Metrics

* Attendance completion rate > 98%
* Attendance submission before cutoff > 95%
* Correction approval turnaround < 24 hours

---

## Product Metrics

* Teacher attendance entry time < 2 minutes per class
* Attendance screen load time < 2 seconds
* Notification delivery success > 99%

---

## Parent Transparency Metrics

* Parent attendance view usage > 70%
* Absence notification open rate > 80%

---

## Business Metrics

* Reduction in attendance administration effort > 70%
* Reduction in manual attendance reporting > 90%
* Increased visibility into absenteeism trends

---

# 14. Out of Scope

The following are excluded from MVP:

* Biometric attendance devices
* RFID attendance
* Face recognition attendance
* GPS attendance verification
* Employee attendance management
* Visitor attendance management
* Bus attendance tracking
* Real-time classroom occupancy monitoring
* Government attendance portal integrations
* AI absenteeism prediction
* Attendance-based fee penalties
* Attendance-linked academic eligibility automation

---

# Architecture Recommendations for Your Stack

### Attendance Source of Truth

Attendance must always be linked to:

```text
Academic Year
    ↓
Enrollment
    ↓
Class/Section
    ↓
Timetable (optional)
    ↓
Attendance Session
    ↓
Attendance Record
```

Never store attendance independently of enrollment.

---

### Database Design

For PostgreSQL, create a unique constraint:

```sql
UNIQUE (
  tenant_id,
  attendance_date,
  section_id,
  timetable_slot_id
)
```

This prevents duplicate attendance sessions.

---

### High-Scale Optimization

Partition:

```text
attendance_records
```

by:

```text
academic_year_id
```

once schools exceed ~100,000 attendance records annually.

---

This delivers the functionality expected by more than 95% of K-12 schools while keeping implementation manageable for a solo founder.
