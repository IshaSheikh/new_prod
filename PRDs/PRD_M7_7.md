# Product Requirements Document (PRD)

# Module 7.7 – Gradebook and Report Cards

---

# 1. Module Objective

The Gradebook and Report Cards module manages the complete academic assessment lifecycle, from assessment creation and marks entry to result publication and report card generation.

This module serves as the official academic record system for students and enables schools to:

* Define assessments and examinations
* Capture marks and grades
* Apply grading schemes
* Calculate weighted results
* Generate report cards
* Publish results to students and parents
* Maintain historical academic records

The module must support transparency, auditability, and flexibility across different K-12 grading systems while ensuring that published academic results are reliable and tamper-resistant.

---

# 2. Why Schools Need It

Academic evaluation is one of the most important responsibilities of a school.

Without a centralized gradebook:

* Marks are stored in spreadsheets
* Calculation errors occur frequently
* Report card preparation becomes time-consuming
* Parents receive delayed academic updates
* Academic records become difficult to audit

Schools need this module to:

### Standardize Assessment Processes

Create a consistent evaluation framework.

### Reduce Administrative Work

Automate calculations and report card generation.

### Improve Parent Transparency

Provide timely access to results.

### Improve Academic Monitoring

Identify struggling students early.

### Maintain Compliance

Preserve official academic records and publication history.

### Support School Growth

Handle thousands of assessments and report cards efficiently.

---

# 3. Primary Users

## School Admin

Primary academic administrator.

Responsibilities:

* Configure grading systems
* Manage assessment structures
* Publish results

---

## Principal

Academic oversight.

Responsibilities:

* Verify results
* Approve publications
* Monitor academic performance

---

## Teacher

Primary marks entry user.

Responsibilities:

* Record marks
* Verify grades
* Submit results

---

## Parent

Guardian user.

Responsibilities:

* View published report cards
* Monitor academic progress

---

## Student

Learner user.

Responsibilities:

* View published results
* Download report cards

---

## Super Admin

Platform oversight.

Responsibilities:

* Audit academic processes
* Support schools

---

# 4. User Stories

## Assessment Management

### US-001

As a School Admin,

I want to create assessments,

so academic evaluations can be conducted.

---

### US-002

As a School Admin,

I want to define assessment components,

so marks can be split into exams, projects, practicals, and assignments.

---

## Marks Entry

### US-003

As a Teacher,

I want to enter marks quickly,

so grading becomes efficient.

---

### US-004

As a Teacher,

I want bulk marks upload,

so I can avoid manual entry.

---

### US-005

As a Teacher,

I want automatic grade calculation,

so grading remains consistent.

---

## Verification

### US-006

As a Principal,

I want results verified before publication,

so errors are minimized.

---

### US-007

As a School Admin,

I want incomplete marks identified,

so publication does not occur prematurely.

---

## Report Cards

### US-008

As a School Admin,

I want report cards generated automatically,

so administrative effort is reduced.

---

### US-009

As a Parent,

I want to view report cards online,

so I can monitor academic progress.

---

### US-010

As a Student,

I want to download my report card,

so I can keep a personal copy.

---

## Revisions

### US-011

As a School Admin,

I want corrected report cards versioned,

so historical records remain intact.

---

### US-012

As a Principal,

I want an audit trail for mark changes,

so academic integrity is maintained.

---

# 5. Business Rules

## Tenant Isolation

BR-001

All academic records belong to exactly one tenant.

---

BR-002

Results cannot be accessed across tenants.

---

## Assessment Rules

BR-003

Assessments must belong to:

* Academic Year
* Grade
* Section
* Subject

---

BR-004

Assessments may contain multiple components.

Examples:

* Theory
* Practical
* Project
* Assignment

---

BR-005

Assessment components may be weighted.

Example:

* Theory = 70%
* Practical = 30%

---

BR-006

Total component weight must equal 100%.

---

## Marks Rules

BR-007

Marks may be stored as:

* Raw Marks
* Weighted Marks
* Converted Grades

---

BR-008

Marks must not exceed configured maximum marks.

