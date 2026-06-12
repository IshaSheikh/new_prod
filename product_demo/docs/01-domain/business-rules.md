# Business Rules

Authoritative business rules governing platform behavior across all modules. Rules are grouped by domain. These rules must be enforced at the application layer, database layer (constraints, triggers), and API layer (validation guards).

---

## Tenant and Platform Rules

**BR-PLATFORM-001** Every tenant-owned table must include `tenant_id UUID NOT NULL`. Exceptions are approved global tables: `users`, `permissions`, `roles` (when globally scoped).

**BR-PLATFORM-002** No user may access records belonging to another tenant. Cross-tenant joins are prohibited. Every API query must automatically filter by `tenant_id`.

**BR-PLATFORM-003** A user may belong to multiple tenants. Permissions are granted through memberships — not directly on the user record.

**BR-PLATFORM-004** Only active subscriptions can access business modules. Expired subscriptions enter a grace period. Suspended subscriptions become read-only.

**BR-PLATFORM-005** Tenant suspension must immediately invalidate all active sessions for that tenant.

**BR-PLATFORM-006** Domain names are globally unique. No two tenants can share the same subdomain or custom domain.

**BR-PLATFORM-007** Audit logs are immutable. Audit records cannot be edited or deleted.

**BR-PLATFORM-008** Sensitive operations must generate audit entries: user creation/deletion, role assignment, subscription changes, tenant suspension.

---

## User and Membership Rules

**BR-USER-001** Email addresses must be globally unique across all users.

**BR-USER-002** Inactive users cannot authenticate.

**BR-USER-003** Every membership must have at least one active role.

**BR-USER-004** A membership may hold multiple roles (e.g., Teacher + Librarian).

**BR-USER-005** Role changes require audit logging.

**BR-USER-006** User deactivation immediately invalidates all active sessions for that user.

**BR-USER-007** The last active School Admin in a tenant cannot be removed or disabled. At least one active School Admin must exist at all times.

**BR-USER-008** Password reset tokens expire after 30 minutes. Only the latest token is valid if multiple are requested.

**BR-USER-009** Invitation links expire after 7 days. If the invited email belongs to an existing user, a membership is created rather than a new user account.

**BR-USER-010** Parent access is scoped to linked children only. Parents cannot see any data for unlinked students.

**BR-USER-011** Teacher access is scoped to assigned classes, sections, and subjects only. Teachers cannot access unassigned student records.

---

## School Onboarding Rules

**BR-ONBOARD-001** A school cannot go live until all mandatory setup steps are complete: school profile, active academic year, grade structure, section structure, fee categories configured, at least one School Admin, at least one Teacher.

**BR-ONBOARD-002** Every tenant must have exactly one active school profile.

**BR-ONBOARD-003** Only one academic year can be active at a time.

**BR-ONBOARD-004** Academic year dates cannot overlap.

**BR-ONBOARD-005** Term dates must fall inside the parent academic year and cannot overlap.

**BR-ONBOARD-006** Grade names must be unique within a tenant.

**BR-ONBOARD-007** Each Grade + Section combination must be unique per academic year.

**BR-ONBOARD-008** A section cannot exist without a grade.

**BR-ONBOARD-009** Academic years cannot be deleted after students are enrolled.

**BR-ONBOARD-010** One campus must be designated as the primary campus. Deletion of the primary campus is blocked until another primary is designated.

**BR-ONBOARD-011** Branding file uploads must be PNG, JPG, or SVG, and must not exceed 5 MB.

**BR-ONBOARD-012** Section capacity reduction below the current enrolled student count is blocked.

---

## Student and Enrollment Rules

**BR-SIS-001** Each student receives a unique student number within the tenant. Student numbers cannot be reused.

**BR-SIS-002** Student profiles cannot be permanently deleted after enrollment. Soft delete only.

**BR-SIS-003** A student may have only one primary enrollment per academic year (unless dual enrollment is explicitly enabled by school configuration).

**BR-SIS-004** Enrollment must reference academic year, grade, and section.

**BR-SIS-005** Enrollment history must never be deleted.

**BR-SIS-006** A student cannot transition directly from Inquiry to Enrolled.

**BR-SIS-007** Withdrawal requires a reason. Transfer requires a destination.

**BR-SIS-008** One guardian must be marked as primary per student.

**BR-SIS-009** Parents may be linked to multiple students. A student's primary guardian cannot be removed until another primary is designated.

