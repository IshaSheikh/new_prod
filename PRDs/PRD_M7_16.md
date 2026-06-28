# Module 7.16 HR and Payroll

The HR and Payroll module manages staff profiles, staff attendance, leave requests, payroll processing, pay components, and statutory records. It is a later-stage ERP expansion module intended to support school staff operations in a structured, auditable, and finance-aligned way.

The module enables schools to:
- Maintain staff employment records
- Track staff attendance
- Manage leave requests and balances
- Configure salary structures and payroll components
- Process payroll cycles
- Store statutory and compliance-related staff details
- Generate payroll summaries and payout records

The module must integrate with user management, school structure, attendance policy, and finance controls. It must not replace the platform’s core RBAC, tenant isolation, or audit foundations, and it should remain clearly separated from student-facing modules unless explicitly required. 

***

## 1. Module Objective

Schools often manage HR and payroll through spreadsheets, disconnected attendance logs, and external payroll tools. Without a structured HR and payroll workflow:
- Staff records become incomplete or outdated
- Leave balances are hard to track
- Payroll runs become error-prone
- Statutory details are inconsistently maintained
- Audit trails for salary changes are weak
- Administrative workload grows with staff count

Schools need this module to:
- Centralize staff records and operational HR data
- Manage attendance and leave in a structured workflow
- Run payroll consistently
- Maintain statutory information safely
- Reduce manual salary processing effort
- Support later-stage ERP maturity for larger schools

***

## 2. Why Schools Need It

### 2.1 Support Staff Administration
Schools need structured staff profiles, employment records, and organization-linked responsibilities to manage operations beyond student workflows.

### 2.2 Reduce Payroll Errors
Manual payroll calculations increase the risk of wrong payouts, missed deductions, and inconsistent salary records.

### 2.3 Improve Leave and Attendance Control
A staff-focused attendance and leave workflow reduces administrative confusion and provides consistent policy enforcement.

### 2.4 Support Compliance
Schools often need to maintain statutory and payroll-related details in a controlled and auditable way.

### 2.5 Improve Operational Visibility
School leaders need insight into staffing, attendance patterns, leave utilization, payroll completion, and payroll cost summaries.

### 2.6 Enable ERP Expansion
This module is a later-stage expansion and should come after core school operations are stable, because it introduces broader ERP complexity than attendance, grades, fees, or timetable workflows. 

***

## 3. Primary Users

### 3.1 School Admin
Primary HR operator. Responsibilities:
- Manage staff records
- Review attendance and leave
- Configure payroll settings
- Run payroll cycles
- Maintain statutory details

### 3.2 Principal
Oversight and approval user. Responsibilities:
- Review staffing summaries
- Approve leave where policy requires
- Monitor attendance and payroll execution
- View payroll cost summaries at authorized scope

### 3.3 Teacher
Staff self-service user. Responsibilities:
- View own profile
- View own attendance
- Apply for leave
- View own payslips and payroll history where enabled

### 3.4 Accountant
Payroll operations user. Responsibilities:
- Configure pay components
- Process payroll
- Review deduction summaries
- Export payroll outputs
- Reconcile payroll records with finance systems where integrated

### 3.5 Super Admin
Platform oversight user. Responsibilities:
- Audit HR and payroll configuration
- Support tenant issues
- Review platform-level operational incidents

### 3.6 Parent
No primary role in this module.

### 3.7 Student
No primary role in this module.

***

## 4. User Stories

### 4.1 User Stories - Staff Profile Management
- **US-001** As a School Admin, I want to create staff profiles, so employment information is centralized.
- **US-002** As a School Admin, I want to maintain role, department, and employment status, so staff records stay operationally useful.
- **US-003** As a Principal, I want to view staffing summaries, so workforce visibility improves.

### 4.2 User Stories - Staff Attendance
- **US-004** As a School Admin, I want to track staff attendance, so work presence is measurable.
- **US-005** As a Teacher, I want to view my own attendance history, so I can confirm recorded attendance.
- **US-006** As a Principal, I want staff attendance summaries, so attendance patterns can be monitored.

### 4.3 User Stories - Leave Management
- **US-007** As a Teacher, I want to request leave, so I can follow the school’s approval workflow.
- **US-008** As a School Admin, I want to review and approve leave requests, so leave policy is enforced consistently.
- **US-009** As a Teacher, I want to view my leave balance, so I know what is available.
- **US-010** As a Principal, I want leave summaries, so staffing gaps can be monitored.

