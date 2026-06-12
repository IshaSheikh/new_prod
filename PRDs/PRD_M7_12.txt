# Module 7.12 Assignments and Homework

The Assignments and Homework module manages the creation, publication, submission, review, and feedback lifecycle of academic work assigned to students. It supports daily teaching workflows by enabling teachers to assign work by class, section, and subject while giving students and parents structured visibility into pending tasks, deadlines, and feedback.

This module enables schools to:
- Create assignments by academic context
- Publish homework and classwork tasks
- Accept text-based and file-based submissions
- Apply deadline and late-submission policies
- Review submissions and provide feedback
- Track completion and submission rates
- Expose assignment visibility to parents and students through controlled portal views

The module must integrate with academic structure, teacher assignments, parent-student relationships, and student enrollment records. It must not create independent academic rosters outside the platform’s subject, section, and timetable model. 

***

## 1. Module Objective

Schools manage assignments and homework every day, but many still depend on notebooks, chat groups, spreadsheets, and informal follow-up. Without a structured assignment workflow:
- Teachers repeat the same communication manually
- Students miss deadlines
- Parents lack visibility into pending academic work
- Submission records are inconsistent
- Feedback becomes hard to track
- Completion reporting is unreliable

Schools need this module to:
- Support daily academic execution
- Standardize assignment creation and submission tracking
- Give students one place to view pending work
- Improve parent transparency into homework and deadlines
- Reduce manual teacher coordination
- Make assignment completion measurable by class, section, and subject

***

## 2. Why Schools Need It

### 2.1 Support Daily Academic Workflow
Assignments and homework are core academic activities. Schools need a structured digital workflow to support regular classroom work and follow-up.

### 2.2 Reduce Teacher Administrative Effort
Teachers should be able to assign work once and have it visible to all relevant students without repeated manual communication.

### 2.3 Improve Student Accountability
Students need clear access to assignment instructions, due dates, submission status, and feedback.

### 2.4 Improve Parent Transparency
Parents should be able to see assigned work, due dates, and completion visibility for linked children where policy allows.

### 2.5 Standardize Submission Tracking
A school needs reliable records of who submitted, when, in what format, and whether work was late.

### 2.6 Improve Academic Monitoring
Principals and School Admins need insight into assignment volume, completion rates, overdue work, and teacher usage.

### 2.7 Support Multi-Channel Learning Contexts
Assignments may include instructions, attachments, worksheets, or text prompts and should support multiple submission modes.

***

## 3. Primary Users

### 3.1 Teacher
Primary assignment owner. Responsibilities:
- Create assignments
- Publish assignments to classes and sections
- Attach files and instructions
- Review submissions
- Provide grading or feedback
- Monitor completion status

### 3.2 Student
Primary submission user. Responsibilities:
- View assignments
- Submit text or files
- Track deadline and submission status
- Review feedback

### 3.3 Parent
Visibility user. Responsibilities:
- View child assignments where enabled
- Monitor pending and overdue homework
- Review submission status and teacher feedback where policy allows

### 3.4 School Admin
Operational oversight user. Responsibilities:
- Configure submission and late rules
- Monitor assignment usage
- Review assignment analytics
- Support staff usage and policy enforcement

### 3.5 Principal
Academic oversight user. Responsibilities:
- Review assignment completion trends
- Monitor teacher adoption
- Observe workload distribution and student compliance

### 3.6 Super Admin
Platform oversight user. Responsibilities:
- Audit workflow behavior
- Support tenant issues
- Monitor reliability and adoption metrics

### 3.7 Accountant
No primary operational role. Responsibilities:
- No direct assignment management access in normal workflow

***

## 4. User Stories

### 4.1 User Stories - Assignment Creation
- **US-001** As a Teacher, I want to create assignments by class, section, and subject, so students receive work relevant to their schedule.
- **US-002** As a Teacher, I want to add title, instructions, due date, and attachments, so assignment expectations are clear.
- **US-003** As a Teacher, I want to save assignments as draft, so I can prepare work before publishing.
- **US-004** As a Teacher, I want to publish assignments to selected students or full sections, so delivery is controlled.

### 4.2 User Stories - Assignment Delivery
- **US-005** As a Student, I want to see all active assignments in one place, so I know what work is pending.
- **US-006** As a Parent, I want to view my child’s assignments, so I can support homework completion.
- **US-007** As a Student, I want assignments sorted by due date and status, so I can prioritize my work.
- **US-008** As a Teacher, I want assignment visibility tied to correct academic context, so the wrong section does not receive work.

