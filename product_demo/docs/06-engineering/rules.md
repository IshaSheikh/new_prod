<!-- Source: PRDs | Last Verified: 2026-06-28 | Owner: Engineering -->
# AI-Assisted Coding Rules

Conventions and constraints for AI-assisted development on this codebase. These rules help maintain consistency when using  , GitHub Copilot, or similar tools.

---

## Always Do

### 1. Include Tenant Context

Every database query must include tenant filtering:

```typescript
// Always
.where('table.tenantId = :tenantId', { tenantId })

// Never
.find({ status: 'active' })  // Missing tenant filter
```

### 2. Use Parameterized Queries

```typescript
// Always
await db.query('SELECT * FROM students WHERE tenant_id = $1', [tenantId]);

// Never
await db.query(`SELECT * FROM students WHERE tenant_id = '${tenantId}'`);
```

### 3. Return Typed DTOs

```typescript
// Always — use typed response classes
async findStudent(id: string): Promise<StudentResponse> { ... }

// Never — expose raw database entities
async findStudent(id: string): Promise<Student> { ... }  // Exposes password_hash, etc.
```

### 4. Log Sensitive Operations

```typescript
// Every sensitive mutation must log to audit_logs
await this.auditService.log({
  tenantId,
  actorUserId,
  action: 'student.status_changed',
  entityTable: 'students',
  entityId: student.id,
  oldValue: { status: oldStatus },
  newValue: { status: newStatus },
});
```

### 5. Validate All Inputs

```typescript
// Always use class-validator in DTOs
export class CreateInvoiceDto {
  @IsUUID()
  @IsNotEmpty()
  studentId: string;

  @IsNumber()
  @IsPositive()
  amount: number;

  @IsDateString()
  dueDate: string;
}
```

---

## Never Do

### 1. Never Accept tenantId from Client

```typescript
// NEVER — tenantId from request body is a security vulnerability
@Post('students')
create(@Body() dto: { tenantId: string, ... }) { }

// ALWAYS — extract from JWT via decorator
@Post('students')
create(@TenantId() tenantId: string, @Body() dto: CreateStudentDto) { }
```

### 2. Never Skip Permission Checks

```typescript
// NEVER — no permission guard
@Get('invoices')
findAll() { }

// ALWAYS — explicit permission requirement
@Get('invoices')
@RequirePermissions('fees.invoices.view')
findAll() { }
```

### 3. Never Hard-Delete Financial Records

```typescript
// NEVER
await this.invoiceRepo.delete({ id: invoiceId });

// NEVER
await db.query('DELETE FROM invoices WHERE id = $1', [invoiceId]);

// The finance module does not support hard deletion of transactions
// Cancellation and reversal workflows must be used instead
```

### 4. Never Expose Sensitive Fields in API Responses

```typescript
// NEVER return raw User entity (includes password_hash)
return user;

// ALWAYS use a response DTO that excludes sensitive fields
return {
  id: user.id,
  email: user.email,
  firstName: user.firstName,
  lastName: user.lastName,
  status: user.status,
};
```

### 5. Never Mutate Published Records

```typescript
// NEVER
await this.reportCardRepo.update(reportCard.id, { overallGrade: 'A2' });

// ALWAYS create a revision
await this.reportCardService.createRevision(reportCard.id, {
  overallGrade: 'A2',
  revisionReason: 'Marks entry correction',
  revisedBy: actorUserId,
});
```

---

## Module-Specific Rules

### Finance Module

- Invoice amounts are snapshots — never recalculate from live fee structures after issuance
- Receipts are immutable — corrections always go through reversal + new receipt
- All payment operations use idempotency keys
- Webhook handlers are idempotent (check for existing record before creating)

### Attendance Module

- Notifications are triggered only after session submission, never during draft
- Attendance on non-instructional days is always blocked
- Post-lock corrections require approval workflow — never direct update

### Gradebook Module

- Marks may only be entered by the assigned teacher (validated server-side, not just in UI)
- Published report cards are immutable — only revisions allowed
- Assessment component weights must sum to 100 before verification is allowed

### Portal Module

- Only published/submitted/approved data from source modules is shown
- Draft records from ANY module are never exposed through portal APIs
- Parent access is always scoped to linked children — server-side enforcement required

---

## Naming Conventions for AI-Generated Code

When asking an AI to generate code for this codebase, include these constraints in the prompt:

```
Generate NestJS code that:
- Uses UUIDs for all IDs
- Includes tenant_id on all repository queries
- Uses class-validator decorators in DTOs
- Extracts tenantId from JWT using @TenantId() decorator (never from body)
- Uses parameterized queries (no string concatenation)
- Returns typed response DTOs (not raw entities)
- Includes error handling with NestJS exceptions
- Follows the module structure: controller → service → repository
- Adds audit log calls for mutations to critical entities
```

---

## Database Migration Rules

When generating database migrations:

```sql
-- Always include
tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
created_by UUID REFERENCES users(id),
updated_by UUID REFERENCES users(id)

-- Always add RLS
ALTER TABLE new_table ENABLE ROW LEVEL SECURITY;
CREATE POLICY new_table_tenant_isolation ON new_table
  USING (tenant_id = current_setting('app.tenant_id', true)::uuid);

-- Always add tenant index
CREATE INDEX idx_new_table_tenant ON new_table (tenant_id);

-- Use CONCURRENTLY for production index creation
CREATE INDEX CONCURRENTLY idx_new_table_column ON new_table (column_name);

-- Never use without parameterization
-- Never use DROP COLUMN in a migration (deprecate instead)
-- Never hard delete from financial tables
```

