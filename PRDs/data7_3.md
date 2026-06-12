<img src="https://r2cdn.perplexity.ai/pplx-full-logo-primary-dark%402x.png" style="height:64px;margin-right:32px"/>

# now pleaase generate the M7_3 data architecture

For M7_3, the right architecture is a **global identity + tenant-scoped authorization** model, because the PRD defines users as platform identities, requires globally unique emails, allows multi-school membership, and says every user action must execute within a tenant context.[^1]
This module should therefore own invitations, role assignment, parent-student links, teacher scope, secure authentication artifacts, and immutable access audits, while keeping `users` and `permissions` global.[^1]

## Entity list

These are the core entities for Module 7.3, based on the PRD’s required data entities and the access-control behaviors it describes.[^1]

- `users` — global platform identity with lifecycle, password hash, and last-login state.[^1]
- `user_tenant_memberships` — tenant-specific access record for a user, because users may belong to multiple schools.[^1]
- `roles` — fixed V1 system roles such as School Admin, Principal, Teacher, Accountant, Parent, Student, Transport Manager, and Librarian.[^1]
- `permissions` — global permission catalog by module and action.[^1]
- `role_permissions` — role-to-permission inheritance matrix.[^1]
- `membership_roles` — role assignment at membership level, because permissions are inherited through roles assigned to memberships.[^1]
- `invitations` and `invitation_roles` — tenant-scoped invitation workflow with expiry and optional multi-role assignment at acceptance time.[^1]
- `password_reset_tokens` — reset workflow with 30-minute token expiry and “latest token wins” behavior.[^1]
- `user_sessions` — active session tracking so user deactivation or role changes can invalidate or refresh access immediately.[^1]
- `parent_student_links` — guardian linkage with relationship status and strict linked-child visibility.[^1]
- `teacher_assignments` — teacher-to-class/section/subject/academic-year scoping.[^1]
- `access_audit_logs` — immutable audit log for permission changes, access changes, and security activity.[^1]


## Table definitions

The schema below implements the PRD requirements for secure login, invitation-based provisioning, multi-role memberships, linked-child access, assigned-class visibility, and auditability, while avoiding direct user-level permission overrides that the MVP explicitly excludes.[^1]