### 4.3 User Stories - Submission Workflow
- **US-009** As a Student, I want to submit assignments as text, so I can complete simple tasks without file upload.
- **US-010** As a Student, I want to submit assignments as files, so I can upload worksheets, photos, PDFs, or documents.
- **US-011** As a Student, I want to edit or resubmit before deadline when allowed, so I can correct mistakes.
- **US-012** As a Teacher, I want to see which students submitted, so I can follow up with non-submitters.

### 4.4 User Stories - Deadlines and Late Rules
- **US-013** As a School Admin, I want deadlines to respect school timezone, so assignment due times are consistent.
- **US-014** As a Teacher, I want configurable late submission handling, so I can allow, block, or flag late work.
- **US-015** As a Student, I want to know whether a submission is on time or late, so I understand my status.
- **US-016** As a Parent, I want overdue homework visibility, so I can help my child respond quickly.

### 4.5 User Stories - Review and Feedback
- **US-017** As a Teacher, I want to review individual submissions, so I can evaluate student work.
- **US-018** As a Teacher, I want to leave grading feedback, so students understand what was done well or incorrectly.
- **US-019** As a Student, I want to view teacher feedback, so I can improve future work.
- **US-020** As a Parent, I want to see feedback on my child’s homework where allowed, so I can monitor academic progress.

### 4.6 User Stories - Monitoring and Reporting
- **US-021** As a School Admin, I want assignment completion reports, so academic execution becomes measurable.
- **US-022** As a Principal, I want submission trends by class and teacher, so I can identify engagement issues.
- **US-023** As a Teacher, I want a summary of pending, submitted, and late assignments, so I can manage classroom follow-up.
- **US-024** As a Super Admin, I want audit visibility for assignment publication and submission activity, so platform support is manageable.

***

## 5. Business Rules

### 5.1 Business Rules - Tenant Isolation
- **BR-001** Every assignment, submission, attachment, and feedback record must belong to exactly one tenant.
- **BR-002** Assignment data cannot be accessed across tenants.
- **BR-003** All assignment queries must enforce tenant scope automatically.

### 5.2 Business Rules - Assignment Context Rules
- **BR-004** Teachers create assignments by class, section, and subject.
- **BR-005** Assignment creation must reference Academic Year Grade Section Subject and assigned teacher context where applicable. 
- **BR-006** Only authorized teachers may create assignments for assigned academic contexts. 
- **BR-007** Assignments may target a full section, a group of students, or an individual student if school policy allows.
- **BR-008** Assignment publication must preserve recipient snapshot at publish time for auditability.

### 5.3 Business Rules - Content and Attachment Rules
- **BR-009** Assignments must support title, instructions, due datetime, and optional attachments.
- **BR-010** Supported attachment types and size limits must be configurable by school or platform policy.
- **BR-011** Assignment instructions may contain rich text or plain text depending on product scope.
- **BR-012** Published assignments must preserve content snapshot for audit and portal visibility.

### 5.4 Business Rules - Submission Rules
- **BR-013** Submissions may be file-based or text-based.
- **BR-014** Assignment submission mode may allow text only, file only, or both.
- **BR-015** A student may submit only if the assignment is published and visible to that student.
- **BR-016** Submission eligibility must reference active enrollment and assignment audience rules.
- **BR-017** Multiple submission attempts may be allowed based on assignment configuration.
- **BR-018** The latest allowed submission version must be identifiable.
- **BR-019** Submission history must remain auditable.

### 5.5 Business Rules - Deadline and Late Rules
- **BR-020** Deadlines are school-timezone aware.
- **BR-021** Late submission rules are configurable.
- **BR-022** Late policy options may include blocked, accepted and flagged, or accepted with teacher override.
- **BR-023** Submission timestamp must be stored in a canonical timezone-safe format and evaluated against school timezone rules.
- **BR-024** Changing deadline after publication must be audited.
- **BR-025** Students and parents must see updated deadline values after authorized changes.

### 5.6 Business Rules - Review and Feedback Rules
- **BR-026** Teachers may review submissions only for their authorized academic scope.
- **BR-027** Feedback may include text comments, status, score, or attachment where enabled.
- **BR-028** Feedback publication to student and parent must follow school visibility policy.
- **BR-029** Reviewed submissions must retain history of grading feedback updates if edited after publication.
- **BR-030** Final feedback state must be clearly distinguishable from draft teacher notes if draft notes are supported.

