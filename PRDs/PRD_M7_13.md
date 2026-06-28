# Module 7.13 Exams and Publishing

The Exams and Publishing module manages formal examination cycles from exam-term setup and scheduling through marksheet preparation, verification, publication, and parent/student result visibility.

This module enables schools to:
- Define exam terms and exam windows
- Schedule exams by class, section, subject, room, and date
- Prevent timetable conflicts during exam scheduling
- Generate marksheets for formal exam cycles
- Group result publication into controlled batches
- Publish results class-wise or school-wide
- Expose results to parents and students only after official publication
- Maintain auditable exam and publication history

The module must integrate with academic structure, teacher assignments, timetable, student enrollment, and gradebook workflows so that exam scheduling and publication use the same academic source of truth as the rest of the platform.

***

## 1. Module Objective

Schools run formal examinations as high-stakes academic events. Without a structured exam workflow:
- Exam schedules are managed manually
- Conflicts between subjects, rooms, and sections occur
- Marksheets are prepared inconsistently
- Result release is delayed
- Parents receive unclear or premature information
- Publication history is difficult to audit

Schools need this module to:
- Standardize exam cycles across the academic year
- Prevent scheduling conflicts before exams go live
- Organize marksheet generation and review
- Control publication timing and scope
- Improve parent transparency through official result release
- Make exam operations measurable and auditable

***

## 2. Why Schools Need It

### 2.1 Formal Exam Control
Examinations require stricter planning and governance than regular classroom assessments, including schedules, publishing controls, and auditability.

### 2.2 Reduced Administrative Work
Schools need bulk exam setup, timetable validation, marksheet preparation, and controlled publication rather than spreadsheet-driven coordination.

### 2.3 Publication Discipline
Parents cannot see results until publication is active, which requires a clear and enforceable publication model consistent with published academic records. 

### 2.4 Conflict Prevention
Exam timetable conflicts must be prevented for sections, rooms, invigilators where applicable, and subject scheduling patterns, following the same scheduling discipline used in academic timetable workflows.

### 2.5 Better Parent Transparency
Once results are officially published, parents and students should have consistent visibility into formal exam outcomes through the portal model used elsewhere in the platform. 

### 2.6 Operational Measurement
School leaders need visibility into exam completion, marksheet readiness, publication turnaround, and exam-cycle execution quality.

***

## 3. Primary Users

### 3.1 School Admin
Primary exam operations owner. Responsibilities:
- Configure exam terms
- Create exam schedules
- Validate exam conflicts
- Coordinate marksheet preparation
- Manage publication batches
- Publish results

### 3.2 Principal
Academic oversight user. Responsibilities:
- Review exam plans
- Monitor marksheet readiness
- Approve publication where policy requires
- Track exam-cycle performance

### 3.3 Teacher
Academic execution user. Responsibilities:
- View assigned exam schedules
- Support subject-level exam readiness
- Enter or verify exam marks where integrated with marks workflows
- Review exam-related academic records within assigned scope

### 3.4 Parent
Result visibility user. Responsibilities:
- View published exam results only after active publication
- Download or view marksheets where enabled

### 3.5 Student
Learner visibility user. Responsibilities:
- View published exam timetable where policy allows
- Access published exam results and marksheets after release

### 3.6 Super Admin
Platform oversight user. Responsibilities:
- Audit exam workflows
- Investigate support issues
- Review publication history when needed

### 3.7 Accountant
No primary operational role. Responsibilities:
- No direct exam management permissions in standard workflow

***

## 4. User Stories

### 4.1 User Stories - Exam Term Setup
- **US-001** As a School Admin, I want to create exam terms, so formal examination cycles are organized within the academic year.
- **US-002** As a School Admin, I want exam terms linked to academic year and classes, so exam operations follow school structure.
- **US-003** As a Principal, I want visibility into planned exam terms, so academic planning can be monitored.

### 4.2 User Stories - Exam Scheduling
- **US-004** As a School Admin, I want to create exam schedules by class, section, and subject, so every exam is planned clearly.
- **US-005** As a School Admin, I want exam timetable conflicts prevented, so students and staff do not receive invalid schedules.
- **US-006** As a School Admin, I want room and time allocation tracked, so exam logistics are manageable.
- **US-007** As a Teacher, I want to view my exam schedule, so I can prepare for invigilation or subject coordination.
- **US-008** As a Student, I want to view my exam timetable, so I know when each subject exam will occur.

