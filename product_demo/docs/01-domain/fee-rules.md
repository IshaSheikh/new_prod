<!-- Source: PRDs | Last Verified: 2026-06-28 | Owner: Engineering -->
# Fee Rules

Detailed rules governing the Fees and Finance module (M7.8). These rules supplement the general business rules in `business-rules.md`.

---

## Fee Structure Rules

**FR-001 — Academic year versioning**
Fee structures must be versioned by academic year. A new academic year requires a new fee structure version. Historical structures are preserved and referenced by previously issued invoices.

**FR-002 — Grade and section specificity**
Fee structures may be defined at:
- School-wide level (no grade/section specified)
- Grade level (applies to all sections in a grade)
- Section level (applies to a specific section)
- Student level (override via `student_fee_assignments`)

More specific assignments override more general ones. A student-level assignment always takes precedence.

**FR-003 — One active assignment per billing context**
Each student must have exactly one active fee structure per billing context (e.g., one for tuition, one for transport). Multiple active assignments for the same context are blocked.

**FR-004 — Optional services as add-ons**
Transport, hostel, activity fees, and other optional services must be modeled as add-on items on student fee assignments, not forced into all student invoices.

**FR-005 — Retroactive protection**
A student changing class or section does not retroactively alter already-issued invoices. The fee structure effective at invoice creation time governs that invoice permanently.

---

## Invoice Rules

**FR-006 — Invoice snapshot immutability**
Invoice totals are stored as snapshots at issuance time:
- `subtotal` — sum of line item charges
- `discount_total` — total discounts applied
- `scholarship_total` — total scholarship reductions
- `tax_total` — total taxes applied
- `penalty_total` — total late fee penalties
- `adjustment_total` — total manual adjustments
- `balance_due` — `subtotal - discount_total - scholarship_total + tax_total + penalty_total + adjustment_total - paid_total + refunded_total`

These amounts are never recalculated from live setup rules after issuance.

**FR-007 — Discount application order**
Discount calculation must follow this deterministic sequence:
1. Line-item discounts (applied to individual charge lines)
2. Invoice-level discounts (applied to the invoice subtotal)
3. Scholarship reductions (reported separately from discounts)
4. Late fee penalties (added, not subtracted)

**FR-008 — Due date explicitness**
Every issued invoice must have an explicit `due_date`. The system derives overdue status when `CURRENT_DATE > due_date AND balance_due > 0`.

**FR-009 — Negative invoices prohibited**
Invoice totals cannot be negative. If credits exceed charges, the excess becomes an unapplied credit balance — not a negative invoice.

**FR-010 — Cancellation requirements**
Cancelling an invoice requires:
- The invoice must be in `draft` or `issued` state (no payments applied)
- A cancellation reason must be recorded
- Cancelled invoices cannot receive new payments

---

## Payment Rules

**FR-011 — Partial payments**
Partial payments are allowed. The system tracks `paid_total` on the invoice and updates state accordingly (to `partially_paid`).

**FR-012 — Payment-invoice independence**
Payment transaction status is independent from invoice status. A payment can fail while the invoice remains `issued` or `overdue`.

**FR-013 — Offline payment fields**
Offline cash/cheque/bank payments must record:
- Payment method
- Amount
- Received date
- Received by (staff user)
- Reference number (cheque number, bank reference, etc.) — when available

**FR-014 — Duplicate detection**
The system computes a `duplicate_check_hash` on each payment transaction using student ID, amount, and a time window. Suspiciously similar transactions trigger a warning before confirmation.

**FR-015 — Idempotency**
Online payments and gateway webhooks must use idempotency keys. Replayed webhook callbacks must be processed exactly once.

**FR-016 — Failed payment visibility**
Failed payment attempts remain permanently visible in the transaction history. They cannot be deleted.

**FR-017 — Overpayment handling**
If a payment amount exceeds the outstanding balance:
- Option 1: Create a `credit_balance` record for the excess amount
- Option 2: Initiate a refund workflow for the excess

The excess must never silently disappear or reduce an unrelated invoice without explicit allocation.

---

## Receipt Rules

**FR-018 — Receipt trigger**
Receipts are generated only for payments in `success` status. Pending or failed payments do not generate receipts.

