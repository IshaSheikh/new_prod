<!-- Source: PRDs | Last Verified: 2026-06-28 | Owner: Engineering -->
# User Journeys

Critical user journeys across the platform, organized by persona. These journeys define the primary flows the product must support to deliver the core value proposition.

---

## Journey 1: School Admin — New School Onboarding

**Goal:** Get a new school from sign-up to operationally live within one day.

**Steps:**

1. **Receives invitation email** from Super Admin
2. **Creates account** via invitation link
3. **Lands on onboarding wizard** — sees progress checklist with 11 steps
4. **Step 1 — School Profile:** Enters school name, contact email, phone, address, timezone
5. **Step 2 — Academic Year:** Creates "2026-2027" academic year with start and end dates
6. **Step 3 — Terms:** Creates Term 1 and Term 2 within the academic year
7. **Step 4 — Grades:** Creates Grade 1 through Grade 10, sets sort order
8. **Step 5 — Sections:** Creates Grade 1-A, 1-B, Grade 2-A, etc., sets capacities
9. **Step 6 — Campuses:** Creates primary campus with address
10. **Step 7 — Branding:** Uploads school logo (PNG/JPG/SVG ≤5MB)
11. **Step 8 — Fee Categories:** Confirms default categories (Tuition, Exam, Transport, etc.)
12. **Step 9 — Communication Settings:** Sets sender email, SMS sender ID (optional)
13. **Step 10 — Invite Teacher:** Sends invitation to at least one teacher
14. **Step 11 — Readiness Check:** System validates all mandatory steps are complete
15. **Go Live:** School transitions to "Live" status — all modules unlocked

**Success:** School is operational by afternoon of the same day.

---

## Journey 2: School Admin — Add Students and Collect First Fee

**Goal:** Register a batch of new students, assign fee structures, and collect the first payment.

**Steps:**

1. **Navigate to Students → Add Student** for each new admission
2. **Fill student profile:** name, DOB, gender, address
3. **Link guardian:** search for existing guardian or create new, mark as primary, enable portal access
4. **Enroll student:** select academic year, grade, and section
5. **Assign fee structure:** select the fee structure for the student's grade
6. **Generate invoices:** bulk generate Term 1 invoices for the grade
7. **Issue invoices:** move invoices from Draft → Issued
8. **Record first payment:** select student, enter cash amount, generate receipt
9. **Share receipt** with parent (or parent downloads via portal)

**Success:** Student is enrolled, invoice is issued, payment is recorded, receipt is available.

---

## Journey 3: Teacher — Daily Attendance

**Goal:** Mark attendance for a class within 5 minutes at the start of the school day.

**Steps:**

1. **Opens app on mobile (Flutter) or web**
2. **Navigates to Attendance** — sees today's sessions for assigned classes
3. **Selects Grade 6-A session** — loads student roster
4. **Clicks "Mark All Present"** — all students set to Present
5. **Marks 2 students as Absent** — taps their names, changes status
6. **Marks 1 student as Late** — adds a remark
7. **Reviews attendance summary** — 32 present, 2 absent, 1 late
8. **Submits attendance** — session moves to Submitted
9. **System triggers automatic absence notifications** to guardians of absent students

**Time:** Under 2 minutes for a class of 35.

---

## Journey 4: Parent — Checking Child's Attendance and Paying a Fee

**Goal:** Parent opens the app, checks today's attendance, and pays an overdue fee.

**Steps:**

1. **Opens Flutter app, logs in with email**
2. **Lands on Parent Dashboard:**
   - Shows Arjun (Grade 1-A): ✓ Present today
   - Shows overdue fee: ₹4,999 due 3 days ago
3. **Taps on Attendance → Views last 30 days:** Attendance rate 94%, 2 absences
4. **Taps on Fees → Views outstanding invoices:**
   - Invoice #INV-2026-1042: ₹4,999 (Tuition Term 1) — Overdue
5. **Taps "Pay Now"** → selects UPI/card payment
6. **Completes payment in gateway** → redirected back to app
7. **Sees confirmation screen** — payment successful, receipt generated
8. **Downloads receipt** as PDF

**Time:** Under 3 minutes end-to-end.

---

## Journey 5: Accountant — Month-End Fee Reconciliation

**Goal:** Reconcile the month's fee collections with the payment gateway settlement report.

**Steps:**

1. **Downloads settlement CSV** from payment gateway dashboard
2. **Opens Platform → Finance → Reconciliation**
3. **Uploads settlement file** — system imports and begins auto-matching
4. **Reviews auto-matched transactions:** 247 matched, 3 mismatched, 1 unreconciled
5. **Investigates mismatches:** one gateway amount differs by ₹5 (gateway fee deducted)
6. **Manually resolves mismatches:** adds note explaining the gateway fee
7. **Reviews unreconciled transaction:** finds it's a double-entry from network retry
8. **Marks as resolved** — flags for refund review
9. **Generates reconciliation report** — shows 99.5% match rate
10. **Views defaulters dashboard:** 18 students with overdue invoices — filters by section
11. **Sends bulk fee reminders** to defaulter parents via Communication Hub