```sql
CREATE EXTENSION IF NOT EXISTS pgcrypto;
CREATE EXTENSION IF NOT EXISTS citext;

CREATE TYPE user_account_status AS ENUM (
    'pending_acceptance',
    'active',
    'suspended',
    'disabled',
    'removed'
);

CREATE TYPE membership_status AS ENUM (
    'pending',
    'active',
    'inactive',
    'removed'
);

CREATE TYPE role_type AS ENUM (
    'super_admin',
    'school_admin',
    'principal',
    'teacher',
    'accountant',
    'parent',
    'student',
    'transport_manager',
    'librarian'
);

CREATE TYPE invitation_status AS ENUM (
    'pending',
    'accepted',
    'expired',
    'revoked'
);

CREATE TYPE password_reset_status AS ENUM (
    'issued',
    'used',
    'expired',
    'revoked'
);

CREATE TYPE parent_student_link_status AS ENUM (
    'requested',
    'verified',
    'active',
    'revoked'
);

CREATE TYPE parent_relationship_type AS ENUM (
    'father',
    'mother',
    'guardian',
    'legal_guardian',
    'other'
);

CREATE TYPE teacher_assignment_status AS ENUM (
    'active',
    'ended',
    'revoked'
);

CREATE TYPE session_status AS ENUM (
    'active',
    'revoked',
    'expired'
);

-- Global identity
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email CITEXT NOT NULL UNIQUE,
    mobile_e164 VARCHAR(20),
    first_name VARCHAR(100) NOT NULL,
    last_name VARCHAR(100),
    password_hash TEXT NOT NULL,
    status user_account_status NOT NULL DEFAULT 'pending_acceptance',
    is_platform_super_admin BOOLEAN NOT NULL DEFAULT FALSE,
    last_login_at TIMESTAMPTZ,
    disabled_at TIMESTAMPTZ,
    removed_at TIMESTAMPTZ,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID NULL REFERENCES users(id),
    updated_by UUID NULL REFERENCES users(id)
);

-- Memberships
CREATE TABLE user_tenant_memberships (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
    user_id UUID NOT NULL REFERENCES users(id) ON DELETE RESTRICT,
    status membership_status NOT NULL DEFAULT 'pending',
    joined_at TIMESTAMPTZ,
    invited_at TIMESTAMPTZ,
    accepted_at TIMESTAMPTZ,
    deactivated_at TIMESTAMPTZ,
    removed_at TIMESTAMPTZ,
    version_no INTEGER NOT NULL DEFAULT 1,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID NULL REFERENCES users(id),
    updated_by UUID NULL REFERENCES users(id),
    CONSTRAINT uq_membership_user_tenant UNIQUE (tenant_id, user_id),
    CONSTRAINT uq_membership_id_tenant UNIQUE (id, tenant_id)
);

-- Roles remain global in V1, tenant_id nullable for Phase 2 extensibility
CREATE TABLE roles (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NULL REFERENCES tenants(id) ON DELETE CASCADE,
    name VARCHAR(100) NOT NULL,
    role_type role_type NOT NULL,
    is_system_role BOOLEAN NOT NULL DEFAULT TRUE,
    is_assignable BOOLEAN NOT NULL DEFAULT TRUE,
    description TEXT,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID NULL REFERENCES users(id),
    updated_by UUID NULL REFERENCES users(id)
);

CREATE TABLE permissions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    code VARCHAR(120) NOT NULL UNIQUE,
    module_name VARCHAR(100) NOT NULL,
    action_name VARCHAR(100) NOT NULL,
    description TEXT,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE TABLE role_permissions (
    role_id UUID NOT NULL REFERENCES roles(id) ON DELETE CASCADE,
    permission_id UUID NOT NULL REFERENCES permissions(id) ON DELETE CASCADE,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID NULL REFERENCES users(id),
    PRIMARY KEY (role_id, permission_id)
);

CREATE TABLE membership_roles (
    tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
    membership_id UUID NOT NULL,
    role_id UUID NOT NULL REFERENCES roles(id) ON DELETE RESTRICT,
    assigned_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    revoked_at TIMESTAMPTZ,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID NULL REFERENCES users(id),
    updated_by UUID NULL REFERENCES users(id),
    PRIMARY KEY (tenant_id, membership_id, role_id),
    FOREIGN KEY (membership_id, tenant_id)
        REFERENCES user_tenant_memberships(id, tenant_id)
        ON DELETE CASCADE
);

CREATE TABLE invitations (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
    email CITEXT NOT NULL,
    user_id UUID NULL REFERENCES users(id) ON DELETE SET NULL,
    token_hash TEXT NOT NULL,
    expires_at TIMESTAMPTZ NOT NULL,
    accepted_at TIMESTAMPTZ,
    revoked_at TIMESTAMPTZ,
    status invitation_status NOT NULL DEFAULT 'pending',
    invited_membership_id UUID NULL REFERENCES user_tenant_memberships(id),
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID NOT NULL REFERENCES users(id),
    updated_by UUID NULL REFERENCES users(id)
);

CREATE TABLE invitation_roles (
    tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
    invitation_id UUID NOT NULL REFERENCES invitations(id) ON DELETE CASCADE,
    role_id UUID NOT NULL REFERENCES roles(id) ON DELETE RESTRICT,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID NULL REFERENCES users(id),
    PRIMARY KEY (tenant_id, invitation_id, role_id)
);

-- Password reset is global because users are global identities
CREATE TABLE password_reset_tokens (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    token_hash TEXT NOT NULL,
    expires_at TIMESTAMPTZ NOT NULL,
    used_at TIMESTAMPTZ,
    revoked_at TIMESTAMPTZ,
    status password_reset_status NOT NULL DEFAULT 'issued',
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID NULL REFERENCES users(id)
);

-- Tenant-context sessions for permission refresh and forced logout
CREATE TABLE user_sessions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NULL REFERENCES tenants(id) ON DELETE CASCADE,
    membership_id UUID NULL REFERENCES user_tenant_memberships(id) ON DELETE CASCADE,
    user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    refresh_token_hash TEXT NOT NULL,
    status session_status NOT NULL DEFAULT 'active',
    ip_address INET,
    user_agent TEXT,
    last_seen_at TIMESTAMPTZ,
    expires_at TIMESTAMPTZ NOT NULL,
    revoked_at TIMESTAMPTZ,
    revoked_reason TEXT,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

-- Parent ↔ student relationship model
CREATE TABLE parent_student_links (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
    parent_membership_id UUID NOT NULL,
    parent_user_id UUID NOT NULL REFERENCES users(id) ON DELETE RESTRICT,
    student_membership_id UUID NOT NULL,
    student_user_id UUID NOT NULL REFERENCES users(id) ON DELETE RESTRICT,
    relationship_type parent_relationship_type NOT NULL,
    status parent_student_link_status NOT NULL DEFAULT 'requested',
    verified_at TIMESTAMPTZ,
    revoked_at TIMESTAMPTZ,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID NULL REFERENCES users(id),
    updated_by UUID NULL REFERENCES users(id),
    FOREIGN KEY (parent_membership_id, tenant_id)
        REFERENCES user_tenant_memberships(id, tenant_id)
        ON DELETE CASCADE,
    FOREIGN KEY (student_membership_id, tenant_id)
        REFERENCES user_tenant_memberships(id, tenant_id)
        ON DELETE CASCADE,
    CONSTRAINT uq_parent_student_link UNIQUE (tenant_id, parent_user_id, student_user_id)
);

-- Teacher scope
CREATE TABLE teacher_assignments (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
    teacher_membership_id UUID NOT NULL,
    teacher_user_id UUID NOT NULL REFERENCES users(id) ON DELETE RESTRICT,
    academic_year_id UUID NOT NULL REFERENCES academic_years(id) ON DELETE RESTRICT,
    grade_level_id UUID NOT NULL REFERENCES grade_levels(id) ON DELETE RESTRICT,
    section_id UUID NOT NULL REFERENCES sections(id) ON DELETE RESTRICT,
    subject_id UUID NOT NULL,
    campus_id UUID NULL REFERENCES campuses(id) ON DELETE RESTRICT,
    status teacher_assignment_status NOT NULL DEFAULT 'active',
    starts_on DATE NOT NULL DEFAULT CURRENT_DATE,
    ends_on DATE,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID NULL REFERENCES users(id),
    updated_by UUID NULL REFERENCES users(id),
    FOREIGN KEY (teacher_membership_id, tenant_id)
        REFERENCES user_tenant_memberships(id, tenant_id)
        ON DELETE CASCADE,
    CONSTRAINT ck_teacher_assignment_dates CHECK (ends_on IS NULL OR ends_on >= starts_on)
);

-- Immutable access/security audit
CREATE TABLE access_audit_logs (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE RESTRICT,
    actor_user_id UUID NULL REFERENCES users(id) ON DELETE SET NULL,
    actor_membership_id UUID NULL REFERENCES user_tenant_memberships(id) ON DELETE SET NULL,
    action VARCHAR(120) NOT NULL,
    target_entity VARCHAR(120) NOT NULL,
    target_id UUID NOT NULL,
    metadata JSONB,
    ip_address INET,
    user_agent TEXT,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

The DDL above keeps `users`, `roles`, `permissions`, and password-reset artifacts aligned to platform identity, while scoping memberships, invitation handling, parent links, teacher assignments, sessions, and access audits to a tenant context as the PRD requires.[^1]

## Constraints and indexes

The key enforcement points for M7_3 are globally unique identity, membership-scoped roles, linked-child-only parent access, assigned-class-only teacher access, expiry-controlled tokens, and immediate session revocation when security state changes.[^1]

**Constraints**

- `users.email` is globally unique, because the PRD explicitly requires global email uniqueness for platform identities.[^1]
- `user_tenant_memberships (tenant_id, user_id)` is unique, because the same user should not have duplicate memberships in one tenant.[^1]
- `membership_roles` allows multiple roles per membership, because the PRD explicitly supports combinations such as Teacher + Librarian.[^1]
- `parent_student_links` is unique on `(tenant_id, parent_user_id, student_user_id)` so duplicate guardian linkage is blocked.[^1]
- `invitations.expires_at` and `password_reset_tokens.expires_at` are mandatory, because invitation links expire after 7 days and reset tokens expire after 30 minutes.[^1]
- `teacher_assignments` should preserve history through status and end date rather than hard delete, because historical academic actions must remain intact even if assignments change later.[^1]

**Recommended trigger rules**

- Add a deferred trigger to ensure every active membership has at least one non-revoked role, because the PRD requires every membership to have at least one role.[^1]
- Add a trigger to block disabling the last active School Admin in a tenant, because the PRD explicitly calls out that edge case.[^1]
- Add a trigger to revoke all active `user_sessions` when a user is disabled or a membership becomes inactive, because deactivation must immediately invalidate active sessions.[^1]
- Add a trigger to expire older password-reset tokens when a new one is issued, because only the latest token should remain valid.[^1]
- Add an audit trigger or service write-path guarantee so role changes, access changes, and security events always create immutable `access_audit_logs` entries.[^1]

```sql
CREATE UNIQUE INDEX uq_roles_global_system
ON roles (role_type)
WHERE tenant_id IS NULL AND is_system_role = TRUE;

