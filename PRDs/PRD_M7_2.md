# Product Requirements Document (PRD)

# Module 7.2 – School Onboarding and Configuration

---

# 1. Module Objective

The School Onboarding and Configuration module is responsible for transforming a newly subscribed school from an empty tenant into a fully operational K-12 institution ready to use the platform.

This module establishes the foundational academic, organizational, branding, and communication structures required before attendance, grades, fees, timetable, and parent communication modules can function.

The onboarding process should guide School Admins through a structured setup workflow and ensure no school goes live with incomplete configuration.

---

# 2. Why Schools Need It

Most school software implementations fail because setup is complex, inconsistent, and dependent on vendor intervention.

Schools need:

### Fast Implementation

Reduce onboarding time from weeks to hours.

### Standardized Setup

Ensure all required academic and organizational structures are configured correctly.

### Reduced Administrative Errors

Prevent invalid class structures, duplicate sections, and academic calendar issues.

### Branding Consistency

Allow schools to use their own identity across portals, reports, and communications.

### Operational Readiness

Guarantee that attendance, fees, grades, and timetable modules work correctly from day one.

### Self-Service Administration

Allow School Admins to configure their institution without technical support.

---

# 3. Primary Users

## School Admin (Primary)

Responsible for configuring the school.

Actions:

* Complete setup wizard
* Configure academic years
* Create classes and sections
* Configure campuses
* Upload branding assets
* Manage communication settings

---

## Principal

Reviews school structure and academic setup.

Actions:

* View configuration
* Validate academic hierarchy
* Review readiness status

---

## Super Admin

Platform-level oversight.

Actions:

* Monitor onboarding progress
* Assist schools
* Force-complete onboarding
* Lock/unlock configurations

---

## Teacher

Participates during readiness validation.

Actions:

* Accept invitations
* Verify assignments

---

# 4. User Stories

## School Profile Setup

### US-001

As a School Admin,

I want to create a school profile,

so that the institution is identifiable within the platform.

---

### US-002

As a School Admin,

I want to configure school contact details,

so parents and staff can communicate effectively.

---

### US-003

As a School Admin,

I want to configure school address information,

so official documents display correct information.

---

## Academic Setup

### US-004

As a School Admin,

I want to create an academic year,

so academic operations can begin.

---

### US-005

As a School Admin,

I want to define terms/semesters,

so reporting periods are properly structured.

---

### US-006

As a School Admin,

I want to configure grade levels,

so students can be assigned correctly.

---

### US-007

As a School Admin,

I want to create sections,

so multiple classrooms can exist within a grade.

---

## Campus Setup

### US-008

As a School Admin,

I want to configure multiple campuses,

so each location can be managed separately.

---

## Branding Setup

### US-009

As a School Admin,

I want to upload a logo,

so the platform reflects the school's identity.

---

### US-010

As a School Admin,

I want to configure colors and branding,

so communications appear professional.

---

## Readiness Validation

### US-011

As a School Admin,

I want the system to verify readiness,

so I know whether the school can go live.

---

### US-012

As a Super Admin,

I want visibility into onboarding progress,

so I can support schools that are stuck.

---

# 5. Business Rules

## School Setup Rules

BR-001

Every tenant must have exactly one active school profile.

---

BR-002

School profile must be completed before activation.

Required:

* School name
* Contact email
* Contact phone
* Address
* Timezone

---

## Academic Year Rules

BR-003

Only one academic year can be active at a time.

---

BR-004

Academic year dates cannot overlap.

---

BR-005

Future academic years may be pre-configured.

---

BR-006

Academic year cannot be deleted after students are enrolled.

---

## Term Rules

BR-007

Each term belongs to a single academic year.

---

BR-008

Term dates must fall inside the academic year.

---

BR-009

Terms cannot overlap.

---

## Grade and Section Rules

BR-010

Grade names must be unique within a tenant.

Examples:

* Grade 1
* Grade 2
* Grade 10

---

BR-011

Section names may repeat across grades.

Example:

Grade 1-A

Grade 2-A

Allowed.

---

BR-012

Each Grade + Section combination must be unique per academic year.

---

BR-013

A section cannot exist without a grade.

---

## Campus Rules

BR-014

A school may operate multiple campuses.

---

BR-015

One campus must be designated as the primary campus.

---

BR-016

Campus deletion is prohibited if active students exist.

---

## Branding Rules

BR-017

Logo uploads must meet supported file formats.

Allowed:

* PNG
* JPG
* SVG

---

BR-018

Maximum logo size: 5 MB.

---

BR-019

Branding changes must be reflected across all portals.

---

## Go-Live Rules

BR-020

A school cannot go live until all mandatory setup steps are completed.

Required:

* School profile
* Academic year
* Grade structure
* Section structure
* Fee categories configured
* At least one School Admin
* At least one Teacher

---

BR-021

Go-live validation runs automatically after every setup change.

---

# 6. State Machine

## School Onboarding Lifecycle

