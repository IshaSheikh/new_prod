# Product Requirements Document (PRD)

# Module 7.4 – Student Information System (SIS)

---

# 1. Module Objective

The Student Information System (SIS) is the authoritative source of truth for all student-related information throughout the student's lifecycle in the school.

This module manages:

* Student admissions and registration
* Student demographic information
* Academic enrollment history
* Guardian relationships
* Student documents
* Health and emergency information
* Student status changes (active, transferred, withdrawn, graduated)

The SIS serves as the foundation for all downstream modules:

* Attendance
* Gradebook
* Exams
* Timetable
* Fees
* Parent Portal
* Transport
* Communication

Every operational module should consume student data from the SIS rather than maintaining its own student records.

---

# 2. Why Schools Need It

Schools typically store student information across spreadsheets, paper files, ERP systems, and disconnected databases.

This causes:

* Duplicate records
* Data inconsistency
* Admission errors
* Parent communication failures
* Compliance issues
* Reporting inaccuracies

The SIS solves this by providing:

### Single Source of Truth

One student record used throughout the platform.

### Complete Student Lifecycle Tracking

Track students from inquiry through graduation.

### Improved Parent Communication

Accurate guardian relationships.

### Operational Efficiency

Eliminate duplicate data entry.

### Regulatory Compliance

Maintain auditable records and document history.

### Better Reporting

Enable accurate attendance, academic, and financial analytics.

---

# 3. Primary Users

## School Admin

Primary owner of student records.

Responsibilities:

* Manage admissions
* Update student information
* Manage enrollment
* Upload documents

---

## Principal

Academic oversight.

Responsibilities:

* Review enrollment statistics
* Monitor student population
* Approve transfers and withdrawals

---

## Teacher

Limited student access.

Responsibilities:

* View assigned students
* Access emergency information
* View guardian contacts

---

## Accountant

Financial visibility.

Responsibilities:

* Access student enrollment information
* Access fee-related identifiers

---

## Parent

Guardian access.

Responsibilities:

* View linked child information
* Update selected profile information (optional)

---

## Student

Self-service access.

Responsibilities:

* View profile
* View enrollment details

---

## Super Admin

Platform oversight.

Responsibilities:

* Audit student data integrity
* Support schools

---

# 4. User Stories

## Student Registration

### US-001

As a School Admin,

I want to create a student profile,

so the student can be admitted into the school.

---

### US-002

As a School Admin,

I want to upload student documents,

so records remain centralized.

---

### US-003

As a School Admin,

I want to define mandatory profile fields,

so data collection aligns with school requirements.

---

## Enrollment Management

### US-004

As a School Admin,

I want to enroll a student into a class and section,

so academic activities can begin.

---

### US-005

As a School Admin,

I want to transfer a student,

so enrollment history remains accurate.

---

### US-006

As a School Admin,

I want to withdraw a student,

so access and reporting remain accurate.

---

## Guardian Management

### US-007

As a School Admin,

I want to link multiple guardians,

so all authorized contacts are available.

---

### US-008

As a Parent,

I want to access my child's information,

so I can monitor their education.

---

## Student Records

### US-009

As a Teacher,

I want to view student emergency contacts,

so I can respond quickly during emergencies.

---

### US-010

As a Principal,

I want to view enrollment history,

so I can analyze retention and transfers.

---

## Health Records

### US-011

As a School Admin,

I want to record health alerts,

so staff can respond appropriately.

---

### US-012

As a Teacher,

I want visibility into critical health alerts,

so I can ensure student safety.

---

# 5. Business Rules

## Tenant Isolation

BR-001

Every student record must belong to exactly one tenant.

---

BR-002

Student data cannot be accessed across tenants.

---

## Student Identity Rules

BR-003

Each student receives a unique student number within the tenant.

---

BR-004

Student numbers cannot be reused.

---

BR-005

A student profile cannot be permanently deleted after enrollment.

