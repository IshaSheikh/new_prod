# RLS Policies

PostgreSQL Row-Level Security (RLS) policies for all tenant-owned tables. RLS is the second line of defense after application-layer tenant enforcement.

---

## How It Works

Before executing any tenant-scoped query, the application sets a session-level variable:

```sql
-- Set within each transaction/request context
SET LOCAL app.tenant_id = 'tenant-uuid-here';
```

Every RLS policy uses `current_setting('app.tenant_id', true)::uuid` to filter records. The `true` argument means the function returns NULL (rather than throwing an error) if the variable is not set — this allows super-admin operations without a tenant context.

---

## Super Admin Bypass

The application service role used by super-admin operations bypasses RLS entirely using `SET ROLE` or a dedicated service account. School-level service accounts have RLS enforced at all times.

```sql
-- Service role used by business module APIs (RLS enforced)
CREATE ROLE app_school_user;
-- Service role used by platform operations (RLS bypassed)
CREATE ROLE app_platform_admin BYPASSRLS;
```

---

## Platform Core Policies

```sql
-- tenants: accessible to the tenant itself and super admins
ALTER TABLE tenants ENABLE ROW LEVEL SECURITY;
CREATE POLICY tenants_isolation ON tenants
  FOR ALL
  USING (
    id = current_setting('app.tenant_id', true)::uuid
    OR current_setting('app.tenant_id', true) IS NULL
  );

-- tenant_domains: tenant scoped
ALTER TABLE tenant_domains ENABLE ROW LEVEL SECURITY;
CREATE POLICY tenant_domains_isolation ON tenant_domains
  FOR ALL
  USING (tenant_id = current_setting('app.tenant_id', true)::uuid);

-- user_tenant_memberships: tenant scoped
ALTER TABLE user_tenant_memberships ENABLE ROW LEVEL SECURITY;
CREATE POLICY memberships_isolation ON user_tenant_memberships
  FOR ALL
  USING (tenant_id = current_setting('app.tenant_id', true)::uuid);

-- membership_roles: tenant scoped
ALTER TABLE membership_roles ENABLE ROW LEVEL SECURITY;
CREATE POLICY membership_roles_isolation ON membership_roles
  FOR ALL
  USING (tenant_id = current_setting('app.tenant_id', true)::uuid);

-- subscriptions: tenant scoped
ALTER TABLE subscriptions ENABLE ROW LEVEL SECURITY;
CREATE POLICY subscriptions_isolation ON subscriptions
  FOR ALL
  USING (tenant_id = current_setting('app.tenant_id', true)::uuid);

-- tenant_invitations: tenant scoped
ALTER TABLE tenant_invitations ENABLE ROW LEVEL SECURITY;
CREATE POLICY invitations_isolation ON tenant_invitations
  FOR ALL
  USING (tenant_id = current_setting('app.tenant_id', true)::uuid);

-- auth_sessions: user + tenant scoped
ALTER TABLE auth_sessions ENABLE ROW LEVEL SECURITY;
CREATE POLICY sessions_isolation ON auth_sessions
  FOR ALL
  USING (
    tenant_id = current_setting('app.tenant_id', true)::uuid
    OR tenant_id IS NULL  -- platform-level sessions
  );

-- audit_logs: tenant scoped (append-only via REVOKE UPDATE DELETE)
ALTER TABLE audit_logs ENABLE ROW LEVEL SECURITY;
CREATE POLICY audit_logs_isolation ON audit_logs
  FOR ALL
  USING (tenant_id = current_setting('app.tenant_id', true)::uuid);

REVOKE UPDATE, DELETE ON audit_logs FROM app_school_user;
```

---

## School Onboarding Policies

```sql
-- Standard pattern applied to all onboarding tables
ALTER TABLE schools ENABLE ROW LEVEL SECURITY;
CREATE POLICY schools_isolation ON schools FOR ALL
  USING (tenant_id = current_setting('app.tenant_id', true)::uuid);

ALTER TABLE school_profiles ENABLE ROW LEVEL SECURITY;
CREATE POLICY school_profiles_isolation ON school_profiles FOR ALL
  USING (tenant_id = current_setting('app.tenant_id', true)::uuid);

ALTER TABLE academic_years ENABLE ROW LEVEL SECURITY;
CREATE POLICY academic_years_isolation ON academic_years FOR ALL
  USING (tenant_id = current_setting('app.tenant_id', true)::uuid);

ALTER TABLE terms ENABLE ROW LEVEL SECURITY;
CREATE POLICY terms_isolation ON terms FOR ALL
  USING (tenant_id = current_setting('app.tenant_id', true)::uuid);

ALTER TABLE grade_levels ENABLE ROW LEVEL SECURITY;
CREATE POLICY grade_levels_isolation ON grade_levels FOR ALL
  USING (tenant_id = current_setting('app.tenant_id', true)::uuid);

ALTER TABLE sections ENABLE ROW LEVEL SECURITY;
CREATE POLICY sections_isolation ON sections FOR ALL
  USING (tenant_id = current_setting('app.tenant_id', true)::uuid);

ALTER TABLE campuses ENABLE ROW LEVEL SECURITY;
CREATE POLICY campuses_isolation ON campuses FOR ALL
  USING (tenant_id = current_setting('app.tenant_id', true)::uuid);

ALTER TABLE branding_assets ENABLE ROW LEVEL SECURITY;
CREATE POLICY branding_assets_isolation ON branding_assets FOR ALL
  USING (tenant_id = current_setting('app.tenant_id', true)::uuid);
```

