<!-- Source: PRDs | Last Verified: 2026-06-28 | Owner: Engineering -->
# Multi-Tenant Strategy

## Approach: Shared Database, Schema-Level Isolation with Row-Level Security

The platform uses a **shared database, shared schema** multi-tenancy model with PostgreSQL Row-Level Security (RLS) as the primary isolation mechanism, supplemented by application-layer tenant context enforcement.

This approach was chosen over per-tenant databases or per-tenant schemas because:
- It is more cost-efficient at scale (no N×database overhead)
- PostgreSQL RLS provides reliable, database-level enforcement
- It supports 1,000+ tenants without architectural changes
- Operational overhead (migrations, backups, monitoring) is significantly lower

---

## Tenant Isolation Layers

### Layer 1: Domain Routing

Every request is routed to the correct tenant based on the incoming hostname:

```
greenfield.schoolsaas.com → tenant_id: abc-123
portal.greenfieldschool.edu → tenant_id: abc-123 (custom domain)
riverside.schoolsaas.com → tenant_id: def-456
```

Domain resolution is handled in the API middleware before authentication:

```typescript
// NestJS middleware (simplified)
async function resolveTenant(req: Request, res: Response, next: NextFunction) {
  const host = req.hostname;
  const tenant = await tenantDomainRepo.findByDomain(host);
  if (!tenant) throw new NotFoundException('Tenant not found');
  req.tenantId = tenant.tenant_id;
  next();
}
```

### Layer 2: JWT Claims

After authentication, every JWT includes:
```json
{
  "sub": "user-uuid",
  "tenant_id": "tenant-uuid",
  "membership_id": "membership-uuid",
  "roles": ["teacher"],
  "session_id": "session-uuid",
  "iat": 1234567890,
  "exp": 1234567890
}
```

JWTs without a `tenant_id` are valid only for super-admin platform operations.

### Layer 3: Application Guard

Before any business logic executes, a NestJS guard chain enforces:

```
JwtAuthGuard
    └─> TenantGuard (validates tenant_id matches token + request)
          └─> SubscriptionGuard (checks subscription is active/not expired)
                └─> RoleGuard (validates membership role)
                      └─> PermissionGuard (validates specific permission code)
                            └─> ResourceScopeGuard (teacher assignments, parent links, etc.)
```

### Layer 4: Database Row-Level Security (RLS)

Every tenant-owned table has RLS enabled. The application sets the tenant context before executing queries:

```sql
-- Set at connection/transaction start
SET app.tenant_id = 'tenant-uuid-here';

-- Example RLS policy on students table
CREATE POLICY tenant_isolation ON students
  USING (tenant_id = current_setting('app.tenant_id')::uuid);
```

This ensures that even if application code has a bug and omits a `WHERE tenant_id = ?` clause, the database will not return records from another tenant.

---

## Required Table Structure

Every tenant-owned table must follow this pattern:

```sql
CREATE TABLE example_table (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
    -- ... business columns ...
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID REFERENCES users(id),
    updated_by UUID REFERENCES users(id)
);

ALTER TABLE example_table ENABLE ROW LEVEL SECURITY;

CREATE POLICY example_table_tenant_isolation ON example_table
  USING (tenant_id = current_setting('app.tenant_id', true)::uuid);

CREATE INDEX idx_example_table_tenant ON example_table (tenant_id);
CREATE INDEX idx_example_table_tenant_created ON example_table (tenant_id, created_at DESC);
```

### Global Tables (No tenant_id Required)

The following tables are global and do not use tenant isolation:
- `users` — global platform identity
- `permissions` — global permission catalog
- `roles` — when using global system roles (tenant_id is nullable for Phase 2 custom roles)
- `subscription_plans` — global plan catalog
- `platform_features` — global feature catalog

---

## Tenant Provisioning Flow

```
Super Admin creates tenant
    └─> Tenant record created (status: draft)
          └─> Subdomain reserved in tenant_domains
                └─> Default system roles seeded for tenant
                      └─> School Admin invited by email
                            └─> School Admin completes setup wizard
                                  └─> Validation passes → Tenant goes live (status: active)
```

