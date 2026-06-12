# Coding Standards

Engineering conventions for the platform codebase. These standards apply to all modules and must be consistent across the NestJS API, Next.js web app, and Flutter mobile app.

---

## General Principles

1. **Readability over cleverness** — code is read 10× more than it is written
2. **Explicit over implicit** — never rely on magic or undocumented behavior
3. **Security by default** — assume hostile input; validate everything
4. **Fail loudly** — unexpected states should throw errors, not fail silently
5. **Audit everything sensitive** — every mutation to a critical entity generates an audit log
6. **Tenant-first** — every query, every API, every function must be tenant-aware

---

## NestJS API Standards

### Module Structure

Each business module follows this directory structure:

```
src/
  modules/
    attendance/
      attendance.module.ts
      attendance.controller.ts
      attendance.service.ts
      attendance.repository.ts
      dto/
        create-attendance-session.dto.ts
        bulk-submit-attendance.dto.ts
      entities/
        attendance-session.entity.ts
        attendance-record.entity.ts
      guards/
        attendance-scope.guard.ts
      types/
        attendance.types.ts
```

### Controller Conventions

```typescript
@Controller('attendance')
@UseGuards(JwtAuthGuard, TenantGuard, SubscriptionGuard)
export class AttendanceController {
  
  @Post('sessions')
  @RequirePermissions('attendance.mark')
  @HttpCode(HttpStatus.CREATED)
  async createSession(
    @TenantId() tenantId: string,
    @CurrentUser() user: AuthUser,
    @Body() dto: CreateAttendanceSessionDto,
  ): Promise<AttendanceSessionResponse> {
    return this.attendanceService.createSession(tenantId, user.id, dto);
  }
}
```

Rules:
- Always use `@TenantId()` decorator to extract tenant from JWT — never from request body
- Always use `@RequirePermissions()` to declare access requirements
- Return typed response DTOs — never return raw entities with sensitive fields
- Use `@HttpCode()` explicitly for non-200 responses

### Service Layer Conventions

```typescript
@Injectable()
export class AttendanceService {
  constructor(
    private readonly attendanceRepo: AttendanceRepository,
    private readonly auditService: AuditService,
  ) {}

  async createSession(
    tenantId: string,
    actorUserId: string,
    dto: CreateAttendanceSessionDto,
  ): Promise<AttendanceSession> {
    // 1. Validate tenant context
    // 2. Validate business rules (no duplicate session, instructional day check)
    // 3. Execute operation
    // 4. Emit audit log
    // 5. Return result
  }
}
```

Rules:
- Services never access the HTTP layer — they receive plain parameters
- Services must not have cross-module dependencies on repositories; use module public service methods
- All business rule validations happen in the service, not the controller
- Throw typed exceptions from `@nestjs/common` with descriptive messages

### Repository Conventions

```typescript
@Injectable()
export class AttendanceRepository {
  constructor(
    @InjectDataSource() private readonly dataSource: DataSource,
  ) {}

  async findSessionsByDate(
    tenantId: string,
    date: Date,
    sectionId: string,
  ): Promise<AttendanceSession[]> {
    return this.dataSource
      .getRepository(AttendanceSession)
      .createQueryBuilder('s')
      .where('s.tenantId = :tenantId', { tenantId })
      .andWhere('s.attendanceDate = :date', { date })
      .andWhere('s.sectionId = :sectionId', { sectionId })
      .getMany();
  }
}
```

Rules:
- **Always** include `WHERE tenant_id = :tenantId` on every query
- Never use raw string interpolation in SQL — always use parameterized queries
- Use TypeORM query builder for complex queries; avoid raw SQL except for reporting
- Set `SET LOCAL app.tenant_id = ?` before every transaction

### DTO Conventions

```typescript
export class CreateAttendanceSessionDto {
  @IsUUID()
  @IsNotEmpty()
  academicYearId: string;

  @IsUUID()
  @IsNotEmpty()
  sectionId: string;

  @IsDateString()
  attendanceDate: string;

  @IsEnum(AttendanceMode)
  attendanceMode: AttendanceMode;
}
```

Rules:
- All DTOs use `class-validator` decorators
- Use `@IsUUID()` for all UUID fields
- Never trust client-provided `tenantId` — always use the JWT-derived tenant
- Transform and validate all inputs with `ValidationPipe` globally

### Error Handling

