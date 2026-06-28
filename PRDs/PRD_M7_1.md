# Product Requirements Document (PRD)

# Module 7.1 – Multi-Tenant Platform Core

# 1. Module Objective

Build the foundational multi-tenant architecture that enables a single SaaS platform to securely serve multiple K-12 schools while ensuring complete tenant isolation, centralized platform administration, subscription management, role-based access control, and auditability.

This module is the backbone of the entire school management system and governs:

* School onboarding
* User access control
* Tenant isolation
* Subscription lifecycle
* Security and compliance
* Cross-school scalability

Without this module, no other business modules (attendance, grades, fees, timetable, communication) can operate safely in a multi-school environment.

# 2. Why Schools Need It

Schools require:

### Secure Data Separation

Each school's data must remain completely isolated from other schools.

### Centralized Administration

Platform operators need a single control center for managing all schools.

### Controlled User Access

Different staff members require different permissions.

### Subscription Management

Schools subscribe to plans and features.

### Compliance and Accountability

All critical actions must be traceable through audit logs.

### Scalability

The platform should support:

* 10 schools
* 100 schools
* 1,000 schools

without architectural changes.


# 3. Primary Users

## Platform Level

### Super Admin

Platform owner/operator.

Responsibilities:

* Create tenants
* Manage subscriptions
* Suspend tenants
* Manage tenants’ members
* View platform analytics
* Manage platform-wide settings

## Tenant Level
### School Admin

Responsible for school-level administration.

Responsibilities:

* Manage users
* Assign roles
* Configure school settings
* View & request subscriptions changes

### Principal

Limited administrative access.

Responsibilities:

* View staff
* Monitor operations
* Access reports


### Teacher

Access only educational data within tenant.
Access students and academic details 

### Accountant

Access finance-related modules only.

### Parent

Access child-related records only.

### Student

Access personal records only.


# 4. User Stories

## Tenant Creation

### US-001

As a Super Admin,
I want to create a new school tenant,
so that the school can start using the platform.

### US-002

As a Super Admin,
I want to assign a subdomain,
so that users can access their school portal.

Example:
greenfield.schoolsaas.com

### US-003

As a Super Admin,
I want to assign a custom domain,
so that schools can use their own branding.

Example:
portal.greenfieldschool.edu

## Membership Management

### US-004

As a School Admin,
I want to invite staff users,
so they can access the platform.

### US-005

As a School Admin,
I want to assign roles,
so users receive correct permissions.

### US-006

As a School Admin,
I want to deactivate memberships,
so former employees lose access.

## Subscription Management

### US-007

As a Super Admin,
I want to activate subscriptions,
so schools can use premium features.

### US-008

As a Super Admin,
I want to suspend subscriptions,
so expired schools lose access.

## Audit Logging

### US-009

As a School Admin,
I want to view audit logs,
so I can investigate changes.

### US-010

As a Super Admin,
I want to see platform-wide logs,
so I can monitor security events.

# 5. Business Rules

## Tenant Isolation

BR-001

Every business table must contain:

sql
tenant_id UUID NOT NULL

except approved global tables.

BR-002

No user may access records belonging to another tenant.

BR-003

All API queries must automatically filter by tenant.

BR-004

Cross-tenant joins are prohibited.


## Membership Rules

BR-005

A user may belong to multiple tenants.

Example:

Teacher works at two schools.
A parents may have two different kids at two different institutes(students will use parents contact).


BR-006

Permissions are granted through memberships.

Not directly through users.

---

BR-007

Users must have at least one active membership to access a tenant.

---

## Role Rules

BR-008

Super Admin is platform-wide only.

---

BR-009

School Admin is tenant-specific.

---

BR-010

Roles are assigned through memberships.

---

## Audit Rules

BR-011

Audit logs are immutable.

---

BR-012

Audit records cannot be edited.

---

BR-013

Sensitive operations must generate audit entries.

Examples:

* User creation
* User deletion
* Role assignment
* Subscription changes
* Tenant suspension

---

## Subscription Rules

BR-014

Only active subscriptions can access business modules.

---

BR-015

Expired subscriptions enter grace period.

---

BR-016

Suspended subscriptions become read-only.

---

# 6. State Machine

## Tenant Lifecycle

```text

Draft
  |
  v
Provisioning
  |
  v
Active
  |
  +----> Suspended
  |              |
  |              v
  |           Reactivated
  |
  +-------> Cancelled
```

---

### Tenant States

#### Draft

Tenant created but incomplete.

---

#### Provisioning

Domain setup and onboarding in progress.

---

#### Active

Fully operational.

---

#### Suspended

Access restricted.

Read-only mode.

---

#### Cancelled

Subscription terminated.

