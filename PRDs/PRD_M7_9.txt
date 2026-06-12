**Module 7.9 Parent Portal and Student Portal**, written in the same structured style as your reference modules, with numbered user stories, business rules, edge cases, risks, and success metrics. The design aligns with your platform’s established architecture: strict tenant isolation, role-based access, parent-student relationship controls, and read-only downstream consumption of attendance, grades, timetable, and fee data from source modules rather than duplicating records.

# 7.9 Parent Portal and Student Portal

## 1. Module objective

The Parent Portal and Student Portal module provides a secure self-service experience for families and students to view school information, monitor academic and operational status, and complete a limited set of approved actions without relying on office staff.

The module must expose a unified, role-aware portal experience for:
- Dashboard
- Attendance summary
- Grade summary and report cards
- Timetable
- Assignments or homework summaries
- Fee balance and payments
- Notices and announcements
- Documents
- Support and contact workflows

The objective is to:
- Reduce front-office calls and repetitive support queries.
- Improve parent transparency across academics, attendance, timetable, and finance.
- Encourage faster fee payments through clear balance visibility.
- Give students controlled access to their own operational and academic data.
- Make school-family communication measurable through portal usage and engagement metrics.

The portal is a presentation and action layer, not a source-of-truth module. It must read published or permitted records from SIS, timetable, attendance, gradebook, fees, and communication modules rather than maintaining separate copies of core school data.***

## 2. Why schools need it

Schools typically spend significant time answering repetitive questions from parents and students, such as attendance status, timetable changes, fee dues, result availability, receipt downloads, and notice access. That creates unnecessary administrative workload and delays communication.

Schools need this module to:
- Give parents one place to monitor all linked children.
- Give students one place to view their own approved data.
- Reduce dependence on phone calls, printed notices, and front-desk inquiries.
- Improve trust by making attendance, academic results, fees, and documents visible.
- Increase fee collection efficiency by surfacing dues, due dates, and receipts.
- Support measurable parent engagement through usage and response analytics.
- Ensure portal access follows strict relationship-based authorization and school-level data privacy controls. 
The portal also strengthens the value of downstream modules because attendance, grades, timetable, and finance become visible to the end users they are meant to serve.***

## 3. Primary users

### 3.1 Parent
Primary guardian user. Responsibilities:
- View all linked children under one login.
- Switch between children and compare status quickly.
- View attendance, grades, timetable, fees, notices, and documents.
- Download receipts, report cards, and approved documents.
- Submit limited forms or support requests where enabled.
- Make fee payments where online payment is configured.

### 3.2 Student
Learner self-service user. Responsibilities:
- View own attendance, grades, timetable, assignments, notices, and documents.
- Download own approved files and report cards.
- View own fee information only where school policy allows.
- Submit limited self-service forms if enabled.

### 3.3 School Admin
Portal administrator at school level. Responsibilities:
- Configure portal visibility and section access.
- Enable or disable student portal features.
- Manage portal announcements and content policies.
- Monitor usage analytics and support requests.
- Control publication rules through source modules.

### 3.4 Principal
Academic and operational oversight user. Responsibilities:
- Monitor parent and student portal adoption.
- Review transparency outcomes and communication effectiveness.
- Observe academic and attendance visibility at a high level.

### 3.5 Super Admin
Platform oversight user. Responsibilities:
- Audit tenant-safe portal access patterns.
- Investigate support incidents.
- Monitor portal reliability and security issues.

### 3.6 Teacher
Limited indirect portal stakeholder. Responsibilities:
- Publish source data through upstream modules such as attendance, grades, assignments, and notices.
- View whether parent-facing publication has occurred where relevant.

### 3.7 Accountant
Finance-related indirect portal stakeholder. Responsibilities:
- Enable accurate fee balances, receipts, and payment visibility through the finance module.
- Monitor whether parents can access published financial information.

***

## 4. User stories

### 4.1 Parent dashboard and child switching
- **US-001** As a Parent, I want to log in once and see all my linked children, so I do not need separate accounts for each child.
- **US-002** As a Parent, I want to switch quickly between children, so I can monitor each child’s attendance, results, timetable, and fees.
- **US-003** As a Parent, I want a dashboard summary of attendance, dues, recent notices, and upcoming events, so I can understand the most important updates immediately.
- **US-004** As a Parent, I want child-level alerts such as overdue fees, absence alerts, or newly published report cards, so I can act quickly.

