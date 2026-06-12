<img src="https://r2cdn.perplexity.ai/pplx-full-logo-primary-dark%402x.png" style="height:64px;margin-right:32px"/>

# You are my PostgreSQL data architect.

Task:
have you skimmed through all the files to (M7_1 to M7_16) to understand the modules.
based on each of the modules please carry out the following tasks

Design a production-ready multi-tenant PostgreSQL schema for the module:
 
Global rules:

- Every tenant-owned table must have tenant_id
- Include created_at, updated_at, created_by, updated_by where relevant
- Include soft delete only where justified
- Add indexes based on likely query patterns
- Protect PII
- Include foreign key relationships
- Avoid polymorphic chaos
- Use clear enum strategy where needed
 
Product facts:
- This is a K-12 school management SaaS
- Parents can have multiple children
- Students belong to class-section and academic year
- Attendance, fees, and report cards are core workflows
 
Output:

1. Entity list
2. Table definitions
3. Constraints
4. Indexes
5. Sample records
6. Query patterns
7. Migration sequence
8. Security-sensitive columns

give me outputs for module M7_1 for now and we will continue

For M7_1, the right database design is a **platform-core** schema with global identities, tenant-scoped memberships, domain routing, subscription control, and immutable audit logs, because this module is the backbone for tenant isolation, access control, subscription lifecycle, and auditability across all schools.[^1]
I am also treating users as global and access as membership-based, because Module 7.1 explicitly allows one user to belong to multiple tenants and says permissions are granted through memberships rather than directly on users.[^1]

## Entity list

These are the production-ready entities for Module 7.1, aligned to tenant creation, domain setup, membership management, subscription control, RBAC, and audit logging.[^1]

- `users` — global identity for staff, parents, and students across one or many tenants.[^1]
- `tenants` — school tenant master with lifecycle state such as draft, provisioning, active, suspended, and cancelled.[^1]
- `tenant_settings` — per-tenant operational defaults such as timezone and security-related tenant settings.[^1]
- `tenant_domains` — tenant subdomains and custom domains with verification tracking and primary-domain rules.[^1]
- `subscription_plans` — global plan catalog for pricing and limits.[^1]
- `platform_features` and `subscription_plan_features` — normalized feature catalog instead of opaque JSON-only feature control.[^1]
- `subscriptions` — tenant subscription lifecycle with active, grace, expired, cancelled, and suspended states.[^1]
- `permissions` — global permission catalog by module and action.[^1]
- `tenant_roles` — tenant-scoped roles such as school admin, principal, teacher, and accountant.[^1]
- `tenant_role_permissions` — many-to-many mapping from tenant roles to permissions.[^1]
- `user_tenant_memberships` — user access to a tenant, including lifecycle state and tenant membership history.[^1]
- `membership_roles` — role assignment through memberships, which follows the PRD rule that roles are assigned through memberships.[^1]
- `tenant_invitations` — invitation flow for onboarding users into a tenant.[^1]
- `auth_sessions` — tenant-context sessions so suspended tenants can have sessions invalidated immediately.[^1]
- `audit_logs` — immutable change/event records for sensitive operations such as user creation, role assignment, subscription change, and tenant suspension.[^1]


## Table definitions

I split platform-level privilege from tenant roles by keeping `users.is_platform_super_admin` as a platform flag and keeping all school roles tenant-scoped, because Module 7.1 says Super Admin is platform-wide only while School Admin is tenant-specific.[^1]
The DDL below implements the M7_1 entities and hardens them for production with lifecycle enums, composite foreign keys, append-only audit structure, and clean tenant boundaries.[^1]

