# QA Checklists

Quality assurance checklists for releases, feature deployments, and module launches.

---

## Pre-Release Checklist (Every Release)

### Code Quality

- [ ] All CI checks passing (lint, unit tests, integration tests)
- [ ] Security tests passing (cross-tenant isolation + permission escalation)
- [ ] No TypeScript compilation errors
- [ ] No outstanding `TODO` comments in changed files
- [ ] Database migrations reviewed and reversible
- [ ] No hardcoded secrets or environment-specific values in code

### Security

- [ ] New endpoints have `@RequirePermissions()` or are explicitly public
- [ ] New tables have `tenant_id NOT NULL` and RLS policy
- [ ] No raw string interpolation in SQL queries
- [ ] New routes return typed DTOs (no raw entity exposure)
- [ ] File uploads validate MIME type server-side (not just extension)
- [ ] New audit-sensitive operations log to `audit_logs`

### Database

- [ ] Migrations applied to staging and tested
- [ ] New indexes reviewed for query patterns
- [ ] No breaking schema changes (no column drops, type changes without cast)
- [ ] RLS policies verified on new tables

### API

- [ ] All new endpoints documented in OpenAPI spec
- [ ] Error responses follow standard error contract
- [ ] Pagination implemented for list endpoints
- [ ] Response times verified: list endpoints < 500ms, single record < 200ms

---

## Module Launch Checklist

Use this checklist when launching a new module or major feature.

### M7.1 — Platform Core

- [ ] Tenant creation creates default roles, settings, and onboarding checklist
- [ ] Domain uniqueness enforced globally
- [ ] Subdomain routing resolves correct tenant
- [ ] Tenant suspension immediately revokes all active sessions
- [ ] RLS policies verified for all platform tables
- [ ] Audit log entries generated for: user create/disable, role change, subscription change, tenant suspend

### M7.2 — School Onboarding

- [ ] Academic year overlap constraint active and tested
- [ ] Only one active academic year allowed per tenant
- [ ] Grade name uniqueness enforced
- [ ] Section uniqueness per academic year/grade enforced
- [ ] Primary campus uniqueness enforced
- [ ] Go-live blocked until all mandatory checklist items complete
- [ ] Branding file size limit (5MB) enforced

### M7.3 — RBAC

- [ ] All system roles seeded per tenant
- [ ] Last School Admin cannot be removed
- [ ] Invitation links expire after 7 days
- [ ] Password reset tokens expire after 30 minutes (latest token only)
- [ ] User deactivation immediately revokes sessions
- [ ] Role changes audit-logged

### M7.4 — SIS

- [ ] Student number uniqueness enforced per tenant
- [ ] Enrollment history never deleted (no hard delete)
- [ ] One primary guardian per student enforced
- [ ] Status transitions follow state machine rules
- [ ] Student status change always creates `student_status_history` row
- [ ] Guardian linkage required before parent portal access
- [ ] Documents are versioned (replace creates new version)

### M7.6 — Attendance

- [ ] Duplicate session blocked (same date/section/period)
- [ ] Non-instructional day blocked
- [ ] One status per student per session
- [ ] Submitted attendance not editable by teacher
- [ ] Locked attendance corrections require approval
- [ ] Absence notifications trigger only after submission (not draft)
- [ ] Only enrolled students appear in roster

### M7.7 — Gradebook

- [ ] Component weights must sum to 100%
- [ ] Marks exceed max check active
- [ ] Only assigned teacher can enter marks
- [ ] Published report cards immutable (revision creates new version)
- [ ] Students/parents only see published results
- [ ] Mark changes audit-logged

### M7.8 — Fees

- [ ] Issued invoices cannot be edited
- [ ] Invoice amounts stored as snapshots (not recalculated)
- [ ] Only issued invoices accept payments
- [ ] Receipts immutable after issuance
- [ ] Refund amount cannot exceed confirmed payments
- [ ] Overpayment creates credit balance (not negative invoice)
- [ ] Idempotency keys enforced for online payments
- [ ] Webhook processing is idempotent
- [ ] Finance audit log entries generated for all critical actions
- [ ] Hard delete blocked on financial transactions

### M7.9 — Portal

- [ ] Only published/submitted data shown (no drafts)
- [ ] Parent access scoped to linked children only (server-side)
- [ ] Student access scoped to self only
- [ ] Document download permission checked at request time
- [ ] Login, child-switch, payment, and download events audited
- [ ] Portal sessions have configured expiry

### M7.10 — Communication

- [ ] Attendance alerts not triggered on draft sessions
- [ ] Fee reminders use official balance from finance module
- [ ] Result notifications only after publication
- [ ] Audience snapshots persisted at send time
- [ ] Sent messages not deletable (only archived)
- [ ] Webhook callbacks processed idempotently

---

## Performance Validation Checklist

Before production deployment of any module, verify against PRD-defined performance targets:

| Endpoint | Target | How to Test |
|----------|--------|-------------|
| Student list (500 students) | < 2 seconds | Load test with k6 |
| Teacher attendance entry | < 1 second per save | Manual test with class of 40 |
| Invoice list (1000 invoices) | < 2 seconds | Load test |
| Report card generation (500 students) | < 10 minutes total | Background job timing |
| Parent portal dashboard | < 2 seconds | Simulated parent load |
| Timetable view | < 2 seconds | Manual test |

---

## Regression Test Scenarios

Run these scenarios after any change to shared infrastructure (auth, database, tenant resolution):

1. **Multi-school user:** User with memberships in 2 schools — verify correct data isolation after tenant selection
2. **Suspended tenant:** Suspend tenant → verify all sessions revoked, GET returns 403, POST returns 403
3. **Subscription expiry:** Expire subscription → verify read-only mode active
4. **Last admin protection:** Attempt to disable last School Admin → verify blocked
5. **Parent child switching:** Parent with 2 children → verify switching shows correct child's data
6. **Teacher scope:** Teacher attempts to access unassigned section's attendance → verify blocked

---

## Release Sign-Off

Before every production deployment, the following must sign off:

| Role | Responsibility |
|------|---------------|
| Lead Engineer | Code review complete, tests passing, no known critical bugs |
| QA Lead | Regression tests passed, new feature tested |
| Security Review | No new security vulnerabilities in changed code |
| Product Owner | Feature meets requirements, UX acceptable |

For financial module changes, an additional sign-off is required:
| Finance Operations | Verify billing calculations are correct on staging data |
