<!-- Source: PRDs | Last Verified: 2026-06-28 | Owner: Engineering -->
# State Machines

All significant lifecycle entities in the platform follow explicit state machines. Transitions are controlled — no direct database updates outside the transition rules below.

---

## Platform and Tenant

### Tenant Lifecycle

```
Draft
  └─> Provisioning
        └─> Active
              ├─> Suspended ──> Active (Reactivated)
              └─> Cancelled
```

| State | Meaning |
|-------|---------|
| `draft` | Tenant created but setup incomplete |
| `provisioning` | Domain setup and onboarding in progress |
| `active` | Fully operational |
| `suspended` | Access restricted; read-only mode |
| `cancelled` | Subscription terminated; data retained per policy |

### Subscription Lifecycle

```
Trial
  └─> Active
        ├─> Grace Period ──> Active (renewed)
        ├─> Expired
        ├─> Suspended
        └─> Cancelled
```

### Membership Lifecycle

```
Invited
  └─> Pending Acceptance
        └─> Active
              ├─> Disabled
              └─> Removed
```

---

## School Onboarding

### School Onboarding Lifecycle

```
Draft
  └─> Profile Configured
        └─> Academic Setup Complete
              └─> Organizational Setup Complete
                    └─> Ready For Validation
                          ├─> Validation Failed ──> Needs Action ──> Ready For Validation
                          └─> Live
```

### Academic Year Lifecycle

```
Draft
  └─> Scheduled
        └─> Active
              └─> Completed
                    └─> Archived
```

### Campus Lifecycle

```
Draft
  └─> Active
        └─> Inactive
```

---

## Students

### Student Admission Lifecycle

```
Inquiry
  └─> Applicant
        └─> Admitted
              └─> Enrolled
                    └─> Active
                          ├─> Transferred
                          ├─> Withdrawn
                          ├─> Graduated
                          └─> Deceased
```

### Enrollment Lifecycle

```
Draft
  └─> Pending Approval
        └─> Active
              ├─> Completed
              ├─> Transferred
              └─> Withdrawn
```

### Document Lifecycle

```
Uploaded
  └─> Verified
        ├─> Replaced (new version created)
        └─> Archived
```

---

## Timetable

### Timetable Lifecycle

```
Draft
  └─> Under Review
        └─> Approved
              └─> Published
                    └─> Archived
```

### Subject Lifecycle

```
Active
  ├─> Inactive
  └─> Archived
```

---

## Attendance

### Attendance Session Lifecycle

```
Draft
  └─> Submitted
        └─> Locked
              └─> Corrected Pending
                    ├─> Corrected Approved
                    └─> Corrected Rejected
```

### Attendance Correction Lifecycle

```
Requested
  └─> Pending Review
        ├─> Approved
        └─> Rejected
```

### Notification Lifecycle

```
Pending
  └─> Queued
        ├─> Sent
        └─> Failed
```

---

## Gradebook

### Assessment Lifecycle

```
Draft
  └─> Active
        └─> Marks Entered
              └─> Verified
                    └─> Published
                          └─> Revised
```

### Marks Entry Lifecycle

```
Draft
  └─> Entered
        └─> Submitted
              └─> Verified
```

### Report Card Lifecycle

```
Generated
  └─> Verified
        └─> Published
              ├─> Revised (new version; original preserved)
              └─> Withdrawn
```

### Result Publication Lifecycle

```
Scheduled
  └─> Published
        ├─> Revised
        └─> Withdrawn
```

---

## Fees and Finance

### Invoice State Machine

| State | Allowed Transitions |
|-------|-------------------|
| `draft` | `issued`, `cancelled` |
| `issued` | `partially_paid`, `paid`, `overdue`, `cancelled` |
| `partially_paid` | `paid`, `overdue`, `cancelled` (policy-controlled) |
| `paid` | `refunded_partial`, `refunded_full` |
| `overdue` | `partially_paid`, `paid`, `cancelled` (controlled) |
| `cancelled` | Terminal |
| `refunded_partial` | `refunded_full` |
| `refunded_full` | Terminal |

### Payment Transaction State Machine

| State | Allowed Transitions |
|-------|-------------------|
| `initiated` | `pending`, `success`, `failed`, `cancelled` |
| `pending` | `success`, `failed`, `expired` |
| `success` | `reconciled`, `refund_initiated` |
| `failed` | Terminal |
| `cancelled` | Terminal |
| `expired` | Terminal |
| `reconciled` | `refund_initiated` |
| `refund_initiated` | `refunded_partial`, `refunded_full`, `refund_failed` |
| `refunded_partial` | `refunded_full` |
| `refunded_full` | Terminal |
| `refund_failed` | `refund_initiated` |