```sql
CREATE EXTENSION IF NOT EXISTS pgcrypto;
CREATE EXTENSION IF NOT EXISTS citext;

CREATE TYPE tenant_status AS ENUM ('draft', 'provisioning', 'active', 'suspended', 'cancelled');
CREATE TYPE domain_type AS ENUM ('subdomain', 'custom');
CREATE TYPE domain_verification_status AS ENUM ('pending', 'verified', 'failed');
CREATE TYPE user_status AS ENUM ('active', 'disabled');
CREATE TYPE membership_status AS ENUM ('invited', 'accepted', 'active', 'disabled', 'removed');
CREATE TYPE role_status AS ENUM ('active', 'archived');
CREATE TYPE subscription_status AS ENUM ('trial', 'active', 'grace_period', 'expired', 'cancelled', 'suspended');
CREATE TYPE billing_cycle AS ENUM ('monthly', 'annual');
CREATE TYPE invitation_status AS ENUM ('pending', 'accepted', 'expired', 'revoked');
CREATE TYPE session_status AS ENUM ('active', 'revoked', 'expired');

CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email CITEXT NOT NULL UNIQUE,
    mobile_e164 VARCHAR(20),
    first_name VARCHAR(100) NOT NULL,
    last_name VARCHAR(100),
    password_hash TEXT NOT NULL,
    status user_status NOT NULL DEFAULT 'active',
    is_platform_super_admin BOOLEAN NOT NULL DEFAULT FALSE,
    last_login_at TIMESTAMPTZ,
    deleted_at TIMESTAMPTZ,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID NULL REFERENCES users(id),
    updated_by UUID NULL REFERENCES users(id)
);

CREATE TABLE tenants (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    code VARCHAR(50) NOT NULL UNIQUE,
    name VARCHAR(255) NOT NULL,
    status tenant_status NOT NULL DEFAULT 'draft',
    timezone VARCHAR(64) NOT NULL DEFAULT 'Asia/Kolkata',
    address_line1 VARCHAR(255),
    address_line2 VARCHAR(255),
    city VARCHAR(100),
    state VARCHAR(100),
    postal_code VARCHAR(20),
    country_code CHAR(2) NOT NULL DEFAULT 'IN',
    onboarding_completed_at TIMESTAMPTZ,
    activated_at TIMESTAMPTZ,
    suspended_at TIMESTAMPTZ,
    cancelled_at TIMESTAMPTZ,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID NULL REFERENCES users(id),
    updated_by UUID NULL REFERENCES users(id)
);

CREATE TABLE tenant_settings (
    tenant_id UUID PRIMARY KEY REFERENCES tenants(id) ON DELETE CASCADE,
    enforce_mfa_for_admins BOOLEAN NOT NULL DEFAULT FALSE,
    session_idle_timeout_minutes INTEGER NOT NULL DEFAULT 60,
    allow_custom_domain BOOLEAN NOT NULL DEFAULT TRUE,
    read_only_mode BOOLEAN NOT NULL DEFAULT FALSE,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID NULL REFERENCES users(id),
    updated_by UUID NULL REFERENCES users(id)
);

CREATE TABLE tenant_domains (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
    domain_type domain_type NOT NULL,
    domain VARCHAR(255) NOT NULL,
    is_primary BOOLEAN NOT NULL DEFAULT FALSE,
    verification_status domain_verification_status NOT NULL DEFAULT 'pending',
    verification_token_hash TEXT,
    verified_at TIMESTAMPTZ,
    removed_at TIMESTAMPTZ,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID NULL REFERENCES users(id),
    updated_by UUID NULL REFERENCES users(id),
    CONSTRAINT uq_tenant_domain UNIQUE (domain)
);

CREATE TABLE subscription_plans (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    code VARCHAR(50) NOT NULL UNIQUE,
    name VARCHAR(100) NOT NULL,
    max_students INTEGER,
    max_staff INTEGER,
    monthly_price NUMERIC(12,2) NOT NULL,
    annual_price NUMERIC(12,2) NOT NULL,
    is_active BOOLEAN NOT NULL DEFAULT TRUE,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE TABLE platform_features (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    code VARCHAR(100) NOT NULL UNIQUE,
    module_name VARCHAR(100) NOT NULL,
    feature_name VARCHAR(100) NOT NULL,
    description TEXT,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE TABLE subscription_plan_features (
    plan_id UUID NOT NULL REFERENCES subscription_plans(id) ON DELETE CASCADE,
    feature_id UUID NOT NULL REFERENCES platform_features(id) ON DELETE CASCADE,
    is_enabled BOOLEAN NOT NULL DEFAULT TRUE,
    limit_value INTEGER,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    PRIMARY KEY (plan_id, feature_id)
);

CREATE TABLE permissions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    code VARCHAR(100) NOT NULL UNIQUE,
    module_name VARCHAR(100) NOT NULL,
    action_name VARCHAR(100) NOT NULL,
    description TEXT,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE TABLE tenant_roles (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
    code VARCHAR(50) NOT NULL,
    name VARCHAR(100) NOT NULL,
    status role_status NOT NULL DEFAULT 'active',
    is_system_role BOOLEAN NOT NULL DEFAULT FALSE,
    description TEXT,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID NULL REFERENCES users(id),
    updated_by UUID NULL REFERENCES users(id),
    CONSTRAINT uq_tenant_role_code UNIQUE (tenant_id, code),
    CONSTRAINT uq_tenant_role_id_tenant UNIQUE (id, tenant_id)
);

CREATE TABLE tenant_role_permissions (
    tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
    role_id UUID NOT NULL,
    permission_id UUID NOT NULL REFERENCES permissions(id) ON DELETE CASCADE,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID NULL REFERENCES users(id),
    PRIMARY KEY (tenant_id, role_id, permission_id),
    FOREIGN KEY (role_id, tenant_id)
        REFERENCES tenant_roles(id, tenant_id)
        ON DELETE CASCADE
);

CREATE TABLE user_tenant_memberships (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
    user_id UUID NOT NULL REFERENCES users(id) ON DELETE RESTRICT,
    status membership_status NOT NULL DEFAULT 'invited',
    joined_at TIMESTAMPTZ,
    disabled_at TIMESTAMPTZ,
    removed_at TIMESTAMPTZ,
    invited_by UUID NULL REFERENCES users(id),
    accepted_at TIMESTAMPTZ,
    version_no INTEGER NOT NULL DEFAULT 1,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_by UUID NULL REFERENCES users(id),
    CONSTRAINT uq_membership_user_tenant UNIQUE (tenant_id, user_id),
    CONSTRAINT uq_membership_id_tenant UNIQUE (id, tenant_id)
);

CREATE TABLE membership_roles (
    tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
    membership_id UUID NOT NULL,
    role_id UUID NOT NULL,
    assigned_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    assigned_by UUID NULL REFERENCES users(id),
    PRIMARY KEY (tenant_id, membership_id, role_id),
    FOREIGN KEY (membership_id, tenant_id)
        REFERENCES user_tenant_memberships(id, tenant_id)
        ON DELETE CASCADE,
    FOREIGN KEY (role_id, tenant_id)
        REFERENCES tenant_roles(id, tenant_id)
        ON DELETE CASCADE
);

CREATE TABLE tenant_invitations (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
    email CITEXT NOT NULL,
    invite_token_hash TEXT NOT NULL,
    status invitation_status NOT NULL DEFAULT 'pending',
    expires_at TIMESTAMPTZ NOT NULL,
    accepted_at TIMESTAMPTZ,
    accepted_membership_id UUID NULL REFERENCES user_tenant_memberships(id),
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID NULL REFERENCES users(id),
    updated_by UUID NULL REFERENCES users(id)
);

CREATE TABLE subscriptions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
    plan_id UUID NOT NULL REFERENCES subscription_plans(id) ON DELETE RESTRICT,
    billing_cycle billing_cycle NOT NULL,
    status subscription_status NOT NULL DEFAULT 'trial',
    start_date DATE NOT NULL,
    end_date DATE NOT NULL,
    grace_until DATE,
    cancelled_at TIMESTAMPTZ,
    read_only_from TIMESTAMPTZ,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID NULL REFERENCES users(id),
    updated_by UUID NULL REFERENCES users(id),
    CONSTRAINT ck_subscription_dates CHECK (
        end_date >= start_date
        AND (grace_until IS NULL OR grace_until >= end_date)
    )
);

CREATE TABLE auth_sessions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NULL REFERENCES tenants(id) ON DELETE CASCADE,
    user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    membership_id UUID NULL REFERENCES user_tenant_memberships(id) ON DELETE CASCADE,
    refresh_token_hash TEXT NOT NULL,
    status session_status NOT NULL DEFAULT 'active',
    ip_address INET,
    user_agent TEXT,
    last_seen_at TIMESTAMPTZ,
    expires_at TIMESTAMPTZ NOT NULL,
    revoked_at TIMESTAMPTZ,
    revocation_reason TEXT,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE TABLE audit_logs (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NULL REFERENCES tenants(id) ON DELETE RESTRICT,
    actor_user_id UUID NULL REFERENCES users(id) ON DELETE SET NULL,
    actor_membership_id UUID NULL REFERENCES user_tenant_memberships(id) ON DELETE SET NULL,
    action VARCHAR(100) NOT NULL,
    entity_table VARCHAR(100) NOT NULL,
    entity_id UUID NOT NULL,
    old_value JSONB,
    new_value JSONB,
    ip_address INET,
    user_agent TEXT,
    request_id UUID,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);
```


