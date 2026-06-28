<!-- Source: PRDs | Last Verified: 2026-06-28 | Owner: Engineering -->
# Seed Data

Required seed data that must be present before any tenant can be onboarded or any school can operate. This data is inserted during initial platform setup and when a new tenant is provisioned.

---

## Platform-Level Seed Data

### Permissions

Core permission codes used across all modules. These are seeded globally and never tenant-specific.

```sql
-- Platform/Tenant Management
INSERT INTO permissions (code, module_name, action_name, description) VALUES
('platform.tenant.create', 'platform', 'tenant.create', 'Create a new tenant'),
('platform.tenant.suspend', 'platform', 'tenant.suspend', 'Suspend a tenant'),
('platform.tenant.activate', 'platform', 'tenant.activate', 'Activate a tenant'),
('platform.subscription.manage', 'platform', 'subscription.manage', 'Manage tenant subscriptions'),
('platform.audit.view', 'platform', 'audit.view', 'View platform audit logs');

-- User Management
INSERT INTO permissions (code, module_name, action_name) VALUES
('users.invite', 'users', 'invite'),
('users.view', 'users', 'view'),
('users.edit', 'users', 'edit'),
('users.disable', 'users', 'disable'),
('users.roles.assign', 'users', 'roles.assign'),
('users.parent_links.manage', 'users', 'parent_links.manage'),
('users.teacher_assignments.manage', 'users', 'teacher_assignments.manage');

-- School Configuration
INSERT INTO permissions (code, module_name, action_name) VALUES
('school.profile.view', 'school', 'profile.view'),
('school.profile.edit', 'school', 'profile.edit'),
('school.academic_year.manage', 'school', 'academic_year.manage'),
('school.grades.manage', 'school', 'grades.manage'),
('school.sections.manage', 'school', 'sections.manage'),
('school.campuses.manage', 'school', 'campuses.manage'),
('school.branding.manage', 'school', 'branding.manage');

-- Students
INSERT INTO permissions (code, module_name, action_name) VALUES
('students.view', 'students', 'view'),
('students.create', 'students', 'create'),
('students.edit', 'students', 'edit'),
('students.enroll', 'students', 'enroll'),
('students.transfer', 'students', 'transfer'),
('students.withdraw', 'students', 'withdraw'),
('students.documents.manage', 'students', 'documents.manage'),
('students.health.view', 'students', 'health.view'),
('students.health.manage', 'students', 'health.manage');

-- Attendance
INSERT INTO permissions (code, module_name, action_name) VALUES
('attendance.view', 'attendance', 'view'),
('attendance.mark', 'attendance', 'mark'),
('attendance.submit', 'attendance', 'submit'),
('attendance.lock', 'attendance', 'lock'),
('attendance.correction.request', 'attendance', 'correction.request'),
('attendance.correction.approve', 'attendance', 'correction.approve'),
('attendance.configure', 'attendance', 'configure'),
('attendance.reports.view', 'attendance', 'reports.view');

-- Timetable
INSERT INTO permissions (code, module_name, action_name) VALUES
('timetable.subjects.manage', 'timetable', 'subjects.manage'),
('timetable.rooms.manage', 'timetable', 'rooms.manage'),
('timetable.create', 'timetable', 'create'),
('timetable.review', 'timetable', 'review'),
('timetable.publish', 'timetable', 'publish'),
('timetable.view', 'timetable', 'view');

-- Gradebook
INSERT INTO permissions (code, module_name, action_name) VALUES
('gradebook.assessments.manage', 'gradebook', 'assessments.manage'),
('gradebook.marks.enter', 'gradebook', 'marks.enter'),
('gradebook.marks.verify', 'gradebook', 'marks.verify'),
('gradebook.results.publish', 'gradebook', 'results.publish'),
('gradebook.report_cards.generate', 'gradebook', 'report_cards.generate'),
('gradebook.report_cards.view', 'gradebook', 'report_cards.view'),
('gradebook.grading_schemes.manage', 'gradebook', 'grading_schemes.manage');

-- Fees
INSERT INTO permissions (code, module_name, action_name) VALUES
('fees.categories.manage', 'fees', 'categories.manage'),
('fees.structures.manage', 'fees', 'structures.manage'),
('fees.assignments.manage', 'fees', 'assignments.manage'),
('fees.invoices.generate', 'fees', 'invoices.generate'),
('fees.invoices.view', 'fees', 'invoices.view'),
('fees.invoices.cancel', 'fees', 'invoices.cancel'),
('fees.payments.record', 'fees', 'payments.record'),
('fees.payments.view', 'fees', 'payments.view'),
('fees.receipts.view', 'fees', 'receipts.view'),
('fees.receipts.download', 'fees', 'receipts.download'),
('fees.refunds.manage', 'fees', 'refunds.manage'),
('fees.reconciliation.manage', 'fees', 'reconciliation.manage'),
('fees.reports.view', 'fees', 'reports.view');

-- Communication
INSERT INTO permissions (code, module_name, action_name) VALUES
('communication.notices.create', 'communication', 'notices.create'),
('communication.notices.send', 'communication', 'notices.send'),
('communication.templates.manage', 'communication', 'templates.manage'),
('communication.delivery.view', 'communication', 'delivery.view'),
('communication.channels.configure', 'communication', 'channels.configure');

-- Portal
INSERT INTO permissions (code, module_name, action_name) VALUES
('portal.configure', 'portal', 'configure'),
('portal.analytics.view', 'portal', 'analytics.view'),
('portal.support.manage', 'portal', 'support.manage');
```

