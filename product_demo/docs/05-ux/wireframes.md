<!-- Source: PRDs | Last Verified: 2026-06-28 | Owner: Engineering -->
# Wireframes

Screen inventory and wireframe descriptions for the platform. Wireframes are organized by module and user role.

---

## Navigation Structure

### Web Application (Next.js)

**Top-level navigation (School Admin / Principal):**
```
Dashboard | Students | Academics | Attendance | Fees | Reports | Settings
```

**Sidebar for Settings:**
```
School Profile | Academic Years | Grades & Sections | Campuses | Branding |
Users | Roles | Subscription
```

**Mobile App (Flutter):**
```
Bottom tab bar:
Home | Attendance | [Module-specific] | Notifications | Profile
```

---

## Screen Inventory by Module

---

### M7.1 / M7.2 — Onboarding and Setup

**SCR-001: Onboarding Wizard**
- Step-by-step wizard with progress bar (10 steps)
- Each step shows a form with relevant fields
- "Save and Continue" progresses to next step
- "Save Draft" allows partial completion
- Sidebar shows checklist with completion indicators (✓ / ● / ○)

**SCR-002: Readiness Dashboard**
- Completion percentage ring chart
- List of blocking requirements (in red)
- List of completed steps (in green)
- "Go Live" button (enabled when all blocking items cleared)

**SCR-003: School Profile Screen**
- School name, legal name, contact email, phone, website
- Address fields (address line 1, city, state, postal code, country)
- Timezone selector
- Save button

**SCR-004: Academic Year Setup**
- Create form: name, start date, end date
- Calendar preview of dates
- List of existing academic years with status badges
- "Set as Active" action

**SCR-005: Grade Structure Screen**
- Drag-and-drop grade ordering
- Add grade button (name + code fields)
- Reorder indicators
- Section counts per grade shown as chips

**SCR-006: Section Management Screen**
- Table: Grade | Section Name | Capacity | Student Count | Actions
- "Add Section" button opens inline form
- Filter by grade
- Edit capacity in-place

---

### M7.3 — Users and RBAC

**SCR-007: User Directory**
- Search bar + filters (Role, Status)
- Table: Name | Email | Role | Status | Last Login | Actions
- Bulk actions: Disable, Send Reminder
- "Invite User" CTA button (opens modal)

**SCR-008: Invite User Modal**
- Email input
- Role multi-select (checkboxes)
- "Send Invitation" button
- Shows "Invitation sent to [email]" confirmation

**SCR-009: User Profile Screen**
Tabs:
1. Profile (name, email, phone, avatar)
2. Memberships (tenant list with roles)
3. Roles (current role assignment with edit)
4. Activity (last login, recent actions)

**SCR-010: Role Management Screen**
- List of system roles with permission counts
- Expand role to see permission matrix
- Permission checkboxes by module

**SCR-011: Parent-Student Linking Screen**
- Search parent by name or email
- Search student by name or student number
- Relationship type dropdown
- "Mark as Primary" checkbox
- Active links table with remove action

---

### M7.4 — Student Information System

**SCR-012: Student List Screen**
- Search bar + filters (Grade, Section, Status, Campus)
- Table: Student # | Name | Grade | Section | Status | Actions
- Bulk actions: Export, Assign Fee Structure
- "Add Student" CTA

**SCR-013: Student Profile Screen**
Tabs:
1. Overview (photo, basic details, enrollment status)
2. Profile (demographics, address, custom fields)
3. Enrollment (history table with timeline)
4. Guardians (linked guardians with portal access indicators)
5. Documents (upload + version history)
6. Health (alerts, medical notes — restricted by role)
7. Attendance (attendance history summary)
8. Fees (invoice list + payment history)
9. Audit History (status change timeline)

**SCR-014: Enrollment Management Screen**
- Current enrollment card
- "Transfer" and "Withdraw" actions
- Historical enrollment timeline

**SCR-015: Guardian Management Screen**
- Guardian cards: name, relationship, phone, email
- Primary guardian badge
- "Link Guardian" (search existing or create new)
- Portal access toggle per guardian

---

### M7.5 — Timetable

**SCR-016: Timetable Builder**
- Weekly grid (rows = periods, columns = Mon–Sat)
- Each cell shows subject + teacher name + room
- Empty cells are click-to-assign
- Teacher and room conflict indicators (orange/red badges)
- "Validate and Publish" button

**SCR-017: Subject Management Screen**
- Table: Subject Code | Subject Name | Group | Status | Actions
- Add Subject form (modal)
- Subject group assignment

**SCR-018: Room Management Screen**
- Table: Room Code | Name | Campus | Capacity | Type | Status
- Utilization percentage column
- Add Room button

**SCR-019: Timetable Validation Screen**
- Conflict list (teacher conflicts, room conflicts, section conflicts)
- Each conflict shows: day, period, teacher/room name, conflicting sections
- "Fix" quick action for each conflict
- "All Clear — Publish" button when no blocking conflicts

**SCR-020: Student/Teacher Timetable View**
- Weekly calendar view (Mon–Sat)
- Color-coded by subject
- Period times on Y-axis
- "Today" indicator
- Export/print action

---

### M7.6 — Attendance

**SCR-021: Teacher Attendance Screen**
- Date and session info at top
- Student roster (photo + name)
- Status chips: Present / Absent / Late / Excused / Half Day
- "Mark All Present" toggle
- Remarks field (expandable per student)
- Submit button
- Student count summary at bottom