### 4.4 User Stories - Payroll Processing
- **US-011** As an Accountant, I want salary structures and pay components, so payroll can be processed consistently.
- **US-012** As an Accountant, I want to run payroll for a defined period, so staff payouts are prepared accurately.
- **US-013** As a School Admin, I want payroll status visibility, so I can track whether payroll is draft, approved, or completed.
- **US-014** As a Teacher, I want to access my payslip, so I can view salary details.

### 4.5 User Stories - Statutory and Audit
- **US-015** As a School Admin, I want to store statutory details for staff, so compliance-related records are organized.
- **US-016** As an Accountant, I want deduction and statutory summaries, so payroll review is easier.
- **US-017** As a Super Admin, I want payroll and HR audit logs, so sensitive changes can be investigated.

***

## 5. Business Rules

### 5.1 Business Rules - Tenant Isolation
- **BR-001** Every staff profile, leave record, attendance record, payroll record, and statutory detail must belong to exactly one tenant.
- **BR-002** HR and payroll data cannot be accessed across tenants.
- **BR-003** All HR and payroll APIs and queries must automatically enforce tenant scope.

### 5.2 Staff Profile Rules
- **BR-004** Staff profiles must reference platform user identities where applicable, rather than creating separate login identities outside RBAC. 
- **BR-005** Employment status must be tracked using controlled states.
- **BR-006** Historical employment records must remain searchable.
- **BR-007** Sensitive payroll and statutory fields must be role-protected.

### 5.3 Attendance Rules
- **BR-008** Staff attendance must be separate from student attendance workflows.
- **BR-009** Only active staff may receive active attendance sessions.
- **BR-010** Attendance corrections must be auditable.
- **BR-011** Attendance-related payroll effects must use effective payroll period rules and must not mutate closed payroll records.

### 5.4 Leave Rules
- **BR-012** Leave requests must include type, date range, and reason where policy requires.
- **BR-013** Leave balances must be policy-driven and configurable.
- **BR-014** Approved leave must affect leave balances using effective dated rules.
- **BR-015** Rejected leave must preserve request history.
- **BR-016** Overlapping approved leave requests for the same staff member must be blocked.

### 5.5 Payroll Rules
- **BR-017** Payroll must be processed by payroll period.
- **BR-018** Payroll records must preserve salary snapshot at processing time.
- **BR-019** Closed payroll periods must not be recalculated silently.
- **BR-020** Salary component changes must be auditable.
- **BR-021** Payroll may require approval before finalization depending on school policy.
- **BR-022** Payslips must be visible only after payroll is finalized or published according to school policy.

### 5.6 Statutory Rules
- **BR-023** Statutory details must be stored securely and access must be restricted to authorized roles.
- **BR-024** Changes to statutory details must be audited.
- **BR-025** Statutory data retention must follow tenant policy configuration where supported.

### 5.7 Reporting and Audit Rules
- **BR-026** Every create, edit, approve, reject, process, finalize, and publish action must be auditable.
- **BR-027** HR and payroll reporting must support filtering by period, staff status, department, and payroll status.
- **BR-028** Historical payroll, leave, and attendance records must remain searchable.

***

## 6. State Machine

### 6.1 State Machine - Staff Profile Lifecycle
```text
Draft
  v
Active
  |------ Suspended
  |         v
  |------ Reactivated
  |------ Terminated
  v
Archived
```

### 6.2 State Machine - Leave Request Lifecycle
```text
Draft
  v
Submitted
  |------ Approved
  |------ Rejected
  |------ Cancelled
  v
Closed
```

### 6.3 State Machine - Payroll Run Lifecycle
```text
Draft
  v
Calculated
  |------ Approved
  |         v
  |------ Finalized
  |------ Reopened
  v
Archived
```

### 6.4 State Machine - Payslip Lifecycle
```text
Pending
  v
Generated
  v
Published
  |------ Revised
  |         v
  |------ Published
```

***

## 7. Required Data Entities

### 7.1 Required Data Entities - staff_profiles
Field | Type
--- | ---
id | UUID
tenantid | UUID
schoolid | UUID
userid | UUID NULL
staffcode | VARCHAR
firstname | VARCHAR
lastname | VARCHAR
department | VARCHAR NULL
designation | VARCHAR NULL
employmentstatus | ENUM
joiningdate | DATE
exitdate | DATE NULL
createdat | TIMESTAMP
updatedat | TIMESTAMP

### 7.2 Required Data Entities - staff_attendance
Field | Type
--- | ---
id | UUID
tenantid | UUID
staffprofileid | UUID
attendancedate | DATE
status | ENUM
recordedby | UUID
createdat | TIMESTAMP
updatedat | TIMESTAMP

