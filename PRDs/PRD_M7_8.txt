Below is a full PRD for **Module 7.8 Fees and Finance**, designed for your multi-tenant K-12 platform and aligned with the platform patterns already established for onboarding, RBAC, SIS, attendance, and gradebook: centralized source-of-truth modules, strict school-level isolation, role-scoped access, and immutable published records.

# 7.8 Fees and Finance PRD

## 1. Module objective

The Fees and Finance module automates fee setup, invoice generation, payment collection, receipt issuance, refund handling, and financial reconciliation for each school tenant.

It exists to:
- Reduce manual fee administration.
- Improve parent visibility into dues, payments, and balances.
- Increase on-time collections.
- Create auditable, measurable school finance operations.
- Provide finance-ready records without duplicating student master data from the SIS. 
The module must support:
- Multi-school operation with strict tenant isolation.
- Student-specific fee assignment through fee structures.
- Monthly and term-based billing.
- Partial payments and mixed payment channels.
- Immutable receipts and auditable corrections.
- Reconciliation between system transactions and actual settlement records.

***

## 2. Why schools need it

Schools usually manage fees across spreadsheets, paper receipts, bank messages, and separate accountant workflows. That creates billing errors, unclear balances, delayed parent communication, and weak accountability.

Schools need this module to:
- Standardize fee policies across classes, terms, and categories.
- Automatically generate invoices at scale.
- Track outstanding balances student by student.
- Allow parents to view dues and payment history clearly.
- Reduce front-desk workload for accountants and admins.
- Maintain a trustworthy audit trail for issued receipts, failed payments, refunds, and reversals.
- Monitor collection efficiency through dashboards and defaulter tracking.

This module is also a direct business-value driver because fee collection is one of the most visible administrative pain points in K-12 operations.

***

## 3. Primary users

| Role | Why they use the module |
|---|---|
| Super Admin | Monitor tenant-level usage, investigate incidents, support configuration issues, view audit data when authorized. |
| School Admin | Configure fee categories, fee structures, billing cycles, penalties, and finance policies for the school. |
| Principal | View collection performance, defaulter trends, waiver summaries, and operational finance KPIs. |
| Teacher | View fee status when relevant to student follow-up, but should not manage financial records by default. |
| Accountant | Core operational user for invoice issuance, payment recording, reconciliation, refunds, and ledger review. |
| Parent | View invoices, balances, due dates, receipts, payment history, and make payments. |
| Student | View own fee status and receipts where student self-service is enabled. |

Primary operational ownership sits with School Admin and Accountant. Student identity, enrollment, and guardian linkage should come from the SIS rather than being recreated inside finance. 
***

## 4. User stories

### Fee setup
- As a School Admin, I want to create fee categories, so billing is organized into tuition, transport, books, exam fees, lab fees, and other heads.
- As a School Admin, I want to define fee structures by academic year, grade, section, or student group, so invoicing can be generated consistently.
- As a School Admin, I want to assign a fee structure to a student, so the student is billed correctly.
- As a School Admin, I want to configure monthly, term-based, one-time, and optional fees, so the school can support different charging models.
- As a School Admin, I want to define late fee rules, so overdue payments are handled automatically.

### Billing
- As an Accountant, I want to generate invoices in bulk, so manual billing work is minimized.
- As an Accountant, I want invoices to include line items, discounts, scholarships, taxes if applicable, and prior balance adjustments, so the payable amount is accurate.
- As an Accountant, I want to issue invoices immediately or schedule them, so billing can align with school policy.
- As an Accountant, I want to cancel draft invoices and reverse issued invoices through controlled flows, so mistakes are corrected without deleting history.

### Payments
- As a Parent, I want to see all unpaid and overdue invoices in one place, so I understand what is due.
- As a Parent, I want to make full or partial payments online, so I can pay conveniently.
- As an Accountant, I want to record offline payments such as cash, cheque, bank transfer, or POS, so all collections are captured in one ledger.
- As an Accountant, I want failed payment attempts to remain visible, so disputed or incomplete transactions are auditable.
- As an Accountant, I want overpayments to become either unapplied credit or a controlled refund, so balances stay correct.