Data retained according to policy.

---

## Membership Lifecycle

```text
Invited
   |
   v
Pending Acceptance
   |
   v
Accepted
   |
   v
Active
   |
   +-----> Disabled
   |
   +-----> Removed
```

---

## Subscription Lifecycle

```text
Trial
  |
  v
Active
  |
  +-----> Grace Period
  |             |
  |             v
  |         Active
  |
  +-----> Expired
  |
  +-----> Cancelled
```

---

# 7. Required Data Entities

## tenants

| Field      | Type      |
| -------| ------|
| id         | UUID      |
| name       | VARCHAR   |
| code       | VARCHAR   |
| status     | ENUM      |
| timezone   | VARCHAR   |
| Address    | VARCHAR   |
| created_at | TIMESTAMP |
| updated_at | TIMESTAMP |

---

## tenant_domains

| Field               | Type    |
| ----------------| ----|
| id                  | UUID    |
| tenant_id           | UUID    |
| domain              | VARCHAR |
| is_primary          | BOOLEAN |
| verification_status | ENUM    |

---

## subscription_plans

| Field         | Type    |
| ----------| ------- |
| id            | UUID    |
| name          | VARCHAR |
| max_students  | INTEGER |
| max_staff     | INTEGER |
| monthly_price | DECIMAL |
| annual_price  | DECIMAL |
| feature_flags | JSONB   |



## subscriptions

| Field       | Type |
| ----------- | ---- |
| id          | UUID |
| tenant_id   | UUID |
| plan_id     | UUID |
| status      | ENUM |
| start_date  | DATE |
| end_date    | DATE |
| grace_until | DATE |

---

## users

| Field         | Type      |
| ------------- | --------- |
| id            | UUID      |
| email         | VARCHAR   |
| mobile        | VARCHAR   |
| first_name    | VARCHAR   |
| last_name     | VARCHAR   |
| status        | ENUM      |
| last_login_at | TIMESTAMP |

---

## user_tenant_memberships

| Field     | Type      |
| --------- | --------- |
| id        | UUID      |
| user_id   | UUID      |
| tenant_id | UUID      |
| role_id   | UUID      |
| status    | ENUM      |
| joined_at | TIMESTAMP |

---

## roles

| Field                | Type    |
| -------------------- | ------- |
| id                   | UUID    |
| tenant_id (nullable) | UUID    |
| name                 | VARCHAR |
| role_type            | ENUM    |

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

## audit_logs

| Field         | Type      |
| ------------- | --------- |
| id            | UUID      |
| tenant_id     | UUID      |
| actor_user_id | UUID      |
| action        | VARCHAR   |
| entity_type   | VARCHAR   |
| entity_id     | UUID      |
| old_value     | JSONB     |
| new_value     | JSONB     |
| ip_address    | VARCHAR   |
| user_agent    | TEXT      |
| created_at    | TIMESTAMP |

---

# 8. Permissions Matrix

| Action               | Super Admin | School Admin | Principal | Teacher | Accountant | Parent | Student |
| -------------------- | ----------- | ------------ | --------- | ------- | ---------- | ------ | ------- |
| Create Tenant        | ✓           | ✗            | ✗         | ✗       | ✗          | ✗      | ✗       |
| Suspend Tenant       | ✓           | ✗            | ✗         | ✗       | ✗          | ✗      | ✗       |
| Manage Subscription  | ✓           | ✗            | ✗         | ✗       | ✗          | ✗      | ✗       |
| Invite User          | ✗           | ✓            | ✗         | ✗       | ✗          | ✗      | ✗       |
| Assign Role          | ✗           | ✓            | ✗         | ✗       | ✗          | ✗      | ✗       |
| Remove Membership    | ✗           | ✓            | ✗         | ✗       | ✗          | ✗      | ✗       |
| View Tenant Settings | ✓           | ✓            | ✓         | ✗       | ✗          | ✗      | ✗       |
| Edit Tenant Settings | ✗           | ✓            | ✗         | ✗       | ✗          | ✗      | ✗       |
| View Audit Logs      | ✓           | ✓            | Limited   | ✗       | ✗          | ✗      | ✗       |
| Export Audit Logs    | ✓           | ✓            | ✗         | ✗       | ✗          | ✗      | ✗       |

---

# 9. APIs Needed

## Tenant APIs

### POST /platform/tenants

Create tenant

---

### GET /platform/tenants

List tenants

---

### GET /platform/tenants/{tenantId}

Tenant details

---

### PATCH /platform/tenants/{tenantId}

Update tenant

---

### POST /platform/tenants/{tenantId}/suspend

Suspend tenant

---

### POST /platform/tenants/{tenantId}/activate

Activate tenant

---

## Domain APIs