**Success:** Month-end reconciliation complete in under 2 hours (vs. full day previously).

---

## Journey 6: Teacher — Enter Marks for Assessments

**Goal:** Enter exam marks for all students in assigned subjects.

**Steps:**

1. **Opens Gradebook → Assessments**
2. **Selects "Term 1 Mathematics" assessment for Grade 8-A**
3. **Views component breakdown:** Theory (70%) and Practical (30%)
4. **Enters marks for Theory component:**
   - Sees student roster (34 students)
   - Enters raw marks (out of 70) for each student
   - 2 students marked "Absent" — system records without marks
5. **Enters marks for Practical component:** same process
6. **Reviews auto-calculated weighted grades** — system converts to final grade using school's grading scheme
7. **Submits marks for verification**
8. **Principal verifies** → Assessment moves to Verified status

---

## Journey 7: School Admin — Publish Report Cards

**Goal:** Generate and publish Term 1 report cards for all students.

**Steps:**

1. **Opens Gradebook → Result Publications**
2. **Creates new publication:** "Term 1 Results 2026"
3. **Selects scope:** All grades (school-wide)
4. **Runs validation:** system checks for missing marks — shows 3 missing entries
5. **Notifies subject teachers** to enter missing marks
6. **Re-runs validation:** all marks complete ✓
7. **Generates report cards** for all students — background job (takes 5–10 minutes for 500 students)
8. **Previews sample report card** — verifies layout, branding, grades
9. **Schedules publication** for Friday 3:00 PM
10. **Publication goes live** at scheduled time — parents receive notification
11. **Parents access report cards** in portal — download as PDF

---

## Journey 8: Principal — Morning Operations Review

**Goal:** Get a 5-minute overview of school operations at 9 AM.

**Steps:**

1. **Opens platform on web**
2. **Views Operations Dashboard:**
   - Today's attendance: 92% (so far) — 4 missing submissions
   - Outstanding fees: ₹2,34,500 overdue
   - Pending leave approvals: 2
   - Pending attendance corrections: 1
3. **Clicks on "4 missing attendance submissions"** → sees Grade 7-B and Grade 10-A not submitted
4. **Sends reminder** to respective teachers via Communication Hub
5. **Approves 2 leave requests** for staff
6. **Reviews attendance correction** for Grade 6-A → approves
7. **Looks at this week's fee collection trend** — on track vs last week

**Time:** Under 5 minutes.

---

## Journey 9: Parent — Using the Portal After School Communication

**Goal:** Parent reads school notice, checks child's timetable for tomorrow.

**Steps:**

1. **Opens portal app → sees notification badge**
2. **Opens Notices screen** — reads "Annual Sports Day Announcement"
3. **Marks notice as read**
4. **Navigates to Timetable → Tomorrow (Thursday)**
   - Period 1: Mathematics (08:30–09:15)
   - Period 2: Science Lab (09:30–10:15)
   - Period 4: English (11:00–11:45)
5. **Navigates to Assignments** — sees 1 pending assignment due tomorrow
6. **Contacts school via Support Request** — asks about an upcoming fee revision

---

## Journey 10: School Admin — Suspension of a Teacher Account

**Goal:** A teacher leaves mid-year; remove their access immediately.

**Steps:**

1. **Opens Users → Staff directory**
2. **Searches for the teacher's name**
3. **Views profile → Membership section**
4. **Clicks "Disable Membership"** — confirms action
5. **System immediately:**
   - Revokes all active sessions for this user in this tenant
   - Teacher's access tokens expire in 15 minutes (next refresh fails)
   - Teacher cannot submit attendance or enter marks
6. **Audit log records:** actor, action, timestamp, previous state
7. **System warns:** 3 pending teacher assignments remain — admin reassigns to substitute

---

## Edge-Case Journeys

### Student Transfers Mid-Year

1. Admin opens the student's profile → Enrollment tab
2. Creates a transfer: selects destination information
3. Status moves to Transferred; exit date recorded
4. Historical attendance, grades, and fees remain accessible in read-only mode
5. New enrollment can be created if the student joins a different section at the same school

### Parent Dispute on Attendance Record

1. Parent notices incorrect absence on portal
2. Parent submits Support Request: "My child was present on June 15"
3. School Admin reviews attendance record — confirms error
4. Requests correction (requires reason)
5. If attendance is locked: correction goes through approval workflow
6. Correction approved → attendance record updated → audit trail preserved
7. Parent sees corrected record on portal (no revision markers visible to parents)