---

BR-009

Only assigned teachers may enter marks.

---

BR-010

Marks entry deadlines may be configured.

---

BR-011

Bulk uploads must be validated before import.

---

## Grading Rules

BR-012

Each school may configure grading schemes.

---

BR-013

Grading schemes support:

* Percentage-based
* Grade-based
* GPA-based

---

BR-014

Grade conversion must be automatic.

---

## Verification Rules

BR-015

Results cannot move to verification until all mandatory marks are entered.

---

BR-016

Verification requires authorized academic staff.

---

BR-017

Verified results become read-only for teachers.

---

## Report Card Rules

BR-018

Report cards cannot be published with missing mandatory marks.

---

BR-019

Report cards are generated from approved templates.

---

BR-020

Published report cards become immutable.

---

BR-021

Any modification after publication creates a new version.

---

BR-022

Previous versions remain permanently accessible to administrators.

---

## Publication Rules

BR-023

Students can only view published results.

---

BR-024

Parents can only view published results.

---

BR-025

Publication date may be scheduled.

---

BR-026

Publication events must be audited.

---

## Audit Rules

BR-027

Every mark change must be logged.

---

BR-028

Every grade recalculation must be logged.

---

BR-029

Every report card revision must be logged.

---

# 6. State Machine

## Assessment Lifecycle

```text
Assessment Draft
      |
      v
Assessment Active
      |
      v
Marks Entered
      |
      v
Verified
      |
      v
Published
      |
      +------> Revised
```

---

## Marks Entry Lifecycle

```text
Draft
   |
   v
Entered
   |
   v
Submitted
   |
   v
Verified
```

---

## Report Card Lifecycle

```text
Generated
    |
    v
Verified
    |
    v
Published
    |
    +------> Revised
```

---

## Result Publication Lifecycle

```text
Scheduled
    |
    v
Published
    |
    +------> Revised
    |
    +------> Withdrawn
```

---

# 7. Required Data Entities

## assessments

| Field            | Type    |
| ---------------- | ------- |
| id               | UUID    |
| tenant_id        | UUID    |
| academic_year_id | UUID    |
| grade_level_id   | UUID    |
| section_id       | UUID    |
| subject_id       | UUID    |
| assessment_name  | VARCHAR |
| assessment_type  | ENUM    |
| total_marks      | DECIMAL |
| status           | ENUM    |

---

## assessment_components

| Field             | Type    |
| ----------------- | ------- |
| id                | UUID    |
| tenant_id         | UUID    |
| assessment_id     | UUID    |
| component_name    | VARCHAR |
| max_marks         | DECIMAL |
| weight_percentage | DECIMAL |

---

## marks_entries

| Field                   | Type    |
| ----------------------- | ------- |
| id                      | UUID    |
| tenant_id               | UUID    |
| assessment_component_id | UUID    |
| student_id              | UUID    |
| raw_marks               | DECIMAL |
| weighted_marks          | DECIMAL |
| grade                   | VARCHAR |
| entered_by              | UUID    |
| verified_by             | UUID    |

---

## grading_schemes

| Field        | Type    |
| ------------ | ------- |
| id           | UUID    |
| tenant_id    | UUID    |
| scheme_name  | VARCHAR |
| grading_type | ENUM    |
| is_active    | BOOLEAN |

---

## grading_scheme_ranges

| Field             | Type    |
| ----------------- | ------- |
| id                | UUID    |
| grading_scheme_id | UUID    |
| min_score         | DECIMAL |
| max_score         | DECIMAL |
| grade             | VARCHAR |
| grade_point       | DECIMAL |

---

## report_cards

| Field            | Type      |
| ---------------- | --------- |
| id               | UUID      |
| tenant_id        | UUID      |
| student_id       | UUID      |
| academic_year_id | UUID      |
| template_id      | UUID      |
| version_number   | INTEGER   |
| overall_grade    | VARCHAR   |
| status           | ENUM      |
| published_at     | TIMESTAMP |

---

## report_card_templates

