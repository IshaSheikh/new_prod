<!-- Source: PRDs | Last Verified: 2026-06-28 | Owner: Engineering -->
# Product Thesis

## What We're Building

A multi-tenant K-12 School Management SaaS platform that allows a single operator to serve hundreds of schools securely, with complete data isolation between them. The platform digitizes the full lifecycle of school operations — from tenant onboarding through admissions, academic structure, attendance, gradebook, fees, communication, parent/student portals, transport, library, HR/payroll, and analytics.

## The Core Problem

K-12 schools in emerging markets manage their operations across spreadsheets, paper registers, WhatsApp groups, and disconnected software tools. This creates:

- **Inconsistent data** — student records exist in multiple places with no single source of truth
- **Operational inefficiency** — attendance, fee collection, report cards, and timetables require excessive manual work
- **Poor parent transparency** — parents receive information late, informally, and unreliably
- **Compliance risk** — audit trails don't exist, financial records can't be reconstructed, student histories are fragmented
- **Scaling bottleneck** — every new school requires vendor re-implementation because no self-service setup exists

## Our Thesis

Schools don't need another app. They need an **operating system for school administration** — one that is:

1. **Self-service from day one** — a school signs up, configures its academic structure, and starts recording attendance, collecting fees, and communicating with parents on the same day
2. **Multi-tenant by architecture** — complete data isolation, tenant-aware APIs, subdomain/custom domain routing, and row-level security in PostgreSQL
3. **Modular by design** — schools adopt the platform at their own pace, module by module; every module is independently useful but becomes more valuable when combined
4. **Audit-first** — every action that matters is logged immutably; receipts can't be edited; attendance can't be modified without approval workflows; academic records are versioned
5. **Parent-transparent** — parents have a portal that exposes attendance, grades, timetables, fee balances, and receipts without requiring a phone call to the school office
6. **Affordable for the market** — priced for commercial K-12 schools in Tier 1–3 cities, not enterprise schools with dedicated IT teams

## Why This Works

The wedge is **fee collection and parent communication** — two pain points every school experiences daily, every principal cares about, and every parent notices. Schools that start with fees + parent portal quickly see value and expand to attendance, gradebook, and beyond.

The business model scales because:
- Tenant isolation means one deployment serves unlimited schools
- Self-service onboarding removes implementation cost
- Module expansion increases ARPU without proportional cost growth
- Data lock-in increases retention (historical records, financial history, multi-year student data)

## The Platform Stack

- **Backend:** NestJS on Node.js
- **Database:** PostgreSQL with Row-Level Security (RLS)
- **Frontend Web:** Next.js
- **Mobile:** Flutter
- **Infrastructure:** AWS (S3 for files, SES/SNS for communication, ECS/RDS for compute/data)
- **Auth:** JWT + Refresh Tokens with tenant-context claims

## Long-Term Vision

Starting with K-12 schools in India and expanding to similar markets where school management is fragmented, paper-heavy, and underserved by existing software. The platform becomes the default operating system for a school — the system of record for students, staff, finances, and communication — making it deeply embedded in school operations and highly defensible.