CREATE UNIQUE INDEX uq_roles_tenant_name
ON roles (tenant_id, name)
WHERE tenant_id IS NOT NULL;

CREATE UNIQUE INDEX uq_mobile_when_present
ON users (mobile_e164)
WHERE mobile_e164 IS NOT NULL;

CREATE UNIQUE INDEX uq_active_invitation_per_tenant_email
ON invitations (tenant_id, email)
WHERE status = 'pending';

CREATE INDEX idx_memberships_tenant_status
ON user_tenant_memberships (tenant_id, status, user_id);

CREATE INDEX idx_memberships_user_status
ON user_tenant_memberships (user_id, status, tenant_id);

CREATE INDEX idx_membership_roles_membership
ON membership_roles (tenant_id, membership_id, role_id);

CREATE INDEX idx_permissions_module_action
ON permissions (module_name, action_name);

CREATE INDEX idx_invitations_tenant_status_expiry
ON invitations (tenant_id, status, expires_at);

CREATE INDEX idx_password_reset_user_status_expiry
ON password_reset_tokens (user_id, status, expires_at DESC);

CREATE INDEX idx_sessions_user_tenant_status
ON user_sessions (user_id, tenant_id, status, expires_at);

CREATE INDEX idx_parent_links_parent
ON parent_student_links (tenant_id, parent_user_id, status);

