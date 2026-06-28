# Product Requirements Document (PRD)

# Module 7.3 – User Management and Role-Based Access Control (RBAC)

---

# 1. Module Objective

The User Management and RBAC module controls identity, authentication, authorization, and relationship management across the entire K-12 platform.

Its purpose is to ensure that every user only sees and performs actions appropriate to their role, school, and assigned responsibilities while maintaining strict tenant isolation.

This module is a foundational platform service that all business modules (attendance, grades, fees, timetable, parent communication, transport, library, reporting) depend on.

The module must support:

* User lifecycle management
* Role assignment
* Permission enforcement
* Parent-student relationships
* Teacher-class-subject relationships
* Multi-school memberships
* Secure authentication
* Auditability

---

# 2. Why Schools Need It

Schools operate with highly sensitive student and financial information.

Without strong RBAC:

* Teachers could access unrelated students
* Parents could see other children's records
* Accountants could access confidential academic notes
* Former employees might retain access
* Data privacy regulations could be violated

Schools need RBAC to:

### Protect Student Data

Ensure only authorized users can access student records.

### Reduce Administrative Work

Automatically assign permissions through roles.

### Improve Parent Transparency

Give parents access only to their children.

### Improve Operational Efficiency

Teachers see only relevant classes and subjects.

### Support Compliance

Maintain audit trails and controlled access.

### Enable Growth

Allow schools to add staff without manually configuring permissions.

---

# 3. Primary Users

## Platform Super Admin

Platform-wide authority.

Responsibilities:

* Manage tenant administrators
* Monitor access policies
* Investigate security issues

---

## School Admin

Primary owner of user management inside a school.

Responsibilities:

* Invite users
* Assign roles
* Disable access
* Manage relationships

---

## Principal

Academic oversight.

Responsibilities:

* View staff access
* Review assignments

---

## Teacher

Academic user.

Responsibilities:

* Access assigned classes
* Manage attendance and grades

---

## Accountant

Finance user.

Responsibilities:

* Access fee and payment data

---

## Parent

Guardian user.

Responsibilities:

* Access linked students

---

## Student

Learner user.

Responsibilities:

* Access own records

---

## Transport Manager

Transportation operations.

Responsibilities:

* Manage routes and vehicles

---

## Librarian

Library operations.

Responsibilities:

* Manage books and circulation

---

# 4. User Stories

## User Provisioning

### US-001

As a School Admin,

I want to invite a user,

so they can access the platform.


---

### US-002

As a School Admin,

I want to resend invitations,

so users who missed emails can join.

---

### US-003

As a School Admin,

I want to disable users,

so former employees lose access.

---

## Role Management

### US-004

As a School Admin,

I want to assign roles,

so users receive correct permissions.

---

### US-005

As a School Admin,

I want to change roles,

so responsibilities can evolve.

---

## Parent Management

### US-006

As a School Admin,

I want to link parents to students,

so guardians can access their children.

---

### US-007

As a Parent,

I want to see all my children,

so I can monitor them from one account.

---

## Teacher Assignment

### US-008

As a School Admin,

I want to assign teachers to classes,

so access is automatically scoped.

---

### US-009

As a Teacher,

I want to view only assigned classes,

so my dashboard remains relevant.

---

## Authentication

### US-010

As a User,

I want to reset my password,

so I can regain access.

---

### US-011

As a User,

I want secure login,

so my account remains protected.

---

## Security

### US-012

As a Super Admin,

I want audit logs for permission changes,

so unauthorized access can be investigated.

---

# 5. Business Rules

## Tenant Isolation

BR-001

Every user action must execute within a tenant context.

---

BR-002

Users cannot access data outside their tenant membership.

---

BR-003

Cross-tenant user visibility is prohibited except for Super Admin.

---

## User Rules

BR-004

Users are platform identities.

A user may belong to multiple schools.

---

BR-005

Email addresses must be globally unique.

---

BR-006

Mobile numbers should be unique where possible.

---

BR-007

Inactive users cannot authenticate.

---

## Role Rules