### Receipts and reconciliation
- As an Accountant, I want receipts generated automatically for successful payments, so parents receive proof of payment immediately.
- As an Accountant, I want receipts to remain immutable after issuance, so the school has a reliable audit record.
- As an Accountant, I want bank settlement and gateway payment references matched against system transactions, so reconciliation is fast and accurate.
- As a School Admin, I want every reversal, refund, and manual adjustment logged with reason and actor, so finance operations are auditable.

### Visibility and reporting
- As a Principal, I want a defaulters dashboard by class, section, and aging bucket, so intervention is targeted.
- As a Parent, I want to download receipts and view payment history, so I do not need to contact the office repeatedly.
- As a School Admin, I want collection KPIs by fee category and time period, so school operations become measurable.
- As a Super Admin, I want tenant-safe audit access for support cases, so platform issues can be investigated without breaking school isolation.

***

## 5. Business rules

### Tenant and identity
- Every finance record must belong to exactly one tenant and one school.
- No invoice, payment, receipt, refund, or ledger entry may reference entities from another tenant.
- Student, parent, class, section, and academic-year references must come from platform master modules, especially SIS and school configuration.- Soft deletion is allowed only for setup records where no financial transaction has occurred; financial transactions themselves must never be hard deleted.

### Fee structure rules
- Each student must be assigned one active fee structure per billing context.
- A fee structure may inherit defaults from grade or academic year and then allow student-level overrides.
- Fee structures must be versioned by academic year.
- Optional services such as transport, hostel, or activity fees must be modeled as add-on items, not forced into all student invoices.
- A student changing class or section does not retroactively alter already issued invoices.

### Invoice rules
- Invoices may be generated as draft or issued directly.
- Only issued invoices can accept payments.
- Invoices may be monthly, term-based, one-time, or ad hoc.
- Each invoice must contain one or more line items.
- Discounts may be fixed amount or percentage.
- Scholarships may be modeled separately from standard discounts for reporting clarity.
- Discount application order must be deterministic: line-item discount first, invoice-level discount second, scholarship third, penalty last.
- Invoice totals must be stored as calculated snapshots, not recalculated from mutable setup rules after issuance.
- Due dates must be explicit on every issued invoice.
- Overdue status is system-derived when current date passes due date and outstanding amount remains greater than zero.
- Cancelled invoices cannot receive new payments.

### Payment rules
- Partial payments are allowed.
- A single payment may be allocated to one or many invoices only if the school enables split allocation; otherwise one payment maps to one invoice.
- Payment status must be independent from invoice status. A payment transaction can fail while the invoice remains issued or overdue.
- Failed online payments must remain auditable and visible in transaction history.
- Offline payment recording requires payment method, amount, received date, receiver, and reference number when available.
- Duplicate payment detection should warn users when amount, student, reference, and timestamp are suspiciously similar.
- Overpayment must create unapplied credit or trigger a refund workflow; it must never silently disappear.

### Receipt rules
- Receipts are generated only for successful or confirmed payments.
- Receipts must be immutable after issuance.
- Corrections require reversal, adjustment entry, or refund flow; direct editing of an issued receipt is not allowed.
- Receipt numbering must be unique per tenant, with configurable school-level prefix and sequence rules.

### Refund and adjustment rules
- Refunds may be partial or full.
- Refunds require reason, approver where configured, original payment reference, and refund mode.
- A refund must create reversing ledger entries and update invoice balance state.
- Negative invoice totals are not allowed; credit notes or unapplied credits should be used instead.
- Manual adjustments require mandatory reason capture and actor audit.

### Reconciliation rules
- Reconciliation compares internal payment transactions with external settlement evidence such as gateway settlement reports or bank statement imports.
- A transaction can be unreconciled, matched, mismatched, or manually resolved.
- Reconciliation must not alter original payment records; it adds reconciliation state and references.
- Finance reports must distinguish between collected, settled, refunded, waived, overdue, and written-off amounts.

***

## 6. State machine

### Invoice state machine