### 4.3 User Stories - Marksheets
- **US-009** As a School Admin, I want marksheets generated for exam cycles, so formal exam records are centralized.
- **US-010** As a Teacher, I want subject marks reflected in marksheets within my authorized scope, so academic records are accurate.
- **US-011** As a Principal, I want marksheet readiness visibility, so publication is not activated prematurely.
- **US-012** As a School Admin, I want incomplete marksheets identified, so follow-up happens before publication.

### 4.4 User Stories - Publication Management
- **US-013** As a School Admin, I want publication batches, so result release can be controlled in groups.
- **US-014** As a School Admin, I want publication to happen class-wise or school-wide, so release strategy matches school policy.
- **US-015** As a Principal, I want publication approval before release where required, so academic governance is maintained.
- **US-016** As a Parent, I want exam results visible only after official publication, so I see final and reliable information.
- **US-017** As a Student, I want published results available in one place, so I can track formal exam performance.

### 4.5 User Stories - Visibility and History
- **US-018** As a Parent, I want to view marksheets after publication, so I can monitor my child’s performance.
- **US-019** As a Student, I want to download published marksheets where enabled, so I can keep a copy.
- **US-020** As a School Admin, I want publication history tracked, so I can audit what was released and when.
- **US-021** As a Super Admin, I want exam and publication actions audited, so platform support is manageable.

### 4.6 User Stories - Reporting
- **US-022** As a Principal, I want exam-cycle summaries, so I can monitor operational completion.
- **US-023** As a School Admin, I want class-wise exam completion and publication dashboards, so delays are visible.
- **US-024** As a Teacher, I want outstanding marksheet dependencies shown, so missing academic inputs can be resolved.

***

## 5. Business Rules

### 5.1 Business Rules - Tenant Isolation
- **BR-001** Every exam term, exam schedule, exam subject, marksheet, and publication batch must belong to exactly one tenant.
- **BR-002** Exam and publication data cannot be accessed across tenants.
- **BR-003** All exam-related APIs and queries must enforce tenant scope automatically.

### 5.2 Business Rules - Exam Term Rules
- **BR-004** Exam terms must belong to a single academic year.
- **BR-005** Exam terms may be configured for specific grades, sections, or school-wide use depending on school policy.
- **BR-006** Archived academic years cannot receive new exam-term changes, consistent with academic structure controls. 
- **BR-007** Exam term status changes must be auditable.

### 5.3 Business Rules - Exam Scheduling Rules
- **BR-008** Each exam schedule must reference Academic Year Grade Section Subject Exam Term Date Time Slot.
- **BR-009** Exam timetable conflicts must be prevented.
- **BR-010** A section cannot have overlapping exams.
- **BR-011** A room cannot be assigned to overlapping exam slots if room allocation is enabled.
- **BR-012** A teacher or invigilator cannot be assigned to overlapping exam duties if invigilation tracking is enabled.
- **BR-013** Validation must occur before exam schedule publication, following the same conflict-prevention principle used in timetable workflows. 
- **BR-014** Published exam schedules must preserve version history.

### 5.4 Business Rules - Exam Subject Rules
- **BR-015** Exam subjects must belong to valid subject and academic context records.
- **BR-016** Exam subjects may vary by exam term and class configuration.
- **BR-017** Only authorized academic staff may create or modify exam subject mappings.
- **BR-018** Students must only see exam subjects assigned to their academic context.

### 5.5 Business Rules - Marksheet Rules
- **BR-019** Marksheets must be generated from official exam-cycle academic records.
- **BR-020** Marksheets cannot be marked ready if mandatory subject entries are incomplete.
- **BR-021** Marksheet generation must preserve calculation or data snapshot at the time of readiness or publication where applicable, consistent with published academic record principles. 
- **BR-022** A published marksheet must not be silently overwritten.
- **BR-023** Any correction after publication must create a tracked revision or controlled republication workflow if enabled by school policy.

### 5.6 Business Rules - Publication Rules
- **BR-024** Publication may happen class-wise or school-wide.
- **BR-025** Publication batches must group marksheets or result sets into a controlled release unit.
- **BR-026** Parents cannot see results until publication is active.
- **BR-027** Students cannot see results until publication is active.
- **BR-028** Publication activation may be immediate or scheduled.
- **BR-029** Publication must be blocked if required marksheets are incomplete.
- **BR-030** Publication scope and activation time must be audited.
- **BR-031** Only authorized users may activate or withdraw publication.

### 5.7 Business Rules - Parent and Student Visibility Rules
- **BR-032** Parent access must be limited to linked children only, following relationship-based access controls. 
- **BR-033** Student access must be limited to self only.
- **BR-034** Draft exam schedules and unpublished results must not be visible to parents or students.
- **BR-035** Historical published results may remain visible according to school retention policy.

