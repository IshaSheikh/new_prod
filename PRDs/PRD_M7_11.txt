# Module 7.11 Admissions Workflow

The Admissions Workflow module manages the full pre-enrollment lifecycle from inquiry capture to applicant evaluation, offer management, admission decision, and final enrollment handoff. It enables schools to convert prospective families into active students through a structured, auditable, and configurable workflow.

This module enables schools to:
- Capture and track admission inquiries
- Convert inquiries into applicants
- Collect application data and required documents
- Evaluate applicants through configurable review steps
- Record assessment or interview results
- Issue time-bound offers
- Capture acceptance and admission decisions
- Handoff admitted applicants into student enrollment workflows

The module must support:
- Grade-specific application requirements
- Duplicate applicant detection
- Offer expiry tracking
- Admission pipeline visibility
- Configurable statuses and approval controls
- Auditability across the full applicant lifecycle

The Admissions Workflow module must integrate with the Student Information System for final admitted student creation and enrollment activation. It must not create disconnected student records outside the platform’s core identity and enrollment architecture. 

***

## 1. Module Objective

Schools often manage admissions through spreadsheets, paper forms, email chains, phone calls, and fragmented staff coordination. Without a structured admissions workflow:
- Inquiry follow-up becomes inconsistent
- Applicant documents go missing
- Duplicate applications are created
- Evaluation progress is hard to track
- Offer decisions are delayed
- Enrollment handoff becomes error-prone
- Admission reporting becomes unreliable

Schools need this module to:
- Standardize inquiry-to-enrollment conversion
- Reduce paperwork and manual tracking
- Improve applicant processing speed
- Prevent duplicate applicant records
- Ensure grade-specific application requirements are enforced
- Track offer issuance and expiry clearly
- Make admissions performance measurable

***

## 2. Why Schools Need It

### 2.1 Reduce Manual Admission Work
Admissions is one of the most paperwork-heavy school processes. A structured workflow reduces manual coordination, repetitive data entry, and document chasing.

### 2.2 Improve Inquiry Conversion
Schools need visibility into how many inquiries convert into applicants, offers, admissions, and enrollments.

### 2.3 Standardize Application Handling
A commercial K-12 product must support consistent forms, document requirements, and evaluation flow by grade or program.

### 2.4 Prevent Duplicate Records
Prospective families often submit multiple inquiries or repeat applications. Duplicate detection improves data quality and avoids admissions confusion.

### 2.5 Improve Parent Experience
Families need a clear, organized process for submitting applications, uploading documents, and responding to offers.

### 2.6 Support Controlled Decision-Making
Admission decisions, offer release, and acceptance tracking should be structured and auditable rather than managed through informal communication.

### 2.7 Enable Enrollment Handoff
Once an applicant is admitted, their data should flow into enrollment without re-entering the same information unnecessarily.

***

## 3. Primary Users

### 3.1 School Admin
Primary admissions operator. Responsibilities:
- Capture and manage inquiries
- Convert inquiries into applicants
- Configure application requirements
- Verify submitted documents
- Track application progress
- Issue offers
- Complete admission decisions
- Trigger enrollment handoff

### 3.2 Principal
Decision and oversight user. Responsibilities:
- Review applicant pipeline
- Approve selective admissions where needed
- Monitor conversion and seat utilization
- Review assessment outcomes
- Approve final admission decisions if policy requires

### 3.3 Teacher
Evaluation user where applicable. Responsibilities:
- Participate in applicant assessment or interview review
- Submit evaluation remarks
- View assigned applicant review tasks

### 3.4 Parent
Applicant-side user where parent self-service is enabled. Responsibilities:
- Submit inquiry
- Complete application form
- Upload required documents
- Track application status
- Respond to offer
- Complete admission acceptance requirements

### 3.5 Student
Applicant-side subject where applicable. Responsibilities:
- Participate in assessment or interview processes
- Appear in applicant records as the candidate for admission

### 3.6 Super Admin
Platform oversight user. Responsibilities:
- Monitor admissions usage across tenants
- Support school configuration issues
- Audit workflow history when required

***

## 4. User Stories