### 7.3 Required Data Entities - leave_requests
Field | Type
--- | ---
id | UUID
tenantid | UUID
staffprofileid | UUID
leavetype | ENUM
startdate | DATE
enddate | DATE
reason | TEXT NULL
status | ENUM
submittedat | TIMESTAMP NULL
reviewedby | UUID NULL
reviewedat | TIMESTAMP NULL
createdat | TIMESTAMP
updatedat | TIMESTAMP

### 7.4 Required Data Entities - leave_balances
Field | Type
--- | ---
id | UUID
tenantid | UUID
staffprofileid | UUID
leaveyear | VARCHAR
leavetype | ENUM
allocateddays | DECIMAL
useddays | DECIMAL
remainingdays | DECIMAL
updatedat | TIMESTAMP

### 7.5 Required Data Entities - payroll_runs
Field | Type
--- | ---
id | UUID
tenantid | UUID
schoolid | UUID
payrollperiod | VARCHAR
status | ENUM
processedby | UUID
approvedby | UUID NULL
finalizedat | TIMESTAMP NULL
createdat | TIMESTAMP
updatedat | TIMESTAMP

### 7.6 Required Data Entities - payroll_items
Field | Type
--- | ---
id | UUID
tenantid | UUID
payrollrunid | UUID
staffprofileid | UUID
grossamount | DECIMAL
deductionamount | DECIMAL
netamount | DECIMAL
status | ENUM
createdat | TIMESTAMP
updatedat | TIMESTAMP

### 7.7 Required Data Entities - statutory_details
Field | Type
--- | ---
id | UUID
tenantid | UUID
staffprofileid | UUID
detailtype | ENUM
detailvalue | TEXT
status | ENUM
createdat | TIMESTAMP
updatedat | TIMESTAMP

### 7.8 Additional Supporting Entities
- `salary_structures`
- `salary_components`
- `payroll_adjustments`
- `payslips`
- `staff_hr_audit_logs`

***

## 8. Permissions Matrix

| Capability | Super Admin | School Admin | Principal | Teacher | Accountant | Parent | Student |
|---|---|---|---|---|---|---|---|
| View HR dashboard | Yes | Yes | Yes | Self only where enabled | Limited payroll summary | No | No |
| Create staff profile | Limited Support | Yes | Yes if allowed | No | No | No | No |
| Edit staff profile | Limited Audit | Yes | Yes limited | Self limited fields only if enabled | No | No | No |
| Record staff attendance | Limited Audit | Yes | Yes if allowed | Self only if self-marking enabled | No | No | No |
| Submit leave request | No | Yes | Yes | Yes self | No | No | No |
| Approve leave | No | Yes | Yes if allowed | No | No | No | No |
| Create payroll run | No | Yes | Yes summary only | No | Yes | No | No |
| Approve payroll run | No | Yes | Yes if allowed | No | Yes if policy allows | No | No |
| View payslip | Limited Audit | Yes | Yes summary only | Self only | Yes | No | No |
| Manage statutory details | Limited Audit | Yes | Limited | No | Yes if authorized | No | No |
| View audit history | Limited Audit | Yes | Limited | No | Limited payroll only | No | No |

***

## 9. APIs Needed

### 9.1 APIs Needed - Staff Profile APIs
- `POST /hr/staff-profiles`
- `GET /hr/staff-profiles`
- `GET /hr/staff-profiles/{id}`
- `PATCH /hr/staff-profiles/{id}`

### 9.2 APIs Needed - Staff Attendance APIs
- `POST /hr/staff-attendance`
- `GET /hr/staff-attendance`
- `PATCH /hr/staff-attendance/{id}`

### 9.3 APIs Needed - Leave APIs
- `POST /hr/leave-requests`
- `GET /hr/leave-requests`
- `GET /hr/leave-requests/{id}`
- `POST /hr/leave-requests/{id}/approve`
- `POST /hr/leave-requests/{id}/reject`
- `GET /hr/leave-balances`

### 9.4 APIs Needed - Payroll APIs
- `POST /hr/payroll-runs`
- `GET /hr/payroll-runs`
- `GET /hr/payroll-runs/{id}`
- `POST /hr/payroll-runs/{id}/calculate`
- `POST /hr/payroll-runs/{id}/approve`
- `POST /hr/payroll-runs/{id}/finalize`