| State | Meaning | Allowed transitions |
|---|---|---|
| `draft` | Invoice prepared but not yet active | `issued`, `cancelled` |
| `issued` | Invoice active and payable | `partially_paid`, `paid`, `overdue`, `cancelled` |
| `partially_paid` | Some amount paid, balance remains | `paid`, `overdue`, `cancelled` (only if policy allows with reversal) |
| `paid` | Fully settled | `refunded_partial`, `refunded_full` |
| `overdue` | Due date passed and balance remains | `partially_paid`, `paid`, `cancelled` (controlled only) |
| `cancelled` | Invoice voided before active financial completion | No normal forward transitions |
| `refunded_partial` | Some paid amount refunded | `refunded_full` if remaining paid amount is fully refunded |
| `refunded_full` | All paid amount refunded | Terminal |

### Payment transaction state machine

| State | Meaning | Allowed transitions |
|---|---|---|
| `initiated` | Payment flow started | `pending`, `success`, `failed`, `cancelled` |
| `pending` | Awaiting gateway/bank confirmation | `success`, `failed`, `expired` |
| `success` | Payment confirmed | `reconciled`, `refund_initiated` |
| `failed` | Payment attempt unsuccessful | Terminal |
| `cancelled` | User or operator cancelled payment | Terminal |
| `expired` | Payment window expired | Terminal |
| `reconciled` | Matched with settlement source | `refund_initiated` |
| `refund_initiated` | Refund requested | `refunded_partial`, `refunded_full`, `refund_failed` |
| `refunded_partial` | Part refunded | `refunded_full` |
| `refunded_full` | Fully refunded | Terminal |
| `refund_failed` | Refund attempt failed | `refund_initiated` |

### Receipt state model
- `issued`
- `reversed`

A receipt should not support edit state. If an issued receipt is wrong, the system creates a reversal entry and then issues a new correct receipt.

***

## 7. Required data entities

### Core entities
- `fee_categories`
- `fee_structures`
- `fee_structure_lines`
- `student_fee_assignments`
- `invoices`
- `invoice_line_items`
- `discounts`
- `scholarships`
- `payment_transactions`
- `payment_allocations`
- `receipts`
- `refunds`
- `ledger_entries`
- `late_fee_rules`
- `reconciliation_batches`
- `reconciliation_items`
- `adjustments`
- `credit_balances`

### Suggested entity definitions

| Entity | Purpose | Key fields |
|---|---|---|
| `fee_categories` | Master list of charge types | id, tenant_id, school_id, code, name, type, optional_flag, active |
| `fee_structures` | Billing template for a school context | id, academic_year_id, grade_id, section_id nullable, billing_frequency, currency, status, version |
| `fee_structure_lines` | Charge rows inside a structure | id, fee_structure_id, fee_category_id, amount, due_rule, recurrence_type, tax_rule nullable |
| `student_fee_assignments` | Active fee plan per student | id, student_id, fee_structure_id, effective_from, effective_to nullable, custom_override_json, status |
| `invoices` | Bill header | id, student_id, invoice_no, period_start, period_end, due_date, subtotal, discount_total, scholarship_total, penalty_total, paid_total, balance_due, state |
| `invoice_line_items` | Detailed charges per invoice | id, invoice_id, fee_category_id, description, quantity, unit_amount, line_discount, line_total |
| `discounts` | Concessions or promotional reductions | id, scope_type, scope_id, discount_type, value, reason, approval_required, active |
| `scholarships` | Named student aid records | id, student_id, scholarship_code, type, value, sponsor nullable, start_date, end_date, status |
| `payment_transactions` | Each collection attempt or confirmation | id, student_id, invoice_id nullable, amount, currency, method, channel, gateway_ref nullable, bank_ref nullable, received_at, status |
| `payment_allocations` | Maps money to invoices | id, payment_transaction_id, invoice_id, allocated_amount |
| `receipts` | Immutable proof of payment | id, receipt_no, payment_transaction_id, issued_at, issued_by, total_amount, status |
| `refunds` | Refund workflow record | id, payment_transaction_id, refund_no, amount, reason, mode, requested_by, approved_by nullable, status |
| `ledger_entries` | Double-entry or operational ledger rows | id, source_type, source_id, entry_type, debit, credit, posted_at, account_code nullable, immutable_hash |
| `late_fee_rules` | Penalty configuration | id, school_id, rule_name, trigger_days, mode, value, cap_amount nullable, active |
| `reconciliation_batches` | Imported or manual recon session | id, source_type, batch_date, uploaded_by, status |
| `reconciliation_items` | Match results per external row | id, reconciliation_batch_id, payment_transaction_id nullable, external_ref, external_amount, match_status, resolved_by nullable |
| `adjustments` | Non-refund corrections | id, student_id, invoice_id nullable, amount, adjustment_type, reason, created_by |
| `credit_balances` | Unapplied credits available for future use | id, student_id, source_payment_transaction_id, available_amount, status |