### 4.1 User Stories - Inquiry Management
- **US-001** As a School Admin, I want to capture admission inquiries, so prospective students enter a structured pipeline.
- **US-002** As a School Admin, I want to record inquiry source, so I can track where leads are coming from.
- **US-003** As a School Admin, I want to convert a valid inquiry into an applicant, so formal application processing can begin.
- **US-004** As a Parent, I want to submit an inquiry, so I can express interest in admission.

### 4.2 User Stories - Application Management
- **US-005** As a School Admin, I want grade-specific application forms, so required information varies appropriately by grade.
- **US-006** As a Parent, I want to complete an application form, so the school can evaluate my child.
- **US-007** As a School Admin, I want mandatory fields enforced before application submission, so incomplete records do not move forward.
- **US-008** As a School Admin, I want application status visibility, so I can track pending and completed applications efficiently.

### 4.3 User Stories - Document Collection
- **US-009** As a Parent, I want to upload required documents, so the school can review the application.
- **US-010** As a School Admin, I want document requirements to vary by grade, so admissions policies are applied correctly.
- **US-011** As a School Admin, I want to verify or reject submitted documents, so invalid submissions can be corrected.
- **US-012** As a Parent, I want to know which documents are missing, so I can complete the application faster.

### 4.4 User Stories - Duplicate Detection
- **US-013** As a School Admin, I want duplicate applicants flagged by phone, email, and date of birth similarity, so duplicate records can be reviewed before processing.
- **US-014** As a School Admin, I want the system to warn before creating a duplicate applicant, so data quality is preserved.
- **US-015** As a Principal, I want duplicate cases resolved visibly, so admissions reporting remains accurate.

### 4.5 User Stories - Evaluation and Assessment
- **US-016** As a School Admin, I want applicants moved into evaluation after required documents are complete, so readiness is controlled.
- **US-017** As a Teacher, I want to record assessment or interview outcomes, so applicant evaluation is documented.
- **US-018** As a Principal, I want to review evaluation summaries, so admission decisions are evidence-based.
- **US-019** As a School Admin, I want failed or incomplete evaluations clearly tracked, so follow-up actions are organized.

### 4.6 User Stories - Offers and Admission Decisions
- **US-020** As a School Admin, I want to issue admission offers, so selected applicants can confirm acceptance.
- **US-021** As a Parent, I want to view offer details and expiry date, so I know when to respond.
- **US-022** As a School Admin, I want accepted and expired offers clearly tracked, so seat planning is accurate.
- **US-023** As a Principal, I want final admission approval visibility, so admissions remain governed properly.

### 4.7 User Stories - Enrollment Handoff
- **US-024** As a School Admin, I want accepted applicants converted into admitted students, so final enrollment can begin.
- **US-025** As a School Admin, I want admitted applicant data handed off into student enrollment without duplicate data entry, so the transition is efficient.
- **US-026** As a Parent, I want accepted admission to move smoothly into enrollment, so I do not repeat the same information unnecessarily.

### 4.8 User Stories - Reporting and Oversight
- **US-027** As a School Admin, I want admissions funnel reports, so I can measure inquiry-to-enrollment performance.
- **US-028** As a Principal, I want seat demand visibility by grade, so planning decisions improve.
- **US-029** As a Super Admin, I want audit visibility into admissions workflow actions, so support and compliance are manageable.

***

## 5. Business Rules

### 5.1 Business Rules - Tenant Isolation
- **BR-001** Every inquiry, applicant, form, document, evaluation result, offer, and contract must belong to exactly one tenant.
- **BR-002** Admissions records cannot be accessed across tenants.
- **BR-003** All admissions APIs and queries must automatically enforce tenant scope.

### 5.2 Business Rules - Inquiry Rules
- **BR-004** An inquiry may exist before a formal applicant record is created.
- **BR-005** Inquiry source must be stored when available.
- **BR-006** An inquiry may be converted into one applicant only.
- **BR-007** Closed or invalid inquiries must remain searchable for reporting.

### 5.3 Business Rules - Applicant Rules
- **BR-008** Duplicate applicants should be flagged by phone, email, and date-of-birth similarity.
- **BR-009** Duplicate detection must run before creating or converting an applicant.
- **BR-010** A flagged duplicate must require review before progressing if school policy enables strict duplicate control.
- **BR-011** An applicant must reference target academic year and target grade.
- **BR-012** Applicant records must remain auditable across status changes.
- **BR-013** Applicant deletion after workflow progression is prohibited. Soft delete only before significant processing, if allowed.

