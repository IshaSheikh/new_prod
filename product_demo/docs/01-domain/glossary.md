<!-- Source: PRDs | Last Verified: 2026-06-28 | Owner: Engineering -->
# Glossary

Definitions for core terms used across the platform, PRDs, API contracts, and database schema.

---

## Platform Concepts

**Tenant**
A single school organization registered on the platform. Every tenant has its own isolated data, subdomain or custom domain, subscription, and user memberships. No data crosses tenant boundaries.

**Super Admin**
A platform-level operator with access across all tenants. Not a school-level role. Responsible for tenant creation, subscription management, and platform-level audit.

**Subscription**
A commercial agreement between the platform and a tenant. Controls which features are available and enforces limits on students and staff. States: Trial → Active → Grace Period → Expired/Cancelled/Suspended.

**Tenant Domain**
A subdomain (e.g., `greenfield.schoolsaas.com`) or custom domain (e.g., `portal.greenfieldschool.edu`) used to route users to the correct tenant context.

**Feature Flag**
A per-subscription or per-tenant capability toggle. Used to enable modules like transport, library, or HR/payroll based on the tenant's plan.

---

## Users and Access

**User**
A global platform identity identified by a unique email address. A user may belong to multiple tenants simultaneously (e.g., a teacher working at two schools).

**Membership**
A user's relationship with a specific tenant. Access, roles, and permissions are granted through memberships — not directly on the user record. States: Invited → Accepted → Active → Disabled/Removed.

**Role**
A named collection of permissions assigned to a membership. Roles are tenant-scoped in V1 and use a fixed set: School Admin, Principal, Teacher, Accountant, Parent, Student, Transport Manager, Librarian.

**Permission**
A granular action code (e.g., `attendance.mark`, `fees.generate_invoice`) that controls what a user can do. Permissions are global and inherited through roles.

**Invitation**
A time-bound (7 days) link sent to an email address to join a tenant. If the email already belongs to a platform user, a membership is created. Otherwise, a new user account is created on acceptance.

**Parent-Student Link**
An explicit relationship between a guardian user and a student. Parents can only see portal data for linked children. One parent may be linked to multiple students; one student should have one primary guardian link.

**Teacher Assignment**
A scoped relationship between a teacher membership and a specific academic year, grade, section, and subject. Teachers only see data for their assigned contexts.

---

## School Structure

**School**
The institutional entity within a tenant. Currently one school per tenant, with a structured onboarding lifecycle from Draft to Live.

**Academic Year**
A defined calendar period (e.g., 2026–2027) during which academic operations occur. Only one academic year can be active at a time. Academic years cannot overlap.

**Term / Semester**
A sub-period within an academic year (e.g., Term 1, Term 2). Terms must fall within their parent academic year and cannot overlap.

**Grade Level**
An academic stage (e.g., Grade 1, Grade 2, Grade 10). Grade names are unique within a tenant. Grades are shared across academic years.

**Section**
A classroom division within a grade for a specific academic year (e.g., Grade 1-A). Each Grade + Section combination must be unique per academic year.

**Campus**
A physical location the school operates from. A school may have multiple campuses. One campus must be designated as primary.

---

## Students

**Student**
The primary academic entity in the SIS. Identified by a unique student number (never reused). Lifecycle: Inquiry → Applicant → Admitted → Enrolled → Active → Transferred/Withdrawn/Graduated/Deceased.

**Student Number**
A unique, non-reusable identifier within a tenant. Format example: `GFS-2026-0042`. Separate from the database UUID primary key.

**Admission Number**
An optional school-assigned sequential admission identifier.

**Enrollment**
A student's assignment to an academic year, grade, and section. A student may have only one primary active enrollment per academic year. Enrollment history is never deleted.

**Guardian**
A contact person associated with a student (father, mother, guardian, etc.). One guardian can be linked to multiple students. One guardian must be marked as primary per student.

**Student Status History**
An immutable audit trail of every status change on a student record. Required for transfers, withdrawals, and graduation events.

---

## Academic Operations

**Subject**
A teachable topic (e.g., Mathematics, English, Physics). Subject codes are unique within a tenant. Subjects are reusable across academic years.

**Subject Group**
A logical category of related subjects (e.g., Languages, Sciences, Arts).

**Period**
A time slot in the school day (e.g., Period 1: 08:30–09:15). Periods belong to a Period Scheme and must not overlap within the same scheme.

**Period Scheme**
A named set of ordered periods defining a school day structure. Referenced by timetables.

**Timetable**
A versioned schedule assigning subjects, teachers, and rooms to day-of-week + period combinations for a specific grade-section in an academic year. Only one version can be published at a time. States: Draft → Under Review → Approved → Published → Archived.

**Room**
A physical space identified by a code within a campus (e.g., `R-101`). Rooms cannot be double-booked in published timetables.

---

## Attendance

**Attendance Session**
A single attendance-taking event for a specific date, grade, section, and optionally timetable slot. States: Draft → Submitted → Locked.

**Attendance Record**
A student's attendance status within a session. One record per student per session.

**Attendance Mode**
Configurable per school: Daily (one session per day per section) or Period-wise (one session per period per section per day).

**Attendance Correction**
A formal change request for a submitted or locked attendance record. Requires a reason. Post-lock corrections require School Admin approval.

**Instructional Day**
A calendar day designated as a school day. Attendance can only be taken on instructional days.

---

## Assessment and Grades

**Assessment**
An evaluation event for a specific academic context (year, grade, section, subject). Assessments have components (theory, practical, project, assignment).

