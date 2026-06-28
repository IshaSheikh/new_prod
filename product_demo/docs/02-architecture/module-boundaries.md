<!-- Source: PRDs | Last Verified: 2026-06-28 | Owner: Engineering -->
# Module Boundaries

Defines the ownership, responsibilities, and dependency relationships between platform modules. Each module owns its data, enforces its own business rules, and exposes data to other modules only through well-defined interfaces.

---

## Module Dependency Tree

```
M7.1 — Multi-Tenant Platform Core
  │
  └─> M7.2 — School Onboarding and Configuration
        │
        └─> M7.3 — User Management and RBAC
              │
              └─> M7.4 — Student Information System (SIS)
                    │
                    ├─> M7.5 — Academic Structure and Timetable
                    │     │
                    │     └─> M7.6 — Attendance Management
                    │
                    ├─> M7.7 — Gradebook and Report Cards
                    │     │
                    │     └─> M7.13 — Exams and Publishing
                    │
                    ├─> M7.8 — Fees and Finance
                    │     │
                    │     └─> M7.14 — Transport Management (fee hooks)
                    │
                    ├─> M7.9 — Parent and Student Portal
                    │
                    ├─> M7.10 — Communication Hub
                    │
                    ├─> M7.11 — Admissions Workflow
                    │     └─> (handoff into M7.4 SIS)
                    │
                    ├─> M7.12 — Assignments and Homework
                    │
                    ├─> M7.14 — Transport Management
                    │
                    ├─> M7.15 — Library
                    │
                    └─> M7.16 — HR and Payroll
                          │
                          └─> M7.17 — Analytics and KPI Dashboards
                                (consumes from all modules, read-only)
```

---

## Module Ownership Table

| Module | Owner Entity | Data It Owns | Consumes From |
|--------|-------------|--------------|---------------|
| M7.1 — Platform Core | `tenants`, `users`, `memberships`, `subscriptions`, `audit_logs` | Global | None |
| M7.2 — Onboarding | `schools`, `academic_years`, `terms`, `grade_levels`, `sections`, `campuses`, `branding_assets` | School setup | M7.1 |
| M7.3 — RBAC | `roles`, `permissions`, `invitations`, `parent_student_links`, `teacher_assignments`, `user_sessions` | Access control | M7.1, M7.2 |
| M7.4 — SIS | `students`, `student_profiles`, `student_enrollments`, `guardians`, `guardian_student_links`, `student_documents`, `student_health_notes` | Student master | M7.1, M7.2, M7.3 |
| M7.5 — Timetable | `subjects`, `rooms`, `periods`, `timetables`, `timetable_slots` | Academic schedule | M7.2, M7.3, M7.4 |
| M7.6 — Attendance | `attendance_sessions`, `attendance_records`, `attendance_corrections` | Attendance records | M7.4, M7.5 |
| M7.7 — Gradebook | `assessments`, `marks_entries`, `grading_schemes`, `report_cards` | Academic results | M7.4, M7.5 |
| M7.8 — Fees | `invoices`, `payment_transactions`, `receipts`, `refunds`, `ledger_entries` | Financial records | M7.2, M7.3, M7.4 |
| M7.9 — Portal | Portal sessions, preferences, notices, support requests | Portal state | All modules (read) |
| M7.10 — Comms | `notices`, `outbound_messages`, `templates`, `automation_rules` | Communication records | M7.3, M7.4, M7.6, M7.7, M7.8 |
| M7.11 — Admissions | `admission_inquiries`, `applicants`, `offers`, `assessment_results` | Admission pipeline | M7.2, M7.3 → handoff to M7.4 |
| M7.12 — Assignments | `assignments`, `assignment_submissions`, `grading_feedback` | Assignment records | M7.4, M7.5 |
| M7.13 — Exams | `exam_terms`, `exam_schedules`, `marksheets`, `publication_batches` | Exam operations | M7.4, M7.5, M7.7 |
| M7.14 — Transport | `vehicles`, `routes`, `stops`, `student_transport_assignments` | Transport records | M7.4 → hooks to M7.8 |
| M7.15 — Library | `books`, `book_copies`, `issues`, `returns`, `fines` | Library circulation | M7.4 |
| M7.16 — HR | `staff_profiles`, `leave_requests`, `payroll_runs`, `payslips` | HR/payroll records | M7.1, M7.2, M7.3 |
| M7.17 — Analytics | Dashboard definitions, KPI snapshots, export jobs | Read-only views | All modules |

