# System Context

## Overview

The platform is a multi-tenant K-12 School Management SaaS. A single deployment serves multiple school tenants with complete logical isolation. Each tenant accesses the platform through its own subdomain or custom domain.

---

## System Diagram

```
┌─────────────────────────────────────────────────────────────────────┐
│                        External Users                               │
│                                                                     │
│  ┌──────────────┐  ┌──────────────┐  ┌───────────────────────────┐ │
│  │  Web Browser │  │ Flutter App  │  │  Super Admin Dashboard    │ │
│  │  (Next.js)   │  │ (iOS/Android)│  │  (Web)                    │ │
│  └──────┬───────┘  └──────┬───────┘  └───────────┬───────────────┘ │
└─────────┼────────────────┼──────────────────────┼─────────────────┘
          │                │                        │
          ▼                ▼                        ▼
┌─────────────────────────────────────────────────────────────────────┐
│                     AWS CloudFront / ALB                            │
│                  (TLS termination, routing)                         │
└─────────────────────────────────┬───────────────────────────────────┘
                                  │
                    ┌─────────────▼──────────────┐
                    │     NestJS API Server       │
                    │     (ECS Fargate)           │
                    │                             │
                    │  ┌──────────────────────┐  │
                    │  │  Tenant Resolution   │  │
                    │  │  Middleware          │  │
                    │  └──────────────────────┘  │
                    │                             │
                    │  ┌──────────────────────┐  │
                    │  │  JWT Auth Guard      │  │
                    │  └──────────────────────┘  │
                    │                             │
                    │  ┌──────────────────────┐  │
                    │  │  RBAC Guard          │  │
                    │  └──────────────────────┘  │
                    │                             │
                    │  ┌──────────────────────┐  │
                    │  │  Module Controllers  │  │
                    │  │  (17 modules)        │  │
                    │  └──────────────────────┘  │
                    └─────────────┬───────────────┘
                                  │
          ┌───────────────────────┼──────────────────────────┐
          │                       │                          │
          ▼                       ▼                          ▼
┌─────────────────┐   ┌────────────────────┐   ┌────────────────────┐
│  PostgreSQL     │   │  AWS S3            │   │  External Services │
│  (RDS)          │   │  (File Storage)    │   │                    │
│                 │   │                    │   │  - Email (SES)     │
│  - RLS          │   │  /tenant-id/       │   │  - SMS Provider    │
│  - Partitioned  │   │    /students/      │   │  - WhatsApp API    │
│  - Indexed      │   │    /documents/     │   │  - Payment Gateway │
└─────────────────┘   │    /branding/      │   │  - Push (FCM/APNs) │
                      └────────────────────┘   └────────────────────┘
```

---

## Primary Actors

| Actor | Interface | Description |
|-------|-----------|-------------|
| Super Admin | Web (Next.js) | Platform operator managing all tenants |
| School Admin | Web (Next.js) | School-level administrator |
| Principal | Web (Next.js) | Academic oversight user |
| Teacher | Web + Flutter | Daily operational user |
| Accountant | Web (Next.js) | Finance operations user |
| Parent | Flutter (primary), Web | Guardian self-service user |
| Student | Flutter (primary), Web | Learner self-service user |
| Transport Manager | Web | Transport operations |
| Librarian | Web | Library circulation |

---

## External Dependencies

| Service | Purpose | Module |
|---------|---------|--------|
| AWS SES | Transactional email delivery | M7.10 Communication Hub |
| SMS Provider (e.g., MSG91, Twilio) | SMS delivery for alerts and OTP | M7.10 Communication Hub |
| WhatsApp Business API | WhatsApp message delivery | M7.10 Communication Hub |
| Firebase Cloud Messaging (FCM) | Push notifications for Android | M7.9/M7.10 |
| Apple Push Notification Service (APNs) | Push notifications for iOS | M7.9/M7.10 |
| Payment Gateway (Razorpay or similar) | Online fee payment processing | M7.8 Fees and Finance |
| AWS S3 | Document and file storage | M7.4, M7.8, M7.7, M7.9 |
| AWS CloudFront | CDN for static assets and S3 presigned URLs | All file access |

---

## Tenant Request Flow

Every authenticated request carries the tenant context through the following steps:

1. **Domain Resolution** — Middleware extracts the subdomain or custom domain from the request host header and resolves `tenant_id` from the `tenant_domains` table.

2. **Authentication** — JWT is verified. The token includes `user_id`, `tenant_id`, and `membership_id` claims. Tokens without a tenant claim are valid only for super-admin platform operations.

3. **Session Validation** — The session record is checked against `auth_sessions`. If the tenant is suspended, all sessions for that tenant are immediately invalid.

4. **Subscription Gating** — The tenant's subscription status is checked. Expired or cancelled subscriptions block business module access. Suspended subscriptions return read-only responses.

5. **RBAC Authorization** — The user's permissions are resolved from their membership roles. Permission checks happen at the guard layer before the controller.

6. **Database Context** — `current_setting('app.tenant_id')` is set for the session. PostgreSQL Row-Level Security (RLS) policies use this setting to automatically filter all queries.

---

## Data Isolation Model

```
Platform Level (Global)
├── users
├── permissions
├── subscription_plans
└── platform_features

Tenant Level (Isolated)
├── tenants
├── tenant_domains
├── tenant_settings
├── subscriptions
├── user_tenant_memberships
├── roles (tenant-scoped)
├── [all business module tables with tenant_id]
└── audit_logs (per tenant)
```

No tenant can access another tenant's data. The RLS policies on all tenant-owned tables enforce this at the database level as a second line of defense.

---

## File Storage Organization

All files are stored in AWS S3 with tenant-partitioned key prefixes:

```
s3://platform-files/
  {tenant_id}/
    students/
      {student_id}/
        documents/
          {document_id}/
            v1/original_filename.pdf
            v2/original_filename.pdf
    branding/
      logos/
      banners/
    report-cards/
      {academic_year_id}/
        {student_id}/
          v1/report_card.pdf
    assignments/
      {assignment_id}/
        attachments/
        submissions/
          {student_id}/
```

Presigned URLs with short expiry (15 minutes) are used for all file access. Files are never served directly from public URLs except for branding assets (logo/favicon) which are cached via CloudFront.