### 4.2 Student self-service
- **US-005** As a Student, I want to log in and view only my own data, so my privacy is preserved.
- **US-006** As a Student, I want a personal dashboard showing today’s timetable, attendance summary, latest notices, and pending items, so I can stay organized.
- **US-007** As a Student, I want to view my own academic results after publication, so I can track my progress.
- **US-008** As a Student, I want to download my report card and approved documents, so I can access them without visiting the office.

### 4.3 Attendance visibility
- **US-009** As a Parent, I want to view my child’s attendance summary and recent absences, so I can monitor regularity.
- **US-010** As a Student, I want to view my attendance history, so I can understand my own attendance pattern.
- **US-011** As a Parent, I want absence and lateness information to appear only after official submission, so I do not see unverified classroom drafts.

### 4.4 Grades and report cards
- **US-012** As a Parent, I want to view published grades and report cards, so I can monitor academic performance.
- **US-013** As a Student, I want to view only published results, so I see official academic records rather than incomplete data.
- **US-014** As a Parent, I want corrected report cards to appear as new versions where applicable, so academic history remains transparent.
- **US-015** As a Parent, I want grade summaries presented clearly by term or assessment period, so I can understand progress without calling the school.

### 4.5 Timetable and assignments
- **US-016** As a Parent, I want to view my child’s published timetable, so I know the school schedule.
- **US-017** As a Student, I want to see today’s and upcoming classes, so I can prepare properly.
- **US-018** As a Parent, I want to see assignments or homework summaries where enabled, so I can support my child’s learning.
- **US-019** As a Student, I want to view assignment deadlines and teacher instructions where published, so I do not miss work.

### 4.6 Fees and payments
- **US-020** As a Parent, I want to see total outstanding balance, due dates, and invoice history, so I know what must be paid.
- **US-021** As a Parent, I want to make online payments from the portal, so I can settle dues quickly.
- **US-022** As a Parent, I want to download receipts and see payment history, so I have proof of payment.
- **US-023** As a Student, I want fee visibility only when school policy allows, so sensitive family financial information is controlled appropriately.

### 4.7 Notices, documents, and support
- **US-024** As a Parent, I want to view school notices in one place, so I do not miss announcements.
- **US-025** As a Student, I want to see student-relevant notices and circulars, so I stay informed.
- **US-026** As a Parent, I want to download approved student documents, so I do not need to request them manually each time.
- **US-027** As a Parent, I want to submit support or contact requests through the portal, so issues are routed properly.
- **US-028** As a School Admin, I want portal requests and engagement to be measurable, so I can track service effectiveness.

### 4.8 Administration and control
- **US-029** As a School Admin, I want to configure which portal sections are enabled for parents and students, so the portal fits school policy.
- **US-030** As a School Admin, I want publication in source modules to control what becomes visible in the portal, so end users only see approved information.
- **US-031** As a Principal, I want to review portal usage metrics, so I can evaluate transparency and engagement outcomes.
- **US-032** As a Super Admin, I want audit logs for portal access and critical downloads, so platform security and support are manageable.

***

## 5. Business rules

### 5.1 Tenant isolation
- **BR-001** Every portal session must belong to exactly one tenant and one school context.
- **BR-002** Parent and student portal data must never be accessible across tenants.
- **BR-003** Portal queries must resolve tenant scope from authenticated identity and linked relationships, not from client-supplied school identifiers.

### 5.2 Identity and relationships
- **BR-004** Parent portal access is allowed only when a guardian is linked to at least one student through the platform relationship model. - **BR-005** A Parent may be linked to multiple students within the same tenant.
- **BR-006** A Parent sees all linked children under one login unless school policy restricts certain relationships.
- **BR-007** A Student may see only their own data.
- **BR-008** Relationship changes must immediately affect portal visibility after access rules are refreshed.
- **BR-009** Historical records remain visible according to school policy even if a student changes section or class within the same tenant.
- **BR-010** Parent access to a withdrawn, transferred, or graduated student must follow school retention policy and configured visibility rules.