---

## Student Information System Policies

```sql
ALTER TABLE students ENABLE ROW LEVEL SECURITY;
CREATE POLICY students_isolation ON students FOR ALL
  USING (tenant_id = current_setting('app.tenant_id', true)::uuid);

ALTER TABLE student_profiles ENABLE ROW LEVEL SECURITY;
CREATE POLICY student_profiles_isolation ON student_profiles FOR ALL
  USING (tenant_id = current_setting('app.tenant_id', true)::uuid);

ALTER TABLE student_enrollments ENABLE ROW LEVEL SECURITY;
CREATE POLICY student_enrollments_isolation ON student_enrollments FOR ALL
  USING (tenant_id = current_setting('app.tenant_id', true)::uuid);

ALTER TABLE guardians ENABLE ROW LEVEL SECURITY;
CREATE POLICY guardians_isolation ON guardians FOR ALL
  USING (tenant_id = current_setting('app.tenant_id', true)::uuid);

ALTER TABLE guardian_student_links ENABLE ROW LEVEL SECURITY;
CREATE POLICY guardian_student_links_isolation ON guardian_student_links FOR ALL
  USING (tenant_id = current_setting('app.tenant_id', true)::uuid);

-- Health notes: especially sensitive — enforce RLS + role-level filtering in application
ALTER TABLE student_health_notes ENABLE ROW LEVEL SECURITY;
CREATE POLICY student_health_notes_isolation ON student_health_notes FOR ALL
  USING (tenant_id = current_setting('app.tenant_id', true)::uuid);

-- Status history: append-only
ALTER TABLE student_status_history ENABLE ROW LEVEL SECURITY;
CREATE POLICY student_status_history_isolation ON student_status_history FOR ALL
  USING (tenant_id = current_setting('app.tenant_id', true)::uuid);

REVOKE UPDATE, DELETE ON student_status_history FROM app_school_user;
```

---

## Timetable Policies

```sql
ALTER TABLE subjects ENABLE ROW LEVEL SECURITY;
CREATE POLICY subjects_isolation ON subjects FOR ALL
  USING (tenant_id = current_setting('app.tenant_id', true)::uuid);

ALTER TABLE timetables ENABLE ROW LEVEL SECURITY;
CREATE POLICY timetables_isolation ON timetables FOR ALL
  USING (tenant_id = current_setting('app.tenant_id', true)::uuid);

ALTER TABLE timetable_slots ENABLE ROW LEVEL SECURITY;
CREATE POLICY timetable_slots_isolation ON timetable_slots FOR ALL
  USING (tenant_id = current_setting('app.tenant_id', true)::uuid);

-- Published timetable slots visible to portal users (students/parents)
-- Application enforces: only status='published' timetables accessible via portal
```

---

## Attendance Policies

```sql
ALTER TABLE attendance_sessions ENABLE ROW LEVEL SECURITY;
CREATE POLICY attendance_sessions_isolation ON attendance_sessions FOR ALL
  USING (tenant_id = current_setting('app.tenant_id', true)::uuid);

ALTER TABLE attendance_records ENABLE ROW LEVEL SECURITY;
CREATE POLICY attendance_records_isolation ON attendance_records FOR ALL
  USING (tenant_id = current_setting('app.tenant_id', true)::uuid);

-- Corrections are append-only after creation
ALTER TABLE attendance_corrections ENABLE ROW LEVEL SECURITY;
CREATE POLICY attendance_corrections_isolation ON attendance_corrections FOR ALL
  USING (tenant_id = current_setting('app.tenant_id', true)::uuid);

-- Audit logs are immutable
ALTER TABLE attendance_audit_logs ENABLE ROW LEVEL SECURITY;
CREATE POLICY attendance_audit_logs_isolation ON attendance_audit_logs FOR ALL
  USING (tenant_id = current_setting('app.tenant_id', true)::uuid);

REVOKE UPDATE, DELETE ON attendance_audit_logs FROM app_school_user;
```

---

## Finance Policies

