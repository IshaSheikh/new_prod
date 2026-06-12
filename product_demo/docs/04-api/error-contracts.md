# Error Contracts

Standardized error response format used across all API endpoints.

---

## Standard Error Response Shape

```json
{
  "statusCode": 400,
  "error": "BAD_REQUEST",
  "message": "Validation failed",
  "details": [
    {
      "field": "email",
      "code": "INVALID_FORMAT",
      "message": "Must be a valid email address"
    }
  ],
  "requestId": "req_01JXYZ123456",
  "timestamp": "2026-06-15T08:30:00.000Z"
}
```

| Field | Type | Description |
|-------|------|-------------|
| `statusCode` | integer | HTTP status code |
| `error` | string | Machine-readable error code |
| `message` | string | Human-readable summary |
| `details` | array | Optional field-level validation errors |
| `requestId` | string | Unique request ID for support tracing |
| `timestamp` | ISO 8601 | When the error occurred |

---

## HTTP Status Codes

| Code | Error | When Used |
|------|-------|-----------|
| `400` | `BAD_REQUEST` | Invalid request body, missing required fields, validation failures |
| `401` | `UNAUTHORIZED` | Missing or invalid/expired JWT token |
| `402` | `PAYMENT_REQUIRED` | Subscription expired, plan limit exceeded |
| `403` | `FORBIDDEN` | Valid token but insufficient permissions |
| `404` | `NOT_FOUND` | Resource does not exist or is not accessible to this tenant |
| `409` | `CONFLICT` | Duplicate record, unique constraint violation, state conflict |
| `422` | `UNPROCESSABLE_ENTITY` | Business rule violation (invalid state transition, etc.) |
| `429` | `TOO_MANY_REQUESTS` | Rate limit exceeded |
| `500` | `INTERNAL_SERVER_ERROR` | Unexpected server error |
| `503` | `SERVICE_UNAVAILABLE` | External service temporarily unavailable |

---

## Application Error Codes

### Authentication and Authorization

| Code | Description |
|------|-------------|
| `AUTH_TOKEN_MISSING` | No Authorization header or token cookie |
| `AUTH_TOKEN_INVALID` | JWT signature verification failed |
| `AUTH_TOKEN_EXPIRED` | JWT has expired |
| `AUTH_SESSION_REVOKED` | Session has been explicitly revoked |
| `AUTH_TENANT_SUSPENDED` | Tenant is suspended; only read operations allowed |
| `AUTH_SUBSCRIPTION_EXPIRED` | Subscription has expired |
| `AUTH_PERMISSION_DENIED` | Missing required permission |
| `AUTH_INVALID_TENANT` | Requested tenant does not match token |
| `AUTH_INVITATION_EXPIRED` | Invitation token has expired (7 day limit) |
| `AUTH_RESET_TOKEN_EXPIRED` | Password reset token expired (30 minute limit) |
| `AUTH_RESET_TOKEN_USED` | Password reset token already used |

### User and Membership

| Code | Description |
|------|-------------|
| `USER_EMAIL_EXISTS` | Email address already registered |
| `USER_LAST_ADMIN` | Cannot remove the last active School Admin |
| `USER_INACTIVE` | User account is disabled |
| `MEMBERSHIP_EXISTS` | User already has a membership in this tenant |
| `MEMBERSHIP_NOT_FOUND` | No active membership found |
| `ROLE_NOT_FOUND` | Role does not exist |
| `ROLE_REQUIRES_AT_LEAST_ONE` | Membership must have at least one role |

### School Configuration

| Code | Description |
|------|-------------|
| `ACADEMIC_YEAR_OVERLAP` | Proposed academic year dates overlap with existing year |
| `ACADEMIC_YEAR_ACTIVE_EXISTS` | An active academic year already exists |
| `TERM_OUTSIDE_YEAR` | Term dates fall outside the parent academic year |
| `TERM_OVERLAP` | Term dates overlap with another term in the same year |
| `GRADE_NAME_EXISTS` | Grade name already exists in this tenant |
| `SECTION_EXISTS` | This grade-section combination already exists for this academic year |
| `CAMPUS_PRIMARY_REQUIRED` | Cannot delete primary campus without assigning a new primary |
| `SCHOOL_NOT_LIVE` | Operation requires school to be in live status |
| `ONBOARDING_INCOMPLETE` | Cannot go live with incomplete setup steps |

### Students