### Tenant Seeding

When a tenant is created, the following are automatically provisioned:
1. Default system roles (School Admin, Principal, Teacher, Accountant, Parent, Student, Transport Manager, Librarian)
2. Default onboarding checklist steps
3. Default school settings (language: en, currency: INR, timezone: Asia/Kolkata)
4. Default attendance statuses (Present, Absent, Late, Excused, Half Day)
5. Default fee category types

---

## Subscription Gating

Subscription state controls module access:

| Subscription State | Module Access |
|-------------------|---------------|
| `trial` | Full access (configurable limits on students/staff) |
| `active` | Full access per plan features |
| `grace_period` | Full access (for up to 15 days after expiry) |
| `expired` | Read-only access to all data; no new operations |
| `suspended` | Read-only; all active sessions invalidated immediately |
| `cancelled` | Read-only until data retention period expires |

Implementation in the subscription guard:

```typescript
@Injectable()
class SubscriptionGuard implements CanActivate {
  canActivate(context: ExecutionContext) {
    const { tenantId } = context.switchToHttp().getRequest();
    const subscription = await subscriptionService.getCurrent(tenantId);
    
    if (!subscription || ['expired', 'cancelled'].includes(subscription.status)) {
      throw new ForbiddenException('Subscription inactive');
    }
    
    if (subscription.status === 'suspended') {
      // Allow GET requests (read-only), block mutations
      const method = context.switchToHttp().getRequest().method;
      if (method !== 'GET') throw new ForbiddenException('Tenant suspended');
    }
    
    return true;
  }
}
```

---

## Performance Considerations

### Indexing Strategy

Every tenant-owned table must have a composite index on `(tenant_id, id)` and `(tenant_id, created_at DESC)` as a minimum. Business queries add additional composite indexes:

```sql
-- Pattern: tenant_id first, then business columns
CREATE INDEX idx_students_tenant_status ON students (tenant_id, status, student_number);
CREATE INDEX idx_invoices_tenant_student ON invoices (tenant_id, student_id, state);
CREATE INDEX idx_attendance_tenant_date ON attendance_sessions (tenant_id, attendance_date, section_id);
```

### Connection Pooling

Use PgBouncer in transaction mode for connection pooling. The `SET app.tenant_id = ?` command is executed within each transaction, not at the connection level, to ensure correct isolation with pooled connections.

### Tenant Size Tiers

Tables with very high row counts (attendance_records, ledger_entries, audit_logs) are partitioned by `academic_year_id` or time period to maintain query performance as large tenants grow.

---

## Security Boundaries

### What a Tenant CAN Access
- All records in all tables where `tenant_id = their tenant_id`
- Global lookup data (subscription plans, platform feature flags)
- Their own users (via memberships)

### What a Tenant CANNOT Access
- Any other tenant's data (enforced by RLS + application guard)
- Platform-level audit logs from other tenants
- Other tenants' user accounts (users are global, but access is through memberships)
- Super admin operations (separate role flag on user)

### Cross-Tenant Prevention Checklist
- [ ] Every business table has `tenant_id NOT NULL`
- [ ] RLS enabled on every business table with tenant filter policy
- [ ] App middleware sets `app.tenant_id` before every query
- [ ] No raw SQL queries without tenant filter in WHERE clause
- [ ] No ORM queries without `.where({ tenantId })` scope
- [ ] Foreign key joins cannot cross tenant boundaries (enforced by RLS)
- [ ] Automated security tests include cross-tenant access test cases

---

## Tenant Data Retention

On cancellation or deletion of a tenant:
1. Tenant status transitions to `cancelled`
2. Active subscriptions are terminated
3. Data is retained for a configurable retention period (default: 365 days after cancellation)
4. After retention period, a cascade deletion job removes all tenant data
5. S3 files are purged from the tenant's prefix

Soft deletes (using `deleted_at`) are used for critical entities (students, branding assets, financial records) to maintain audit trails even within a tenant's lifetime.