### 5.7 Business Rules - Parent and Student Visibility Rules
- **BR-031** Students see only their own assignments and submissions.
- **BR-032** Parents see assignments only for linked children and only where school policy enables parent visibility.
- **BR-033** Parent-child assignment visibility must follow relationship-based access controls. 
- **BR-034** Draft assignments must not be visible to students or parents.
- **BR-035** Archived assignments may remain visible historically according to school policy.

### 5.8 Business Rules - Audit and Reporting Rules
- **BR-036** Every create, edit, publish, unpublish, submit, resubmit, review, and feedback publish action must be auditable.
- **BR-037** Assignment analytics must distinguish on-time, late, missing, and reviewed submissions.
- **BR-038** Historical assignment records must remain searchable by class, section, subject, teacher, student, and date range.
- **BR-039** The Assignments and Homework module must not directly modify official gradebook results unless a separate integration is explicitly configured. 

***

## 6. State Machine

### 6.1 State Machine - Assignment Lifecycle
```text
Draft
  v
Published
  |------ Updated
  |         v
  |------ Published
  |------ Closed
  v
Archived
```

### 6.2 State Machine - Submission Lifecycle
```text
Not Submitted
  v
Submitted
  |------ Resubmitted
  |------ Late Submitted
  v
Under Review
  |------ Reviewed
  |------ Returned
  v
Closed
```

### 6.3 State Machine - Feedback Lifecycle
```text
Draft
  v
Published
  |------ Revised
  |         v
  |------ Published
```

***

## 7. Required Data Entities

### 7.1 Required Data Entities - assignments
Field | Type
--- | ---
id | UUID
tenantid | UUID
schoolid | UUID
academicyearid | UUID
gradelevelid | UUID
sectionid | UUID
subjectid | UUID
teacheruserid | UUID
title | VARCHAR
instructions | TEXT
submissionmode | ENUM
duedatetime | TIMESTAMP
latesubmissionpolicy | ENUM
status | ENUM
publishedat | TIMESTAMP NULL
createdat | TIMESTAMP
updatedat | TIMESTAMP

### 7.2 Required Data Entities - assignment_submissions
Field | Type
--- | ---
id | UUID
tenantid | UUID
assignmentid | UUID
studentid | UUID
submissiontext | TEXT NULL
submittedat | TIMESTAMP NULL
submissionstatus | ENUM
islate | BOOLEAN
attemptnumber | INTEGER
latestversion | BOOLEAN
createdat | TIMESTAMP
updatedat | TIMESTAMP

### 7.3 Required Data Entities - attachments
Field | Type
--- | ---
id | UUID
tenantid | UUID
entitytype | ENUM
entityid | UUID
filename | VARCHAR
filepath | VARCHAR
filetype | VARCHAR
filesize | BIGINT
uploadedby | UUID
createdat | TIMESTAMP

### 7.4 Required Data Entities - grading_feedback
Field | Type
--- | ---
id | UUID
tenantid | UUID
assignmentsubmissionid | UUID
teacheruserid | UUID
feedbacktext | TEXT NULL
score | DECIMAL NULL
feedbackstatus | ENUM
publishedat | TIMESTAMP NULL
createdat | TIMESTAMP
updatedat | TIMESTAMP

### 7.5 Additional Supporting Entities
- `assignment_audiences`
- `assignment_submission_versions`
- `assignment_events`
- `assignment_visibility_settings`
- `late_submission_overrides`
- `assignment_review_statuses`

### 7.6 Required Data Entities - assignment_audiences
Field | Type
--- | ---
id | UUID
tenantid | UUID
assignmentid | UUID
audiencetype | ENUM
audienceref | UUID
recipientcount | INTEGER
resolvedsnapshot | JSONB
createdat | TIMESTAMP

### 7.7 Required Data Entities - assignment_submission_versions
Field | Type
--- | ---
id | UUID
tenantid | UUID
assignmentsubmissionid | UUID
versionnumber | INTEGER
submissiontext | TEXT NULL
submittedat | TIMESTAMP
islate | BOOLEAN
createdat | TIMESTAMP

***

## 8. Permissions Matrix