### POST /platform/tenants/{tenantId}/domains

Add domain

---

### DELETE /platform/domains/{domainId}

Remove domain

---

## Membership APIs

### POST /tenants/{tenantId}/members/invite

Invite member

---

### GET /tenants/{tenantId}/members

List members

---

### PATCH /memberships/{membershipId}

Update membership

---

### DELETE /memberships/{membershipId}

Remove membership

---

## Role APIs

### POST /roles

Create role

---

### POST /roles/{roleId}/permissions

Assign permissions

---

### POST /memberships/{membershipId}/role

Assign role

---

## Subscription APIs

### POST /subscriptions

Create subscription

---

### POST /subscriptions/{id}/activate

Activate

---

### POST /subscriptions/{id}/cancel

Cancel

---

### GET /subscriptions

List subscriptions

---

## Audit APIs

### GET /audit-logs

List logs

---

### GET /audit-logs/export

Export logs

---

# 10. Screens Needed

## Super Admin Dashboard

Displays:

* Total schools
* Active schools
* Suspended schools
* Revenue overview
* Subscription health

---

## Tenant Onboarding Wizard

Steps:

1. School information
2. Domain setup
3. Subscription selection
4. Admin invitation
5. Confirmation

---

## Tenant Settings

Sections:

* School profile
* Domain settings
* Security settings
* Branding

---

## Membership Management

Features:

* User search
* Invite user
* Role assignment
* Deactivate member

---

## Role Management

Features:

* Create role
* Assign permissions
* Clone role

---

## Subscription Management

Features:

* Plan details
* Renewal dates
* Billing status

---

## Audit Log Viewer

Filters:

* Date range
* User
* Action
* Entity type

---

# 11. Edge Cases

### EC-001

User belongs to multiple schools.

System must force tenant context selection.

---

### EC-002

School changes custom domain.

DNS verification must be repeated.

---

### EC-003

Subscription expires during school hours.

System enters grace period.

---

### EC-004

School Admin deletes own membership.

Prevent self-lockout.

---

### EC-005

Last School Admin removed.

Operation blocked.

At least one active School Admin required.

---

### EC-006

Duplicate domain registration.

Domain must be globally unique.

---

### EC-007

Concurrent role updates.

Use optimistic locking/versioning.

---

### EC-008

Tenant suspended with active sessions.

Immediately invalidate all sessions.

---

# 12. Risks

### Security Risk

Tenant data leakage.

Mitigation:

* PostgreSQL Row-Level Security (RLS)
* Tenant-scoped repositories
* Automated security tests

---

### Authorization Risk

Privilege escalation.

Mitigation:

* RBAC
* Permission middleware
* Audit monitoring

---

### Scalability Risk

Large tenant counts.

Mitigation:

* Proper indexing
* Partitioned audit tables
* Read replicas

---

### Compliance Risk

Insufficient auditability.

Mitigation:

* Immutable audit logs
* Export capabilities

---

# 13. Success Metrics

### Operational Metrics

* Tenant creation < 5 minutes
* Admin invitation success > 98%
* Domain verification success > 95%

### Security Metrics

* Cross-tenant access incidents = 0
* Unauthorized access incidents = 0

### Reliability Metrics

* Tenant provisioning success > 99.9%
* Role assignment success > 99.9%

### Business Metrics

* New school onboarding completed within 1 day
* Subscription activation within 10 minutes
* Churn caused by access issues < 1%

---

# 14. Out of Scope

The following are not part of the Multi-Tenant Platform Core module:

* Attendance management
* Student information system
* Gradebook
* Exam management
* Timetable generation
* Fee collection workflows
* Parent communication workflows
* Learning management features
* Payroll
* Library management
* Transport management
* Mobile application features
* Analytics and BI dashboards beyond tenant administration

---

## Technical Architecture Decisions (Mandatory)

For your stack (Next.js + NestJS + PostgreSQL + AWS + Flutter), I recommend:

* **Tenant isolation:** PostgreSQL Row-Level Security (RLS) + mandatory `tenant_id`
* **Authentication:** JWT + Refresh Tokens
* **Authorization:** RBAC with permissions table
* **Audit logging:** Event-driven audit service in NestJS
* **Domain routing:** Middleware-based tenant resolution from subdomain/custom domain
* **Soft deletes:** Use `deleted_at` instead of hard delete for critical entities
* **Tenant context:** Required in every API request and stored in JWT claims after tenant selection
* **Indexes:** `(tenant_id, id)` and `(tenant_id, created_at)` on all business tables
* **Infrastructure:** Separate S3 prefixes, cache namespaces, and queue namespaces per tenant

This module should be implemented first before any attendance, grades, fees, or timetable functionality.