```typescript
// Use domain-specific exceptions
throw new ConflictException({
  statusCode: 409,
  error: 'CONFLICT',
  message: 'An attendance session already exists for this date and section',
  code: 'ATTENDANCE_SESSION_EXISTS',
});

// Use UnprocessableEntityException for business rule violations
throw new UnprocessableEntityException({
  statusCode: 422,
  error: 'UNPROCESSABLE_ENTITY',
  message: 'Cannot submit attendance on a non-instructional day',
  code: 'ATTENDANCE_NON_INSTRUCTIONAL_DAY',
});
```

---

## Database Conventions

### Naming

- Tables: `snake_case` plurals (e.g., `attendance_sessions`, `student_enrollments`)
- Columns: `snake_case` (e.g., `created_at`, `tenant_id`)
- Indexes: `idx_{table}_{columns}` (e.g., `idx_students_tenant_status`)
- Unique indexes: `uq_{table}_{columns}` (e.g., `uq_student_number`)
- Foreign keys: `fk_{table}_{referenced_table}` (via named constraints)
- Enums: `{table}_{column}_enum` or descriptive type name

### Mandatory Columns

Every tenant-owned table must include:
```sql
id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
tenant_id UUID NOT NULL REFERENCES tenants(id),
created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
created_by UUID REFERENCES users(id),
updated_by UUID REFERENCES users(id)
```

### Migration Conventions

- Migrations live in `src/database/migrations/`
- Naming: `{timestamp}_{description}.ts` (e.g., `20260615_add_attendance_corrections.ts`)
- Each migration is reversible where possible (includes `down()` method)
- Never modify existing migrations — create new ones
- Run migrations in CI before deploying; never skip in production

---

## TypeScript Standards

- Use `strict: true` in tsconfig
- No `any` types — use `unknown` and narrow explicitly
- Prefer interfaces for object types, enums for finite sets
- Use `readonly` for properties that shouldn't change
- Use optional chaining (`?.`) and nullish coalescing (`??`) consistently
- Avoid `as` type assertions — prefer type guards

```typescript
// BAD
const tenantId = (req as any).tenantId;

// GOOD
function isTenantRequest(req: unknown): req is TenantRequest {
  return typeof (req as TenantRequest).tenantId === 'string';
}
```

---

## Security Standards

### Input Validation
- All API inputs validated with `class-validator` before reaching service layer
- File uploads: validate MIME type from file content (not extension), enforce size limits server-side
- No user-controlled SQL fragments or query parameters concatenated as strings

### Authentication
- JWT secret stored in environment variable (never in code)
- Refresh tokens stored as `argon2` hashes (never plaintext)
- Invitation and reset tokens stored as `sha256` hashes

### Password Storage
- Passwords hashed with `argon2id` with memory cost ≥ 65536, iterations ≥ 3
- Never log password values even during debugging

### Audit Logging
The following operations must always produce an audit log entry:
```
user.created | user.disabled | user.deleted
membership.role_changed | membership.disabled
student.status_changed
invoice.issued | invoice.cancelled
payment.confirmed
receipt.issued | receipt.reversed
refund.approved
attendance.session_locked | attendance.correction_approved
report_card.published
tenant.suspended | tenant.activated
subscription.changed
```

### PII Handling
- Health notes encrypted at application level before storage
- Government ID numbers stored with field-level encryption
- PII fields never logged or included in error messages
- API responses mask PII for roles that don't need it

---

## Next.js Standards

- Use Server Components by default; Client Components only for interactive elements
- API calls use server-side fetch with credential headers — never expose tokens to browser unnecessarily
- Form validation with `react-hook-form` + `zod`
- State management with React Query (TanStack Query) for server state
- All protected routes use middleware to validate tenant context and redirect

---

## Flutter Standards

- State management: Riverpod
- HTTP: Dio with interceptors for auth token refresh
- Navigation: GoRouter
- Forms: `reactive_forms`
- Offline capability: limited to viewing cached data (not offline-first)
- Accessibility: all interactive elements have semantic labels

---

## Code Review Checklist

Before approving any PR, verify:

- [ ] All new tables have `tenant_id NOT NULL` and RLS policy
- [ ] No raw SQL without parameterized queries
- [ ] No `tenantId` accepted from client request body
- [ ] Sensitive operations have audit log calls
- [ ] New DTOs have `class-validator` decorators
- [ ] Error responses use standard error shape from `error-contracts.md`
- [ ] No secrets, passwords, or tokens in code or comments
- [ ] Tests cover happy path and key failure scenarios
- [ ] Migration is reversible (has `down()` method)