| Capability | Super Admin | School Admin | Principal | Teacher | Accountant | Parent | Student |
|---|---|---|---|---|---|---|---|
| View assignment dashboard | Yes | Yes | Yes | Yes own scope | No | No | No |
| Create assignment | Limited Support | Yes | Yes if allowed | Yes assigned scope | No | No | No |
| Edit draft assignment | Limited Support | Yes | Yes if allowed | Yes own draft | No | No | No |
| Publish assignment | No | Yes | Yes if allowed | Yes assigned scope | No | No | No |
| View submissions | Limited Audit | Yes | Yes | Yes assigned scope | No | Child only if enabled | Self only |
| Submit assignment | No | No | No | No | No | No | Yes |
| Resubmit assignment | No | No | No | No | No | No | Yes if allowed |
| Review submission | No | Yes | Yes | Yes assigned scope | No | No | No |
| Publish feedback | No | Yes | Yes if allowed | Yes assigned scope | No | Child only if enabled | Self only |
| View assignment history | Limited Audit | Yes | Yes | Yes own scope | No | Child only if enabled | Self only |
| Configure late rules | Limited Support | Yes | Yes if allowed | No | No | No | No |

***

## 9. APIs Needed

### 9.1 APIs Needed - Assignment APIs
- `POST /assignments`
- `GET /assignments`
- `GET /assignments/{id}`
- `PATCH /assignments/{id}`
- `POST /assignments/{id}/publish`
- `POST /assignments/{id}/close`
- `POST /assignments/{id}/archive`

### 9.2 APIs Needed - Assignment Audience APIs
- `POST /assignments/{id}/resolve-audience`
- `GET /assignments/{id}/audience`
- `POST /assignments/{id}/update-deadline`

### 9.3 APIs Needed - Submission APIs
- `POST /assignments/{id}/submit`
- `POST /assignments/{id}/resubmit`
- `GET /assignments/{id}/submissions`
- `GET /assignments/submissions/{submissionId}`
- `PATCH /assignments/submissions/{submissionId}`

### 9.4 APIs Needed - Feedback APIs
- `POST /assignments/submissions/{submissionId}/feedback`
- `PATCH /assignments/submissions/{submissionId}/feedback`
- `POST /assignments/submissions/{submissionId}/feedback/publish`
- `GET /assignments/submissions/{submissionId}/feedback`

### 9.5 APIs Needed - Student and Parent View APIs
- `GET /student/my-assignments`
- `GET /student/my-assignments/{id}`
- `GET /parent/child/{studentId}/assignments`
- `GET /parent/child/{studentId}/assignments/{id}`

### 9.6 APIs Needed - Reporting APIs
- `GET /assignments/reports/completion`
- `GET /assignments/reports/late-submissions`
- `GET /assignments/reports/teacher-activity`
- `GET /assignments/search`

### 9.7 APIs Needed - Attachment APIs
- `POST /attachments/upload`
- `GET /attachments/{id}`
- `DELETE /attachments/{id}`

***

## 10. Screens Needed

### 10.1 Screens Needed - Teacher Assignment Dashboard
Displays:
- Draft assignments
- Published assignments
- Pending reviews
- Late submissions
- Class-wise completion summary

### 10.2 Screens Needed - Assignment Composer Screen
Features:
- Title and instructions
- Class section subject selection
- Due datetime
- Submission mode
- Attachment upload
- Draft and publish actions

### 10.3 Screens Needed - Assignment List Screen
Features:
- Filter by class
- Filter by subject
- Filter by status
- Search assignments
- Bulk close or archive actions if allowed

### 10.4 Screens Needed - Submission Review Screen
Features:
- Student submission list
- Submission timestamp
- Late flag
- View text or files
- Review status
- Add feedback

### 10.5 Screens Needed - Student Assignment Screen
Features:
- Pending assignments
- Due date sorting
- Submit text
- Upload files
- Submission status
- Feedback visibility

### 10.6 Screens Needed - Parent Assignment View
Displays:
- Child assignments
- Pending and overdue work
- Submission status
- Feedback summary where allowed

### 10.7 Screens Needed - Assignment Analytics Dashboard
Displays:
- Completion rate
- Late submission rate
- Assignment volume by teacher
- Class-wise trends
- Student compliance trends

### 10.8 Screens Needed - Assignment Configuration Screen
Features:
- Late submission rules
- Parent visibility settings
- Attachment type limits
- Submission mode defaults

***

## 11. Edge Cases