### 5.3 Data visibility
- **BR-011** The portal is read-only by default.
- **BR-012** Allowed exceptions to read-only access include fee payment, document download, support/contact submission, and approved form submission.
- **BR-013** The portal must only display data that is published, approved, submitted, or otherwise marked visible by the source module.
- **BR-014** Draft attendance, draft grades, draft report cards, and unpublished timetables must not appear in the portal.
- **BR-015** Attendance data shown in the portal must come from the official attendance module and only after submission or approval rules are satisfied. 
- **BR-016** Grade and report card data shown in the portal must come from published academic records only.
- **BR-017** Timetable data shown in the portal must come from the active published timetable version only. 
- **BR-018** Fee balances, invoices, receipts, and payment status shown in the portal must come from the finance module’s official records.
- **BR-019** The portal must not create or edit source academic, attendance, timetable, or finance records directly.

### 5.4 Notifications and notices
- **BR-020** Portal notices may be audience-scoped by role, grade, section, or individual student.
- **BR-021** A notice must have a visibility period or publication status before appearing in the portal.
- **BR-022** Read tracking may be captured for notices if enabled by school policy.
- **BR-023** Sensitive notices must not be visible to unauthorized guardians or unrelated students.

### 5.5 Documents and downloads
- **BR-024** Downloadable documents must be permission-checked at request time.
- **BR-025** Document access must be audited for sensitive files.
- **BR-026** If a document is versioned upstream, the portal must display the latest visible version and preserve prior versions where policy requires. 
- **BR-027** Expired, revoked, or archived documents must not be downloadable unless explicitly allowed.

### 5.6 Payments and finance actions
- **BR-028** Only authorized parent users may initiate fee payments for linked students.
- **BR-029** Student users may view or pay fees only if enabled by school policy.
- **BR-030** Payment attempts must not alter invoice state until confirmed by the finance module.
- **BR-031** Failed online payment attempts must remain visible in transaction history where applicable.
- **BR-032** Receipts available in the portal must be immutable finance outputs, not editable portal artifacts.

### 5.7 Support and forms
- **BR-033** Support requests submitted through the portal must generate trackable records.
- **BR-034** Support categories, routing rules, and SLAs may be configurable at school level.
- **BR-035** Portal-submitted forms must be limited to explicitly approved workflows.
- **BR-036** Form submissions must be versioned or timestamped and auditable.

### 5.8 Security and audit
- **BR-037** Every login, child-switch event, sensitive page access, payment initiation, and protected document download must be auditable.
- **BR-038** Session expiration rules must be stricter for shared-device use cases common in school contexts.
- **BR-039** The portal must support device-safe logout and token revocation.
- **BR-040** Unauthorized access attempts must be logged and rate-limited.
- **BR-041** Parents and students must never see internal staff-only comments, confidential health notes, or restricted administrative data unless explicitly allowed by policy. 
***

## 6. State machine

### 6.1 Parent portal account lifecycle
```text
Invited
  v
Pending Activation
  v
Active
  |------ Suspended
  |         v
  |------ Reactivated
  v
Disabled
```

### 6.2 Student portal account lifecycle
```text
Provisioned
  v
Pending Activation
  v
Active
  |------ Suspended
  |         v
  |------ Reactivated
  v
Disabled
```

### 6.3 Support request lifecycle
```text
Submitted
  v
Acknowledged
  v
In Progress
  |------ Waiting for User
  |         v
  |------ Reopened
  v
Resolved
  v
Closed
```

### 6.4 Notice publication visibility lifecycle
```text
Draft
  v
Scheduled
  v
Published
  v
Expired
  |------ Archived
```

### 6.5 Portal document visibility lifecycle
```text
Uploaded
  v
Verified
  v
Visible
  |------ Replaced
  |         v
  |------ Archived
  v
Revoked
```

***

## 7. Required data entities

### 7.1 Core entities
- `portal_users`
- `portal_profiles`
- `portal_sessions`
- `parent_student_links`
- `portal_dashboards`
- `portal_widgets`
- `portal_notices`
- `portal_notice_audiences`
- `portal_notice_reads`
- `portal_documents`
- `portal_download_logs`
- `portal_support_requests`
- `portal_support_messages`
- `portal_feature_flags`
- `portal_access_policies`
- `portal_audit_logs`

### 7.2 Suggested entity definitions