### Cross-module dependencies
- Student and guardian links must come from SIS.- Academic year, class, section, and school settings must come from onboarding/configuration.
- Role enforcement must come from the RBAC service, not be hardcoded inside finance screens. 
***

## 8. Permissions matrix

Legend:
- V = view
- C = create/configure
- E = edit/update
- A = approve
- R = reconcile/reverse
- P = pay
- D = download

| Capability | Super Admin | School Admin | Principal | Teacher | Accountant | Parent | Student |
|---|---|---:|---:|---:|---:|---:|---:|
| View fee categories | V | V/C/E | V | - | V | - | - |
| Manage fee structures | V | V/C/E/A | V | - | V | - | - |
| Assign fee structure to student | V | V/C/E | V | - | V | - | - |
| Generate invoices | V | V/C/E | V | - | V/C/E | - | - |
| Issue invoices | V | V/A | V | - | V/C/E | - | - |
| Cancel draft invoice | V | V/A | - | - | V/E | - | - |
| View student ledger | V | V | V | Limited | V | V (own child only) | V (self only) |
| Record offline payment | V | V | - | - | V/C/E | - | - |
| Initiate online payment | - | - | - | - | Optional | V/P | V/P optional |
| View payment transactions | V | V | V summary | - | V/C/E | V own | V own |
| Issue receipt | V | V | - | - | V/C | - | - |
| Download receipt | V | V | V | - | V | V/D own | V/D self |
| Process refund | V | V/A | Optional V | - | V/C | - | - |
| Reconcile transactions | V | V | V summary | - | V/C/R | - | - |
| View defaulters dashboard | V | V | V | Limited assigned students | V | - | - |
| View audit trail | V | V | Limited | - | Limited finance records | - | - |

Notes:
- Parent access must be restricted to linked children only, which follows the platform’s parent-student relationship model. 
- Teacher access should default to limited visibility and should never include payment editing.
- Super Admin access is platform-support oriented and should be governed by audit logging and tenant-safe access controls. 
***

## 9. APIs needed

### Fee configuration
| Method | Endpoint | Purpose |
|---|---|---|
| POST | `/api/v1/fee-categories` | Create fee category |
| GET | `/api/v1/fee-categories` | List fee categories |
| PATCH | `/api/v1/fee-categories/{id}` | Update fee category |
| POST | `/api/v1/fee-structures` | Create fee structure |
| GET | `/api/v1/fee-structures` | List fee structures |
| GET | `/api/v1/fee-structures/{id}` | Get structure details |
| PATCH | `/api/v1/fee-structures/{id}` | Update structure |
| POST | `/api/v1/students/{studentId}/fee-assignment` | Assign fee structure to student |
| GET | `/api/v1/students/{studentId}/fee-assignment` | View active assignment |

### Billing
| Method | Endpoint | Purpose |
|---|---|---|
| POST | `/api/v1/invoices/generate` | Bulk-generate invoices |
| POST | `/api/v1/invoices` | Create ad hoc invoice |
| GET | `/api/v1/invoices` | Filter invoice list |
| GET | `/api/v1/invoices/{id}` | Get invoice details |
| POST | `/api/v1/invoices/{id}/issue` | Move draft to issued |
| POST | `/api/v1/invoices/{id}/cancel` | Cancel invoice through rule-based flow |
| POST | `/api/v1/invoices/{id}/recalculate-preview` | Preview recalculation before issue |
| GET | `/api/v1/students/{studentId}/ledger` | Student fee ledger |