**Assessment Component**
A sub-part of an assessment with its own maximum marks and weight percentage. The sum of all component weights for one assessment must equal 100%.

**Marks Entry**
A student's marks record for one assessment component. Stores raw marks, weighted marks, and final grade. Only the assigned teacher may enter marks.

**Grading Scheme**
A school-configured mapping from score ranges to grade codes and grade points. Supports percentage-based, letter grade, and GPA systems.

**Report Card**
A published, immutable academic record for a student covering one academic period. Generated from verified marks. Any correction after publication creates a new versioned revision.

**Result Publication**
A controlled batch that makes report cards visible to parents and students. Only published results are visible through the portal.

---

## Fees and Finance

**Fee Category**
A named type of charge (e.g., Tuition, Transport, Exam, Lab, Books). Optional categories can be enabled per student as add-ons.

**Fee Structure**
A versioned billing template for a school context (academic year, grade, section). Defines recurring or one-time charges per fee category.

**Invoice**
A billing record issued to a student with a snapshot of charges, discounts, scholarships, penalties, and due date. Invoice amounts are immutable snapshots; they are not recalculated from live rules after issuance.

**Invoice State**
`draft → issued → partially_paid / paid / overdue → refunded_partial / refunded_full / cancelled`

**Payment Transaction**
A record of every collection attempt or confirmation — online or offline. Failed transactions remain visible for audit.

**Payment Allocation**
A mapping from a payment transaction to one or more invoices, tracking the allocated amount per invoice.

**Receipt**
An immutable proof-of-payment record issued for every successful or confirmed payment. Receipt corrections require reversal + reissuance; direct editing is not allowed.

**Refund**
A partial or full reversal of a confirmed payment. Requires reason, approver (where configured), and refund mode.

**Reconciliation**
The process of matching internal payment transactions against external settlement records (gateway reports, bank statements). Reconciliation adds state and references without altering original transactions.

**Ledger Entry**
An immutable accounting row created for every invoice, payment, refund, adjustment, or reversal. Used for financial history reconstruction.

**Credit Balance**
Unapplied funds available for future invoice allocation. Created when a payment exceeds invoice obligations.

---

## Communication

**Notice**
A message published to a defined audience (school-wide, grade, section, role, or individual). States: Draft → Scheduled → Queued → Sent → Archived.

**Outbound Message**
A single delivery attempt of a notice or triggered alert to one recipient via one channel (email, SMS, WhatsApp, push, in-app).

**Delivery Channel**
The method used to send a message. Supported: In-App, Email, SMS, WhatsApp, Push Notification.

**Message Template**
A reusable message definition with placeholder variables (student name, due amount, date, etc.). Templates are categorized by module and channel.

**Automation Rule**
A trigger-based communication rule that fires when a defined event occurs (attendance submission, invoice due date, payment success, result publication).

---

## Portal

**Parent Portal**
A secure self-service interface for guardians to view attendance, grades, fees, timetable, notices, and documents for linked children.

**Student Portal**
A self-service interface for students to view their own academic and operational data.

**Portal Session**
A tracked login session for portal users, with device information and expiry. Sessions can be revoked individually or all at once.

**Support Request**
A parent or student-submitted ticket routed to the school's administrative queue.

---

## Admissions

**Inquiry**
An initial expression of interest in admission. May be created by staff or submitted by a parent online.

**Applicant**
A prospective student converted from an inquiry. Has application form, documents, and evaluation records.

**Offer**
A time-bound admission offer issued to an evaluated applicant. Must be accepted before the expiry date.

**Enrollment Handoff**
The process of converting an accepted applicant into an admitted student record in the SIS.

---

## Transport

**Route**
A named path a school vehicle follows, with an ordered list of stops.

**Stop**
A named pickup or drop location on a route.

**Student Transport Assignment**
A student's assignment to a specific route and stop with effective dates. Only one active assignment per student per assignment type is allowed.

---

## Library

**Book**
A catalog entry in the library (title, author, ISBN, category).

**Book Copy**
A physical instance of a book that can be issued. Each copy has a unique code and tracks availability status.

**Issue**
A lending record associating a book copy with a borrower student, with an issue date and due date.

**Fine**
A penalty amount applied when a book copy is returned late. Fines are policy-driven and can be waived by authorized staff.

---

## HR and Payroll

**Staff Profile**
An employment record for a school staff member, optionally linked to a platform user account.

**Leave Request**
A formal request from a staff member for approved leave. States: Draft → Submitted → Approved/Rejected.

**Payroll Run**
A processed payroll calculation for one pay period. States: Draft → Calculated → Approved → Finalized.

**Payslip**
A per-employee earnings record generated from a finalized payroll run. Published to staff after payroll is complete.

---

## Technical Terms

**Tenant Context**
The runtime scope of every authenticated request. Set from JWT claims after tenant selection. All database queries must filter by tenant context.

**Row-Level Security (RLS)**
A PostgreSQL feature used to enforce tenant isolation at the database layer. Every query automatically filters to the authenticated tenant's records.

**Soft Delete**
Marking a record as deleted via a `deleted_at` timestamp rather than physically removing it. Used only for setup records where full deletion would break audit trails (e.g., students after enrollment, branding assets, fee categories).

**Audit Log**
An immutable, append-only record of a significant event. Audit logs cannot be edited or deleted.

**Idempotency Key**
A unique client-generated token on payment operations to prevent duplicate processing if a request is retried.

**Snapshot**
A point-in-time copy of calculated or template data stored in the record itself (e.g., invoice totals, result data in report cards). Snapshots prevent historical records from being affected by future configuration changes.