### Receipt State Model

```
Issued
  └─> Reversed (creates new correct receipt)
```

> Receipts do not support an edit state. Corrections require reversal + new issuance.

### Refund Lifecycle

```
Requested
  └─> Approved
        └─> Processing
              ├─> Completed
              └─> Failed ──> Requested (retry)
```

---

## Communication

### Notice Lifecycle

```
Draft
  ├─> Scheduled
  │     └─> Queued
  │           └─> Sent
  │                 └─> Archived
  └─> Pending Approval
        ├─> Approved ──> Scheduled
        └─> Rejected ──> Draft
```

### Outbound Message Lifecycle

```
Queued
  └─> Sending
        ├─> Sent
        │     ├─> Delivered
        │     └─> Read
        └─> Failed
              └─> Retrying ──> Sending
```

### Template Lifecycle

```
Draft
  └─> Active
        └─> Inactive ──> Active (reactivated)
              └─> Archived
```

---

## Portal

### Parent / Student Portal Account Lifecycle

```
Invited
  └─> Pending Activation
        └─> Active
              ├─> Suspended ──> Reactivated ──> Active
              └─> Disabled
```

### Portal Support Request Lifecycle

```
Submitted
  └─> Acknowledged
        └─> In Progress
              ├─> Waiting for User ──> Reopened ──> In Progress
              └─> Resolved
                    └─> Closed
```

### Portal Notice Visibility Lifecycle

```
Draft
  └─> Scheduled
        └─> Published
              └─> Expired
                    └─> Archived
```

---

## Admissions

### Applicant Admission Lifecycle

```
Inquiry
  └─> Applicant
        └─> Documents Pending
              └─> Evaluation
                    └─> Offered
                          ├─> Accepted ──> Admitted ──> Enrolled
                          ├─> Rejected ──> Closed
                          └─> Expired ──> Closed (or Reissued)
```

### Application Form Lifecycle

```
Draft
  └─> Submitted
        ├─> Returned for Correction ──> Resubmitted ──> Submitted
        └─> Verified
```

### Offer Lifecycle

```
Draft
  └─> Issued
        ├─> Accepted ──> Closed
        ├─> Rejected ──> Closed
        └─> Expired ──> Closed (or Reissued)
```

---

## Transport

### Vehicle Lifecycle

```
Draft ──> Active ──> Inactive ──> Reactivated ──> Active
                  └─> Archived
```

### Route Lifecycle

```
Draft ──> Active ──> Inactive ──> Reactivated ──> Active
                  └─> Archived
```

### Student Transport Assignment Lifecycle

```
Draft ──> Active ──> Suspended ──> Reactivated ──> Active
                  └─> Ended ──> Archived
```

---

## Library

### Book Copy Lifecycle

```
Available
  └─> Issued
        ├─> Returned ──> Available
        ├─> Overdue ──> Returned ──> Available
        ├─> Lost ──> Archived
        └─> Damaged ──> Available (after review) or Archived
```

### Issue Lifecycle

```
Active
  ├─> Overdue
  ├─> Returned ──> Closed
  └─> Lost ──> Closed
```

### Fine Lifecycle

```
Pending
  └─> Due
        ├─> Paid ──> Closed
        └─> Waived ──> Closed
```

---

## HR and Payroll

### Staff Profile Lifecycle

```
Draft ──> Active ──> Suspended ──> Reactivated ──> Active
                  └─> Terminated ──> Archived
```

### Leave Request Lifecycle

```
Draft
  └─> Submitted
        ├─> Approved ──> Closed
        ├─> Rejected ──> Closed
        └─> Cancelled ──> Closed
```

### Payroll Run Lifecycle

```
Draft
  └─> Calculated
        └─> Approved
              ├─> Finalized ──> Archived
              └─> Reopened ──> Calculated
```

### Payslip Lifecycle

```
Pending
  └─> Generated
        └─> Published
              └─> Revised ──> Published
```

---

## Analytics

### KPI Refresh Lifecycle

```
Pending
  └─> Running
        ├─> Succeeded ──> Completed
        └─> Failed ──> Retrying ──> Running
```

### Export Job Lifecycle

```
Requested
  └─> Queued
        └─> Generating
              ├─> Completed ──> Expired
              └─> Failed
```