| Entity | Purpose | Key fields |
|---|---|---|
| `portal_users` | Portal identity record for parent or student account | id, tenant_id, school_id, user_id, role_type, status, last_login_at |
| `portal_profiles` | Portal preferences and display metadata | id, portal_user_id, preferred_language, notification_preferences_json, profile_status |
| `portal_sessions` | Session tracking and device visibility | id, portal_user_id, device_info, ip_address, started_at, expires_at, revoked_at |
| `parent_student_links` | Effective guardian-child visibility map consumed by portal | id, tenant_id, parent_user_id, student_id, relationship_type, is_primary, active |
| `portal_dashboards` | Configured dashboard views per role | id, tenant_id, role_type, layout_json, active |
| `portal_widgets` | Role-based dashboard cards and summaries | id, dashboard_id, widget_code, widget_config_json, active |
| `portal_notices` | Portal-visible notices and circulars | id, tenant_id, school_id, title, body, published_at, expires_at, status |
| `portal_notice_audiences` | Audience scoping for notices | id, notice_id, audience_type, audience_ref_id |
| `portal_notice_reads` | Notice read tracking | id, notice_id, portal_user_id, read_at |
| `portal_documents` | Portal-eligible files and metadata | id, tenant_id, student_id nullable, document_type, source_module, source_ref_id, version_no, visibility_status |
| `portal_download_logs` | Audit trail for downloads | id, portal_user_id, document_id, downloaded_at, ip_address |
| `portal_support_requests` | Support or contact submissions | id, tenant_id, school_id, portal_user_id, student_id nullable, category, subject, description, status, priority |
| `portal_support_messages` | Threaded communication on requests | id, support_request_id, sender_type, message_body, created_at |
| `portal_feature_flags` | School-level feature enablement | id, tenant_id, school_id, feature_code, enabled |
| `portal_access_policies` | Fine-grained visibility rules | id, tenant_id, school_id, policy_code, role_type, config_json, active |
| `portal_audit_logs` | Portal activity trail | id, tenant_id, portal_user_id, event_type, event_ref, created_at, metadata_json |

### 7.3 Cross-module dependencies
- Student identity and guardian relationship data must come from SIS and RBAC-linked relationship models. 
- Attendance summaries must come from attendance records already submitted or approved.
- Grade summaries and report cards must come from published gradebook outputs. 
- Timetable must come from the active published timetable version. 
- Fee balance, invoices, payment transactions, and receipts must come from the finance module.

***

## 8. Permissions matrix

Legend:
- `V` = View
- `A` = Action allowed
- `C` = Configure
- `M` = Monitor
- `-` = No access

| Capability | Super Admin | School Admin | Principal | Teacher | Accountant | Parent | Student |
|---|---:|---:|---:|---:|---:|---:|---:|
| View portal config | V | V/C | V | - | - | - | - |
| Configure portal sections | V | V/C | V | - | - | - | - |
| View parent portal dashboard | V | V | V | - | Limited finance context | V | - |
| View student portal dashboard | V | V | V | - | - | Child only | V |
| Switch between linked children | V | V | - | - | - | A | - |
| View attendance | V | V | V | Limited assigned students only | - | V child only | V self only |
| View grades/report cards | V | V | V | Limited assigned students only | - | V child only | V self only |
| View timetable | V | V | V | V own classes | - | V child only | V self only |
| View fees and receipts | V | V | V summary | - | V | V child only | Policy-based |
| Initiate fee payment | - | - | - | - | Optional assist only | A | Policy-based |
| Download documents | V | V | V | - | - | A child only | A self only |
| Submit support request | V | V | M | - | - | A | A |
| View support queue | V | V | V | - | - | Own requests only | Own requests only |
| View portal analytics | V | V | V | - | - | - | - |
| View portal audit logs | V | V | Limited | - | Limited finance events | - | - |

Notes:
- Parent access must always be restricted to linked students only. 
- Student access must always be restricted to self only.
- Teacher and Accountant portal access is operationally indirect and should remain limited.

***

## 9. APIs needed

### 9.1 Authentication and session APIs
- **API-001** `POST /api/v1/portal/auth/login` — Parent or student portal login.
- **API-002** `POST /api/v1/portal/auth/logout` — Revoke current session.
- **API-003** `POST /api/v1/portal/auth/refresh` — Refresh session token.
- **API-004** `POST /api/v1/portal/auth/activate` — Activate invited portal account.
- **API-005** `POST /api/v1/portal/auth/forgot-password` — Start password reset.
- **API-006** `POST /api/v1/portal/auth/reset-password` — Complete password reset.
- **API-007** `GET /api/v1/portal/sessions` — List active sessions for current portal user.
- **API-008** `POST /api/v1/portal/sessions/{id}/revoke` — Revoke a specific session.