**BR-SIS-010** Guardian linkage is required before parent portal access is granted.

**BR-SIS-011** Student documents are versioned. Replacing a document creates a new version; prior versions are retained.

**BR-SIS-012** Critical health notes are visible to teachers assigned to the student. Confidential health notes are role-protected.

**BR-SIS-013** All student status transitions must create an immutable record in `student_status_history`.

---

## Academic Structure and Timetable Rules

**BR-TIMETABLE-001** Subject codes must be unique within a tenant.

**BR-TIMETABLE-002** Period timings must not overlap within the same period scheme.

**BR-TIMETABLE-003** A room cannot be assigned to overlapping timetable slots.

**BR-TIMETABLE-004** A teacher cannot be assigned to overlapping timetable slots.

**BR-TIMETABLE-005** A section cannot have overlapping periods.

**BR-TIMETABLE-006** Only one timetable version can be published per grade-section at a time. Publishing automatically archives the previously active version.

**BR-TIMETABLE-007** Conflict validation must occur before publication. Publication is blocked if blocking conflicts exist.

**BR-TIMETABLE-008** Archived academic years cannot receive timetable changes.

**BR-TIMETABLE-009** Removing a subject is blocked while it is referenced in an active published timetable.

---

## Attendance Rules

**BR-ATT-001** Attendance must be recorded against academic year, grade, section, and date.

**BR-ATT-002** Period-wise attendance additionally requires timetable slot, subject, and teacher assignment references.

**BR-ATT-003** Only active enrolled students appear in attendance sessions.

**BR-ATT-004** Each student may have only one attendance status per attendance session.

**BR-ATT-005** Attendance on non-instructional days (holidays, closures) is blocked.

**BR-ATT-006** Duplicate attendance sessions for the same date/section/period are blocked.

**BR-ATT-007** Submitted attendance cannot be edited by teachers. Corrections after lock require School Admin approval.

**BR-ATT-008** All attendance corrections require a reason and are preserved in history.

**BR-ATT-009** Absence and late notifications are triggered only after official attendance submission, not during draft.

**BR-ATT-010** Attendance percentage calculations exclude non-instructional days.

**BR-ATT-011** Transferred or withdrawn students remain in historical attendance reports.

---

## Assessment and Gradebook Rules

**BR-GRADE-001** Assessments must belong to academic year, grade, section, and subject.

**BR-GRADE-002** The sum of all assessment component weight percentages must equal exactly 100%.

**BR-GRADE-003** Marks must not exceed the configured maximum marks for the component.

**BR-GRADE-004** Only the assigned teacher may enter marks for an assessment.

**BR-GRADE-005** Results cannot move to verification until all mandatory marks are entered.

**BR-GRADE-006** Verified results become read-only for teachers.

**BR-GRADE-007** Report cards cannot be published with missing mandatory marks.

**BR-GRADE-008** Published report cards are immutable. Any correction after publication creates a new versioned revision.

**BR-GRADE-009** Previous report card versions remain permanently accessible to administrators.

**BR-GRADE-010** Students and parents can only view published results.

**BR-GRADE-011** Every mark change, grade recalculation, and report card revision must be logged immutably.

**BR-GRADE-012** Grading scheme changes after publication do not affect already-published report cards.

---

## Fee and Finance Rules

**BR-FEE-001** Every finance record must belong to exactly one tenant and one school. Cross-tenant finance access is prohibited.

**BR-FEE-002** Student, guardian, class, section, and academic year references must come from the SIS and onboarding modules — not be recreated inside finance.

**BR-FEE-003** Fee structures must be versioned by academic year.

**BR-FEE-004** Each student must have one active fee structure per billing context.

**BR-FEE-005** A student changing class or section does not retroactively alter already-issued invoices.

**BR-FEE-006** Only issued invoices can accept payments. Draft and cancelled invoices cannot be paid.

**BR-FEE-007** Invoice amounts are stored as snapshots at issuance time and must not be recalculated from live setup rules after issuance.

**BR-FEE-008** Overdue status is system-derived when the current date passes the due date and outstanding balance remains.

**BR-FEE-009** Partial payments are allowed. A payment may be allocated to one or multiple invoices.

**BR-FEE-010** Failed online payments remain auditable and visible in transaction history.

**BR-FEE-011** Overpayments must create unapplied credit or trigger a refund workflow — they must never silently disappear.