### 5.4 Business Rules - Application Form Rules
- **BR-014** Application form requirements vary by grade.
- **BR-015** Mandatory application fields may differ by grade and school configuration.
- **BR-016** Application cannot move forward until all mandatory fields are completed.
- **BR-017** Application form version used at submission time must be retained for auditability.

### 5.5 Business Rules - Document Rules
- **BR-018** Required documents may vary by grade.
- **BR-019** Application cannot move to evaluation until all mandatory documents are submitted or explicitly waived by authorized staff.
- **BR-020** Submitted documents must be versioned.
- **BR-021** Documents cannot be modified after upload. Replacement creates a new version.
- **BR-022** Document verification outcome must be recorded.
- **BR-023** Rejected documents must include a reason.

### 5.6 Business Rules - Evaluation Rules
- **BR-024** Evaluation may include interview, assessment, observation, or manual review depending on school policy.
- **BR-025** Evaluation cannot begin until application and mandatory document requirements are satisfied unless an override is granted.
- **BR-026** Assessment results must be stored per applicant.
- **BR-027** Evaluation decisions must be auditable.
- **BR-028** Applicants may be marked not selected, waitlisted, or recommended for offer depending on school policy.

### 5.7 Business Rules - Offer Rules
- **BR-029** Offers may only be issued to applicants in eligible evaluation states.
- **BR-030** Offer expiry dates must be tracked.
- **BR-031** Expired offers cannot be accepted unless reissued.
- **BR-032** Offer acceptance must be timestamped.
- **BR-033** Offer rejection or expiry must be retained in history.
- **BR-034** One active offer per applicant is allowed by default unless school policy supports multiple offer rounds.

### 5.8 Business Rules - Admission and Enrollment Rules
- **BR-035** Accepted offers may move to admitted state only after required acceptance conditions are completed.
- **BR-036** Admitted applicants must be converted into student and enrollment records through official SIS and enrollment workflows, not through separate ad hoc record creation. 
- **BR-037** Admissions workflow must not bypass academic year, grade, and section enrollment rules enforced by downstream modules. 
- **BR-038** Historical admissions data must remain accessible after enrollment conversion.

### 5.9 Business Rules - Audit and Reporting Rules
- **BR-039** Every status transition must be audited.
- **BR-040** Every offer issuance, acceptance, expiry, and reissue must be audited.
- **BR-041** Admissions funnel reporting must use historical pipeline state changes.
- **BR-042** Search and reporting must support inquiry, applicant, grade, status, and date-based filters.

***

## 6. State Machine

### 6.1 State Machine - Applicant Admission Lifecycle
```text
Inquiry
  v
Applicant
  v
Documents Pending
  v
Evaluation
  v
Offered
  |------ Expired
  |         v
  |------ Closed
  |------ Rejected
  v
Accepted
  v
Admitted
  v
Enrolled
```

### 6.2 State Machine - Application Form Lifecycle
```text
Draft
  v
Submitted
  |------ Returned For Correction
  |         v
  |------ Resubmitted
  v
Verified
```

### 6.3 State Machine - Document Lifecycle
```text
Uploaded
  v
Under Review
  |------ Verified
  |------ Rejected
  |         v
  |------ Reuploaded
  v
Archived
```

### 6.4 State Machine - Offer Lifecycle
```text
Draft
  v
Issued
  |------ Accepted
  |------ Rejected
  |------ Expired
  v
Closed
```

***

## 7. Required Data Entities

### 7.1 Required Data Entities - admission_inquiries
Field | Type
--- | ---
id | UUID
tenantid | UUID
schoolid | UUID
academicyearid | UUID
studentname | VARCHAR
parentname | VARCHAR
email | VARCHAR
phone | VARCHAR
dateofbirth | DATE NULL
targetgradeid | UUID
inquirysource | VARCHAR
status | ENUM
createdby | UUID NULL
createdat | TIMESTAMP
updatedat | TIMESTAMP

### 7.2 Required Data Entities - applicants
Field | Type
--- | ---
id | UUID
tenantid | UUID
schoolid | UUID
inquiryid | UUID NULL
academicyearid | UUID
targetgradeid | UUID
firstname | VARCHAR
lastname | VARCHAR
dateofbirth | DATE
gender | VARCHAR NULL
parentname | VARCHAR
email | VARCHAR
phone | VARCHAR
status | ENUM
duplicateriskscore | DECIMAL NULL
duplicatereviewstatus | ENUM NULL
createdat | TIMESTAMP
updatedat | TIMESTAMP