### 9.2 Dashboard APIs
- **API-009** `GET /api/v1/portal/dashboard` — Get role-based dashboard summary.
- **API-010** `GET /api/v1/portal/children` — List linked children for current parent.
- **API-011** `POST /api/v1/portal/context/select-child` — Switch active child context for parent view.
- **API-012** `GET /api/v1/portal/widgets` — Get dashboard widgets for current role.

### 9.3 Attendance APIs
- **API-013** `GET /api/v1/portal/attendance/summary` — Attendance summary for selected child or self.
- **API-014** `GET /api/v1/portal/attendance/history` — Attendance history list.
- **API-015** `GET /api/v1/portal/attendance/calendar` — Calendar view of attendance statuses.

### 9.4 Grades APIs
- **API-016** `GET /api/v1/portal/grades/summary` — Published grade summary.
- **API-017** `GET /api/v1/portal/report-cards` — List published report cards.
- **API-018** `GET /api/v1/portal/report-cards/{id}` — Report card detail.
- **API-019** `GET /api/v1/portal/report-cards/{id}/download` — Download report card file or PDF.

### 9.5 Timetable and assignments APIs
- **API-020** `GET /api/v1/portal/timetable` — Get active published timetable.
- **API-021** `GET /api/v1/portal/timetable/today` — Get today’s schedule.
- **API-022** `GET /api/v1/portal/assignments` — List published assignments or homework.
- **API-023** `GET /api/v1/portal/assignments/{id}` — Assignment detail.

### 9.6 Finance APIs
- **API-024** `GET /api/v1/portal/fees/summary` — Outstanding balance and upcoming dues.
- **API-025** `GET /api/v1/portal/invoices` — Invoice list.
- **API-026** `GET /api/v1/portal/invoices/{id}` — Invoice detail.
- **API-027** `POST /api/v1/portal/payments/session` — Initiate online payment.
- **API-028** `GET /api/v1/portal/payments/history` — Payment and receipt history.
- **API-029** `GET /api/v1/portal/receipts/{id}/download` — Download receipt.

### 9.7 Notices and documents APIs
- **API-030** `GET /api/v1/portal/notices` — List notices visible to current user.
- **API-031** `GET /api/v1/portal/notices/{id}` — Notice detail.
- **API-032** `POST /api/v1/portal/notices/{id}/mark-read` — Record read status where enabled.
- **API-033** `GET /api/v1/portal/documents` — List visible documents.
- **API-034** `GET /api/v1/portal/documents/{id}/download` — Download document.

### 9.8 Support APIs
- **API-035** `POST /api/v1/portal/support-requests` — Create support request.
- **API-036** `GET /api/v1/portal/support-requests` — List own requests.
- **API-037** `GET /api/v1/portal/support-requests/{id}` — Get request thread.
- **API-038** `POST /api/v1/portal/support-requests/{id}/messages` — Add message to request.

### 9.9 Admin APIs
- **API-039** `GET /api/v1/admin/portal/config` — View school portal configuration.
- **API-040** `PATCH /api/v1/admin/portal/config` — Update school portal configuration.
- **API-041** `GET /api/v1/admin/portal/analytics` — Portal usage analytics.
- **API-042** `GET /api/v1/admin/portal/audit-logs` — Portal audit trail.

### 9.10 API design notes
- All portal APIs must enforce tenant and relationship-based authorization server-side.
- Parent-child context switching must never permit access to unlinked students.
- All download endpoints must perform permission checks at request time.
- Use pagination for notice lists, history pages, and support threads.
- Cache dashboard summaries carefully, but never cache across users or tenant boundaries.

***

## 10. Screens needed

### 10.1 Parent portal screens
- **SCR-001 Parent Login Screen** — Login, password reset, account activation.
- **SCR-002 Parent Dashboard** — Child switcher, attendance snapshot, due fees, latest notices, quick downloads.
- **SCR-003 Child Profile Overview** — Basic student identity, class, section, academic year, selected profile info.
- **SCR-004 Attendance Summary Screen** — Present/absent/late totals, recent records, monthly calendar.
- **SCR-005 Grade Summary Screen** — Assessment summaries, term-level overview, status of published results.
- **SCR-006 Report Cards Screen** — List of report cards, version labels, download action.
- **SCR-007 Timetable Screen** — Weekly timetable, today’s schedule, active timetable version view.
- **SCR-008 Assignments Screen** — Homework and assignment summaries with due dates.
- **SCR-009 Fee Balance and Payment Screen** — Outstanding dues, invoice list, pay-now actions.
- **SCR-010 Receipts and Payment History Screen** — Completed payments, failed attempts, receipt downloads.
- **SCR-011 Notices Screen** — Announcements list with filters.
- **SCR-012 Documents Screen** — Student documents, downloadable files, version display.
- **SCR-013 Support and Contact Screen** — Request submission, ticket list, message thread.