```sql
ALTER TABLE invoices ENABLE ROW LEVEL SECURITY;
CREATE POLICY invoices_isolation ON invoices FOR ALL
  USING (tenant_id = current_setting('app.tenant_id', true)::uuid);

ALTER TABLE payment_transactions ENABLE ROW LEVEL SECURITY;
CREATE POLICY payment_transactions_isolation ON payment_transactions FOR ALL
  USING (tenant_id = current_setting('app.tenant_id', true)::uuid);

-- Receipts are immutable after issuance
ALTER TABLE receipts ENABLE ROW LEVEL SECURITY;
CREATE POLICY receipts_isolation ON receipts FOR ALL
  USING (tenant_id = current_setting('app.tenant_id', true)::uuid);

REVOKE UPDATE, DELETE ON receipts FROM app_school_user;

-- Ledger entries are immutable
ALTER TABLE ledger_entries ENABLE ROW LEVEL SECURITY;
CREATE POLICY ledger_entries_isolation ON ledger_entries FOR ALL
  USING (tenant_id = current_setting('app.tenant_id', true)::uuid);

REVOKE UPDATE, DELETE ON ledger_entries FROM app_school_user;

ALTER TABLE refunds ENABLE ROW LEVEL SECURITY;
CREATE POLICY refunds_isolation ON refunds FOR ALL
  USING (tenant_id = current_setting('app.tenant_id', true)::uuid);

ALTER TABLE finance_audit_logs ENABLE ROW LEVEL SECURITY;
CREATE POLICY finance_audit_logs_isolation ON finance_audit_logs FOR ALL
  USING (tenant_id = current_setting('app.tenant_id', true)::uuid);

REVOKE UPDATE, DELETE ON finance_audit_logs FROM app_school_user;
```

---

## Gradebook Policies

```sql
ALTER TABLE assessments ENABLE ROW LEVEL SECURITY;
CREATE POLICY assessments_isolation ON assessments FOR ALL
  USING (tenant_id = current_setting('app.tenant_id', true)::uuid);

ALTER TABLE marks_entries ENABLE ROW LEVEL SECURITY;
CREATE POLICY marks_entries_isolation ON marks_entries FOR ALL
  USING (tenant_id = current_setting('app.tenant_id', true)::uuid);

-- Report cards are immutable after publication
ALTER TABLE report_cards ENABLE ROW LEVEL SECURITY;
CREATE POLICY report_cards_isolation ON report_cards FOR ALL
  USING (tenant_id = current_setting('app.tenant_id', true)::uuid);

-- Mark audit logs are immutable
ALTER TABLE mark_change_audit_logs ENABLE ROW LEVEL SECURITY;
CREATE POLICY mark_change_audit_logs_isolation ON mark_change_audit_logs FOR ALL
  USING (tenant_id = current_setting('app.tenant_id', true)::uuid);

REVOKE UPDATE, DELETE ON mark_change_audit_logs FROM app_school_user;
```

---

## Portal Policies

```sql
ALTER TABLE portal_users ENABLE ROW LEVEL SECURITY;
CREATE POLICY portal_users_isolation ON portal_users FOR ALL
  USING (tenant_id = current_setting('app.tenant_id', true)::uuid);

ALTER TABLE portal_sessions ENABLE ROW LEVEL SECURITY;
CREATE POLICY portal_sessions_isolation ON portal_sessions FOR ALL
  USING (tenant_id = current_setting('app.tenant_id', true)::uuid);

ALTER TABLE portal_notices ENABLE ROW LEVEL SECURITY;
CREATE POLICY portal_notices_isolation ON portal_notices FOR ALL
  USING (tenant_id = current_setting('app.tenant_id', true)::uuid);

ALTER TABLE portal_audit_logs ENABLE ROW LEVEL SECURITY;
CREATE POLICY portal_audit_logs_isolation ON portal_audit_logs FOR ALL
  USING (tenant_id = current_setting('app.tenant_id', true)::uuid);

REVOKE UPDATE, DELETE ON portal_audit_logs FROM app_school_user;
```

---

## Global Tables (No RLS)

These tables are global and should not have tenant-scoped RLS. Access control is managed at the application layer:

- `users` — global identity; access controlled through membership queries
- `permissions` — global permission catalog; read-only for all authenticated users
- `roles` — global role definitions (tenant_id IS NULL for system roles); filtered by tenant in application
- `subscription_plans` — global catalog; read-only
- `platform_features` — global catalog; read-only

---

## Verification Checklist

Before adding a new table to the schema, verify:

- [ ] `tenant_id UUID NOT NULL` column exists
- [ ] `ALTER TABLE ... ENABLE ROW LEVEL SECURITY;` executed
- [ ] `CREATE POLICY ... USING (tenant_id = current_setting('app.tenant_id', true)::uuid)` created
- [ ] `CREATE INDEX idx_{table}_tenant ON {table} (tenant_id)` created
- [ ] If the table is audit/immutable: `REVOKE UPDATE, DELETE ON {table} FROM app_school_user`