CREATE INDEX idx_parent_links_student
ON parent_student_links (tenant_id, student_user_id, status);

CREATE INDEX idx_teacher_assignments_scope
ON teacher_assignments (tenant_id, teacher_user_id, academic_year_id, status);

CREATE INDEX idx_access_audit_tenant_created
ON access_audit_logs (tenant_id, created_at DESC);

CREATE INDEX idx_access_audit_actor_created
ON access_audit_logs (actor_user_id, created_at DESC);

CREATE INDEX idx_access_audit_target
ON access_audit_logs (tenant_id, target_entity, target_id, created_at DESC);
```

These indexes match the module’s dominant query patterns: user directory search, invitation management, role resolution, linked-child lookups, assigned-class lookups, session revocation, and access-audit review.[^1]

## Sample records and query patterns

These samples represent the main M7_3 flows: invite a school admin or teacher, activate membership, assign roles, link a parent to a student, and scope a teacher to one academic context.[^1]

```sql
INSERT INTO users (
    id, email, mobile_e164, first_name, last_name, password_hash, status
) VALUES
(
    '61000000-0000-0000-0000-000000000001',
    'teacher1@greenfield.edu',
    '+919876543210',
    'Meena',
    'Krishnan',
    'argon2_hash_teacher',
    'active'
),
(
    '61000000-0000-0000-0000-000000000002',
    'parent1@gmail.com',
    '+919812345678',
    'Lakshmi',
    'Narayan',
    'argon2_hash_parent',
    'active'
);

INSERT INTO user_tenant_memberships (
    id, tenant_id, user_id, status, joined_at, created_by
) VALUES
(
    '62000000-0000-0000-0000-000000000001',
    '10000000-0000-0000-0000-000000000001',
    '61000000-0000-0000-0000-000000000001',
    'active',
    now(),
    '00000000-0000-0000-0000-000000000002'
);

INSERT INTO membership_roles (
    tenant_id, membership_id, role_id, created_by
) VALUES
(
    '10000000-0000-0000-0000-000000000001',
    '62000000-0000-0000-0000-000000000001',
    '70000000-0000-0000-0000-000000000004',
    '00000000-0000-0000-0000-000000000002'
);
```

**Query patterns**

1. **Resolve effective permissions for a logged-in user inside one tenant**[^1]
```sql
SELECT DISTINCT p.code
FROM user_tenant_memberships m
JOIN membership_roles mr
  ON mr.membership_id = m.id AND mr.tenant_id = m.tenant_id