### 10.2 Student portal screens
- **SCR-014 Student Login Screen**
- **SCR-015 Student Dashboard**
- **SCR-016 Student Attendance Screen**
- **SCR-017 Student Grades and Report Cards Screen**
- **SCR-018 Student Timetable Screen**
- **SCR-019 Student Assignments Screen**
- **SCR-020 Student Notices Screen**
- **SCR-021 Student Documents Screen**
- **SCR-022 Student Fee View Screen** — Only if enabled by school policy.
- **SCR-023 Student Support Screen** — If enabled.

### 10.3 School administration screens
- **SCR-024 Portal Configuration Screen** — Enable or disable sections for parents and students.
- **SCR-025 Portal Audience Policy Screen** — Control access rules for notices, documents, fee visibility, and student self-service.
- **SCR-026 Portal Usage Analytics Screen** — Logins, active users, read rates, downloads, payment initiation metrics.
- **SCR-027 Portal Support Queue Screen** — Track and respond to incoming support requests.
- **SCR-028 Portal Audit Log Screen** — Sensitive activity monitoring.

***

## 11. Edge cases

- **EC-001** Parent is linked to multiple children across different grades and sections; dashboard must aggregate without mixing records.
- **EC-002** Parent-child relationship is removed after account creation; portal access must update immediately.
- **EC-003** Student transfers section mid-term; historical timetable, attendance, and grades must remain linked to original periods where applicable. 
- **EC-004** Student withdraws from school; portal visibility must follow retention policy rather than exposing unrestricted historical access. 
- **EC-005** Parent has two children in the same school with different fee statuses; payment actions must clearly target the correct child.
- **EC-006** Gradebook contains draft results; portal must show nothing until publication. 
- **EC-007** Attendance is recorded but not yet submitted; portal must not display provisional absence data. 
- **EC-008** Timetable draft exists alongside active version; portal must show only active published timetable.
- **EC-009** Report card is revised after publication; portal must show new version while preserving prior version history if enabled. 
- **EC-010** Parent account is active before guardian link is completed; access must remain blocked until linkage exists. 
- **EC-011** Student self-service is disabled school-wide; student portal login should be blocked or feature-limited according to policy.
- **EC-012** Failed payment attempt occurs during gateway callback delay; portal must not show receipt until finance confirmation is complete.
- **EC-013** Sensitive document is revoked after earlier visibility; future download attempts must be blocked.
- **EC-014** Parent uses shared family device and forgets logout; auto-expiry and session management must reduce exposure.
- **EC-015** Same guardian exists across multiple tenants or schools in the future; tenant context must remain explicit and isolated.
- **EC-016** Notice is intended for only one grade or section; unrelated users must not see it.
- **EC-017** Support request references a student who has become inactive; thread history must remain accessible according to policy.
- **EC-018** Student portal fee visibility is disabled while parent fee visibility is enabled; role-specific presentation must differ correctly.
- **EC-019** A document has multiple versions from SIS or report-card modules; portal must present the correct visible version and status. 
- **EC-020** Parent tries to open a bookmarked URL for an unlinked student; server must deny access even if the client path is manipulated.

***

## 12. Risks

### 12.1 Security risk
- **RK-001** Unauthorized access to another child’s records.  
  **Mitigation:** Relationship-based authorization, tenant scoping, audited child-switch events, server-side permission checks.

### 12.2 Data consistency risk
- **RK-002** Portal shows stale or unpublished data.  
  **Mitigation:** Read only from approved source states, cache invalidation on publish events, no local source copies. 

### 12.3 Privacy risk
- **RK-003** Sensitive staff-only or confidential student data becomes visible to guardians or students.  
  **Mitigation:** Explicit portal visibility flags, role-based filtering, policy-driven field exposure. 

### 12.4 Payment trust risk
- **RK-004** Parent sees incorrect fee balance or payment outcome.  
  **Mitigation:** Finance module as single source of truth, payment status reconciliation, immutable receipt rendering.