| Field           | Type    |
| --------------- | ------- |
| id              | UUID    |
| tenant_id       | UUID    |
| template_name   | VARCHAR |
| template_schema | JSONB   |
| is_active       | BOOLEAN |

---

## result_publications

| Field            | Type      |
| ---------------- | --------- |
| id               | UUID      |
| tenant_id        | UUID      |
| academic_year_id | UUID      |
| publication_name | VARCHAR   |
| publication_date | TIMESTAMP |
| status           | ENUM      |

---

## mark_change_audit_logs

| Field          | Type      |
| -------------- | --------- |
| id             | UUID      |
| tenant_id      | UUID      |
| marks_entry_id | UUID      |
| old_value      | JSONB     |
| new_value      | JSONB     |
| changed_by     | UUID      |
| changed_at     | TIMESTAMP |

---

# 8. Permissions Matrix

| Action                | Super Admin | School Admin | Principal | Teacher           | Accountant | Parent         | Student        |
| --------------------- | ----------- | ------------ | --------- | ----------------- | ---------- | -------------- | -------------- |
| View Assessments      | ✓           | ✓            | ✓         | Assigned Subjects | ✗          | Published Only | Published Only |
| Create Assessments    | ✓           | ✓            | ✓         | ✗                 | ✗          | ✗              | ✗              |
| Edit Assessments      | ✓           | ✓            | ✓         | Limited           | ✗          | ✗              | ✗              |
| Enter Marks           | ✓           | ✓            | Optional  | Assigned Subjects | ✗          | ✗              | ✗              |
| Verify Marks          | ✓           | ✓            | ✓         | ✗                 | ✗          | ✗              | ✗              |
| Publish Results       | ✓           | ✓            | ✓         | ✗                 | ✗          | ✗              | ✗              |
| Generate Report Cards | ✓           | ✓            | ✓         | ✗                 | ✗          | ✗              | ✗              |
| View Report Cards     | ✓           | ✓            | ✓         | Assigned Students | ✗          | Own Child      | Self           |
| View Audit Logs       | ✓           | ✓            | ✓         | ✗                 | ✗          | ✗              | ✗              |

---

# 9. APIs Needed

## Assessment APIs

### POST /assessments

Create assessment

---

### GET /assessments

List assessments

---

### PATCH /assessments/{id}

Update assessment

---

### POST /assessments/{id}/activate

Activate assessment

---

## Assessment Component APIs

### POST /assessment-components

Create component

---

### GET /assessment-components

List components

---

## Marks APIs

### POST /marks-entries/bulk

Bulk marks entry

---

### PATCH /marks-entries/{id}

Update marks

---

### GET /marks-entries

List marks

---

### POST /marks-entries/submit

Submit marks

---

### POST /marks-entries/verify

Verify marks

---

## Grading APIs

### POST /grading-schemes

Create grading scheme

---

### GET /grading-schemes

List grading schemes

---

### PATCH /grading-schemes/{id}

Update grading scheme

---

## Report Card APIs

### POST /report-cards/generate

Generate report cards

---

### GET /report-cards

List report cards

---

### GET /report-cards/{id}

Get report card

---

### POST /report-cards/{id}/publish

Publish report card

---

### POST /report-cards/{id}/revise

Create revised version

---

## Publication APIs

### POST /result-publications

Create publication

---

### POST /result-publications/{id}/publish

Publish results

---

### POST /result-publications/{id}/withdraw

Withdraw publication

---

# 10. Screens Needed

## Assessment Management Screen

Features:

* Create assessments
* Manage components
* Configure weights
* Assessment status tracking

---

## Marks Entry Screen

Features:

* Student roster
* Bulk marks entry
* Validation checks
* Auto-save

---

## Marks Verification Screen

Features:

* Review entered marks
* Missing marks alerts
* Verification actions

---

## Grading Scheme Screen

Features:

* Grade boundaries
* GPA configuration
* Scheme preview

---

## Report Card Template Builder

Features:

* School branding
* Layout customization
* Subject grouping
* Teacher remarks section

---

## Report Card Management Screen

Features:

* Generate reports
* Publish reports
* Revision history

---

## Parent Report Card View

Displays:

* Published results
* Subject performance
* Teacher comments

---

## Student Academic Dashboard

Displays:

* Current grades
* Historical results
* Download report cards

---

## Academic Analytics Dashboard

Displays:

* Grade distribution
* Subject performance
* Pass/fail rates
* Class rankings (if enabled)

---

# 11. Edge Cases

### EC-001

Teacher enters marks after deadline.

System blocks entry or requires override.

---

### EC-002

Student joins after assessment creation.

Student automatically added to assessment roster.

---

### EC-003

Student transfers section mid-term.

Historical marks remain linked to original enrollment.

---

### EC-004

Assessment component weight exceeds 100%.

Publication blocked.

---

### EC-005

Marks exceed maximum marks.

Save blocked.

---

### EC-006

Teacher enters marks for unassigned subject.

Operation blocked.

---

### EC-007

Report card published with discovered error.

New version created.

Previous version retained.

---

### EC-008

Student absent for assessment.

Special status recorded.

Marks not treated as zero automatically.

---

### EC-009

Grading scheme changed after publication.

Published report cards remain unchanged.

---

### EC-010

Parent account linked after publication.

Historical report cards become accessible.

---

# 12. Risks

## Academic Integrity Risk

Unauthorized mark changes.

**Mitigation**

* Audit logs
* Verification workflow
* Role restrictions

---

## Calculation Risk

Incorrect weighted grade calculations.

**Mitigation**

* Centralized grading engine
* Automated validation tests

---

## Operational Risk

Missing marks delaying publication.

**Mitigation**

* Completion dashboard
* Missing marks alerts

---

## Compliance Risk

Published records modified without history.

**Mitigation**

* Immutable published versions
* Revision tracking

---

## Scalability Risk

Large schools processing millions of marks.

**Mitigation**

* Bulk operations
* Optimized indexing
* Background report generation

---

# 13. Success Metrics

## Operational Metrics

* Marks entry completion before deadline > 95%
* Report card generation success > 99%
* Verification turnaround < 48 hours

---

## Product Metrics

* Marks entry screen load time < 2 seconds
* Report card generation < 5 minutes per grade
* Bulk import success rate > 98%

---

## Parent Transparency Metrics

* Parent report card access rate > 80%
* Student report card download rate > 70%

---

## Business Metrics

* Reduction in report card preparation effort > 80%
* Reduction in grading calculation errors > 95%
* Increase in parent engagement with academic results

---

# 14. Out of Scope

The following are excluded from MVP:

* AI-generated teacher comments
* Outcome-based education analytics
* Learning competency tracking
* University transcript generation
* National board integration
* External examination authority synchronization
* Online exam delivery
* Question paper management
* Answer sheet scanning and evaluation
* AI grading
* Predictive academic performance models
* Scholarship eligibility engines
* Advanced ranking algorithms

---

# Architecture Recommendations for a Solo Founder

### Critical Dependency Chain

```text
Student Information System
        ↓
Academic Structure
        ↓
Teacher Assignments
        ↓
Assessments
        ↓
Marks Entry
        ↓
Grade Calculation
        ↓
Report Cards
        ↓
Parent & Student Portal
```

No module should bypass this flow.

---

### Grade Calculation Strategy

Store all three values:

```text
Raw Marks
Weighted Marks
Final Grade
```

Never recalculate published report cards from live grading rules.

Persist calculated results at publication time.

---

### Report Card Versioning

Use immutable versions:

```text
Version 1 → Published
Version 2 → Revised
Version 3 → Revised
```

Never overwrite published records.

---

### MVP Recommendation

Build in this order:

1. Assessments
2. Marks Entry
3. Grading Schemes
4. Verification Workflow
5. Report Card Generation
6. Result Publication
7. Parent/Student Result View

Avoid advanced ranking, competency frameworks, and AI-generated comments in V1. Focus on accurate marks management, secure publication workflows, and professional report card generation, which are the features most schools expect from a commercial K-12 gradebook system.