### Payments
| Method | Endpoint | Purpose |
|---|---|---|
| POST | `/api/v1/payments/offline` | Record cash/cheque/bank payment |
| POST | `/api/v1/payments/online/session` | Create online payment session |
| POST | `/api/v1/payments/webhook` | Gateway callback endpoint |
| GET | `/api/v1/payments` | List transactions |
| GET | `/api/v1/payments/{id}` | Payment detail |
| POST | `/api/v1/payments/{id}/allocate` | Allocate payment to invoices |
| POST | `/api/v1/payments/{id}/retry` | Retry failed payment flow |
| GET | `/api/v1/payments/{id}/receipt` | Fetch receipt |

### Refunds and reconciliation
| Method | Endpoint | Purpose |
|---|---|---|
| POST | `/api/v1/refunds` | Initiate refund |
| GET | `/api/v1/refunds` | List refunds |
| POST | `/api/v1/refunds/{id}/approve` | Approve refund |
| POST | `/api/v1/refunds/{id}/complete` | Complete refund |
| POST | `/api/v1/reconciliation/batches` | Upload reconciliation batch |
| GET | `/api/v1/reconciliation/batches/{id}` | View batch results |
| POST | `/api/v1/reconciliation/items/{id}/match` | Manually match row |
| POST | `/api/v1/reconciliation/items/{id}/resolve` | Resolve mismatch |

### Reporting
| Method | Endpoint | Purpose |
|---|---|---|
| GET | `/api/v1/reports/fees/defaulters` | Defaulters list with filters |
| GET | `/api/v1/reports/fees/collections` | Collection KPI summary |
| GET | `/api/v1/reports/fees/aging` | Aging analysis |
| GET | `/api/v1/reports/fees/category-breakdown` | Revenue by category |
| GET | `/api/v1/reports/fees/refunds` | Refund monitoring |
| GET | `/api/v1/audit/finance` | Finance audit trail |

### API design notes
- Every endpoint must enforce tenant context from auth claims, never from client-selected tenant ID.
- Use idempotency keys for payment initiation, webhook processing, refund completion, and receipt issuance.
- Webhooks must be signature-validated and replay-safe.
- Pagination, filtering, and export jobs are required for large schools.

***

## 10. Screens needed

### Admin and accountant screens
1. **Fee setup**
- Fee categories
- Fee structures
- Recurrence rules
- Late fee rules
- Discount and scholarship setup
- Assignment rules by class/section/student

2. **Student fee assignment**
- Search student
- View current structure
- Apply override
- View effective dates
- Preview upcoming invoices

3. **Invoice list**
- Filters by academic year, class, section, status, due date, category
- Bulk actions: generate, issue, export, remind
- Quick summary totals

4. **Invoice detail**
- Header summary
- Line items
- Payment history
- Receipt links
- Refunds and adjustments
- Audit timeline

5. **Student fee ledger**
- Chronological ledger of invoices, payments, refunds, adjustments, credits
- Current balance
- Aging bucket
- Download statement

6. **Payment collection**
- Record offline payment
- Scan/search student
- Select invoices
- Allocate amount
- Generate receipt

7. **Payment reconciliation**
- Import settlement file
- Match status
- Exceptions queue
- Manual resolution panel

8. **Receipt view**
- Printable receipt
- Immutable version display
- Reversal reference if applicable

9. **Defaulters dashboard**
- By class, section, grade, aging, category
- Follow-up status
- Reminder actions
- Trend view

10. **Finance analytics dashboard**
- Daily collection
- Month-to-date collection
- Collection vs billed
- Overdue aging
- Refund rate
- Scholarship/discount totals

### Parent and student screens
11. **My fees**
- Outstanding dues
- Upcoming due dates
- Paid invoices
- Overdue alerts

12. **Invoice detail and pay**
- View line items
- Choose payment mode
- Partial/full payment
- Download invoice

13. **Receipts and payment history**
- Transaction list
- Receipt download
- Refund status if any

### Principal screen
14. **Finance oversight dashboard**
- High-level collection metrics
- Defaulter heatmap
- Trend summaries
- Waiver and refund monitoring

***

## 11. Edge cases