### 7.3 Required Data Entities - application_forms
Field | Type
--- | ---
id | UUID
tenantid | UUID
schoolid | UUID
applicantid | UUID
formversionid | UUID
gradeid | UUID
formdata | JSONB
status | ENUM
submittedat | TIMESTAMP NULL
verifiedat | TIMESTAMP NULL
createdat | TIMESTAMP
updatedat | TIMESTAMP

### 7.4 Required Data Entities - documents_submitted
Field | Type
--- | ---
id | UUID
tenantid | UUID
schoolid | UUID
applicantid | UUID
documenttype | VARCHAR
storagepath | VARCHAR
versionno | INTEGER
status | ENUM
reviewreason | TEXT NULL
uploadedat | TIMESTAMP
reviewedat | TIMESTAMP NULL
reviewedby | UUID NULL

### 7.5 Required Data Entities - assessment_results
Field | Type
--- | ---
id | UUID
tenantid | UUID
schoolid | UUID
applicantid | UUID
assessmenttype | ENUM
score | DECIMAL NULL
resultstatus | ENUM
remarks | TEXT NULL
evaluatedby | UUID
evaluatedat | TIMESTAMP

### 7.6 Required Data Entities - offers
Field | Type
--- | ---
id | UUID
tenantid | UUID
schoolid | UUID
applicantid | UUID
offerno | VARCHAR
issuedat | TIMESTAMP
expiresat | TIMESTAMP
status | ENUM
acceptedat | TIMESTAMP NULL
rejectedat | TIMESTAMP NULL
createdby | UUID
updatedat | TIMESTAMP

### 7.7 Required Data Entities - enrollment_contracts
Field | Type
--- | ---
id | UUID
tenantid | UUID
schoolid | UUID
applicantid | UUID
offerid | UUID
contractstatus | ENUM
signedat | TIMESTAMP NULL
acceptedterms | BOOLEAN
createdat | TIMESTAMP
updatedat | TIMESTAMP

### 7.8 Additional Supporting Entities
- `application_form_versions`
- `application_requirements`
- `document_requirements`
- `admissions_evaluations`
- `offer_events`
- `admissions_audit_logs`
- `duplicate_match_candidates`
- `admissions_notes`

***

## 8. Permissions Matrix

| Capability | Super Admin | School Admin | Principal | Teacher | Accountant | Parent | Student |
|---|---|---|---|---|---|---|---|
| View admissions dashboard | Yes | Yes | Yes | Limited assigned reviews | No | No | No |
| Create inquiry | Limited Support | Yes | Yes if allowed | No | No | Yes if enabled | No |
| Convert inquiry to applicant | No | Yes | Yes if allowed | No | No | No | No |
| Edit applicant | Limited Support | Yes | Yes if allowed | No | No | Limited own application if enabled | No |
| Configure application forms | Limited Support | Yes | Review only | No | No | No | No |
| Upload applicant documents | No | Yes | No | No | No | Yes if enabled | No |
| Verify documents | No | Yes | Yes if allowed | No | No | No | No |
| Record assessment results | No | Yes | Yes | Yes assigned only | No | No | No |
| Issue offer | No | Yes | Yes if allowed | No | No | No | No |
| Accept or reject offer | No | No | No | No | No | Yes if enabled | No |
| Convert admitted to enrolled | No | Yes | Yes if allowed | No | No | No | No |
| View admissions history | Limited Audit | Yes | Yes | Limited assigned applicants | No | Limited own application if enabled | No |
| View audit logs | Limited Audit | Yes | Limited | No | No | No | No |

***

## 9. APIs Needed

### 9.1 APIs Needed - Inquiry APIs
- `POST /admissions/inquiries`
- `GET /admissions/inquiries`
- `GET /admissions/inquiries/{id}`
- `PATCH /admissions/inquiries/{id}`
- `POST /admissions/inquiries/{id}/convert-applicant`