### 5.8 Business Rules - Audit and Reporting Rules
- **BR-036** Every create, edit, validate, publish, withdraw, and republish action must be auditable.
- **BR-037** Exam-cycle reports must support filtering by exam term, grade, section, subject, and publication status.
- **BR-038** Searchable history must preserve exam schedule versions and publication events.
- **BR-039** The Exams and Publishing module must not bypass gradebook publication controls when consuming published academic results. 

***

## 6. State Machine

### 6.1 State Machine - Exam Term Lifecycle
```text
Draft
  v
Scheduled
  v
Active
  v
Completed
  v
Archived
```

### 6.2 State Machine - Exam Schedule Lifecycle
```text
Draft
  v
Under Validation
  |------ Validation Failed
  |         v
  |------ Draft
  v
Published
  v
Archived
```

### 6.3 State Machine - Marksheet Lifecycle
```text
Pending
  v
Generated
  v
Verified
  v
Published
  |------ Revised
  |         v
  |------ Published
```

### 6.4 State Machine - Publication Batch Lifecycle
```text
Draft
  v
Ready
  v
Scheduled
  |------ Active
  |         v
  |------ Completed
  |------ Withdrawn
```

***

## 7. Required Data Entities

### 7.1 Required Data Entities - exam_terms
Field | Type
--- | ---
id | UUID
tenantid | UUID
schoolid | UUID
academicyearid | UUID
termname | VARCHAR
description | TEXT NULL
startdate | DATE
enddate | DATE
status | ENUM
createdby | UUID
createdat | TIMESTAMP
updatedat | TIMESTAMP

### 7.2 Required Data Entities - exam_schedules
Field | Type
--- | ---
id | UUID
tenantid | UUID
schoolid | UUID
examtermid | UUID
academicyearid | UUID
gradelevelid | UUID
sectionid | UUID
examdate | DATE
starttime | TIME
endtime | TIME
roomid | UUID NULL
status | ENUM
versionnumber | INTEGER
publishedat | TIMESTAMP NULL
createdat | TIMESTAMP
updatedat | TIMESTAMP

### 7.3 Required Data Entities - exam_subjects
Field | Type
--- | ---
id | UUID
tenantid | UUID
examscheduleid | UUID
subjectid | UUID
teacherassignmentid | UUID NULL
maximummarks | DECIMAL NULL
status | ENUM
createdat | TIMESTAMP
updatedat | TIMESTAMP

### 7.4 Required Data Entities - marksheets
Field | Type
--- | ---
id | UUID
tenantid | UUID
schoolid | UUID
examtermid | UUID
studentid | UUID
gradelevelid | UUID
sectionid | UUID
status | ENUM
generatedat | TIMESTAMP NULL
verifiedat | TIMESTAMP NULL
publishedat | TIMESTAMP NULL
versionnumber | INTEGER
createdat | TIMESTAMP
updatedat | TIMESTAMP

### 7.5 Required Data Entities - publication_batches
Field | Type
--- | ---
id | UUID
tenantid | UUID
schoolid | UUID
examtermid | UUID
publicationtype | ENUM
scopelevel | ENUM
scopevalue | VARCHAR NULL
status | ENUM
scheduledat | TIMESTAMP NULL
activatedat | TIMESTAMP NULL
createdby | UUID
createdat | TIMESTAMP
updatedat | TIMESTAMP

### 7.6 Additional Supporting Entities
- `exam_schedule_slots`
- `exam_schedule_conflicts`
- `marksheet_subject_entries`
- `publication_batch_items`
- `exam_publication_history`
- `exam_audit_logs`

### 7.7 Required Data Entities - publication_batch_items
Field | Type
--- | ---
id | UUID
tenantid | UUID
publicationbatchid | UUID
marksheetid | UUID
publicationstatus | ENUM
createdat | TIMESTAMP

### 7.8 Required Data Entities - exam_schedule_conflicts
Field | Type
--- | ---
id | UUID
tenantid | UUID
examscheduleid | UUID
conflicttype | ENUM
conflictref | VARCHAR
status | ENUM
detectedat | TIMESTAMP
resolvedat | TIMESTAMP NULL

***

## 8. Permissions Matrix