- Student changes school section after invoice issuance.
- Student withdraws mid-term after partial payment.
- Student transfers out with unused advance balance.
- Fee structure changes after invoices were already issued.
- Discount added after payment already happened.
- Same student has transport fee for only part of the year.
- Parent pays one amount intended for multiple siblings.
- Gateway marks payment success but webhook delivery is delayed.
- Webhook is replayed twice.
- Offline cash payment is entered twice by mistake.
- Cheque payment is recorded, receipt issued, then cheque bounces.
- Refund requested for a partially allocated payment.
- Invoice is fully paid, then one line item is disputed.
- School wants to waive late fee without waiving tuition.
- Negative balance appears because refund exceeds confirmed collected amount.
- Settlement amount differs from captured amount because of gateway charges.
- Parent has access to multiple linked children and must not mix balances incorrectly.
- Financial year closes while previous invoice remains overdue.
- Receipt number sequence breaks because of failed issuance attempt.
- School imports a bank reference that matches more than one pending payment.
- Tenant admin tries to export data outside their school scope.

***

## 12. Risks

| Risk | Impact | Mitigation |
|---|---|---|
| Incorrect fee configuration | Large-scale wrong billing | Versioned structures, preview before issue, approval workflow |
| Weak tenant isolation | Severe data breach | Tenant-scoped auth, row-level enforcement, audit review |
| Payment duplication | Parent distrust, ledger errors | Idempotency keys, duplicate detection, webhook replay protection |
| Mutable receipts | Audit and compliance failure | Immutable receipt model, reversal-only corrections |
| Poor reconciliation workflow | Accountant workload stays high | Import tools, match suggestions, exceptions queue |
| Over-complex V1 finance accounting | Delivery delays for solo founder | Keep V1 focused on billing, collection, receipts, refunds, reconciliation |
| Parent confusion on balances | Higher support burden | Clear ledger, invoice-level breakdown, status explanations |
| Dependency mismatch with SIS/RBAC | Broken access and bad student links | Shared master IDs, contract-first integration, role policy tests |
| Regulatory or tax variation by school | Rework later | Configurable fields, region-aware abstractions, country-neutral core |

***

## 13. Success metrics

### Product metrics
- 90%+ of active students have an assigned fee structure before billing cycle start.
- 95%+ of issued invoices are generated without manual correction.
- Less than 2% of payment records require manual finance support intervention.
- 95%+ of successful payments produce receipt issuance within 30 seconds.
- Less than 1% unreconciled successful transactions after 3 business days.

### Operational metrics
- Reduce invoice preparation time by at least 70% versus spreadsheet workflow.
- Reduce receipt-generation time at collection desk to under 1 minute.
- Reduce parent balance-related support queries by at least 40%.
- Cut month-end reconciliation effort by at least 50%.
- Enable defaulter identification by class/section in under 10 seconds.

### Business metrics
- Improve on-time fee collection rate.
- Increase digital payment share over time.
- Improve school retention by making fee operations more reliable and transparent.
- Increase module adoption among commercial schools because finance automation is a purchase-driving capability.

### Quality metrics
- Zero cross-tenant data leakage incidents.
- 100% audit trail coverage for invoice issue, payment record, receipt issue, refund, reversal, and reconciliation actions.
- No hard deletion of financial transactions in production.

***

## 14. Out of scope

The following should be excluded from V1 of this module:
- Full general ledger accounting suite.
- Payroll processing.
- Vendor payments and procurement.
- Tax filing engine.
- Multi-currency accounting.
- Advanced franchise or trust-level consolidated accounting.
- AI-based collection prediction.
- Loan management or student financing products.
- Inventory-linked bookstore billing.
- Hostel and transport operations beyond fee billing hooks.
- Budgeting and expense management.
- Deep accounting integrations beyond export or basic connectors.
- Complex sibling wallet/shared family credit in V1 unless urgently required by your target schools.

### V1 recommendation
For a solo founder, the highest-value V1 is:
1. Fee categories and fee structures.
2. Student fee assignment.
3. Invoice generation and issuance.
4. Offline and online payments.
5. Immutable receipts.
6. Refunds and adjustments.
7. Reconciliation.
8. Defaulters and collection dashboards.

That sequence keeps scope commercially strong while staying consistent with the platform’s document-first, auditable module architecture. 
Would you like this converted next into the same PRD text format as your attached module files, with formal section wording and IDs like `US-001`, `BR-001`, and `API-001`?