---

## Cross-Module Contract Rules

### 1. SIS is the Student Master
All modules that reference students must use `student_id` from the SIS (`students` table). No module may create its own parallel student identity or store redundant student demographic data.

**Enforced for:** Attendance, Gradebook, Fees, Portal, Communication, Transport, Library, Exams, Assignments, Admissions (at handoff).

### 2. Academic Structure is the Schedule Master
Attendance, Gradebook, Exams, and Assignments must consume academic year, grade, section, subject, and timetable references from the Onboarding and Timetable modules. They must not maintain their own schedule records.

**Enforced for:** M7.6 (Attendance), M7.7 (Gradebook), M7.12 (Assignments), M7.13 (Exams).

### 3. Finance Reads Student/Guardian from SIS
The Fees module references students and guardians from the SIS. It does not store student demographic data.

**Enforced for:** M7.8 (Fees).

### 4. Portal is Read-Only from Source Modules
The Parent and Student Portal (M7.9) reads data from attendance, gradebook, timetable, fees, and communication modules. The portal does not create, modify, or store operational academic, attendance, timetable, or financial records.

**Exceptions (write allowed):** Fee payment initiation, support request submission, document download (logged), form submissions (where explicitly approved).

### 5. Communication Hub Does Not Source Its Own Data
The Communication Hub (M7.10) fires messages based on events and data from other modules. It does not maintain student, guardian, or financial data independently.

### 6. Analytics is Strictly Read-Only
M7.17 consumes from all modules through read-optimized queries and materialized views. It does not modify any operational records.

### 7. Admissions Handoff is One-Way
When an applicant is admitted, their data flows into the SIS (M7.4) through a controlled handoff operation. Admissions records are preserved but do not receive updates from the SIS after handoff.

---

## What Each Module Must NOT Do

| Module | Must Not |
|--------|---------|
| M7.6 Attendance | Create its own student list or academic schedule |
| M7.7 Gradebook | Modify published report cards without creating a revision |
| M7.8 Fees | Store student demographic data; recalculate issued invoices from live rules |
| M7.9 Portal | Expose draft/unpublished data from any source module |
| M7.10 Communication | Send attendance alerts before official submission; duplicate student identity |
| M7.12 Assignments | Automatically update report card results without explicit integration |
| M7.13 Exams | Bypass gradebook publication controls |
| M7.14 Transport | Retroactively alter settled finance records when assignments change |
| M7.17 Analytics | Modify operational records; recalculate source module data independently |

---

## Integration Patterns

### Event-Driven Triggers (Communication Hub)
Events emitted by source modules trigger the Communication Hub:
- `attendance.session.submitted` → triggers absence/late alerts
- `invoice.issued` → triggers fee notification
- `payment.confirmed` → triggers receipt notification
- `invoice.overdue` → triggers overdue reminder
- `report_card.published` → triggers result notification
- `timetable.changed` → triggers timetable change alert

### Portal Data Consumption
The portal aggregates data from source modules at request time. Summary data (attendance percentage, outstanding balance) is cached with short TTLs (2–5 minutes). Real-time reads are used for time-sensitive data (payment status, latest notice).

### Analytics Materialization
KPI dashboards use materialized views or precomputed snapshots refreshed on a schedule:
- Daily attendance rate: refreshed at end of day
- Outstanding fee amount: refreshed hourly
- Exam completion rate: refreshed on demand
- Portal engagement: refreshed daily