**FR-019 — Receipt immutability**
Receipts cannot be edited after issuance. The only correction mechanism is:
1. Create a reversal entry linking the new receipt to the original
2. Issue a corrected receipt

**FR-020 — Receipt numbering**
Receipt numbers are unique per tenant and school. Schools can configure a prefix (e.g., `RCP-`) and the sequence is auto-incremented. Gaps in the sequence are acceptable but must not be re-used.

**FR-021 — Receipt timing**
Receipts must be generated within 30 seconds of payment confirmation for online payments and within the same session for offline payments.

---

## Late Fee Rules

**FR-022 — Late fee calculation modes**
Schools can configure late fees in three modes:
- `fixed_amount` — a flat penalty applied after the trigger date
- `percentage` — a percentage of the outstanding balance
- `per_day` — a daily rate applied for each day overdue

**FR-023 — Late fee cap**
A maximum cap on total late fees per invoice can be configured. Once the cap is reached, no additional late fees are added.

**FR-024 — Late fee application**
Late fees are applied through the invoice line item system as a `penalty` line type, preserving the original charge amounts intact.

**FR-025 — Late fee waiver**
Late fees can be waived per invoice by authorized staff. The waiver creates a manual adjustment record with a mandatory reason.

---

## Refund Rules

**FR-026 — Refund eligibility**
Refunds can only be initiated against payments in `success` or `reconciled` status.

**FR-027 — Refund amount limit**
Total refund amount across all refunds for one payment must not exceed the confirmed collected amount. The system enforces this before allowing refund initiation.

**FR-028 — Partial refunds**
Partial refunds are supported. After a partial refund, the payment moves to `refunded_partial` status. After full refund, it moves to `refunded_full`.

**FR-029 — Refund modes**
Supported refund modes:
- `original_method` — reversal to original payment method (online gateway, card)
- `bank_transfer` — manual bank transfer (offline)
- `cash` — cash refund at the counter
- `cheque` — cheque issued
- `credit_balance` — applied as a credit balance for future invoices

**FR-030 — Refund approval**
Schools can configure refund approval workflows. When configured, refunds require explicit approval from an authorized School Admin before processing.

---

## Reconciliation Rules

**FR-031 — Reconciliation scope**
Reconciliation matches internal `payment_transactions` against external settlement data (gateway settlement file, bank statement import).

**FR-032 — Match statuses**
Each reconciliation item has one of:
- `unreconciled` — not yet matched
- `matched` — internal and external amounts align
- `mismatched` — amounts or references don't match
- `manually_resolved` — resolved by authorized staff with a reason

**FR-033 — Non-mutation principle**
Reconciliation must not alter original `payment_transactions` records. It adds `match_status` and resolution references on the reconciliation layer.

**FR-034 — Settlement difference handling**
When gateway settlement differs from captured amount (e.g., due to gateway fees), the difference is recorded in `reconciliation_items` with notes. This does not create a credit or debit in the student ledger automatically — manual resolution is required.

---

## Ledger and Audit Rules

**FR-035 — Ledger completeness**
Every invoice issuance, payment, refund, adjustment, and reversal must create corresponding immutable `ledger_entries`. The ledger is the source of truth for financial history reconstruction.

**FR-036 — Ledger immutability**
Ledger entries cannot be updated or deleted after creation. Corrections are made through new, reversing ledger entries.

**FR-037 — Finance audit coverage**
The following actions must generate `finance_audit_logs` entries:
- Invoice issue
- Invoice cancellation
- Payment recorded
- Payment confirmed (webhook)
- Receipt issued
- Receipt reversed
- Refund requested
- Refund approved
- Refund completed
- Manual adjustment applied
- Discount or scholarship applied
- Reconciliation batch uploaded
- Reconciliation item resolved

---

## Reporting Definitions

**FR-038 — Collection efficiency**
Defined as: `(paid_total for period / issued_total for period) × 100`
Excludes draft invoices. Includes partial payments.

**FR-039 — Outstanding amount**
Sum of `balance_due` across all `issued`, `partially_paid`, and `overdue` invoices.

**FR-040 — Aging buckets**
- Current: due date in the future
- 1–30 days overdue
- 31–60 days overdue
- 61–90 days overdue
- 90+ days overdue

**FR-041 — Defaulter definition**
A student with at least one invoice in `overdue` status as of the reporting date.