**SCR-022: Admin Attendance Dashboard**
- Today's date and summary (e.g., "87% — 3 classes not submitted")
- Grid of classes × sections showing submission status
- Missing submissions highlighted
- "Send Reminder" action
- Date picker for historical review

**SCR-023: Student Attendance Profile**
- Attendance percentage gauge
- Monthly calendar heatmap (green = present, red = absent, amber = late)
- Trend chart (3 months)
- Status breakdown pie chart

**SCR-024: Attendance Correction Screen**
- List of pending corrections
- Each item: Student name | Date | Old Status | New Status | Reason | Requested By
- "Approve" / "Reject" actions with confirmation

---

### M7.7 — Gradebook

**SCR-025: Assessment Management Screen**
- Table: Assessment Name | Subject | Grade | Section | Status | Deadline
- Completion percentage column
- "Create Assessment" button
- Filter by academic year, grade, subject

**SCR-026: Marks Entry Screen**
- Student roster table
- Columns per assessment component (Theory, Practical, etc.)
- In-line editing with tab navigation
- Auto-calculated weighted marks column
- Final grade column (auto from grading scheme)
- "Mark Absent" action per student
- Save draft / Submit buttons
- Progress: X of Y students entered

**SCR-027: Report Card Management Screen**
- Publication status overview
- Grade-wise readiness table (missing marks counts)
- "Generate Report Cards" button
- Preview sample report card
- Schedule / Publish actions

**SCR-028: Parent Report Card View**
- Report card header: school branding, student photo, academic year
- Subject-wise marks table with grades
- Term-wise comparison (if available)
- Principal and teacher remarks
- Download PDF button

---

### M7.8 — Fees

**SCR-029: Fee Structure Setup**
- Structure name, academic year, grade/section, billing frequency
- Line items table: Category | Description | Amount | Recurrence
- "Add Line Item" button
- Total preview
- Activate/Archive controls

**SCR-030: Invoice List Screen**
- Filters: Academic Year, Grade, Section, State, Due Date range
- Table: Invoice # | Student | Period | Due Date | Amount | Paid | Balance | State
- Bulk actions: Issue, Send Reminder, Export
- Summary totals at bottom

**SCR-031: Student Fee Ledger**
- Chronological list of invoices, payments, credits, adjustments
- Current balance display (prominent)
- Aging bucket (current / 1-30 / 31-60 / 61-90 / 90+)
- "Record Payment" and "Generate Invoice" quick actions

**SCR-032: Payment Collection Screen**
- Student search (name or student number)
- Shows outstanding invoices for selected student
- Amount field (auto-fills with balance due)
- Payment method selector
- Reference number field
- "Record Payment + Generate Receipt" button

**SCR-033: Defaulters Dashboard**
- Class × Section heatmap (darker = more overdue)
- Table: Student | Grade | Section | Overdue Amount | Days Overdue | Last Payment
- Bulk "Send Reminder" action
- Export to CSV

**SCR-034: Reconciliation Screen**
- Import area (drag CSV/XLSX)
- Match status overview: Matched | Mismatched | Unreconciled (counts)
- Table of reconciliation items with match status
- Mismatched items highlighted with expand for details
- "Manually Resolve" action per item

---

### M7.9 — Parent Portal

**SCR-035: Parent Dashboard**
- Child switcher (if multiple children)
- Child card: photo, name, grade, section
- Today's attendance status (large, colored)
- Overdue fee alert (if any)
- Latest notice snippet
- Quick links: Attendance | Fees | Timetable | Assignments

**SCR-036: Fee Balance Screen**
- Total outstanding amount (large display)
- Invoice list with due dates and amounts
- "Pay" button per invoice
- Payment history tab

**SCR-037: Parent Attendance Timeline**
- Month-view calendar with color codes
- Attendance percentage for current month
- Last 5 entries list (date, status, class)

**SCR-038: Parent Notices Screen**
- Sorted by date (most recent first)
- Unread count badge
- Tap to expand full notice
- Attachment download button if present

---

### M7.10 — Communication Hub

**SCR-039: Notice Composer**
- Title field
- Body rich text editor
- Audience builder: school-wide / grade / section / role / individual
- Channel selection checkboxes: In-App | Email | SMS | WhatsApp | Push
- Priority dropdown
- Attachment upload
- Schedule toggle + date-time picker
- Preview button
- Send / Schedule button

**SCR-040: Delivery Tracking Screen**
- Notice summary at top
- Delivery status tabs: All | Delivered | Failed | Read
- Table: Recipient | Channel | Status | Timestamp | Failure Reason
- Retry failed action
- Export list

**SCR-041: Template Management Screen**
- Templates grid with category badges
- Preview template with mock data
- Create/Edit form with placeholder hint panel
- Activate/Archive toggle

---

### Super Admin (Platform Dashboard)

**SCR-042: Platform Dashboard**
- Summary cards: Total Schools | Active | Suspended | Revenue (MRR)
- Subscription health chart
- Recently created tenants list
- Platform-wide audit log feed

**SCR-043: Tenant Management Screen**
- Table: School Name | Code | Status | Subscription | Admins | Created
- Quick actions: Suspend | Activate | View Details
- "Create Tenant" button

**SCR-044: Tenant Detail Screen**
- School info
- Onboarding progress (completion %)
- Subscription details
- Domain configuration
- Staff members list
- Recent audit events

