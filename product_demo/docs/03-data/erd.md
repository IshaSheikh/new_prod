<!-- Source: PRDs | Last Verified: 2026-06-28 | Owner: Engineering -->
# Entity Relationship Diagram

High-level ERD organized by domain module. Full DDL definitions are in `postgres-schema.md`.

---

## Platform Core (M7.1)

```
users ──┐
        ├─< user_tenant_memberships >─── tenants ──── tenant_domains
        │           │                        │
        │           └─< membership_roles      └── tenant_settings
        │                    │                └── subscriptions ──── subscription_plans
        │                    └── roles                                      │
        │                         └─< role_permissions                      └── subscription_plan_features
        │                                  │                                        │
        │                                  └── permissions              platform_features
        │
        ├─< tenant_invitations
        ├─< password_reset_tokens
        ├─< auth_sessions
        └─< audit_logs
```

---

## School Onboarding (M7.2)

```
tenants
  └── schools ──── school_profiles
             │ └── school_settings
             │ └── branding_themes ──── branding_assets
             │ └── communication_settings
             │
             └─< academic_years ──── terms
             └─< grade_levels
             └─< sections ──── grade_levels
             │           └── academic_years
             └─< campuses
             └─< onboarding_checklists
             └─< onboarding_validation_runs ──── onboarding_validation_issues
```

---

## User Management / RBAC (M7.3)

```
users ──────────────────────────────────── user_tenant_memberships
        │                                           │
        └─< parent_student_links (as parent)        └─< membership_roles ──── roles
        └─< parent_student_links (as student)
        └─< teacher_assignments ──────── academic_years
                                    └── grade_levels
                                    └── sections
                                    └── subjects (→ M7.5)
```

---

## Student Information System (M7.4)

```
students ──── student_profiles
         │ └── student_custom_field_values ──── student_custom_field_definitions
         │                                            └─< student_custom_field_options
         │
         ├─< student_enrollments ──── academic_years
         │                      └── grade_levels
         │                      └── sections
         │                      └── campuses
         │
         ├─< guardian_student_links ──── guardians (can link to users)
         │
         ├─< student_emergency_contacts
         │
         ├─< student_documents ──── student_document_categories
         │         └─< student_document_versions
         │
         ├─< student_health_notes
         └─< student_status_history
```

---

## Academic Structure and Timetable (M7.5)

```
subjects ──────────────────────────────────────────────── subject_group_members
                                                                  │
         subject_groups ────────────────────────────────────────/

academic_teacher_assignments ──── teacher membership
                            │ └── academic_years
                            │ └── grade_levels
                            │ └── sections
                            │ └── subjects
                            └── campuses

rooms ──── campuses

period_schemes ──── academic_years
      └─< periods

timetables ──── academic_years
           │ └── grade_levels
           │ └── sections
           │ └── period_schemes
           └─< timetable_slots ──── periods
                              │ └── subjects
                              │ └── academic_teacher_assignments
                              └── rooms

timetable_publication_history ──── timetables
timetable_validation_runs ──────── timetables
      └─< timetable_validation_issues
```

---

## Attendance (M7.6)

```
attendance_sessions ──── academic_years
                    │ └── grade_levels
                    │ └── sections
                    └── timetable_slots (period-wise mode)

attendance_sessions └─< attendance_records ──── students
                                           │ └── student_enrollments
                                           │ └── attendance_statuses
                                           └─< attendance_corrections ──── attendance_statuses (old/new)
                                           └─< attendance_notifications ──── guardians
```

---

## Gradebook and Report Cards (M7.7)

```
assessments ──── academic_years
           │ └── grade_levels
           │ └── sections
           │ └── subjects
           └─< assessment_components (weight must sum to 100%)
           └─< assessment_student_rosters ──── students
                                         └── student_enrollments
           └─< marks_entries ──── assessment_components
                             └── assessment_student_rosters

grading_schemes └─< grading_scheme_ranges

report_card_templates └─< report_card_template_versions

result_publications ──── academic_years
      └─< result_publication_items ──── report_cards
                                              │
report_cards ──── students                   │
            │ └── academic_years             │
            │ └── report_card_templates      │
            └─< report_card_subject_results ──── subjects
                      └─< report_card_component_results ──── assessments

mark_change_audit_logs ──── marks_entries
report_card_revision_logs ──── report_cards (old + new)
```

---

## Fees and Finance (M7.8)

```
fee_categories ──── schools
fee_structures ──── academic_years
              │ └── grade_levels (optional)
              │ └── sections (optional)
              └─< fee_structure_lines ──── fee_categories

late_fee_rules ──── schools
discounts ──────── schools
scholarships ────── students

student_fee_assignments ──── students
                        │ └── student_enrollments
                        │ └── fee_structures
                        └─< student_fee_assignment_addons ──── fee_categories

invoices ──── students
         │ └── student_enrollments
         │ └── academic_years
         │ └── student_fee_assignments
         └─< invoice_line_items ──── fee_categories
                                └── fee_structure_lines

payment_transactions ──── students
                    │ └── invoices (optional)
                    └─< payment_allocations ──── invoices

receipts ──── payment_transactions
refunds ──── payment_transactions
adjustments ──── students
credit_balances ──── payment_transactions
ledger_entries ──── source records (polymorphic via source_type + source_id)

reconciliation_batches └─< reconciliation_items ──── payment_transactions
```

---

## Communication Hub (M7.10)

```
notices ──── schools
        │ └─< notice_audiences
        └─< outbound_messages ──── recipients
                              └─< communication_delivery_logs

message_templates ──── schools
communication_approvals ──── notices
automation_rules ──── schools
```

---

## Parent and Student Portal (M7.9)

```
portal_users ──── users
             └── memberships
             └─< portal_sessions
             └─< portal_profiles

parent_student_links (from M7.3)

portal_notices ──── schools
          └─< portal_notice_audiences
          └─< portal_notice_reads ──── portal_users

portal_documents ──── students (optional)
portal_download_logs ──── portal_users

portal_support_requests ──── portal_users
                        └─< portal_support_messages

portal_feature_flags ──── schools
portal_access_policies ──── schools
portal_audit_logs ──── portal_users
```

---

## Admissions (M7.11)

```
admission_inquiries ──── schools
      └── academic_years
      └── grade_levels
      └── (converts to applicants)

applicants ──── admission_inquiries
           └── academic_years
           └── grade_levels
           └─< application_forms ──── application_form_versions
           └─< documents_submitted
           └─< assessment_results
           └─< offers └─< enrollment_contracts
           └── (handoff to students in SIS)
```

---

## Transport (M7.14)

```
vehicles ──── schools
routes ──── schools
       └── vehicles
       └── drivers
       └─< route_stops ──── stops

student_transport_assignments ──── students
                              └── routes
                              └── stops

transport_payments ──── students
                   └── student_transport_assignments
```

---

## Library (M7.15)

```
books ──── schools
      └─< book_copies
            └─< issues ──── students
                       └─< returns
                       └─< fines
```

---

## HR and Payroll (M7.16)

```
staff_profiles ──── schools
               └── users (optional link)
               └─< staff_attendance
               └─< leave_requests
               └─< leave_balances

salary_structures ──── schools
             └─< salary_components

payroll_runs ──── schools
           └─< payroll_items ──── staff_profiles
                            └─< payslips

statutory_details ──── staff_profiles
```

---

## Analytics (M7.17)

```
dashboard_definitions ──── schools
      └─< dashboard_widgets

kpi_definitions
      └─< kpi_snapshots ──── tenants

dashboard_exports ──── dashboard_definitions
kpi_refresh_jobs ──── kpi_definitions
```