JOIN role_permissions rp
  ON rp.role_id = mr.role_id
JOIN permissions p
  ON p.id = rp.permission_id
WHERE m.user_id = $1
  AND m.tenant_id = $2
  AND m.status = 'active'
  AND mr.revoked_at IS NULL;
```

2. **List all children visible to a parent**[^1]
```sql
SELECT psl.student_user_id, u.first_name, u.last_name, psl.relationship_type
FROM parent_student_links psl
JOIN users u ON u.id = psl.student_user_id
WHERE psl.tenant_id = $1
  AND psl.parent_user_id = $2
  AND psl.status = 'active'
ORDER BY u.first_name, u.last_name;
```

3. **List the academic scope for a teacher dashboard**[^1]
```sql
SELECT ta.academic_year_id, ta.grade_level_id, ta.section_id, ta.subject_id, ta.campus_id
FROM teacher_assignments ta
WHERE ta.tenant_id = $1
  AND ta.teacher_user_id = $2
  AND ta.status = 'active'
  AND (ta.ends_on IS NULL OR ta.ends_on >= CURRENT_DATE);
```

4. **Show pending invitations for a tenant user-management screen**[^1]
```sql
SELECT i.id, i.email, i.expires_at, i.status,
       array_agg(r.name ORDER BY r.name) AS invited_roles
FROM invitations i
LEFT JOIN invitation_roles ir
  ON ir.invitation_id = i.id AND ir.tenant_id = i.tenant_id
LEFT JOIN roles r
  ON r.id = ir.role_id
WHERE i.tenant_id = $1
  AND i.status = 'pending'
GROUP BY i.id, i.email, i.expires_at, i.status
ORDER BY i.created_at DESC;
```

5. **Review access and permission changes for investigation**[^1]
```sql
SELECT created_at, actor_user_id, action, target_entity, target_id, metadata
FROM access_audit_logs
WHERE tenant_id = $1
  AND created_at >= $2
  AND created_at < $3
ORDER BY created_at DESC;
```


## Migration sequence and security-sensitive columns

The safest rollout order is to create global identity and permission tables first, then tenant-scoped access relationships, then auth/session artifacts, then relationship-scope tables, and finally immutable audit structures.[^1]

**Migration sequence**

1. Create enums for account status, membership status, roles, invitation state, password reset, parent links, teacher assignments, and sessions.[^1]
2. Create global tables: `users`, `roles`, `permissions`, and `role_permissions`.[^1]
3. Create `user_tenant_memberships` and `membership_roles`, because all tenant-scoped access depends on them.[^1]
4. Create `invitations`, `invitation_roles`, `password_reset_tokens`, and `user_sessions` for provisioning and secure authentication flows.[^1]
5. Create `parent_student_links` and `teacher_assignments` for relationship-based and scope-based access.[^1]
6. Create `access_audit_logs`, then add indexes, triggers, and RLS policies for all tenant-owned tables.[^1]
7. Seed the fixed V1 system roles and their permission mappings, because the PRD recommends shipping fixed roles in MVP and explicitly excludes direct user-level permission overrides.[^1]

**Security-sensitive columns**

- `users.email`, `users.mobile_e164`, `users.first_name`, and `users.last_name` are PII and should be masked in low-privilege admin views.[^1]
- `users.password_hash`, `invitations.token_hash`, `password_reset_tokens.token_hash`, and `user_sessions.refresh_token_hash` are secret-bearing fields and must store hashes only, never raw tokens.[^1]
- `parent_student_links` contains sensitive family relationships and must be readable only through tenant membership and role-aware scope filters.[^1]
- `teacher_assignments` reveals instructional scope and should be writable only by authorized admins or principals with assignment rights.[^1]
- `access_audit_logs.metadata`, `ip_address`, and `user_agent` are security telemetry and should be restricted to privileged support, security, and audit roles.[^1]

This M7_3 design gives you a production-ready RBAC foundation with strict tenant isolation, clean role inheritance, linked-child access control, teacher scope control, secure auth artifacts, and auditable access change history without introducing ABAC or per-user overrides that the MVP explicitly excludes.[^1]

<div align="center">⁂</div>

[^1]: PRD_M7_3.txt