### 9.5 APIs Needed - Payslip APIs
- `GET /hr/payslips`
- `GET /hr/payslips/{id}`
- `POST /hr/payslips/{id}/publish`

### 9.6 APIs Needed - Statutory APIs
- `POST /hr/statutory-details`
- `GET /hr/statutory-details`
- `PATCH /hr/statutory-details/{id}`

### 9.7 APIs Needed - Reporting APIs
- `GET /hr/reports/attendance`
- `GET /hr/reports/leave`
- `GET /hr/reports/payroll`
- `GET /hr/search`

***

## 10. Screens Needed

### 10.1 Screens Needed - HR Dashboard
Displays:
- Active staff count
- Attendance summary
- Leave summary
- Payroll status
- Pending approvals

### 10.2 Screens Needed - Staff Profile Screen
Features:
- Create and edit staff profile
- Employment details
- Department and designation
- Status history

### 10.3 Screens Needed - Staff Attendance Screen
Features:
- Daily attendance view
- Status marking
- Correction workflow
- Attendance history

### 10.4 Screens Needed - Leave Management Screen
Features:
- Submit leave
- Review requests
- Approve or reject
- Leave balance summary

### 10.5 Screens Needed - Payroll Run Screen
Features:
- Create payroll period
- Calculate payroll
- Review payroll items
- Approve and finalize payroll

### 10.6 Screens Needed - Payslip Screen
Displays:
- Earnings
- Deductions
- Net pay
- Publication status
- Download action where enabled

### 10.7 Screens Needed - Statutory Details Screen
Features:
- Store statutory records
- Edit authorized fields
- Audit history visibility

### 10.8 Screens Needed - HR Reports Screen
Displays:
- Staff attendance trends
- Leave utilization
- Payroll cost summary
- Payroll completion status

***

## 11. Edge Cases

- **EC-001** Staff member is terminated during an open payroll period. Final payroll treatment must follow effective-date rules.
- **EC-002** Leave request overlaps approved leave. Submission blocked.
- **EC-003** Staff profile linked to user account after profile creation. Identity linkage must preserve history and permissions. 
- **EC-004** Payroll finalized and later salary structure changes. Historical payroll remains unchanged.
- **EC-005** Attendance corrected after payroll finalization. Closed payroll records remain unchanged unless controlled adjustment workflow is used.
- **EC-006** Statutory detail updated after payroll run generation. Impact applies only according to payroll period policy.
- **EC-007** Teacher attempts to access another staff member’s payslip. Access denied.
- **EC-008** School Admin removed while payroll approval is pending. Approval authority must transfer according to active role policy. 
- **EC-009** Staff member has no linked login account. HR profile remains operational for payroll and attendance workflows.

***

## 12. Risks

### 12.1 Risks - Privacy Risk
Payroll and statutory data are highly sensitive.  
Mitigation: Strict role protection, tenant isolation, audit logging, field-level access control.

### 12.2 Risks - Payroll Integrity Risk
Payroll calculations may become inconsistent or silently altered.  
Mitigation: Snapshot-based payroll records, approval workflow, immutable finalized periods.

### 12.3 Risks - Operational Risk
Leave and attendance policies may be applied inconsistently.  
Mitigation: Configurable rules, approval workflow, history preservation.

### 12.4 Risks - Compliance Risk
Changes to sensitive staff and statutory records may not be reconstructable.  
Mitigation: Immutable audit logs and controlled updates. 

### 12.5 Risks - Scalability Risk
Large staff counts and payroll processing windows create performance pressure.  
Mitigation: Indexed staff and payroll tables, batch processing, period-based partitioning where needed.

### 12.6 Risks - Product Complexity Risk
HR and payroll scope expands beyond manageable solo-founder delivery.  
Mitigation: Keep MVP narrow and position this as a later-stage ERP expansion. 

***

## 13. Success Metrics

### 13.1 Success Metrics - Product Metrics
- Staff profile creation success rate 99
- Leave request processing success rate 98
- Payroll run generation success rate 98
- HR dashboard load time 2 seconds
- Payslip access success rate 99

### 13.2 Success Metrics - Operational Metrics
- Leave approval turnaround within 24 hours
- Payroll completion within planned payroll window 95
- Staff attendance capture completion rate 95
- Payroll correction rate below 2

### 13.3 Success Metrics - Data Quality Metrics
- Duplicate active staff profiles below 0.5
- Overlapping approved leave requests 0
- Finalized payroll records changed without audit 0

### 13.4 Success Metrics - Business Metrics
- Reduction in staff administration workload 50
- Reduction in payroll processing effort 60
- Improved visibility into staffing and payroll execution