---

### Subscription Plans

```sql
INSERT INTO subscription_plans (code, name, max_students, max_staff, monthly_price, annual_price) VALUES
('STARTER', 'Starter', 200, 30, 2999.00, 29999.00),
('GROWTH', 'Growth', 500, 75, 5999.00, 59999.00),
('PRO', 'Professional', 2000, 250, 14999.00, 149999.00),
('ENTERPRISE', 'Enterprise', NULL, NULL, 29999.00, 299999.00);
```

---

## Tenant-Level Seed Data

This data is seeded for each tenant immediately after tenant creation.

### System Roles

```sql
-- These are created for each new tenant with is_system_role = TRUE
-- They reference the fixed role_type enum values

INSERT INTO roles (tenant_id, name, role_type, is_system_role) VALUES
(:tenant_id, 'School Admin', 'school_admin', TRUE),
(:tenant_id, 'Principal', 'principal', TRUE),
(:tenant_id, 'Teacher', 'teacher', TRUE),
(:tenant_id, 'Accountant', 'accountant', TRUE),
(:tenant_id, 'Parent', 'parent', TRUE),
(:tenant_id, 'Student', 'student', TRUE),
(:tenant_id, 'Transport Manager', 'transport_manager', TRUE),
(:tenant_id, 'Librarian', 'librarian', TRUE);
```

### Role-Permission Assignments

Default permission assignments per role (representative — full matrix in code):

```sql
-- School Admin: full access to most modules
-- (school.admin_role_id resolved after role creation above)

-- School Admin permissions (abbreviated)
INSERT INTO role_permissions (role_id, permission_id)
SELECT r.id, p.id FROM roles r, permissions p
WHERE r.tenant_id = :tenant_id
  AND r.role_type = 'school_admin'
  AND p.code IN (
    'users.invite', 'users.view', 'users.edit', 'users.disable',
    'users.roles.assign', 'users.parent_links.manage',
    'school.profile.view', 'school.profile.edit',
    'school.academic_year.manage', 'school.grades.manage',
    'school.sections.manage', 'school.campuses.manage', 'school.branding.manage',
    'students.view', 'students.create', 'students.edit', 'students.enroll',
    'students.transfer', 'students.withdraw', 'students.documents.manage',
    'students.health.view', 'students.health.manage',
    'attendance.view', 'attendance.mark', 'attendance.submit', 'attendance.lock',
    'attendance.correction.request', 'attendance.correction.approve', 'attendance.configure',
    'attendance.reports.view',
    'timetable.subjects.manage', 'timetable.rooms.manage', 'timetable.create',
    'timetable.review', 'timetable.publish', 'timetable.view',
    'gradebook.assessments.manage', 'gradebook.marks.verify', 'gradebook.results.publish',
    'gradebook.report_cards.generate', 'gradebook.report_cards.view',
    'fees.categories.manage', 'fees.structures.manage', 'fees.assignments.manage',
    'fees.invoices.generate', 'fees.invoices.view', 'fees.invoices.cancel',
    'fees.payments.record', 'fees.payments.view', 'fees.receipts.view',
    'fees.refunds.manage', 'fees.reconciliation.manage', 'fees.reports.view',
    'communication.notices.create', 'communication.notices.send',
    'communication.templates.manage', 'communication.delivery.view',
    'communication.channels.configure',
    'portal.configure', 'portal.analytics.view', 'portal.support.manage'
  );

-- Teacher permissions (abbreviated)
INSERT INTO role_permissions (role_id, permission_id)
SELECT r.id, p.id FROM roles r, permissions p
WHERE r.tenant_id = :tenant_id
  AND r.role_type = 'teacher'
  AND p.code IN (
    'students.view',
    'attendance.view', 'attendance.mark', 'attendance.submit',
    'attendance.correction.request', 'attendance.reports.view',
    'timetable.view',
    'gradebook.marks.enter', 'gradebook.report_cards.view'
  );

-- Accountant permissions (abbreviated)
INSERT INTO role_permissions (role_id, permission_id)
SELECT r.id, p.id FROM roles r, permissions p
WHERE r.tenant_id = :tenant_id
  AND r.role_type = 'accountant'
  AND p.code IN (
    'students.view',
    'fees.structures.manage', 'fees.assignments.manage',
    'fees.invoices.generate', 'fees.invoices.view', 'fees.invoices.cancel',
    'fees.payments.record', 'fees.payments.view', 'fees.receipts.view',
    'fees.receipts.download', 'fees.refunds.manage',
    'fees.reconciliation.manage', 'fees.reports.view',
    'communication.notices.create'
  );
```