| Capability | Super Admin | School Admin | Principal | Teacher | Accountant | Parent | Student |
|---|---|---|---|---|---|---|---|
| View exam dashboard | Yes | Yes | Yes | Limited assigned scope | No | No | No |
| Create exam terms | Limited Support | Yes | Yes if allowed | No | No | No | No |
| Edit exam terms | Limited Support | Yes | Yes if allowed | No | No | No | No |
| Create exam schedules | No | Yes | Yes if allowed | No | No | No | No |
| Validate schedules | Limited Audit | Yes | Yes | Limited read-only scope | No | No | No |
| Publish exam schedules | No | Yes | Yes if allowed | No | No | No | No |
| View marksheets | Limited Audit | Yes | Yes | Limited assigned academic scope | No | Child only after publication | Self only after publication |
| Verify marksheets | No | Yes | Yes | Limited assigned scope if enabled | No | No | No |
| Create publication batches | No | Yes | Yes if allowed | No | No | No | No |
| Activate publication | No | Yes | Yes if allowed | No | No | No | No |
| View publication history | Limited Audit | Yes | Yes | Limited own scope | No | No | No |
| Download published marksheet | No | No | No | No | No | Child only if enabled | Self only if enabled |

***

## 9. APIs Needed

### 9.1 APIs Needed - Exam Term APIs
- `POST /exam-terms`
- `GET /exam-terms`
- `GET /exam-terms/{id}`
- `PATCH /exam-terms/{id}`
- `POST /exam-terms/{id}/activate`
- `POST /exam-terms/{id}/complete`

### 9.2 APIs Needed - Exam Schedule APIs
- `POST /exam-schedules`
- `GET /exam-schedules`
- `GET /exam-schedules/{id}`
- `PATCH /exam-schedules/{id}`
- `POST /exam-schedules/{id}/validate`
- `POST /exam-schedules/{id}/publish`
- `POST /exam-schedules/{id}/archive`

### 9.3 APIs Needed - Exam Subject APIs
- `POST /exam-subjects`
- `GET /exam-subjects`
- `PATCH /exam-subjects/{id}`
- `DELETE /exam-subjects/{id}`

### 9.4 APIs Needed - Marksheet APIs
- `POST /marksheets/generate`
- `GET /marksheets`
- `GET /marksheets/{id}`
- `POST /marksheets/{id}/verify`
- `POST /marksheets/{id}/revise`
- `GET /marksheets/{id}/download`

### 9.5 APIs Needed - Publication APIs
- `POST /publication-batches`
- `GET /publication-batches`
- `GET /publication-batches/{id}`
- `PATCH /publication-batches/{id}`
- `POST /publication-batches/{id}/ready`
- `POST /publication-batches/{id}/schedule`
- `POST /publication-batches/{id}/activate`
- `POST /publication-batches/{id}/withdraw`

### 9.6 APIs Needed - Parent and Student Result APIs
- `GET /parent/child/{studentId}/exam-results`
- `GET /student/my-exam-results`
- `GET /parent/child/{studentId}/marksheets/{id}`
- `GET /student/my-marksheets/{id}`

### 9.7 APIs Needed - Reporting APIs
- `GET /exams/reports/completion`
- `GET /exams/reports/publication-status`
- `GET /exams/reports/conflicts`
- `GET /exams/search`

***

## 10. Screens Needed

### 10.1 Screens Needed - Exams Dashboard
Displays:
- Active exam terms
- Schedule readiness
- Conflict counts
- Marksheet readiness
- Publication status
- Pending approvals

### 10.2 Screens Needed - Exam Term Management Screen
Features:
- Create exam term
- Edit exam window
- Assign academic context
- Track status

### 10.3 Screens Needed - Exam Schedule Builder Screen
Features:
- Add subjects
- Assign date and time
- Room assignment
- Conflict indicators
- Draft save
- Publish schedule

### 10.4 Screens Needed - Exam Conflict Validation Screen
Displays:
- Section conflicts
- Room conflicts
- Staff conflicts
- Resolution status
- Validation summary

### 10.5 Screens Needed - Marksheet Readiness Screen
Displays:
- Student-wise marksheet status
- Missing subject entries
- Verified vs pending counts
- Generation status

### 10.6 Screens Needed - Publication Batch Management Screen
Features:
- Create batch
- Choose class-wise or school-wide scope
- Schedule release
- Activate publication
- Withdraw publication

### 10.7 Screens Needed - Parent and Student Result View
Displays:
- Published exam results
- Marksheet access
- Publication date
- Download action if enabled

### 10.8 Screens Needed - Exam Analytics Dashboard
Displays:
- Exam completion
- Conflict rate
- Publication turnaround
- Class-wise readiness
- Batch release status

***

## 11. Edge Cases