- **EC-001** Teacher creates assignment for a section they are not assigned to. Operation blocked.
- **EC-002** Student joins class after assignment publication. Visibility follows configured late-join assignment rule.
- **EC-003** Student transfers section mid-term. Historical assignment records remain linked to original section context where applicable.
- **EC-004** Deadline passes during active file upload. System determines acceptance based on finalized submission timestamp and late policy.
- **EC-005** Student submits text and file when only one mode is allowed. Invalid submission blocked.
- **EC-006** Teacher changes due date after some students already submitted. Existing submissions remain valid and audit history records the change.
- **EC-007** Student resubmits after teacher already reviewed earlier version. Review history must remain traceable.
- **EC-008** Parent is linked after assignment publication. Historical visible assignments become available based on parent visibility rules.
- **EC-009** Assignment is published with missing attachment reference. Publication blocked or flagged based on validation policy.
- **EC-010** Student becomes inactive before deadline. Submission eligibility follows enrollment and status policy.
- **EC-011** Teacher closes assignment manually before due date. Further submissions blocked.
- **EC-012** School timezone changes mid-year. Stored timestamps remain canonical and displayed deadlines follow effective timezone policy.
- **EC-013** Very large file upload exceeds size limit. Upload rejected with clear reason.
- **EC-014** Duplicate submission triggered by unstable network. Idempotency or client retry control prevents unintended duplicates.
- **EC-015** Parent attempts to view unrelated student assignments. Access blocked by relationship-based filtering.

***

## 12. Risks

### 12.1 Risks - Operational Risk
Teachers may bypass the system and continue using informal channels.  
Mitigation: Simple assignment composer, mobile-friendly workflow, clear student and parent visibility.

### 12.2 Risks - Data Integrity Risk
Assignments may be published to the wrong section or subject.  
Mitigation: Strict teacher-scope validation, academic context checks, preview before publish.

### 12.3 Risks - Privacy Risk
Parents may access unrelated student assignment data.  
Mitigation: Relationship-based filtering, tenant enforcement, audited access controls. 

### 12.4 Risks - Submission Reliability Risk
Students may fail to submit due to upload issues or connectivity problems.  
Mitigation: Text submission option, retry-safe uploads, submission confirmation, file validation.

### 12.5 Risks - Compliance Risk
Submission and feedback history may be overwritten without traceability.  
Mitigation: Versioned submissions, audit logs, immutable event history.

### 12.6 Risks - Scalability Risk
Large schools may generate high submission volume with many attachments.  
Mitigation: Object storage for files, pagination, background processing, indexed query paths.

### 12.7 Risks - Academic Workflow Risk
Assignments become disconnected from subject and timetable context.  
Mitigation: Require academic references and assigned-teacher validation against source modules. 

### 12.8 Risks - User Experience Risk
Students may be confused by deadline and late-submission behavior.  
Mitigation: Clear status labels, school-timezone display, late-policy visibility, consistent feedback.

***

## 13. Success Metrics

### 13.1 Success Metrics - Product Metrics
- Assignment creation success rate 99
- Student submission success rate 95
- File upload success rate 99
- Submission review screen load time 2 seconds
- Assignment dashboard load time 2 seconds

### 13.2 Success Metrics - Operational Metrics
- Teacher assignment publishing time under 2 minutes
- Assignment completion rate above 85 for published work
- Feedback publication turnaround within 3 days for standard homework
- Late submission identification accuracy 100
- Assignment review completion before deadline plus 5 days 90

### 13.3 Success Metrics - Parent Transparency Metrics
- Parent assignment visibility usage above 60 where enabled
- Parent overdue assignment view rate above 50 where enabled
- Student assignment access rate above 80

### 13.4 Success Metrics - Academic Monitoring Metrics
- Class-wise assignment completion reporting available for 100 of active sections
- Teacher assignment adoption rate above 70 in enabled schools
- Late submission trend visibility across grades and sections

### 13.5 Success Metrics - Business Metrics
- Reduction in manual homework coordination effort 60
- Increase in parent transparency into daily academic workflow
- Improved visibility into assignment completion and student compliance
- Increased platform stickiness through regular teacher and student usage

***

## 14. Out of Scope

The following are excluded from the Assignments and Homework module MVP:
- Full LMS course delivery
- Real-time collaborative document editing
- AI-generated assignment creation
- AI grading or plagiarism detection
- Peer review workflows
- Rubric-based advanced assessment engines
- Group project management
- Live classroom whiteboard integration
- Video assignment recording and streaming workflows
- External cloud-drive authoring integrations
- Offline-first submission sync
- Auto-grade integration into report card calculations by default
- Competency-based learning analytics
- Gamified badges and rewards systems