```text
Draft
  |
  v
Profile Configured
  |
  v
Academic Setup Complete
  |
  v
Organizational Setup Complete
  |
  v
Ready For Validation
  |
  v
Live
```

---

## Alternative Paths

```text
Ready For Validation
          |
          v
Validation Failed
          |
          v
Needs Action
          |
          +------> Ready For Validation
```

---

## Academic Year Lifecycle

```text
Draft
  |
  v
Scheduled
  |
  v
Active
  |
  v
Completed
  |
  v
Archived
```

---

## Campus Lifecycle

```text
Draft
  |
  v
Active
  |
  v
Inactive
```

---

# 7. Required Data Entities

## schools

| Field       | Type      |
| ----------- | --------- |
| id          | UUID      |
| tenant_id   | UUID      |
| school_code | VARCHAR   |
| status      | ENUM      |
| created_at  | TIMESTAMP |
| updated_at  | TIMESTAMP |

---

## school_profiles

| Field          | Type    |
| -------------- | ------- |
| id             | UUID    |
| tenant_id      | UUID    |
| school_name    | VARCHAR |
| legal_name     | VARCHAR |
| contact_email  | VARCHAR |
| contact_phone  | VARCHAR |
| website        | VARCHAR |
| timezone       | VARCHAR |
| address_line_1 | VARCHAR |
| city           | VARCHAR |
| state          | VARCHAR |
| country        | VARCHAR |
| postal_code    | VARCHAR |

---

## school_settings

| Field                 | Type    |
| --------------------- | ------- |
| id                    | UUID    |
| tenant_id             | UUID    |
| language              | VARCHAR |
| currency              | VARCHAR |
| date_format           | VARCHAR |
| communication_enabled | BOOLEAN |
| attendance_enabled    | BOOLEAN |
| gradebook_enabled     | BOOLEAN |

---

## academic_years

| Field      | Type    |
| ---------- | ------- |
| id         | UUID    |
| tenant_id  | UUID    |
| name       | VARCHAR |
| start_date | DATE    |
| end_date   | DATE    |
| status     | ENUM    |

---

## terms

| Field            | Type    |
| ---------------- | ------- |
| id               | UUID    |
| tenant_id        | UUID    |
| academic_year_id | UUID    |
| name             | VARCHAR |
| start_date       | DATE    |
| end_date         | DATE    |
| status           | ENUM    |

---

## grade_levels

| Field      | Type    |
| ---------- | ------- |
| id         | UUID    |
| tenant_id  | UUID    |
| code       | VARCHAR |
| name       | VARCHAR |
| sort_order | INTEGER |

---

## sections

| Field            | Type    |
| ---------------- | ------- |
| id               | UUID    |
| tenant_id        | UUID    |
| academic_year_id | UUID    |
| grade_level_id   | UUID    |
| name             | VARCHAR |
| capacity         | INTEGER |

---

## campuses

| Field       | Type    |
| ----------- | ------- |
| id          | UUID    |
| tenant_id   | UUID    |
| campus_name | VARCHAR |
| address     | TEXT    |
| is_primary  | BOOLEAN |
| status      | ENUM    |

---

## branding_assets

| Field       | Type      |
| ----------- | --------- |
| id          | UUID      |
| tenant_id   | UUID      |
| asset_type  | ENUM      |
| file_url    | TEXT      |
| uploaded_by | UUID      |
| uploaded_at | TIMESTAMP |

---

## onboarding_checklists

| Field        | Type      |
| ------------ | --------- |
| id           | UUID      |
| tenant_id    | UUID      |
| step_code    | VARCHAR   |
| status       | ENUM      |
| completed_at | TIMESTAMP |

---

# 8. Permissions Matrix

| Action                  | Super Admin | School Admin | Principal | Teacher | Accountant | Parent  | Student |
| ----------------------- | ----------- | ------------ | --------- | ------- | ---------- | ------- | ------- |
| View School Profile     | ✓           | ✓            | ✓         | ✓       | ✓          | Limited | Limited |
| Edit School Profile     | ✓           | ✓            | ✗         | ✗       | ✗          | ✗       | ✗       |
| Configure Academic Year | ✓           | ✓            | View      | ✗       | ✗          | ✗       | ✗       |
| Configure Terms         | ✓           | ✓            | View      | ✗       | ✗          | ✗       | ✗       |
| Configure Grades        | ✓           | ✓            | View      | ✗       | ✗          | ✗       | ✗       |
| Configure Sections      | ✓           | ✓            | View      | ✗       | ✗          | ✗       | ✗       |
| Configure Campuses      | ✓           | ✓            | View      | ✗       | ✗          | ✗       | ✗       |
| Upload Branding Assets  | ✓           | ✓            | ✗         | ✗       | ✗          | ✗       | ✗       |
| View Readiness Status   | ✓           | ✓            | ✓         | ✗       | ✗          | ✗       | ✗       |
| Go Live                 | ✓           | ✓            | ✗         | ✗       | ✗          | ✗       | ✗       |

---

# 9. APIs Needed

