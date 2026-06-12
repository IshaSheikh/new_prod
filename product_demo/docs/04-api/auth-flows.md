# Auth Flows

Authentication and authorization flows for the platform. All flows use JWT + Refresh Tokens as defined in ADR-002.

---

## 1. Standard Login Flow

For users who belong to exactly one tenant:

```
POST /auth/login
{
  "email": "teacher@greenfield.edu",
  "password": "..."
}

Response:
{
  "user": { "id": "...", "firstName": "Meena", "email": "..." },
  "memberships": [
    { "tenantId": "...", "tenantName": "Greenfield School", "roles": ["teacher"] }
  ]
}
```

Since this user has only one active membership, the client immediately calls:

```
POST /auth/select-tenant
{
  "tenantId": "..."
}

Response:
{
  "accessToken": "eyJ...",
  "refreshToken": "opaque-token-here",
  "expiresIn": 900
}
```

---

## 2. Multi-School User Flow

For users who belong to multiple tenants (e.g., a teacher at two schools):

```
POST /auth/login → returns list of memberships

Client shows school selector UI:
  "You belong to multiple schools. Please select:"
  1. Greenfield School (Teacher)
  2. Riverside Academy (Teacher, Librarian)

POST /auth/select-tenant { "tenantId": "riverside-uuid" }
→ Returns access token with tenant_id: riverside-uuid
```

---

## 3. Token Structure

### Access Token (JWT, 15 minutes)

```json
{
  "sub": "user-uuid",
  "tenant_id": "tenant-uuid",
  "membership_id": "membership-uuid",
  "session_id": "session-uuid",
  "roles": ["teacher"],
  "iat": 1234567890,
  "exp": 1234567890
}
```

### Refresh Flow

```
POST /auth/refresh
Authorization: Bearer <expired-access-token>
Cookie: refreshToken=<opaque-refresh-token>

Response:
{
  "accessToken": "eyJ...",
  "refreshToken": "new-opaque-token",
  "expiresIn": 900
}
```

Refresh tokens are rotated on every use. If an old refresh token is presented after rotation, all sessions for that user are revoked (replay attack detection).

---

## 4. Password Reset Flow

```
Step 1: Request reset
POST /auth/password-reset/request
{ "email": "user@school.edu" }
→ Sends reset email with link containing token
→ Token expires in 30 minutes
→ Only latest token is valid (previous tokens revoked)

Step 2: Confirm reset
POST /auth/password-reset/confirm
{
  "token": "reset-token-here",
  "newPassword": "..."
}
→ Token verified and marked as used
→ Password updated
→ All active sessions revoked (security measure)
```

---

## 5. Portal Login Flow

Parent and student portal users use a separate login endpoint. The portal is a limited subset of the platform:

```
POST /api/v1/portal/auth/login
{
  "email": "parent@example.com",
  "password": "..."
}

Response:
{
  "portalUser": { "id": "...", "roleType": "parent" },
  "children": [
    { "studentId": "...", "name": "Arjun Menon", "grade": "Grade 1", "section": "A" }
  ],
  "accessToken": "eyJ...",
  "refreshToken": "..."
}
```

Portal access tokens carry `roleType: "parent"` or `roleType: "student"` instead of school role arrays.

---

## 6. Invitation Acceptance Flow

```
Step 1: Click invitation link
GET /invitations/accept?token=<invitation-token>
→ Validates token is valid and not expired
→ Shows account creation or login form

Step 2a: New user (email not already on platform)
POST /invitations/accept
{
  "token": "...",
  "firstName": "...",
  "lastName": "...",
  "password": "..."
}
→ Creates user account
→ Creates membership
→ Assigns invited roles
→ Returns access token

Step 2b: Existing user (email already on platform)
POST /invitations/accept
{
  "token": "...",
  "password": "..."  // authenticates existing user
}
→ Creates membership for existing user
→ Assigns invited roles
→ Returns access token
```

---

## 7. Session Management

### List Active Sessions

```
GET /auth/sessions
Authorization: Bearer <access-token>

Response:
[
  {
    "id": "session-uuid",
    "deviceInfo": "Chrome on MacOS",
    "ipAddress": "x.x.x.x",
    "lastSeenAt": "...",
    "expiresAt": "...",
    "current": true
  }
]
```

### Revoke a Specific Session

```
POST /auth/sessions/{sessionId}/revoke
Authorization: Bearer <access-token>
```

### Logout (Revoke Current Session)

```
POST /auth/logout
Authorization: Bearer <access-token>
```

---

## 8. Tenant Suspension — Session Invalidation

When a Super Admin suspends a tenant:

1. All `auth_sessions` with `tenant_id = <suspended-tenant>` are updated to `status = 'revoked'`
2. Existing access tokens remain valid for up to 15 minutes (their remaining TTL)
3. On the next refresh attempt, the suspended status is detected and the session is blocked
4. Any request with a valid access token checks subscription status via the subscription guard — suspended tenants return read-only or blocked responses immediately

---

## 9. Guard Chain (NestJS)

```typescript
// Applied to every authenticated route
@UseGuards(JwtAuthGuard, TenantGuard, SubscriptionGuard, RoleGuard, PermissionGuard)
```

Guards execute in order:

1. **JwtAuthGuard** — verifies JWT signature, checks expiry, extracts claims
2. **TenantGuard** — validates tenant_id from JWT matches request host's tenant; checks session is active
3. **SubscriptionGuard** — checks subscription status; applies read-only mode if suspended
4. **RoleGuard** — validates user has required role for the endpoint
5. **PermissionGuard** — validates user has the specific permission code for the action

Each guard throws the appropriate HTTP exception on failure:
- `401 Unauthorized` — missing or invalid token
- `403 Forbidden` — valid token but insufficient permissions
- `402 Payment Required` — subscription expired (with upgrade link)
