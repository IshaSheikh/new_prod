# Dashboard Information Architecture

Information architecture for dashboards and navigation across all user roles.

---

## Role-Based Navigation

### Super Admin (Platform Level)

```
Platform Dashboard
├── Overview (KPIs: total tenants, active, revenue, subscription health)
├── Tenants
│   ├── All Tenants (list with status filters)
│   ├── Create Tenant
│   └── Tenant Detail
│       ├── School Info
│       ├── Onboarding Status
│       ├── Subscription
│       ├── Domains
│       └── Audit Logs
├── Subscriptions
│   ├── Plans
│   └── Active Subscriptions
└── Platform Audit Logs
```

---

### School Admin

```
Home (Operations Dashboard)
├── Attendance Today (% complete, missing submissions)
├── Fee Collection (today's collection, overdue total)
├── Pending Actions (corrections, leave approvals, support requests)
└── Recent Activity Feed

Students
├── Student List (search, filter)
├── Add Student
├── Student Profile
│   ├── Overview
│   ├── Profile
│   ├── Enrollment
│   ├── Guardians
│   ├── Documents
│   ├── Health
│   ├── Attendance
│   ├── Fees
│   └── Audit History
└── Bulk Import

Admissions (M7.11)
├── Admissions Dashboard
├── Inquiries
├── Applicants
└── Funnel Reports

Academics
├── Academic Years
├── Grades & Sections
├── Timetable Management
│   ├── Subjects
│   ├── Rooms
│   ├── Teachers
│   └── Timetable Builder
├── Assessments
│   ├── Assessment List
│   ├── Marks Entry
│   └── Report Cards
└── Exams

Attendance
├── Attendance Dashboard
├── Sessions
├── Corrections Queue
└── Reports

Fees
├── Fee Categories
├── Fee Structures
├── Invoices
├── Payments
├── Receipts
├── Refunds
├── Reconciliation
├── Defaulters Dashboard
└── Finance Reports

Communication
├── Send Notice
├── Scheduled Messages
├── Templates
├── Delivery Reports
└── Channel Settings

Settings
├── School Profile
├── School Settings
├── Branding
├── Campuses
├── Users & Roles
├── Permissions
└── Subscription
```

---

### Principal

```
Home (Academic Dashboard)
├── Attendance Overview (today's rate by grade)
├── Fee Collection Summary
├── Academic Progress (assessment completion, report cards)
└── Pending Approvals

Students → (view only)

Academics
├── Timetable Viewer
├── Assessment Overview
├── Report Card Status
└── Exam Schedule

Attendance
├── Attendance Dashboard (read)
├── Approve Corrections

Fees → Summary views only

Communication → Send school-wide notices

Reports
├── Attendance Analytics
├── Academic Performance
└── Admissions Funnel

Leave Approvals (HR - M7.16)
```

---

### Teacher

```
Home
├── Today's Attendance Sessions
├── Pending Marks Entry
└── Upcoming Assessments

My Classes
├── Assigned Sections
└── Class Roster per Section

Attendance
├── Take Attendance (today)
├── Attendance History
└── Correction Requests

Grades
├── My Assessments
└── Marks Entry

Timetable
└── My Weekly Schedule

Assignments
├── Create Assignment
├── Submitted Assignments
└── Feedback

Notices (receive)
```

---

### Accountant

```
Home
├── Daily Collection Summary
├── Outstanding Amount
└── Pending Actions

Students → (limited: fee-relevant info only)

Fees
├── Fee Categories
├── Fee Structures
├── Student Fee Assignments
├── Invoices (generate, issue, cancel)
├── Payments (record offline, view online)
├── Receipts
├── Refunds
├── Reconciliation
└── Adjustments

Reports
├── Defaulters
├── Collection Analytics
├── Aging Analysis
├── Category Breakdown
└── Fee Audit Trail

Communication
└── Fee Reminders (trigger)
```

---

### Parent (Portal)

```
Dashboard
├── Child Status (attendance today, fee alert, recent notice)
└── Child Switcher (if multiple children)

Attendance
├── Attendance Summary
├── Monthly Calendar
└── History

Grades
├── Assessment Results
└── Report Cards

Timetable
├── Today's Schedule
└── Weekly View

Assignments (where enabled)
└── Pending Assignments

Fees
├── Outstanding Dues
├── Invoice List
├── Pay Online
└── Payment History & Receipts

Notices
└── School Notices

Documents
└── Student Documents

Support
└── Contact School
```

---

### Student (Portal)

```
Dashboard
├── Today's Timetable
├── Pending Assignments
└── Recent Notices

Timetable
└── Weekly Schedule

Assignments
├── Pending
├── Submitted
└── Feedback

Grades (published only)
├── Assessment Results
└── Report Cards

Attendance (own only)
└── My Attendance

Notices
└── Student Notices

Fees (if enabled by school)
└── My Fee Status
```

---

## Operations Dashboard Widgets (School Admin / Principal)

These are the widgets on the primary daily-use dashboard:

| Widget | Data Source | Refresh |
|--------|------------|---------|
| Today's Attendance Rate | Attendance module | Every 30 mins |
| Missing Attendance Submissions | Attendance sessions with status=draft | Every 15 mins |
| Outstanding Fee Amount | Finance module | Hourly |
| Today's Collection | Payment transactions for today | Hourly |
| Defaulters Count | Overdue invoices | Hourly |
| Pending Corrections | Attendance corrections queue | Real-time |
| Active Students | SIS enrollment count | Daily |
| Upcoming Exams (this week) | Exam schedule | Daily |
| Published Report Cards | Result publications | On publish |
| Portal Active Users (last 7 days) | Portal analytics | Daily |

---

## KPI Definitions for Dashboard

| KPI | Definition | Display |
|-----|-----------|---------|
| Attendance Rate | (Present + Late) / Enrolled × 100 for today's submitted sessions | Percentage |
| Collection Efficiency | Total paid / Total issued for current academic year | Percentage |
| Outstanding Amount | Sum of balance_due on issued/partially_paid/overdue invoices | Currency (₹) |
| Report Card Completion | Published report cards / Enrolled students × 100 | Percentage |
| Admissions Conversion | Enrolled / Total Inquiries × 100 for current year | Percentage |
| Portal Engagement | Unique portal logins in last 7 days / Total active parent accounts × 100 | Percentage |

---

## Navigation Depth Limits

- Maximum 3 levels deep from primary navigation
- Every data entry form accessible within 2 clicks from its parent list
- Back navigation always available (breadcrumb trail)
- Quick actions accessible from list rows (hover/swipe menu)
- Keyboard shortcuts for frequent actions (web app)