BR-008

Every membership must have at least one role.

---

BR-009

A membership may hold multiple roles.

Example:

Teacher + Librarian

---

BR-010

Permissions are inherited through assigned roles.

---

BR-011


---

## Parent Rules

BR-012

Parents may be linked to multiple students.

---

BR-013

A Students  should be linked to a single guardian.

---

BR-014

Parents only see linked students.

---

## Teacher Rules

BR-015

Teachers only see assigned classes.

---

BR-016

Teachers only see assigned subjects.

---

BR-017

Teachers cannot access unassigned student records.

---

## Accountant Rules

BR-018

Accountants access financial data only.

---

BR-019

Accountants cannot view confidential academic evaluation notes.

---

## Security Rules

BR-020

Password reset tokens expire after 30 minutes.

---

BR-021

Invitation links expire after 7 days.

---

BR-022

Role changes require audit logging.

---

BR-023

User deactivation immediately invalidates active sessions.

---

# 6. State Machine

## User Lifecycle

```text
Invited
   |
   v
Pending Acceptance
   |
   v
Active
   |
   +-------> Suspended
   |
   +-------> Disabled
   |
   +-------> Removed
```

---

## Membership Lifecycle

```text
Pending
   |
   v
Active
   |
   +-------> Inactive
   |
   +-------> Removed
```

---

## Password Reset Lifecycle

```text
Requested
   |
   v
Token Issued
   |
   +-------> Expired
   |
   +-------> Used
```

---

## Parent-Student Link Lifecycle

```text
Requested
   |
   v
Verified
   |
   v
Active
   |
   +-------> Revoked
```

---

# 7. Required Data Entities

## users

| Field         | Type      |
| ------------- | --------- |
| id            | UUID      |
| email         | VARCHAR   |
| mobile        | VARCHAR   |
| first_name    | VARCHAR   |
| last_name     | VARCHAR   |
| password_hash | TEXT      |
| status        | ENUM      |
| last_login_at | TIMESTAMP |
| created_at    | TIMESTAMP |

---

## user_tenant_memberships

| Field     | Type      |
| --------- | --------- |
| id        | UUID      |
| user_id   | UUID      |
| tenant_id | UUID      |
| status    | ENUM      |
| joined_at | TIMESTAMP |

---

## roles

| Field     | Type      |
| --------- | --------- |
| id        | UUID      |
| tenant_id | UUID NULL |
| name      | VARCHAR   |
| role_type | ENUM      |

---

## permissions

| Field       | Type    |
| ----------- | ------- |
| id          | UUID    |
| code        | VARCHAR |
| module      | VARCHAR |
| description | TEXT    |

---

## role_permissions

| Field         | Type |
| ------------- | ---- |
| role_id       | UUID |
| permission_id | UUID |

---

## membership_roles

| Field         | Type |
| ------------- | ---- |
| membership_id | UUID |
| role_id       | UUID |

---

## invitations

| Field      | Type      |
| ---------- | --------- |
| id         | UUID      |
| tenant_id  | UUID      |
| email      | VARCHAR   |
| role_id    | UUID      |
| token      | TEXT      |
| expires_at | TIMESTAMP |
| status     | ENUM      |

---

## password_reset_tokens

| Field      | Type      |
| ---------- | --------- |
| id         | UUID      |
| user_id    | UUID      |
| token      | TEXT      |
| expires_at | TIMESTAMP |
| used_at    | TIMESTAMP |

---

## parent_student_links

| Field             | Type |
| ----------------- | ---- |
| id                | UUID |
| tenant_id         | UUID |
| parent_user_id    | UUID |
| student_user_id   | UUID |
| relationship_type | ENUM |
| status            | ENUM |

---

## teacher_assignments

| Field            | Type |
| ---------------- | ---- |
| id               | UUID |
| tenant_id        | UUID |
| teacher_user_id  | UUID |
| class_id         | UUID |
| section_id       | UUID |
| subject_id       | UUID |
| academic_year_id | UUID |

---

## access_audit_logs