| Code | Description |
|------|-------------|
| `STUDENT_NUMBER_EXISTS` | Student number already in use |
| `STUDENT_DUPLICATE_DETECTED` | Possible duplicate student detected |
| `STUDENT_ENROLLMENT_EXISTS` | Student already has a primary enrollment for this academic year |
| `STUDENT_INVALID_STATUS_TRANSITION` | Cannot transition directly between these statuses |
| `GUARDIAN_PRIMARY_REQUIRED` | At least one primary guardian is required |
| `GUARDIAN_ALREADY_PRIMARY` | Another guardian is already marked primary |
| `DOCUMENT_VERSION_EXISTS` | A document version with this number already exists |

### Attendance

| Code | Description |
|------|-------------|
| `ATTENDANCE_SESSION_EXISTS` | An attendance session already exists for this date/section/period |
| `ATTENDANCE_NON_INSTRUCTIONAL_DAY` | Cannot take attendance on a non-instructional day |
| `ATTENDANCE_SESSION_SUBMITTED` | Session is submitted; teacher cannot edit directly |
| `ATTENDANCE_SESSION_LOCKED` | Session is locked; corrections require approval |
| `ATTENDANCE_CORRECTION_REASON_REQUIRED` | A reason is required for attendance corrections |

### Gradebook

| Code | Description |
|------|-------------|
| `ASSESSMENT_WEIGHT_SUM` | Component weights do not sum to 100% |
| `MARKS_EXCEED_MAX` | Marks entered exceed configured maximum |
| `MARKS_TEACHER_NOT_ASSIGNED` | Teacher is not assigned to this subject/section |
| `MARKS_INCOMPLETE` | Mandatory marks missing; cannot proceed to verification |
| `REPORT_CARD_IMMUTABLE` | Published report card cannot be edited; create a revision |
| `GRADING_SCHEME_NOT_FOUND` | No active grading scheme found for this context |

### Fees and Finance

| Code | Description |
|------|-------------|
| `INVOICE_NOT_ISSUED` | Payment cannot be applied to a draft or cancelled invoice |
| `INVOICE_FULLY_PAID` | Invoice is already fully paid |
| `PAYMENT_DUPLICATE_DETECTED` | Possible duplicate payment detected |
| `RECEIPT_IMMUTABLE` | Issued receipt cannot be edited; create a reversal |
| `REFUND_EXCEEDS_PAID` | Refund amount exceeds total confirmed payments |
| `FEE_STRUCTURE_ACTIVE_EXISTS` | A student already has an active fee structure for this billing context |
| `INVOICE_NEGATIVE_TOTAL` | Invoice total cannot be negative |

### Communication

| Code | Description |
|------|-------------|
| `NOTICE_AUDIENCE_EMPTY` | Audience resolution returned zero recipients |
| `TEMPLATE_VARIABLE_UNRESOLVED` | Template contains unresolved placeholder variables |
| `NOTICE_ALREADY_SENT` | Notice has already been dispatched and cannot be modified |
| `CHANNEL_NOT_CONFIGURED` | Delivery channel is not configured for this tenant |

---

## Validation Error Detail Format

For 400 validation errors, the `details` array provides field-level information:

```json
{
  "statusCode": 400,
  "error": "BAD_REQUEST",
  "message": "Validation failed",
  "details": [
    {
      "field": "startDate",
      "code": "REQUIRED",
      "message": "Start date is required"
    },
    {
      "field": "endDate",
      "code": "MUST_BE_AFTER",
      "message": "End date must be after start date",
      "params": { "min": "startDate" }
    }
  ]
}
```

---

## Business Rule Violation Format (422)

For 422 errors from business rule violations:

```json
{
  "statusCode": 422,
  "error": "UNPROCESSABLE_ENTITY",
  "message": "Cannot withdraw student: no withdrawal reason provided",
  "code": "STUDENT_INVALID_STATUS_TRANSITION",
  "context": {
    "currentStatus": "active",
    "requestedStatus": "withdrawn",
    "requiredFields": ["withdrawalReason"]
  }
}
```

---

## Idempotent Operation Errors

For payment and financial operations that use idempotency keys:

```json
{
  "statusCode": 409,
  "error": "CONFLICT",
  "message": "A request with this idempotency key has already been processed",
  "code": "IDEMPOTENCY_KEY_EXISTS",
  "originalResponse": {
    "paymentId": "...",
    "status": "success"
  }
}
```

The client should use the `originalResponse` data instead of retrying.