## Constraints and indexes

The key rules to enforce are tenant isolation, globally unique domains, membership-based access, immutable audit logging, active-subscription gating, and no cross-tenant access.[^1]
Module 7.1 also requires support for multi-tenant membership, role assignment through memberships, read-only behavior for suspended subscriptions, and immediate session invalidation when a tenant is suspended.[^1]

**Constraints**

- `users.email` must be globally unique, because a single identity may belong to multiple tenants.[^1]
- `tenant_domains.domain` must be globally unique, because duplicate domain registration is an edge case the PRD blocks.[^1]
- Use a partial unique index so only one non-removed primary domain exists per tenant.[^1]
- Use a partial unique index so only one current subscription in `trial`, `active`, or `grace_period` exists per tenant.[^1]
- Prevent membership self-lockout and “last School Admin removed” with a deferred trigger on `membership_roles` and `user_tenant_memberships`.[^1]
- Make `audit_logs` append-only by revoking `UPDATE` and `DELETE` and optionally adding a trigger that raises on modification.[^1]

```sql
CREATE UNIQUE INDEX uq_tenant_primary_domain
ON tenant_domains (tenant_id)
WHERE is_primary = TRUE AND removed_at IS NULL;

CREATE UNIQUE INDEX uq_tenant_current_subscription
ON subscriptions (tenant_id)
WHERE status IN ('trial', 'active', 'grace_period');

CREATE INDEX idx_tenants_status_created_at
ON tenants (status, created_at DESC);

CREATE INDEX idx_tenant_domains_lookup
ON tenant_domains (domain)
WHERE removed_at IS NULL;

CREATE INDEX idx_memberships_tenant_status
ON user_tenant_memberships (tenant_id, status, user_id);

CREATE INDEX idx_memberships_user_status
ON user_tenant_memberships (user_id, status, tenant_id);

CREATE INDEX idx_membership_roles_membership
ON membership_roles (tenant_id, membership_id);

CREATE INDEX idx_tenant_roles_tenant_status
ON tenant_roles (tenant_id, status, code);

CREATE INDEX idx_subscriptions_tenant_status_dates
ON subscriptions (tenant_id, status, end_date, grace_until);

CREATE INDEX idx_invitations_tenant_email_status
ON tenant_invitations (tenant_id, email, status);

CREATE INDEX idx_auth_sessions_tenant_user_status
ON auth_sessions (tenant_id, user_id, status, expires_at);

CREATE INDEX idx_audit_logs_tenant_created_at
ON audit_logs (tenant_id, created_at DESC);

CREATE INDEX idx_audit_logs_actor_created_at
ON audit_logs (actor_user_id, created_at DESC);

CREATE INDEX idx_audit_logs_entity
ON audit_logs (tenant_id, entity_table, entity_id, created_at DESC);
```