Soft delete only.

---

## Enrollment Rules

BR-006

A student may have only one primary enrollment per academic year.

---

BR-007

A student cannot be enrolled into multiple active sections simultaneously unless school configuration allows dual enrollment.

Default: Not allowed.

---

BR-008

Enrollment must reference:

* Academic Year
* Grade
* Section

---

BR-009

Enrollment history must never be deleted.

---

## Guardian Rules

BR-010

A student may have multiple guardians.

---

BR-011

One guardian must be marked as primary.

---

BR-012

The same guardian may be linked to multiple students.

---

BR-013

Guardians must be linked before parent portal access is granted.

---

## Status Rules

BR-014

Student status transitions must be auditable.

---

BR-015

Students cannot move directly from Inquiry to Enrolled.

---

BR-016

Withdrawal requires a reason.

---

BR-017

Transfer requires destination information.

---

## Document Rules

BR-018

Documents are versioned.

---

BR-019

Documents cannot be modified after upload.

Replacement creates a new version.

---

BR-020

Supported document categories must be configurable.

Examples:

* Birth Certificate
* Transfer Certificate
* Passport
* Aadhaar
* Medical Records

---

## Health Rules

BR-021

Critical health alerts must be visible to authorized staff.

---

BR-022

Health notes are confidential and role-protected.

---

## Configuration Rules

BR-023

Mandatory profile fields are configurable per school.

Examples:

* Religion
* Nationality
* Aadhaar Number
* Student Category

---

# 6. State Machine

## Student Admission Lifecycle

```text
Inquiry
   |
   v
Applicant
   |
   v
Admitted
   |
   v
Enrolled
   |
   +------> Active
                |
                +------> Transferred
                |
                +------> Withdrawn
                |
                +------> Graduated
                |
                +------> Deceased
```

---

## Enrollment Lifecycle

```text
Draft
   |
   v
Pending Approval
   |
   v
Active
   |
   +------> Completed
   |
   +------> Transferred
   |
   +------> Withdrawn
```

---

## Document Lifecycle

```text
Uploaded
    |
    v
Verified
    |
    +------> Replaced
    |
    +------> Archived
```

---

# 7. Required Data Entities

## students

| Field            | Type      |
| ---------------- | --------- |
| id               | UUID      |
| tenant_id        | UUID      |
| student_number   | VARCHAR   |
| admission_number | VARCHAR   |
| status           | ENUM      |
| created_at       | TIMESTAMP |

---

## student_profiles

| Field         | Type    |
| ------------- | ------- |
| id            | UUID    |
| tenant_id     | UUID    |
| student_id    | UUID    |
| first_name    | VARCHAR |
| middle_name   | VARCHAR |
| last_name     | VARCHAR |
| gender        | ENUM    |
| date_of_birth | DATE    |
| nationality   | VARCHAR |
| blood_group   | VARCHAR |
| photo_url     | TEXT    |

---

## student_enrollments

| Field            | Type |
| ---------------- | ---- |
| id               | UUID |
| tenant_id        | UUID |
| student_id       | UUID |
| academic_year_id | UUID |
| grade_level_id   | UUID |
| section_id       | UUID |
| enrollment_date  | DATE |
| status           | ENUM |

---

## guardians

| Field             | Type    |
| ----------------- | ------- |
| id                | UUID    |
| tenant_id         | UUID    |
| first_name        | VARCHAR |
| last_name         | VARCHAR |
| relationship_type | ENUM    |
| mobile            | VARCHAR |
| email             | VARCHAR |
| occupation        | VARCHAR |

---

## guardian_student_links

| Field       | Type    |
| ----------- | ------- |
| id          | UUID    |
| tenant_id   | UUID    |
| guardian_id | UUID    |
| student_id  | UUID    |
| is_primary  | BOOLEAN |

---

## student_documents