| Field         | Type      |
| ------------- | --------- |
| id            | UUID      |
| tenant_id     | UUID      |
| actor_user_id | UUID      |
| action        | VARCHAR   |
| target_entity | VARCHAR   |
| target_id     | UUID      |
| metadata      | JSONB     |
| created_at    | TIMESTAMP |

---

# 8. Permissions Matrix

| Permission                 | Super Admin | School Admin | Principal | Teacher       | Accountant | Parent         | Student   | Transport Manager | Librarian |
| -------------------------- | ----------- | ------------ | --------- | ------------- | ---------- | -------------- | --------- | ----------------- | --------- |
| View Users                 | ✓           | ✓            | Limited   | ✗             | ✗          | ✗              | ✗         | ✗                 | ✗         |
| Create Users               | ✓           | ✓            | ✗         | ✗             | ✗          | ✗              | ✗         | ✗                 | ✗         |
| Edit Users                 | ✓           | ✓            | ✗         | ✗             | ✗          | ✗              | ✗         | ✗                 | ✗         |
| Disable Users              | ✓           | ✓            | ✗         | ✗             | ✗          | ✗              | ✗         | ✗                 | ✗         |
| Assign Roles               | ✓           | ✓            | ✗         | ✗             | ✗          | ✗              | ✗         | ✗                 | ✗         |
| View Student Data          | ✓           | ✓            | ✓         | Assigned Only | ✗          | Linked Only    | Self Only | ✗                 | ✗         |
| View Financial Data        | ✓           | ✓            | Limited   | ✗             | ✓          | Own Child Only | Own Only  | ✗                 | ✗         |
| View Academic Notes        | ✓           | ✓            | ✓         | Assigned Only | ✗          | Limited        | Own Only  | ✗                 | ✗         |
| Manage Parent Links        | ✓           | ✓            | ✗         | ✗             | ✗          | ✗              | ✗         | ✗                 | ✗         |
| Manage Teacher Assignments | ✓           | ✓            | ✓         | ✗             | ✗          | ✗              | ✗         | ✗                 | ✗         |
| Manage Library             | ✓           | ✓            | ✓         | ✗             | ✗          | ✗              | ✗         | ✗                 | ✓         |
| Manage Transport           | ✓           | ✓            | ✓         | ✗             | ✗          | ✗              | ✗         | ✓                 | ✗         |

---

# 9. APIs Needed

## User APIs

### POST /users/invite

Invite user

---

### POST /users/invitations/{token}/accept

Accept invitation

---

### GET /users

List users

---

### GET /users/{id}

Get user

---

### PATCH /users/{id}

Update user

---

### POST /users/{id}/disable

Disable user

---

### POST /users/{id}/enable

Enable user

---

## Authentication APIs

### POST /auth/login

Login

---

### POST /auth/logout

Logout

---

### POST /auth/refresh

Refresh token

---

### POST /auth/password-reset/request

Request password reset

---

### POST /auth/password-reset/confirm

Confirm password reset

---

## Role APIs

### GET /roles

List roles

---

### POST /roles

Create role

---

### PATCH /roles/{id}

Update role

---

### POST /users/{id}/roles

Assign role

---

### DELETE /users/{id}/roles/{roleId}

Remove role

---

## Parent Relationship APIs

### POST /parent-student-links

Create link

---

### GET /parent-student-links

List links

---

### DELETE /parent-student-links/{id}

Remove link

---

## Teacher Assignment APIs

### POST /teacher-assignments

Assign teacher

---

### GET /teacher-assignments

List assignments

---

### DELETE /teacher-assignments/{id}

Remove assignment

---

## Permission APIs

### GET /permissions

List permissions

---

### GET /roles/{id}/permissions

Role permissions

---

# 10. Screens Needed

## User Directory

Features:

* Search users
* Filter by role
* Filter by status
* Bulk actions

---

## User Detail Screen

Sections:

* Profile
* Memberships
* Roles
* Activity

---

## Invitation Management

Features:

* Invite user
* Resend invitation
* Cancel invitation

---