- **EC-001** School Admin schedules two exams for the same section at overlapping times. System blocks publication until conflict is resolved.
- **EC-002** Same room is assigned to two exam slots at the same time. Conflict validation fails.
- **EC-003** Teacher or invigilator is assigned to overlapping exam duties where duty assignment tracking is enabled. Conflict is flagged.
- **EC-004** Exam term is edited after schedules are already published. System requires controlled update or new version.
- **EC-005** Some marksheets are generated but others remain incomplete for a class-wise publication batch. Batch cannot activate until required items are ready.
- **EC-006** Publication is scheduled, but marksheet verification is revoked before activation time. Activation is blocked.
- **EC-007** Parent attempts to access results before publication becomes active. Access denied.
- **EC-008** Student changes section mid-term. Historical exam results remain linked to original exam context and student record continuity rules. [ppl-ai-file-upload.s3.amazonaws]
- **EC-009** Parent is linked after publication is already active. Historical published results become visible according to relationship and visibility rules. 
- **EC-010** Marksheet is corrected after publication. New revision or controlled republication flow is required.
- **EC-011** Class-wise publication is active while school-wide publication remains pending. Visibility is limited to published scope only.
- **EC-012** Schedule validation passes, but room is archived before publication. Final publish must revalidate dependencies.
- **EC-013** Academic year is archived while an exam term is still draft. Further changes are blocked by academic calendar rules. 
- **EC-014** Publication is withdrawn after parents have already viewed results. Audit history remains preserved and current visibility follows withdrawn state policy.

***

## 12. Risks

### 12.1 Risks - Scheduling Risk
Invalid exam schedules create operational disruption.  
Mitigation: Pre-publication validation, conflict engine, dependency checks.

### 12.2 Risks - Publication Risk
Results become visible before official approval.  
Mitigation: Controlled publication batches, explicit active state, permission-gated release, portal visibility rules. 

### 12.3 Risks - Data Integrity Risk
Marksheets are generated from incomplete or inconsistent academic data.  
Mitigation: Readiness checks, verification workflow, dependency validation.

### 12.4 Risks - Privacy Risk
Parents or students access unpublished or unrelated results.  
Mitigation: Publication-state enforcement, relationship-based filtering, tenant isolation. 

### 12.5 Risks - Compliance Risk
Published result history cannot be reconstructed during disputes.  
Mitigation: Immutable publication logs, marksheet versioning, audit trails.

### 12.6 Risks - Scalability Risk
Large exam cycles create heavy generation and publication loads.  
Mitigation: Batch processing, background jobs, indexed search, optimized reporting patterns similar to other large academic modules. 

### 12.7 Risks - Operational Delay Risk
Schools miss publication timelines because marksheet dependencies are unclear.  
Mitigation: Readiness dashboard, missing-item alerts, class-wise progress tracking.

***

## 13. Success Metrics

### 13.1 Success Metrics - Product Metrics
- Exam schedule creation success rate 99
- Conflict detection accuracy 100
- Marksheet generation success rate 99
- Publication batch activation success rate 98
- Exam dashboard load time 2 seconds

### 13.2 Success Metrics - Operational Metrics
- Schedule validation completed before publication 100
- Marksheet readiness completion before planned release 95
- Publication turnaround within 48 hours of marksheet verification 90
- Conflict resolution turnaround within 24 hours for active exam terms
- Class-wise publication completion within planned window 95

### 13.3 Success Metrics - Parent Transparency Metrics
- Parent published result access rate 80
- Student published marksheet access rate 70
- Unauthorized pre-publication result visibility incidents 0

### 13.4 Success Metrics - Data Quality Metrics
- Invalid published schedules 0
- Incomplete marksheets in activated batches 0
- Duplicate publication events for same marksheet below 0.5
- Publication audit coverage 100

### 13.5 Success Metrics - Business Metrics
- Reduction in manual exam administration effort 60
- Improved reliability of formal exam publishing workflows
- Increased parent trust through controlled and timely result release
- Improved measurability of exam-cycle execution across grades and classes

***

## 14. Out of Scope

The following are excluded from the Exams and Publishing module MVP:
- Question paper authoring
- Question bank management
- Online exam delivery
- Answer sheet scanning
- AI proctoring
- Hall ticket generation
- Seating-plan optimization engine
- Advanced invigilation workforce planning
- Board-exam external authority integrations
- University-style exam registration
- Automated rank generation across complex boards
- AI-generated comments
- Full transcript generation beyond exam-cycle marksheets
- Public exam-result portal outside authenticated parent and student access