| Field         | Type      |
| ------------- | --------- |
| id            | UUID      |
| tenant_id     | UUID      |
| student_id    | UUID      |
| document_type | VARCHAR   |
| file_url      | TEXT      |
| version       | INTEGER   |
| uploaded_at   | TIMESTAMP |

---

## student_health_notes

| Field          | Type |
| -------------- | ---- |
| id             | UUID |
| tenant_id      | UUID |
| student_id     | UUID |
| severity       | ENUM |
| note           | TEXT |
| effective_date | DATE |

---

## student_status_history

| Field      | Type      |
| ---------- | --------- |
| id         | UUID      |
| tenant_id  | UUID      |
| student_id | UUID      |
| old_status | ENUM      |
| new_status | ENUM      |
| reason     | TEXT      |
| changed_by | UUID      |
| changed_at | TIMESTAMP |

---

## student_custom_fields

| Field       | Type    |
| ----------- | ------- |
| id          | UUID    |
| tenant_id   | UUID    |
| field_name  | VARCHAR |
| field_type  | ENUM    |
| is_required | BOOLEAN |

---

# 8. Permissions Matrix

| Action              | Super Admin | School Admin | Principal  | Teacher       | Accountant | Parent    | Student |
| ------------------- | ----------- | ------------ | ---------- | ------------- | ---------- | --------- | ------- |
| View Students       | ✓           | ✓            | ✓          | Assigned Only | Limited    | Own Child | Self    |
| Create Student      | ✓           | ✓            | ✗          | ✗             | ✗          | ✗         | ✗       |
| Edit Student        | ✓           | ✓            | Limited    | ✗             | ✗          | ✗         | ✗       |
| Manage Enrollment   | ✓           | ✓            | ✓          | ✗             | ✗          | ✗         | ✗       |
| Upload Documents    | ✓           | ✓            | ✗          | ✗             | ✗          | ✗         | ✗       |
| View Documents      | ✓           | ✓            | ✓          | Assigned Only | Limited    | Own Child | Self    |
| Manage Guardians    | ✓           | ✓            | ✗          | ✗             | ✗          | ✗         | ✗       |
| View Health Notes   | ✓           | ✓            | ✓          | Critical Only | ✗          | ✗         | ✗       |
| Manage Health Notes | ✓           | ✓            | Authorized | ✗             | ✗          | ✗         | ✗       |
| Transfer Student    | ✓           | ✓            | ✓          | ✗             | ✗          | ✗         | ✗       |
| Withdraw Student    | ✓           | ✓            | ✓          | ✗             | ✗          | ✗         | ✗       |

---

# 9. APIs Needed

## Student APIs

### POST /students

Create student

---

### GET /students

List students

---

### GET /students/{id}

Student details

---

### PATCH /students/{id}

Update student

---

### POST /students/{id}/archive

Archive student

---

## Enrollment APIs

### POST /student-enrollments

Create enrollment

---

### GET /student-enrollments

List enrollments

---

### PATCH /student-enrollments/{id}

Update enrollment

---

### POST /student-enrollments/{id}/transfer

Transfer student

---

### POST /student-enrollments/{id}/withdraw

Withdraw student

---

## Guardian APIs

### POST /guardians

Create guardian

---

### GET /guardians

List guardians

---

### POST /guardian-student-links

Link guardian

---

### DELETE /guardian-student-links/{id}

Unlink guardian

---

## Document APIs

### POST /student-documents/upload

Upload document

---

### GET /student-documents

List documents

---

### POST /student-documents/{id}/verify

Verify document

---

## Health APIs

### POST /student-health-notes

Create health note

---

### GET /student-health-notes

List health notes

---

### PATCH /student-health-notes/{id}

Update health note

---

## Status APIs

### GET /student-status-history

Status history

---

# 10. Screens Needed

## Student List Screen

Features:

* Search
* Filters
* Bulk actions
* Export

Filters:

* Grade
* Section
* Status
* Campus

---

## Student Profile Screen

Tabs:

