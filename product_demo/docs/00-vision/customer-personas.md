<!-- Source: PRDs | Last Verified: 2026-06-28 | Owner: Engineering -->
# Customer Personas

## Platform Level

---

### Super Admin (Platform Operator)

**Who they are:** The team managing the SaaS platform itself — typically the founding team or a small platform operations team.

**Goals:**
- Onboard new schools quickly and reliably
- Monitor subscription health across all tenants
- Resolve support incidents without accessing school data inappropriately
- View platform-level analytics (school counts, revenue, subscription health)

**Pain points:**
- Manual tenant setup takes too long
- No visibility into which schools are stuck in onboarding
- Difficult to investigate cross-tenant issues without the right tools

**Key actions:**
- Create and suspend tenants
- Assign subdomains and custom domains
- Manage subscription plans and activation
- View platform-wide audit logs
- Force-complete onboarding for stuck schools

---

## School Level

---

### School Admin

**Who they are:** The person responsible for running the digital side of the school. Usually the principal's assistant, office manager, or a dedicated IT/admin coordinator. Typically 1–3 per school.

**Goals:**
- Get the school configured quickly and correctly
- Keep student records accurate and up to date
- Ensure fees are collected on time
- Reduce repetitive parent queries (attendance, receipts, report cards)

**Pain points:**
- Setting up academic years, grades, sections, and fee structures is time-consuming
- Manually generating invoices and sending reminders is error-prone
- Staff members accidentally modify records they shouldn't
- Cannot easily generate compliance or reporting documents

**Key actions:**
- Complete school onboarding wizard
- Invite teachers, accountants, and other staff
- Configure fee structures and generate invoices
- Manage student enrollments and transfers
- Run attendance correction and approval workflows
- Publish report cards and exam results

---

### Principal

**Who they are:** The academic leader of the school. Focuses on academic quality, staff performance, and parent relationships. Less hands-on with day-to-day data entry.

**Goals:**
- Visibility into school operations without needing to operate the system
- Monitor attendance trends, academic performance, and fee collection health
- Ensure report cards are correct before publication
- Approve timetables and exam schedules

**Pain points:**
- Currently depends on manual reports from office staff
- Cannot easily identify attendance problems or underperforming classes
- No audit trail for key decisions

**Key actions:**
- Review timetables and approve before publication
- Monitor attendance dashboards
- Approve academic results before publication
- Review admissions pipeline
- Oversee leave approvals for staff

---

### Teacher

**Who they are:** The most frequent daily user of the platform. Primary responsibility is attendance, marks entry, and assignment management. Often mobile-first.

**Goals:**
- Mark attendance quickly without disrupting teaching time
- Enter grades efficiently (especially during exam season)
- Assign and track homework without manual follow-up
- Access student emergency contacts when needed

**Pain points:**
- Attendance marking on paper or spreadsheets is slow
- No easy way to communicate assignments to parents
- No visibility into which students have outstanding fee issues (for context)

**Key actions:**
- Take daily or period-wise attendance
- Enter marks for assigned subjects
- Create and publish assignments
- View assigned class timetable

---

### Accountant

**Who they are:** Manages all financial operations for the school. May be a dedicated finance person at larger schools or the school admin wearing two hats at smaller schools.

**Goals:**
- Generate and issue invoices at scale
- Record payments (cash, cheque, online) accurately
- Reconcile collections with bank statements
- Identify defaulters and send reminders

**Pain points:**
- Manual invoice generation for 500+ students is error-prone
- No way to see live outstanding balance by student, class, or section
- Receipt book management is a compliance risk
- Reconciliation is a manual end-of-month nightmare

**Key actions:**
- Configure fee categories and structures
- Generate and issue invoices in bulk
- Record offline and online payments
- Issue immutable receipts
- Process refunds with approval
- Run reconciliation against bank/gateway settlement

---

### Parent

**Who they are:** A guardian (mother, father, or other legal guardian) of one or more students enrolled in the school. May have children in the same or different schools.

**Goals:**
- Know if their child attended school today
- See upcoming fee due dates and pay online
- Access report cards without visiting the office
- Receive reliable notifications, not just WhatsApp forwards

**Pain points:**
- Currently calls the school for every piece of information
- Gets attendance information through informal channels
- Receipts get lost; no digital record
- Can't see multiple children's status in one view

**Key actions:**
- View attendance history for linked children
- Pay fees online
- Download receipts
- View published report cards
- Receive and read school notices

---

### Student

**Who they are:** The learner. Self-service access is typically for secondary school students (Grade 6 and above).

**Goals:**
- View own timetable and assignments
- Check attendance record and grades
- Download report cards

**Pain points:**
- Currently dependent on teachers or parents for this information
- No digital submission option for homework

**Key actions:**
- View timetable
- View published grades and report cards
- Submit assignments
- View attendance

---

### Transport Manager

**Who they are:** Manages school buses, routes, and student transport assignments. May be a dedicated role or handled by a school admin.

**Goals:**
- Assign students to correct routes and stops
- Track vehicle and route capacity
- Ensure transport charges are linked to active assignments

**Key actions:**
- Create vehicles, routes, and stops
- Assign students to routes
- Monitor capacity and route utilization

---

### Librarian

**Who they are:** Manages the school library. May be a dedicated staff role or a teacher with dual responsibility.

**Goals:**
- Track book issues and returns reliably
- Identify overdue books quickly
- Manage fines

**Key actions:**
- Issue books to students
- Record returns
- Apply and waive fines
- View circulation reports

---

## Buyer Personas (Decision Makers)

### School Owner / Trustee

**Who they are:** The owner or governing board of the school. Not a daily user but makes the purchasing decision.

**Goals:**
- Reduce administrative workload and staffing costs
- Improve parent satisfaction and school reputation
- Get financial visibility without relying on the accountant

**Pain points:**
- Cannot see real-time financial health of the school
- High dependency on individual staff members for operational knowledge
- Parents complain about lack of transparency

**What they buy on:**
- Parent portal quality (visible value to parents = competitive differentiator)
- Fee collection efficiency (direct ROI)
- Ease of setup (self-service = low implementation risk)
- Price fit for Indian K-12 market

