# Wedge MVP

## The Wedge Hypothesis

The fastest path to adoption and retention in K-12 SaaS is through the two pain points every school experiences every single day: **fee collection** and **parent communication**.

A school that starts collecting fees digitally and giving parents a portal to view attendance, dues, and receipts sees immediate value — and that value is visible to parents, principals, and owners simultaneously.

## MVP Scope

The Wedge MVP includes the modules required to get a school operational from sign-up to collecting fees and communicating with parents **within one day**.

### Core MVP Modules (Must Ship Together)

| # | Module | Why It's Required |
|---|--------|------------------|
| M7.1 | Multi-Tenant Platform Core | Foundation — no other module works without tenant isolation, RBAC, and subscriptions |
| M7.2 | School Onboarding and Configuration | Required before any operation can start — academic year, grades, sections, campuses |
| M7.3 | User Management and RBAC | Required for inviting staff, assigning roles, and controlling access |
| M7.4 | Student Information System (SIS) | Required before fees can be assigned or attendance can be taken |
| M7.8 | Fees and Finance | Core wedge — invoice generation, payment collection, receipts, and parent visibility |
| M7.9 | Parent and Student Portal | Core wedge — parent self-service reduces front-office burden and drives adoption |
| M7.10 | Communication Hub | Required for fee reminders, absence alerts, and notices |

### Early Expansion Modules (Ships Shortly After Wedge)

| # | Module | Why It Follows |
|---|--------|---------------|
| M7.5 | Academic Structure and Timetable | Required before period-wise attendance or subject-level grades work |
| M7.6 | Attendance Management | High daily frequency — drives daily teacher engagement |
| M7.7 | Gradebook and Report Cards | Major value for parents — report card access drives portal adoption |

## What a School Can Do on Day 1

After completing the onboarding wizard and inviting staff:

1. **Configure** academic year, grade structure, and sections
2. **Add students** with guardian relationships
3. **Assign fee structures** and **generate invoices** for the current term
4. **Record payments** from parents and issue receipts
5. **Invite parents** to the portal to view dues and pay online
6. **Send a welcome notice** to all parents

On Day 2 and beyond:
- Teachers take attendance every morning
- Parents receive absence alerts automatically
- Accountant sees defaulters dashboard in real-time
- Principal views daily attendance and fee collection KPIs

## What Is Explicitly Out of Scope for MVP

The following modules are valuable but not required to deliver the wedge:

- M7.11 — Admissions Workflow (useful but schools can start with existing students)
- M7.12 — Assignments and Homework (useful but not required for core operations)
- M7.13 — Exams and Publishing (useful during exam season, not for day-one operations)
- M7.14 — Transport Management (optional module, not all schools need it)
- M7.15 — Library (optional expansion module)
- M7.16 — HR and Payroll (complex ERP module, later stage)
- M7.17 — Analytics and KPI Dashboards (built on top of operational data, Phase 2)

## Success Criterion for MVP

> A school signs up in the morning, completes onboarding, and has parents viewing attendance, paying fees, and receiving notices by the afternoon of the same day.

## MVP Implementation Sequence

```
Week 1-2: M7.1 — Platform Core (tenant isolation, auth, RBAC, subscriptions)
Week 3-4: M7.2 — School Onboarding (academic year, grades, sections, campuses)
Week 5-6: M7.3 — User Management (invitations, roles, parent-student links)
Week 7-8: M7.4 — Student Information System (students, enrollments, guardians)
Week 9-11: M7.8 — Fees and Finance (structures, invoices, payments, receipts)
Week 12-13: M7.9 — Parent Portal (dashboard, attendance, fees, notices)
Week 14: M7.10 — Communication Hub (notices, alerts, fee reminders)
Week 15-16: M7.6 — Attendance Management
Week 17-18: M7.5 — Academic Structure and Timetable
Week 19-21: M7.7 — Gradebook and Report Cards
```

## Key Constraints

- **Self-service only** — no vendor implementation support required
- **Mobile-first parent experience** — Flutter app must work on low-end Android devices
- **Single deployment per region** — all tenants share infrastructure with strict logical isolation
- **India-first** — INR currency, Indian phone number formats, Indian academic calendar defaults, IST timezone as default