***

## 14. Out of Scope

The following are excluded from the HR and Payroll module MVP:
- Full recruitment and applicant tracking
- Employee onboarding workflow automation
- Performance appraisal systems
- Training and certification management
- Shift rostering and complex workforce scheduling
- Biometric attendance devices
- Payroll bank disbursement integrations
- Government tax filing automation
- Employee reimbursement workflows
- Asset assignment and recovery
- Multi-entity consolidated payroll
- Staff self-service mobile app beyond basic leave and payslip views
- Advanced compensation planning
- Workforce planning analytics
- Contract labor management

# Module 7.17 Analytics and KPI Dashboards

The Analytics and KPI Dashboards module provides cross-functional visibility into school operations through actionable metrics and dashboards spanning attendance, fees, results, admissions, communication, and transport. It is a read-optimized decision-support layer that helps schools measure daily operations without becoming a separate source of truth.

The module enables schools to:
- View operational dashboards across major modules
- Track defined KPIs at school, grade, section, and time-period level
- Identify trends and exceptions quickly
- Support Principal and School Admin decision-making
- Measure adoption, efficiency, and operational performance
- Expose curated role-based dashboards without changing source records

Core metrics include:
- daily attendance rate
- outstanding fee amount
- collection efficiency
- exam completion rate
- portal engagement
- admissions pipeline conversion
- transport occupancy

The module must consume data from source modules rather than recreating business logic independently, preserving the platform’s single-source-of-truth architecture. 

***

## 1. Module Objective

Schools generate data across attendance, fees, results, admissions, communication, and transport, but they often cannot convert it into actionable insight. Without centralized dashboards:
- Leaders depend on manual reports
- Operational issues are discovered late
- Metrics vary across departments
- Daily decisions are less data-driven
- Parent transparency improvements are harder to measure

Schools need this module to:
- Surface consistent operational KPIs
- Provide role-based dashboard views
- Highlight trends and exceptions quickly
- Make school operations measurable at daily, weekly, monthly, and term levels
- Reduce manual reporting effort for admins and principals

***

## 2. Why Schools Need It

### 2.1 Make Operations Measurable
Schools need a consolidated view of attendance, collections, exam readiness, admissions progress, and engagement.

### 2.2 Reduce Manual Reporting
Admins should not need to export and combine multiple spreadsheets to answer basic operational questions.

### 2.3 Improve Decision Quality
Leaders need timely KPI visibility to identify low attendance, weak fee collection, low portal usage, or stalled admissions.

### 2.4 Create Consistent Metrics
Analytics must use standardized definitions across the platform instead of ad hoc calculations in each module.

### 2.5 Support Parent Transparency Goals
Portal engagement and communication performance can help schools measure whether parent-facing transparency is improving.

### 2.6 Support Growth
As the platform expands across more modules, a shared KPI layer becomes increasingly important for school leadership and product value.

***

## 3. Primary Users

### 3.1 School Admin
Primary dashboard user. Responsibilities:
- Monitor daily KPIs
- Review exceptions
- Track operational bottlenecks
- Export reports where allowed

### 3.2 Principal
Executive dashboard user. Responsibilities:
- Monitor school performance
- Compare trends across classes or periods
- Use KPI trends for decisions and follow-up

### 3.3 Super Admin
Platform oversight user. Responsibilities:
- Review adoption and operational health patterns across tenants at authorized scope
- Investigate support incidents tied to data quality or stale metrics

### 3.4 Teacher
Limited dashboard user. Responsibilities:
- View only metrics relevant to assigned sections, classes, or own activity where enabled

### 3.5 Accountant
Finance-focused dashboard user. Responsibilities:
- View fee collection and outstanding amounts
- Track collection efficiency and payment trends

### 3.6 Parent
No operational dashboard role in MVP.

### 3.7 Student
No operational dashboard role in MVP.

***

## 4. User Stories

### 4.1 User Stories - Operational Dashboards
- **US-001** As a School Admin, I want a daily operations dashboard, so I can quickly understand school status.
- **US-002** As a Principal, I want KPI trends across attendance, fees, results, and admissions, so I can make better decisions.
- **US-003** As an Accountant, I want fee collection dashboards, so I can monitor outstanding amounts and collection efficiency.