## Role Management Screen

Features:

* View roles
* Role permission matrix
* Clone role

---

## Parent-Student Linking Screen

Features:

* Search parent
* Search student
* Create relationship

---

## Teacher Assignment Screen

Features:

* Assign class
* Assign subject
* View workload

---

## Access Audit Screen

Features:

* Role change history
* User access changes
* Security activity

---

## User Profile Screen

Self-service features:

* Update profile
* Change password
* View login history

---

# 11. Edge Cases

### EC-001

Teacher assigned to multiple campuses.

System supports multiple assignments.

---

### EC-002

Parent has children in different grades.

Single login shows all linked children.

---

### EC-003

Parent linked twice to same student.

Duplicate relationship blocked.

---

### EC-004

Teacher removed from class with active attendance records.

Historical records remain intact.

---

### EC-005

Last School Admin disabled.

Operation blocked.

At least one active School Admin required.

---

### EC-006

Invitation sent to existing platform user.

System creates membership instead of new user.

---

### EC-007

User belongs to multiple schools.

Tenant selection required after login.

---

### EC-008

Role removed while user session active.

Permissions refreshed immediately.

---

### EC-009

Student becomes inactive.

Parent relationship remains for historical records.

---

### EC-010

Password reset requested multiple times.

Only latest token remains valid.

---

# 12. Risks

## Security Risk

Privilege escalation through misconfigured permissions.

Mitigation:

* Permission middleware
* RBAC validation tests
* Audit logging

---

## Data Privacy Risk

Parents viewing unrelated students.

Mitigation:

* Relationship-based filtering
* Tenant-level enforcement

---

## Operational Risk

Incorrect teacher assignments.

Mitigation:

* Assignment validation
* Approval workflows later

---

## Scalability Risk

Large schools with thousands of users.

Mitigation:

* Indexed user directory
* Pagination
* Caching

---

## Compliance Risk

Unauthorized access not traceable.

Mitigation:

* Immutable access audit logs

---

# 13. Success Metrics

## Security Metrics

* Cross-role access violations = 0
* Cross-tenant access violations = 0
* Unauthorized parent access incidents = 0

---

## Operational Metrics

* User invitation success rate > 98%
* Password reset success rate > 95%
* Role assignment completion < 30 seconds

---

## Product Metrics

* Teacher assignment accuracy > 99%
* Parent linkage completion > 95%

---

## Business Metrics

* Reduction in user administration workload > 60%
* Reduced support tickets related to access control
* Increased parent portal adoption

---

# 14. Out of Scope

The following are excluded from MVP:

* Attribute-Based Access Control (ABAC)
* Direct user-level permission overrides
* SSO (SAML/OAuth Enterprise)
* Biometric authentication
* Multi-factor authentication (Phase 2)
* Delegated parent access approval workflows
* External identity providers
* Organization hierarchy permissions
* Time-based permissions
* Dynamic policy engine
* Cross-school data sharing
* Advanced approval workflows

---

# Architecture Recommendations for Your Stack

### MVP Permission Strategy

Use a strict hierarchy:

```text
Role
   ↓
Permissions
   ↓
Module Access
   ↓
Data Scope Filters
```

Example:

```text
Teacher
  ↓
attendance.mark
grades.enter
grades.view
  ↓
Only Assigned Classes
```

### NestJS Implementation

Use:

```text
JWT Guard
   ↓
Tenant Guard
   ↓
Role Guard
   ↓
Permission Guard
   ↓
Resource Scope Guard
```

### PostgreSQL Design

Every user-related table must include:

```sql
tenant_id UUID NOT NULL
```

except truly global identity tables (`users`, `permissions`).

### MVP Recommendation

Do **not** build custom roles in V1.

Ship these fixed system roles:

* Super Admin
* School Admin
* Principal
* Teacher
* Accountant
* Parent
* Student
* Transport Manager
* Librarian

Add custom roles and permission overrides in Phase 2 after schools are actively using the platform. This dramatically reduces complexity while still meeting the needs of 95% of K-12 schools.