**BR-FEE-012** Receipts are generated only for successful or confirmed payments.

**BR-FEE-013** Receipts are immutable after issuance. Corrections require reversal + new receipt issuance. Direct editing is not allowed.

**BR-FEE-014** Receipt numbers are unique per tenant, with configurable prefix and sequence.

**BR-FEE-015** Refunds require reason, original payment reference, and refund mode. Partial and full refunds are supported.

**BR-FEE-016** Reconciliation adds state to transaction records — it must not mutate original payment records.

**BR-FEE-017** Discount application order is deterministic: line-item discount first → invoice-level discount → scholarship → penalty last.

**BR-FEE-018** Financial transactions must never be hard deleted.

**BR-FEE-019** Duplicate payment detection must warn users when amount, student, reference, and timestamp are suspiciously similar (idempotency key enforcement).

---

## Communication Rules

**BR-COMM-001** Every communication record must belong to exactly one tenant. Messages, audiences, and delivery logs are never accessible across tenants.

**BR-COMM-002** Attendance alerts must only trigger after official attendance submission — not during draft capture.

**BR-COMM-003** Result publication notifications must only trigger after published result status exists in the gradebook module.

**BR-COMM-004** Fee reminders must use official dues and payment states from the finance module.

**BR-COMM-005** Audience snapshots must be persisted at send time for auditability.

**BR-COMM-006** Message deletion is prohibited after dispatch. Corrections require cancellation (pre-send) or follow-up communication (post-send).

**BR-COMM-007** Delivery provider callbacks must be processed idempotently. Retrying failed messages must not create duplicate successful sends.

**BR-COMM-008** High-priority alerts require delivery tracking.

---

## Portal Rules

**BR-PORTAL-001** The portal is read-only by default. Write actions are limited to: fee payment, document download, support/contact submission, and approved form submissions.

**BR-PORTAL-002** The portal must only display published, approved, submitted, or portal-visible data from source modules.

**BR-PORTAL-003** Draft attendance, draft grades, draft report cards, and unpublished timetables must not appear in the portal.

**BR-PORTAL-004** Parent access must always be restricted to linked children only.

**BR-PORTAL-005** Student access must always be restricted to self only.

**BR-PORTAL-006** Document access must be permission-checked at request time.

**BR-PORTAL-007** Every login, child-switch, payment initiation, and protected document download must be auditable.

---

## Admissions Rules

**BR-ADM-001** An inquiry may be converted into one applicant only.

**BR-ADM-002** Duplicate applicants must be flagged by phone, email, and date-of-birth similarity before progression.

**BR-ADM-003** Application form requirements vary by grade. Mandatory fields must be completed before the application can advance.

**BR-ADM-004** All mandatory documents must be submitted or explicitly waived by authorized staff before evaluation begins.

**BR-ADM-005** Offers have explicit expiry dates. Expired offers cannot be accepted unless reissued.

**BR-ADM-006** Admitted applicants must be converted into student and enrollment records through SIS workflows, not through ad hoc record creation.

**BR-ADM-007** Every status transition, offer event, and decision must be auditable.

---

## Transport Rules

**BR-TRANS-001** Vehicle and route capacity limits must be enforced before assignment confirmation. Over-capacity assignments require authorized override.

**BR-TRANS-002** A student may have only one active transport assignment per assignment type.

**BR-TRANS-003** Assignment history must never be deleted. Changes must preserve effective start and end dates.

**BR-TRANS-004** Inactive routes and stops cannot receive new student assignments.

**BR-TRANS-005** Transport payment changes must not retroactively alter settled finance records.

---

## Library Rules

**BR-LIB-001** A book copy can have only one active issue at a time.

**BR-LIB-002** Issues must reference a valid active student borrower.

**BR-LIB-003** Returns must reference an existing active issue.

**BR-LIB-004** Historical issue and return records must never be deleted.

**BR-LIB-005** Fine changes must be auditable. Fine waivers require authorized staff action.

---

## HR and Payroll Rules

**BR-HR-001** Payroll records must preserve the salary snapshot at processing time.

**BR-HR-002** Closed payroll periods must not be recalculated silently. A controlled adjustment workflow is required.

**BR-HR-003** Leave requests with overlapping dates for the same staff member are blocked.

**BR-HR-004** Changes to statutory and payroll-sensitive fields must be audited.

**BR-HR-005** Payslips must not be visible to staff until payroll is finalized or published according to school policy.