**RLS baseline**

- Enable PostgreSQL RLS on every tenant-owned table and bind access to `current_setting('app.tenant_id')::uuid`, because the PRD explicitly recommends Row-Level Security plus mandatory `tenant_id`.[^1]
- Apply the policy to `tenants` only for super-admin or tenant-admin service roles, and to `tenant_domains`, `tenant_settings`, `user_tenant_memberships`, `membership_roles`, `tenant_roles`, `subscriptions`, `tenant_invitations`, `auth_sessions`, and `audit_logs` for tenant-scoped access.[^1]


## Sample records, query patterns, migration sequence

These examples cover the exact M7_1 workflows the PRD calls out: tenant creation, domain resolution, invitation/membership access, subscription checks, and auditability.[^1]

**Sample records**

```sql
INSERT INTO users (id, email, mobile_e164, first_name, last_name, password_hash, is_platform_super_admin)
VALUES
('00000000-0000-0000-0000-000000000001', 'superadmin@schoolsaas.com', '+919900000001', 'Platform', 'Admin', 'argon2_hash_1', TRUE),
('00000000-0000-0000-0000-000000000002', 'admin@greenfield.edu', '+919900000002', 'Anita', 'Rao', 'argon2_hash_2', FALSE);

INSERT INTO tenants (id, code, name, status, timezone, city, state, country_code, created_by)
VALUES
('10000000-0000-0000-0000-000000000001', 'GREENFIELD', 'Greenfield School', 'active', 'Asia/Kolkata', 'Chennai', 'Tamil Nadu', 'IN',
 '00000000-0000-0000-0000-000000000001');

INSERT INTO tenant_domains (id, tenant_id, domain_type, domain, is_primary, verification_status, created_by)
VALUES
('20000000-0000-0000-0000-000000000001', '10000000-0000-0000-0000-000000000001', 'subdomain',
 'greenfield.schoolsaas.com', TRUE, 'verified', '00000000-0000-0000-0000-000000000001');

INSERT INTO subscription_plans (id, code, name, max_students, max_staff, monthly_price, annual_price)
VALUES
('30000000-0000-0000-0000-000000000001', 'PRO', 'Pro Plan', 2000, 250, 14999.00, 149999.00);

INSERT INTO subscriptions (id, tenant_id, plan_id, billing_cycle, status, start_date, end_date, grace_until, created_by)
VALUES
('40000000-0000-0000-0000-000000000001', '10000000-0000-0000-0000-000000000001',
 '30000000-0000-0000-0000-000000000001', 'annual', 'active', '2026-04-01', '2027-03-31', '2027-04-15',
 '00000000-0000-0000-0000-000000000001');
```

