# Test Strategy

Testing approach for the platform, covering all layers from unit tests to end-to-end tests.

---

## Testing Pyramid

```
        /\
       /  \
      / E2E \      ← 10% — Critical user journeys (Playwright)
     /────────\
    / Integration \  ← 30% — API endpoints, DB interactions (Jest + Supertest)
   /──────────────\
  /   Unit Tests   \  ← 60% — Services, guards, utilities, validators (Jest)
 /──────────────────\
```

---

## Test Categories

### 1. Unit Tests

**Framework:** Jest  
**Coverage target:** >80% on service layer  
**Location:** `src/**/*.spec.ts` (co-located with source)

What to unit test:
- Service business logic
- Guards (permission checks, subscription state)
- DTO validation logic
- Utility functions (date calculations, grading formulas, fee computations)
- State machine transitions (valid/invalid transitions)

```typescript
// Example: attendance business rule test
describe('AttendanceService', () => {
  describe('createSession', () => {
    it('should throw when attendance session already exists for date/section', async () => {
      // arrange: mock repo returns existing session
      mockRepo.findByDateSection.mockResolvedValue(existingSession);
      
      // act/assert
      await expect(
        service.createSession(tenantId, userId, duplicateDto)
      ).rejects.toThrow(ConflictException);
    });

    it('should throw when date is a non-instructional day', async () => {
      mockCalendarRepo.isInstructionalDay.mockResolvedValue(false);
      
      await expect(
        service.createSession(tenantId, userId, holidayDto)
      ).rejects.toThrow(UnprocessableEntityException);
    });
  });
});
```

### 2. Integration Tests

**Framework:** Jest + Supertest + Test PostgreSQL (Docker)  
**Coverage target:** All API endpoints with happy path and key error cases  
**Location:** `test/integration/**/*.spec.ts`

What to test:
- Full HTTP request → response flow
- Database interactions with real PostgreSQL (test database)
- RLS policies (cross-tenant access must return empty/forbidden)
- Authentication and authorization flows
- State machine transitions through HTTP

**Critical: Cross-Tenant Isolation Tests**

Every module must have cross-tenant tests:

```typescript
describe('GET /students — cross-tenant isolation', () => {
  it('should not return students from another tenant', async () => {
    // Create student in tenant A
    await createStudent(tenantA, studentData);
    
    // Request from tenant B's token
    const response = await request(app)
      .get('/students')
      .set('Authorization', `Bearer ${tenantBToken}`)
      .expect(200);
    
    // Should return empty array (not tenant A's student)
    expect(response.body.data).toHaveLength(0);
  });
});
```

### 3. End-to-End Tests

**Framework:** Playwright  
**Location:** `e2e/**/*.spec.ts`  
**Environment:** Staging environment

Critical journeys to automate:
1. New school onboarding (full wizard completion + go-live)
2. Invite teacher → teacher marks attendance → admin views dashboard
3. Add student → assign fee structure → generate invoice → record payment → download receipt
4. Parent portal: login → view attendance → view fee → pay
5. Teacher: enter marks → principal verifies → admin publishes report card → parent downloads

---

## Test Coverage Requirements

| Area | Minimum Coverage |
|------|-----------------|
| Service layer (business logic) | 80% line coverage |
| Guard layer (auth/permissions) | 90% line coverage |
| DTO validation | 100% of validators |
| Critical financial operations | 100% of business rules |
| Cross-tenant isolation | 100% of all module endpoints |

---

## Security Test Requirements

The following must be explicitly tested and must all pass before any release:

### Cross-Tenant Access Tests

```typescript
// For every module, test:
// 1. Cannot read another tenant's records
// 2. Cannot write to another tenant's records
// 3. Cannot reference IDs from another tenant

describe('Security: Cross-tenant isolation', () => {
  const modules = ['students', 'invoices', 'attendance', 'report-cards', ...];
  
  modules.forEach(module => {
    it(`GET /${module} should not return another tenant's data`, async () => { ... });
    it(`POST /${module} with another tenant's IDs should fail`, async () => { ... });
  });
});
```

### Permission Escalation Tests

```typescript
describe('Security: Permission escalation', () => {
  it('teacher cannot access financial data', async () => {
    const response = await request(app)
      .get('/invoices')
      .set('Authorization', `Bearer ${teacherToken}`)
      .expect(403);
    
    expect(response.body.error).toBe('FORBIDDEN');
  });

  it('parent cannot see another child\'s attendance', async () => {
    // Parent with link to student A only
    const response = await request(app)
      .get(`/portal/attendance?studentId=${studentB.id}`)
      .set('Authorization', `Bearer ${parentToken}`)
      .expect(403);
  });
});
```

### Authentication Edge Cases

```typescript
describe('Security: Authentication edge cases', () => {
  it('expired access token is rejected', ...);
  it('revoked refresh token triggers session invalidation', ...);
  it('token from suspended tenant is blocked for mutations', ...);
  it('replayed refresh token revokes all user sessions', ...);
});
```

---

## Test Data Strategy

### Test Database Setup

- Each test suite creates isolated tenant data in a test transaction
- Transactions are rolled back after each test (not committed)
- Seed data (permissions, roles, plans) is applied once at test DB setup
- Use factories for creating test entities:

```typescript
// Test factory pattern
const tenant = await TenantFactory.create();
const school = await SchoolFactory.create({ tenantId: tenant.id });
const teacher = await UserFactory.create();
const membership = await MembershipFactory.create({
  tenantId: tenant.id,
  userId: teacher.id,
  role: 'teacher',
});
```

### Fixture Strategy

- Minimal fixtures — create only what the test needs
- No shared global state between tests
- IDs are generated fresh per test (no hardcoded UUIDs in test code)

---

## Financial Operation Test Requirements

Given the compliance and audit requirements, finance tests must cover:

1. **Invoice immutability after issuance:** Attempting to change an issued invoice's amounts returns an error
2. **Receipt immutability:** Attempting to edit an issued receipt returns an error
3. **Duplicate payment detection:** Sending identical payment with same idempotency key returns existing record
4. **Webhook idempotency:** Processing the same webhook payload twice produces no duplicate records
5. **Refund limit enforcement:** Refunding more than the confirmed payment amount is blocked
6. **Overpayment handling:** Overpaying an invoice creates a credit balance, not a negative invoice
7. **Discount application order:** Discounts applied in the correct sequence (line → invoice → scholarship → penalty)
8. **Ledger completeness:** Every invoice/payment/refund/adjustment creates corresponding ledger entries

---

## Attendance Test Requirements

1. **Duplicate session prevention:** Same date/section combination blocked
2. **Non-instructional day blocking:** Holiday and weekend sessions blocked
3. **Correction requires reason:** Correction without reason is rejected (422)
4. **Post-lock correction requires approval:** Direct edit of locked session is blocked (422)
5. **Only enrolled students appear:** Students not enrolled in the section don't appear in roster
6. **Notification only after submission:** Absence alerts not triggered while session is in draft

---

## Continuous Integration

Tests run in this order in CI:

```yaml
stages:
  - lint          # ESLint, Prettier check
  - unit-tests    # Jest unit tests (fast, no DB needed)
  - migrations    # Apply migrations to test DB
  - integration   # Jest integration tests (real test DB)
  - security      # Cross-tenant + permission escalation tests
  - build         # TypeScript compile
  - e2e           # Playwright (on staging environment after deploy)
```

All stages must pass before a PR can be merged. E2E tests run post-deploy to staging before promoting to production.