### 4.2 User Stories - Metric Visibility
- **US-004** As a School Admin, I want daily attendance rate visibility, so absenteeism issues are visible early.
- **US-005** As an Accountant, I want outstanding fee amount and collection efficiency, so finance performance is measurable.
- **US-006** As a Principal, I want exam completion rate, so academic execution delays are visible.
- **US-007** As a School Admin, I want admissions pipeline conversion, so inquiry-to-enrollment performance is measurable.
- **US-008** As a School Admin, I want portal engagement metrics, so parent-facing product adoption is measurable.
- **US-009** As a School Admin, I want transport occupancy metrics, so route efficiency is measurable where transport is enabled.

### 4.3 User Stories - Filtering and Drilldown
- **US-010** As a Principal, I want to filter dashboards by academic year, grade, section, and time range, so I can investigate specific trends.
- **US-011** As a School Admin, I want drilldowns from KPI to underlying operational lists, so I can act on the metric.
- **US-012** As a Teacher, I want only my relevant metrics, so the dashboard stays focused.

### 4.4 User Stories - Reliability and Governance
- **US-013** As a Super Admin, I want KPI refresh health visibility, so stale dashboards can be detected.
- **US-014** As a School Admin, I want metric definitions to be consistent, so staff trust the numbers.
- **US-015** As a Principal, I want dashboards to be read-only, so analytics does not accidentally modify operational data.

***

## 5. Business Rules

### 5.1 Business Rules - Tenant Isolation
- **BR-001** Every dashboard query, aggregate, and metric result must be tenant-scoped.
- **BR-002** Analytics data cannot be visible across tenants except for authorized platform-level views where explicitly designed.
- **BR-003** Source data access for analytics must follow tenant isolation and role-based access controls. 

### 5.2 Source of Truth Rules
- **BR-004** Analytics and KPI Dashboards must consume data from source modules and must not become a separate transaction source.
- **BR-005** KPI definitions must be centrally governed.
- **BR-006** Changes to metric definitions must be versioned and auditable.
- **BR-007** Drilldowns must resolve to authoritative source module records where allowed by permissions.

### 5.3 Metric Rules
- **BR-008** Daily attendance rate must be derived from attendance source data using school attendance rules. 
- **BR-009** Outstanding fee amount and collection efficiency must be derived from finance source data and must not rewrite financial records.
- **BR-010** Exam completion rate must be derived from formal exam or academic result workflows using published operational statuses. 
- **BR-011** Portal engagement must use defined user interaction events and active-user logic.
- **BR-012** Admissions pipeline conversion must use controlled admissions workflow stages.
- **BR-013** Transport occupancy must use active transport assignments and configured route or vehicle capacity.
- **BR-014** KPI calculations must support time-bound filtering.

### 5.4 Refresh and Performance Rules
- **BR-015** Dashboards may be near-real-time or batch-refreshed depending on metric type.
- **BR-016** Metric freshness metadata must be visible for operational trust.
- **BR-017** Expensive KPIs should use materialized aggregates or precomputed tables where needed.
- **BR-018** Dashboard queries must not degrade transactional module performance.

### 5.5 Access and Visibility Rules
- **BR-019** Users may view only dashboards and drilldowns within their authorized role and scope.
- **BR-020** Teachers must not see school-wide confidential finance metrics unless explicitly authorized.
- **BR-021** Parents and students do not receive operational dashboards in MVP.
- **BR-022** Dashboard actions are read-only except for allowed exports or navigation to source screens.

### 5.6 Audit and Reporting Rules
- **BR-023** Dashboard definition changes, scheduled refresh failures, and metric configuration updates must be auditable.
- **BR-024** Exported analytics must reflect the same tenant-scoped and role-scoped filters as on-screen views.
- **BR-025** Historical KPI trends must remain queryable according to retention policy where supported.

***

## 6. State Machine

### 6.1 State Machine - Dashboard Definition Lifecycle
```text
Draft
  v
Active
  |------ Updated
  |         v
  |------ Active
  |------ Archived
```

### 6.2 State Machine - KPI Refresh Lifecycle
```text
Pending
  v
Running
  |------ Succeeded
  |------ Failed
  |------ Retrying
  v
Completed
```

### 6.3 State Machine - Export Job Lifecycle
```text
Requested
  v
Queued
  v
Generating
  |------ Completed
  |------ Failed
  v
Expired
```

***

## 7. Required Data Entities

### 7.1 Required Data Entities - dashboard_definitions
Field | Type
--- | ---
id | UUID
tenantid | UUID
dashboardname | VARCHAR
roleaudience | ENUM
status | ENUM
createdby | UUID
createdat | TIMESTAMP
updatedat | TIMESTAMP