### 9.2 APIs Needed - Applicant APIs
- `POST /admissions/applicants`
- `GET /admissions/applicants`
- `GET /admissions/applicants/{id}`
- `PATCH /admissions/applicants/{id}`
- `POST /admissions/applicants/{id}/run-duplicate-check`
- `POST /admissions/applicants/{id}/move-status`

### 9.3 APIs Needed - Application Form APIs
- `GET /admissions/application-forms/config`
- `POST /admissions/application-forms`
- `GET /admissions/application-forms/{id}`
- `PATCH /admissions/application-forms/{id}`
- `POST /admissions/application-forms/{id}/submit`
- `POST /admissions/application-forms/{id}/verify`

### 9.4 APIs Needed - Document APIs
- `POST /admissions/documents/upload`
- `GET /admissions/applicants/{id}/documents`
- `POST /admissions/documents/{id}/verify`
- `POST /admissions/documents/{id}/reject`
- `POST /admissions/documents/{id}/replace`

### 9.5 APIs Needed - Evaluation APIs
- `POST /admissions/evaluations`
- `GET /admissions/applicants/{id}/evaluations`
- `PATCH /admissions/evaluations/{id}`
- `POST /admissions/evaluations/{id}/complete`

### 9.6 APIs Needed - Offer APIs
- `POST /admissions/offers`
- `GET /admissions/offers`
- `GET /admissions/offers/{id}`
- `POST /admissions/offers/{id}/issue`
- `POST /admissions/offers/{id}/accept`
- `POST /admissions/offers/{id}/reject`
- `POST /admissions/offers/{id}/expire`
- `POST /admissions/offers/{id}/reissue`

### 9.7 APIs Needed - Enrollment Handoff APIs
- `POST /admissions/applicants/{id}/admit`
- `POST /admissions/applicants/{id}/enroll`
- `GET /admissions/applicants/{id}/handoff-preview`

### 9.8 APIs Needed - Reporting APIs
- `GET /admissions/reports/funnel`
- `GET /admissions/reports/source-conversion`
- `GET /admissions/reports/grade-demand`
- `GET /admissions/reports/offer-status`
- `GET /admissions/search`

***

## 10. Screens Needed

### 10.1 Screens Needed - Admissions Dashboard
Displays:
- Inquiry counts
- Applicant pipeline
- Offers pending response
- Grade-wise demand
- Conversion funnel
- Expiring offers

### 10.2 Screens Needed - Inquiry Management Screen
Features:
- Create inquiry
- Edit inquiry
- Inquiry source tracking
- Convert to applicant
- Duplicate warning

### 10.3 Screens Needed - Applicant Detail Screen
Sections:
- Profile
- Application form
- Documents
- Evaluations
- Offers
- Notes
- Activity history

### 10.4 Screens Needed - Application Form Builder Screen
Features:
- Grade-wise form configuration
- Mandatory field setup
- Version management
- Preview form

### 10.5 Screens Needed - Document Review Screen
Features:
- Missing document checklist
- Upload review
- Verify or reject
- Reupload handling
- Version history

### 10.6 Screens Needed - Evaluation Management Screen
Features:
- Assign review
- Record score
- Add remarks
- Complete evaluation
- View decision summary

### 10.7 Screens Needed - Offer Management Screen
Features:
- Generate offer
- Set expiry date
- View offer status
- Reissue expired offer
- Track acceptance

### 10.8 Screens Needed - Enrollment Handoff Screen
Features:
- Admitted applicant preview
- SIS field mapping
- Enrollment readiness
- Final conversion to enrolled student

### 10.9 Screens Needed - Admissions Reports Screen
Displays:
- Funnel conversion
- Source effectiveness
- Grade demand
- Offer acceptance rates
- Processing timelines

### 10.10 Screens Needed - Parent Application Portal Screen
Features:
- Submit inquiry
- Complete form
- Upload documents
- Track status
- View offer
- Accept or reject offer

***

## 11. Edge Cases