### 12.5 Operational risk
- **RK-005** Support requests submitted via portal are not routed or answered promptly.  
  **Mitigation:** Queue ownership, SLA configuration, status tracking, escalation reporting.

### 12.6 Adoption risk
- **RK-006** Parents do not use the portal consistently.  
  **Mitigation:** Mobile-first UX, simple login, clear dashboard value, notices and receipts centralized in one place.

### 12.7 Scalability risk
- **RK-007** High read traffic during report card publication or fee deadlines causes slowdowns.  
  **Mitigation:** Read-optimized APIs, background generation of downloadable files, caching of summaries, indexed tenant-partitioned queries.

### 12.8 Role confusion risk
- **RK-008** Student and parent experiences expose inconsistent data or actions.  
  **Mitigation:** Separate role-based layouts, explicit access policy matrix, school-configured feature flags.

### 12.9 Shared-device risk
- **RK-009** Families access the portal on public or shared devices.  
  **Mitigation:** Shorter session windows, device/session listing, optional OTP or step-up verification for sensitive actions.

***

## 13. Success metrics

### 13.1 Product metrics
- **SM-001** Parent portal monthly active usage rate above 70 percent of active linked guardians.
- **SM-002** Student portal monthly active usage rate above 50 percent where student portal is enabled.
- **SM-003** Dashboard load time under 2 seconds for standard schools.
- **SM-004** Notice page load time under 2 seconds.
- **SM-005** Document download success rate above 99 percent.
- **SM-006** Portal payment initiation success rate above 95 percent for valid attempts.

### 13.2 Parent transparency metrics
- **SM-007** Parent attendance view usage above 70 percent of active guardian users.
- **SM-008** Parent report card view or download rate above 80 percent after publication. 
- **SM-009** Parent receipt download usage above 60 percent for families making online payments.
- **SM-010** Notice read rate above 75 percent for portal-published notices where read tracking is enabled.

### 13.3 Operational metrics
- **SM-011** Reduction in office calls related to attendance, fees, report cards, and timetable by at least 50 percent.
- **SM-012** Reduction in manual receipt reprint requests by at least 60 percent.
- **SM-013** Reduction in parent-facing repetitive support requests by at least 40 percent.
- **SM-014** Support request first-response SLA compliance above 90 percent.

### 13.4 Business metrics
- **SM-015** Improvement in on-time fee payment rate after parent portal rollout.
- **SM-016** Increased digital payment adoption through portal channel.
- **SM-017** Improved school satisfaction and retention due to stronger transparency capabilities.
- **SM-018** Higher perceived value of bundled modules because published attendance, grades, timetable, and finance outputs are visible to end users.

### 13.5 Security and quality metrics
- **SM-019** Zero cross-tenant portal data exposure incidents.
- **SM-020** Zero unauthorized child-record exposure incidents.
- **SM-021** 100 percent audit coverage for login, child-switch, payment initiation, and protected file download events.
- **SM-022** Less than 1 percent of portal sessions ending in authorization error due to system defects.

***

## 14. Out of scope

- **OS-001** Full messaging or chat platform between parents and teachers.
- **OS-002** Social community features such as parent forums or student community feeds.
- **OS-003** Real-time classroom learning management workflows beyond published assignments and summaries.
- **OS-004** Student editing of official academic, attendance, timetable, or finance records.
- **OS-005** Parent editing of student master records except approved limited forms.
- **OS-006** Advanced document e-signature workflows.
- **OS-007** Video conferencing or virtual classroom delivery.
- **OS-008** AI-based parent assistant or chatbot in V1.
- **OS-009** Multi-tenant cross-school family dashboard beyond the current tenant boundary.
- **OS-010** Deep push-notification orchestration engine beyond standard notice and alert delivery.
- **OS-011** Offline-first mobile sync for all portal content in V1.
- **OS-012** Student-to-student collaboration or assignment submission workflows unless separately scoped under LMS.
- **OS-013** Parent-driven attendance correction requests in V1 unless explicitly required by target schools.
- **OS-014** Complex two-way academic feedback threads in V1.
- **OS-015** Full admissions applicant portal; this should remain separate from enrolled parent and student portal scope.

For a solo founder, the best V1 is:
1. Parent login and linked-children dashboard.
2. Student self portal with read-only access.
3. Published attendance, grades, timetable, notices, and documents.
4. Fee balances, payments, and receipts.
5. Support/contact requests.
6. Portal analytics and audit logs.

That gives strong commercial value without turning the portal into a second operational system. 