### 7.2 Required Data Entities - kpi_definitions
Field | Type
--- | ---
id | UUID
tenantid | UUID NULL
kpicode | VARCHAR
kpiname | VARCHAR
modulename | VARCHAR
calculationversion | INTEGER
refreshmode | ENUM
status | ENUM
createdat | TIMESTAMP
updatedat | TIMESTAMP

### 7.3 Required Data Entities - kpi_snapshots
Field | Type
--- | ---
id | UUID
tenantid | UUID
kpidefinitionid | UUID
snapshotdate | DATE
scopelevel | ENUM
scopevalue | VARCHAR NULL
metricvalue | DECIMAL
freshnessat | TIMESTAMP
createdat | TIMESTAMP

### 7.4 Required Data Entities - dashboard_widgets
Field | Type
--- | ---
id | UUID
tenantid | UUID
dashboarddefinitionid | UUID
widgettype | ENUM
title | VARCHAR
displayorder | INTEGER
configjson | JSONB
status | ENUM
createdat | TIMESTAMP
updatedat | TIMESTAMP

### 7.5 Required Data Entities - dashboard_exports
Field | Type
--- | ---
id | UUID
tenantid | UUID
dashboarddefinitionid | UUID
requestedby | UUID
status | ENUM
fileurl | TEXT NULL
expiresat | TIMESTAMP NULL
createdat | TIMESTAMP
updatedat | TIMESTAMP

### 7.6 Additional Supporting Entities
- `kpi_refresh_jobs`
- `dashboard_access_logs`
- `dashboard_filter_presets`
- `metric_definition_history`

***

## 8. Permissions Matrix

| Capability | Super Admin | School Admin | Principal | Teacher | Accountant | Parent | Student |
|---|---|---|---|---|---|---|---|
| View analytics dashboard | Yes | Yes | Yes | Limited scoped view | Finance-scoped view | No | No |
| View KPI drilldown | Limited Audit | Yes | Yes | Limited own scope | Finance-scoped only | No | No |
| Configure dashboard widgets | Limited Support | Yes | Yes if allowed | No | No | No | No |
| Manage KPI definitions | Limited Support | Yes if tenant-scoped config allowed | No | No | No | No | No |
| Export dashboard data | Limited Audit | Yes | Yes | Limited own scope if enabled | Finance-scoped only | No | No |
| View refresh health | Yes | Yes | Yes | No | No | No | No |
| View cross-school analytics | Platform only if explicitly enabled | No | No | No | No | No | No |

***

## 9. APIs Needed

### 9.1 APIs Needed - Dashboard APIs
- `GET /analytics/dashboards`
- `GET /analytics/dashboards/{id}`
- `PATCH /analytics/dashboards/{id}`

### 9.2 APIs Needed - KPI APIs
- `GET /analytics/kpis`
- `GET /analytics/kpis/{id}`
- `GET /analytics/kpis/{id}/snapshots`
- `POST /analytics/kpis/{id}/refresh`

### 9.3 APIs Needed - Widget APIs
- `POST /analytics/widgets`
- `PATCH /analytics/widgets/{id}`
- `DELETE /analytics/widgets/{id}`

### 9.4 APIs Needed - Drilldown APIs
- `GET /analytics/drilldowns/attendance`
- `GET /analytics/drilldowns/fees`
- `GET /analytics/drilldowns/exams`
- `GET /analytics/drilldowns/admissions`
- `GET /analytics/drilldowns/engagement`
- `GET /analytics/drilldowns/transport`

### 9.5 APIs Needed - Export APIs
- `POST /analytics/exports`
- `GET /analytics/exports`
- `GET /analytics/exports/{id}`

### 9.6 APIs Needed - Refresh and Health APIs
- `GET /analytics/refresh-jobs`
- `GET /analytics/health`

***

## 10. Screens Needed

### 10.1 Screens Needed - Executive KPI Dashboard
Displays:
- Daily attendance rate
- Outstanding fee amount
- Collection efficiency
- Exam completion rate
- Portal engagement
- Admissions pipeline conversion
- Transport occupancy

### 10.2 Screens Needed - Attendance Analytics Screen
Displays:
- Daily trend
- Grade and section breakdown
- Absence concentration
- Attendance drilldown

### 10.3 Screens Needed - Fee Analytics Screen
Displays:
- Outstanding amount
- Collection efficiency trend
- Overdue buckets
- Defaulter drilldown

### 10.4 Screens Needed - Academic Progress Dashboard
Displays:
- Exam completion rate
- Publication readiness
- Result trend indicators
- Academic drilldown