## School Profile APIs

### POST /schools/profile

Create school profile

---

### GET /schools/profile

Get profile

---

### PATCH /schools/profile

Update profile

---

## School Settings APIs

### GET /schools/settings

Get settings

---

### PATCH /schools/settings

Update settings

---

## Academic Year APIs

### POST /academic-years

Create academic year

---

### GET /academic-years

List academic years

---

### PATCH /academic-years/{id}

Update academic year

---

### POST /academic-years/{id}/activate

Activate academic year

---

## Terms APIs

### POST /terms

Create term

---

### GET /terms

List terms

---

### PATCH /terms/{id}

Update term

---

## Grade APIs

### POST /grade-levels

Create grade

---

### GET /grade-levels

List grades

---

### PATCH /grade-levels/{id}

Update grade

---

## Section APIs

### POST /sections

Create section

---

### GET /sections

List sections

---

### PATCH /sections/{id}

Update section

---

## Campus APIs

### POST /campuses

Create campus

---

### GET /campuses

List campuses

---

### PATCH /campuses/{id}

Update campus

---

## Branding APIs

### POST /branding-assets/upload

Upload logo/branding

---

### GET /branding-assets

List assets

---

### DELETE /branding-assets/{id}

Delete asset

---

## Readiness APIs

### GET /onboarding/readiness

Readiness report

---

### POST /onboarding/validate

Run validation

---

### POST /onboarding/go-live

Activate school

---

# 10. Screens Needed

## Setup Wizard

Step-based onboarding experience:

1. School Profile
2. Academic Year
3. Terms
4. Grades
5. Sections
6. Campuses
7. Branding
8. Communication Settings
9. Readiness Check
10. Go Live

---

## School Profile Screen

Sections:

* School Information
* Contact Information
* Address Information

---

## Academic Year Setup Screen

Features:

* Create year
* Activate year
* Archive year

---

## Term Management Screen

Features:

* Create term
* Edit term
* View calendar alignment

---

## Grade Structure Screen

Features:

* Create grades
* Reorder grades
* Import grades

---

## Section Management Screen

Features:

* Create sections
* Set capacities
* Assign campuses

---

## Campus Management Screen

Features:

* Multiple campus support
* Primary campus selection

---

## Branding Screen

Features:

* Upload logo
* Preview portal branding
* Configure theme colors

---

## Communication Settings Screen

Features:

* Email settings
* SMS settings
* Notification preferences

---

## Readiness Dashboard

Displays:

* Completion percentage
* Missing requirements
* Validation results
* Go-live eligibility

---

# 11. Edge Cases

### EC-001

Academic year dates overlap.

System blocks save.

---

### EC-002

Admin attempts to activate second academic year.

System prevents activation.

---

### EC-003

Grade deleted after student assignment.

Deletion blocked.

---

### EC-004

Section capacity lower than enrolled students.

Capacity reduction blocked.

---

### EC-005

Primary campus deleted.

System requires replacement primary campus.

---

### EC-006

Logo upload exceeds file size limit.

Upload rejected.

---

### EC-007

Go-live attempted without teacher configured.

Validation fails.

---

### EC-008

School partially configured and abandoned.

System sends reminder notifications.

---

### EC-009

Duplicate section name within same grade and academic year.

Creation blocked.

---

# 12. Risks

## Data Integrity Risk

Incorrect academic structure causes downstream failures.

Mitigation:

* Strong validation rules
* Readiness checks

---

## User Adoption Risk

Setup process becomes overwhelming.

Mitigation:

* Guided wizard
* Progress indicators
* Templates

---

## Configuration Risk

Schools configure invalid academic calendars.

Mitigation:

* Calendar validation engine

---

## Branding Risk

Inconsistent branding across modules.

Mitigation:

* Centralized branding service

---

## Scalability Risk

Large schools may create hundreds of sections.

Mitigation:

* Pagination
* Bulk operations
* Optimized indexing

---

# 13. Success Metrics

## Onboarding Metrics

* School setup completion rate > 90%
* Average onboarding completion < 2 hours
* Go-live success rate > 95%

---

## Product Metrics

* Academic year setup success > 99%
* Grade creation success > 99%
* Branding setup completion > 80%

---

## Operational Metrics

* Support tickets related to onboarding < 10%
* Validation failures resolved within 1 day

---

## Business Metrics

* New schools operational within first day
* Reduced implementation effort by 80%
* Increased activation-to-paid conversion rate

---

# 14. Out of Scope

The following are intentionally excluded from this module:

* Student admissions
* Student enrollment
* Attendance management
* Fee collection workflows
* Gradebook and assessments
* Timetable generation
* Parent communication delivery
* Staff payroll
* Library management
* Transport management
* Hostel management
* Learning Management System (LMS)
* Analytics and reporting modules
* Mobile application-specific onboarding flows

---
A school should be able to sign up in the morning and start recording attendance, collecting fees, and communicating with parents by the afternoon. This is the primary success criterion for Module 7.2.