1. Overview
2. Enrollment
3. Guardians
4. Documents
5. Health
6. Attendance
7. Fees
8. Audit History

---

## Enrollment Management Screen

Features:

* Create enrollment
* Transfer
* Withdraw
* View history

---

## Guardian Management Screen

Features:

* Link guardian
* Manage relationships
* Mark primary guardian

---

## Student Documents Screen

Features:

* Upload
* Preview
* Version history

---

## Health and Emergency Screen

Features:

* Medical alerts
* Emergency contacts
* Critical notifications

---

## Student Lifecycle Dashboard

Displays:

* Active students
* New admissions
* Withdrawals
* Transfers
* Graduations

---

# 11. Edge Cases

### EC-001

Student transfers mid-year.

Enrollment history must remain intact.

---

### EC-002

Student re-joins after withdrawal.

New enrollment created while preserving history.

---

### EC-003

Guardian linked to multiple students.

Single guardian profile reused.

---

### EC-004

Duplicate student admission.

System warns using configurable matching:

* Name
* DOB
* Mobile
* Government ID

---

### EC-005

Primary guardian removed.

System requires reassignment.

---

### EC-006

Student changes section mid-year.

Historical attendance and grades remain linked to original enrollment periods.

---

### EC-007

Missing mandatory profile fields.

Enrollment blocked.

---

### EC-008

Document replaced.

Previous version retained.

---

### EC-009

Health note marked critical.

Teachers assigned to student receive visibility.

---

### EC-010

Student graduates.

Record becomes read-only but remains searchable.

---

# 12. Risks

## Data Integrity Risk

Duplicate student records.

**Mitigation**

* Duplicate detection engine
* Admission validation rules

---

## Privacy Risk

Unauthorized access to health information.

**Mitigation**

* Field-level permissions
* Audit logging

---

## Operational Risk

Incorrect guardian relationships.

**Mitigation**

* Relationship validation
* Primary guardian enforcement

---

## Compliance Risk

Loss of historical records.

**Mitigation**

* Immutable status history
* Soft deletes only

---

## Scalability Risk

Large schools with 10,000+ students.

**Mitigation**

* Pagination
* Indexed search
* Read replicas

---

# 13. Success Metrics

## Data Quality

* Duplicate student rate < 0.5%
* Missing mandatory profile fields < 2%

---

## Operational Metrics

* Student registration time < 5 minutes
* Enrollment creation time < 1 minute
* Guardian linking success > 99%

---

## Product Metrics

* Student search response < 2 seconds
* Document upload success > 99%

---

## Business Metrics

* Reduction in admission administration workload > 60%
* Increased parent profile accuracy > 95%
* Improved reporting accuracy across attendance, fees, and grades

---

# 14. Out of Scope

The following are excluded from the SIS module:

* Attendance tracking
* Gradebook and assessment management
* Timetable generation
* Fee calculation and collection
* Parent communication delivery
* Transport route management
* Library circulation
* Hostel management
* Learning Management System (LMS)
* Behavioral tracking
* Alumni management (Phase 2)
* Government reporting integrations (Phase 2)

---

# Architecture Recommendations (Critical)

For your stack (Next.js + NestJS + PostgreSQL + AWS + Flutter):

### Student ID Strategy

Use:

```text
Tenant Code + Academic Year + Sequence
```

Example:

```text
DPS-2026-004582
```

Store separately from UUID primary keys.

---

### Document Storage

Use AWS S3:

```text
tenant/
  students/
    student-id/
      documents/
```

Never store files in PostgreSQL.

---

### Auditability

Every status change must create:

```text
student_status_history
```

Never update status without history.

---

### Future-Proofing

Design the SIS as the master source for:

* Attendance
* Grades
* Fees
* Timetable
* Parent Portal

All future modules should reference `student_id` from SIS rather than maintaining their own student data. This architectural decision will significantly reduce complexity and prevent data consistency problems as the platform grows.