### 10.5 Screens Needed - Admissions Analytics Screen
Displays:
- Inquiry volume
- Applicant conversion
- Offer conversion
- Enrollment funnel

### 10.6 Screens Needed - Portal Engagement Dashboard
Displays:
- Parent login activity
- Student login activity
- Feature usage trends
- Engagement trend by period

### 10.7 Screens Needed - Transport Analytics Screen
Displays:
- Route occupancy
- Capacity utilization
- Assignment trends
- Over-capacity alerts

### 10.8 Screens Needed - Analytics Configuration Screen
Features:
- Widget configuration
- KPI selection
- Refresh settings
- Metric freshness view

***

## 11. Edge Cases

- **EC-001** Source attendance data corrected after dashboard snapshot generation. KPI refresh must update future snapshots according to refresh policy. 
- **EC-002** Fee record reversed after metric calculation. Dashboard values must reflect finance refresh rules without mutating source records.
- **EC-003** Exam completion dashboard includes unpublished or incomplete academic states incorrectly. KPI definition must use official published workflow statuses. 
- **EC-004** Teacher attempts to view school-wide finance analytics. Access denied by role scope.
- **EC-005** Parent tries to open operational dashboard route. Access denied.
- **EC-006** KPI refresh job fails for one widget while others succeed. Dashboard must surface freshness and partial data state.
- **EC-007** Student transfers section mid-period. Historical KPI trends must follow metric definition for historical enrollment context.
- **EC-008** Transport module disabled for a tenant. Transport occupancy widget remains hidden or empty by configuration.
- **EC-009** Metric definition changes mid-year. Definition versioning must preserve trend interpretability.
- **EC-010** Export requested for unauthorized scope. Export blocked.
- **EC-011** Tenant suspended or read-only. Analytics remains view-only according to tenant status policy. 

***

## 12. Risks

### 12.1 Risks - Trust Risk
Users may lose trust if KPI values differ from source-module reports.  
Mitigation: Central metric definitions, source-of-truth alignment, visible freshness timestamps.

### 12.2 Risks - Performance Risk
Heavy dashboard queries may impact core transactional modules.  
Mitigation: Precomputed aggregates, refresh jobs, read-optimized query patterns.

### 12.3 Risks - Access Control Risk
Users may view metrics outside their role scope.  
Mitigation: RBAC-enforced dashboard and drilldown filtering. 

### 12.4 Risks - Data Freshness Risk
Users may act on stale metrics unknowingly.  
Mitigation: Freshness metadata, refresh monitoring, failure alerts.

### 12.5 Risks - Product Complexity Risk
Analytics scope may expand into full BI tooling too early.  
Mitigation: Keep MVP focused on curated KPI dashboards, not general-purpose analytics.

### 12.6 Risks - Cross-Module Consistency Risk
Metric logic may drift from source workflows over time.  
Mitigation: Shared metric definition governance and versioning. 

***

## 13. Success Metrics

### 13.1 Success Metrics - Product Metrics
- Dashboard load time 2 seconds
- KPI refresh success rate 99
- Drilldown query success rate 98
- Export generation success rate 95

### 13.2 Success Metrics - Operational Metrics
- Daily attendance rate visible by start of school day
- Outstanding fee amount refreshed within agreed finance window
- Exam completion rate available for active exam cycles
- Admissions conversion dashboard updated within agreed refresh cycle

### 13.3 Success Metrics - Adoption Metrics
- School Admin dashboard weekly usage above 80
- Principal dashboard weekly usage above 70
- Finance dashboard usage by accountants above 75

### 13.4 Success Metrics - Data Quality Metrics
- KPI definition mismatch incidents 0
- Unauthorized analytics access incidents 0
- Widgets with stale data beyond SLA below 2

### 13.5 Success Metrics - Business Metrics
- Reduction in manual operational reporting effort 70
- Improved visibility into school performance drivers
- Increased usage of data for attendance, fee, admissions, and exam follow-up

***

## 14. Out of Scope

The following are excluded from the Analytics and KPI Dashboards module MVP:
- General-purpose BI report builder
- Custom SQL query interface
- External data warehouse integrations
- Predictive analytics and forecasting
- AI-generated recommendations
- Natural-language analytics assistant
- Cross-tenant benchmarking by default
- Public dashboards
- Parent-facing operational analytics
- Student-facing operational analytics
- Advanced cohort analysis
- Data science notebooks
- Complex custom visualization builder
- Real-time event streaming dashboards for all modules
- Full data lake or enterprise BI architecture