**Query patterns**

1. Resolve tenant from incoming domain.[^1]
```sql
SELECT td.tenant_id, t.name, t.status
FROM tenant_domains td
JOIN tenants t ON t.id = td.tenant_id
WHERE td.domain = $1
  AND td.removed_at IS NULL
  AND td.verification_status = 'verified';
```

2. List all tenant contexts for one user, because a user may belong to multiple tenants.[^1]
```sql
SELECT m.tenant_id, t.name, m.status
FROM user_tenant_memberships m
JOIN tenants t ON t.id = m.tenant_id
WHERE m.user_id = $1
  AND m.status IN ('accepted', 'active')
ORDER BY t.name;
```

3. Resolve effective permissions inside a tenant through membership roles.[^1]
```sql
SELECT DISTINCT p.code
FROM user_tenant_memberships m
JOIN membership_roles mr
  ON mr.membership_id = m.id AND mr.tenant_id = m.tenant_id
JOIN tenant_role_permissions trp
  ON trp.role_id = mr.role_id AND trp.tenant_id = mr.tenant_id
JOIN permissions p
  ON p.id = trp.permission_id
WHERE m.user_id = $1
  AND m.tenant_id = $2
  AND m.status = 'active';
```

4. Gate business-module access by subscription state and read-only mode.[^1]
```sql
SELECT s.status, s.end_date, s.grace_until, ts.read_only_mode
FROM subscriptions s
JOIN tenant_settings ts ON ts.tenant_id = s.tenant_id
WHERE s.tenant_id = $1
  AND s.status IN ('trial', 'active', 'grace_period', 'suspended');
```

5. Export audit trail for sensitive operations by tenant and date range.[^1]
```sql
SELECT created_at, action, entity_table, entity_id, actor_user_id, old_value, new_value
FROM audit_logs
WHERE tenant_id = $1
  AND created_at >= $2
  AND created_at < $3
ORDER BY created_at DESC;
```

**Migration sequence**

1. Create extensions `pgcrypto` and `citext`, then create enums.[^1]
2. Create global tables first: `users`, `subscription_plans`, `platform_features`, `subscription_plan_features`, and `permissions`.[^1]
3. Create tenant master tables: `tenants`, `tenant_settings`, and `tenant_domains`.[^1]
4. Create RBAC and access tables: `tenant_roles`, `tenant_role_permissions`, `user_tenant_memberships`, `membership_roles`, and `tenant_invitations`.[^1]
5. Create `subscriptions`, `auth_sessions`, and `audit_logs`, then add indexes, triggers, and RLS policies.[^1]
6. Seed default permissions and tenant system roles such as `school_admin`, `principal`, `teacher`, `accountant`, `parent`, and `student`.[^1]

## Security-sensitive columns

Module 7.1 is the security backbone, so the most sensitive columns are identity, auth, domain-verification, session, and audit fields.[^1]
The PRD also explicitly recommends PostgreSQL RLS, JWT tenant context, auditability, and soft delete only for critical entities, so these columns should be protected with least-privilege access, encryption or hashing where appropriate, and strict export controls.[^1]

- `users.email`, `users.mobile_e164`, `users.first_name`, `users.last_name` — PII; restrict broad selects and mask in admin exports.[^1]
- `users.password_hash` — secret credential material; never decrypt, never expose, and store only strong password hashes such as Argon2id.[^1]
- `tenant_invitations.invite_token_hash` — secret token material; store hash only, never raw invitation token.[^1]
- `tenant_domains.verification_token_hash` — sensitive domain verification secret; hash or encrypt at rest.[^1]
- `auth_sessions.refresh_token_hash` — secret session material; store hash only and rotate on refresh.[^1]
- `auth_sessions.ip_address`, `auth_sessions.user_agent`, `audit_logs.ip_address`, `audit_logs.user_agent` — security telemetry and privacy-sensitive data; retain under policy and expose only to authorized admins.[^1]
- `audit_logs.old_value`, `audit_logs.new_value` — may contain indirect PII or security-relevant changes; redact before export when not necessary.[^1]

This is the M7_1 schema baseline I would use before designing M7_2 onward, because the PRD makes this module the first implementation dependency for attendance, grades, fees, timetable, and communication.[^1]

<div align="center">⁂</div>

[^1]: PRD_M7_1.txt