---

### Onboarding Checklist Steps

```sql
INSERT INTO onboarding_checklists (tenant_id, school_id, step_code, step_label, step_order, status)
VALUES
(:tenant_id, :school_id, 'school_profile', 'Configure School Profile', 1, 'not_started'),
(:tenant_id, :school_id, 'academic_year', 'Create Academic Year', 2, 'not_started'),
(:tenant_id, :school_id, 'terms', 'Define Terms or Semesters', 3, 'not_started'),
(:tenant_id, :school_id, 'grades', 'Configure Grade Levels', 4, 'not_started'),
(:tenant_id, :school_id, 'sections', 'Create Sections', 5, 'not_started'),
(:tenant_id, :school_id, 'campuses', 'Configure Campuses', 6, 'not_started'),
(:tenant_id, :school_id, 'branding', 'Upload School Branding', 7, 'not_started'),
(:tenant_id, :school_id, 'fee_categories', 'Configure Fee Categories', 8, 'not_started'),
(:tenant_id, :school_id, 'communication_settings', 'Configure Communication Settings', 9, 'not_started'),
(:tenant_id, :school_id, 'staff_invitation', 'Invite School Admin and Teachers', 10, 'not_started'),
(:tenant_id, :school_id, 'readiness_check', 'Run Readiness Validation', 11, 'not_started'),
(:tenant_id, :school_id, 'go_live', 'Go Live', 12, 'not_started');
```

---

### Default School Settings

```sql
INSERT INTO school_settings (tenant_id, school_id,
    language_code, currency_code, date_format, time_format_24h,
    attendance_enabled, gradebook_enabled, fees_enabled,
    timetable_enabled, parent_portal_enabled, student_portal_enabled,
    communication_enabled)
VALUES
(:tenant_id, :school_id,
    'en', 'INR', 'DD-MM-YYYY', TRUE,
    TRUE, TRUE, TRUE,
    TRUE, TRUE, FALSE,
    TRUE);
```

---

### Default Attendance Statuses

```sql
INSERT INTO attendance_statuses
    (tenant_id, code, name, counts_as_present, counts_as_absent, is_default, sort_order)
VALUES
(:tenant_id, 'PRESENT', 'Present', TRUE, FALSE, TRUE, 1),
(:tenant_id, 'ABSENT', 'Absent', FALSE, TRUE, FALSE, 2),
(:tenant_id, 'LATE', 'Late', FALSE, FALSE, FALSE, 3),
(:tenant_id, 'EXCUSED', 'Excused', FALSE, FALSE, FALSE, 4),
(:tenant_id, 'HALF_DAY', 'Half Day', FALSE, FALSE, FALSE, 5);
```

---

### Default Fee Categories

```sql
INSERT INTO fee_categories (tenant_id, school_id, code, name, category_type, optional_flag)
VALUES
(:tenant_id, :school_id, 'TUITION', 'Tuition Fee', 'tuition', FALSE),
(:tenant_id, :school_id, 'EXAM', 'Examination Fee', 'exam', FALSE),
(:tenant_id, :school_id, 'BOOKS', 'Books and Stationery', 'books', TRUE),
(:tenant_id, :school_id, 'ACTIVITY', 'Activity Fee', 'activity', TRUE),
(:tenant_id, :school_id, 'TRANSPORT', 'Transport Fee', 'transport', TRUE),
(:tenant_id, :school_id, 'LAB', 'Laboratory Fee', 'lab', TRUE);
```

---

### Default Student Document Categories

```sql
INSERT INTO student_document_categories (tenant_id, code, name, requires_verification)
VALUES
(:tenant_id, 'BIRTH_CERT', 'Birth Certificate', TRUE),
(:tenant_id, 'TRANSFER_CERT', 'Transfer Certificate', TRUE),
(:tenant_id, 'PASSPORT', 'Passport', FALSE),
(:tenant_id, 'AADHAAR', 'Aadhaar Card', FALSE),
(:tenant_id, 'MEDICAL', 'Medical Records', FALSE),
(:tenant_id, 'PHOTO', 'Student Photograph', FALSE),
(:tenant_id, 'CASTE_CERT', 'Caste Certificate', FALSE),
(:tenant_id, 'INCOME_CERT', 'Income Certificate', FALSE);
```