- **EC-001** Duplicate applicant created through separate inquiry and direct application. System flags probable duplicate before progression.
- **EC-002** Applicant submits incomplete form. Submission blocked until mandatory fields are completed.
- **EC-003** Application form requirements differ for Grade 1 and Grade 11. System applies grade-specific form and document rules.
- **EC-004** Parent uploads incorrect document. Reviewer rejects with reason and requests reupload.
- **EC-005** Duplicate applicants share same phone and date of birth but different email. System flags for manual review.
- **EC-006** Offer expires before parent response. Acceptance blocked unless offer is reissued.
- **EC-007** Applicant is accepted but required admission conditions remain incomplete. Move to admitted blocked.
- **EC-008** Applicant is admitted but academic year or grade target is no longer available. Enrollment handoff blocked pending resolution.
- **EC-009** Applicant withdraws after offer issuance. Historical offer and decision records remain searchable.
- **EC-010** Multiple assessments recorded for one applicant. Final decision must follow configured evaluation rule.
- **EC-011** Parent submits application twice for same child. Duplicate warning shown.
- **EC-012** Applicant data edited after offer issuance. Offer snapshot remains preserved for audit.
- **EC-013** Document replaced after verification. Previous version retained in history.
- **EC-014** Parent linked or updated after admission. Final enrollment uses latest approved guardian data where policy allows.
- **EC-015** School reaches seat limit for a grade after offer issuance. Enrollment may require override or waitlist handling.

***

## 12. Risks

### 12.1 Risks - Data Integrity Risk
Duplicate or fragmented applicant records reduce reporting accuracy.  
Mitigation: Duplicate detection engine, conversion rules, review workflow.

### 12.2 Risks - Operational Risk
Admissions staff skip required steps and move applicants forward prematurely.  
Mitigation: State-based validation, mandatory field checks, gated progression.

### 12.3 Risks - Configuration Risk
Incorrect grade-specific form or document setup causes blocked or inconsistent applications.  
Mitigation: Form versioning, requirement previews, admin validation.

### 12.4 Risks - Compliance Risk
Offer, decision, or applicant history cannot be reconstructed later.  
Mitigation: Immutable audit logs, status transition logging, snapshot retention.

### 12.5 Risks - Parent Experience Risk
Families face unclear requirements and abandon applications.  
Mitigation: Missing-item tracking, guided status visibility, clear rejection reasons.

### 12.6 Risks - Scalability Risk
Large admission seasons generate high inquiry and document volumes.  
Mitigation: Pagination, indexed search, asynchronous document processing, background duplicate checks.

### 12.7 Risks - Enrollment Handoff Risk
Admitted applicants fail during conversion to enrolled students due to downstream rule mismatches.  
Mitigation: Handoff preview, validation against SIS and enrollment rules, required-field mapping. 

### 12.8 Risks - Security Risk
Sensitive applicant documents are accessed by unauthorized users.  
Mitigation: Role-based permissions, field-level access control, audit logging.

***

## 13. Success Metrics

### 13.1 Success Metrics - Product Metrics
- Inquiry creation success rate 99
- Applicant conversion success rate 98
- Application form submission success rate 95
- Document upload success rate 99
- Admissions dashboard load time 2 seconds

### 13.2 Success Metrics - Operational Metrics
- Average inquiry to applicant conversion time 1 day
- Average application review turnaround 3 days
- Offer issuance turnaround 2 days after evaluation completion
- Duplicate applicant review turnaround 24 hours
- Enrollment handoff completion rate 98

### 13.3 Success Metrics - Data Quality Metrics
- Duplicate applicant rate below 1
- Missing mandatory application fields below 2
- Missing mandatory documents before evaluation below 3
- Offer expiry tracking accuracy 100

### 13.4 Success Metrics - Parent Experience Metrics
- Parent self-service application completion rate 80
- Document reupload resolution rate 90
- Offer response within expiry window 75

### 13.5 Success Metrics - Business Metrics
- Reduction in admission administration workload 60
- Improvement in inquiry-to-enrollment conversion visibility 100
- Reduction in paperwork-related delays 70
- Improved admissions reporting accuracy across grades and academic years

***

## 14. Out of Scope

The following are excluded from the Admissions Workflow module MVP:
- Marketing CRM campaigns
- Automated lead scoring using AI
- Scholarship decision engine
- Hostel admission workflow
- Transport route selection during admission
- Online payment gateway for admission fees if not separately scoped
- Government admission portal integration
- Advanced document OCR extraction
- Biometric applicant verification
- Interview video recording
- Multi-school common admissions across tenants
- Public seat booking engine
- Full student enrollment module logic beyond admissions handoff
- Alumni or sibling priority policy engine in advanced form
- AI-based admission recommendation system