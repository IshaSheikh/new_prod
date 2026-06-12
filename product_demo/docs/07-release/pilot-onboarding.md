# Pilot Onboarding

Guide for onboarding the first 10 pilot schools onto the platform. This document covers the process from initial contact through go-live and 30-day success review.

---

## Pilot School Selection Criteria

Target schools for the pilot should meet these criteria:

| Criterion | Requirement |
|-----------|-------------|
| School size | 150–800 students |
| Technology comfort | Has used some digital tool (WhatsApp groups, Google Sheets, basic ERP) |
| Decision maker access | Direct contact with School Owner or Principal |
| Pain point match | Active pain around fee collection or parent communication |
| Location | Tier 1 or Tier 2 city (reliable internet) |
| Academic structure | Standard K-12 (not montessori or highly customized) |
| Language | English-medium or comfortable with English interface |

---

## Pilot Onboarding Phases

### Phase 1: Pre-Onboarding (Day -7 to Day 0)

**Goal:** Prepare the school and the platform before the onboarding call.

Tasks:
- [ ] Collect school data sheet (name, address, number of students, grades, sections, academic year dates)
- [ ] Create tenant in platform (Super Admin action)
- [ ] Reserve subdomain (e.g., `schoolname.schoolsaas.com`)
- [ ] Activate Trial subscription
- [ ] Send invitation email to School Admin
- [ ] Share pre-onboarding checklist (things school should prepare):
  - List of grades and sections
  - Fee structure (what they charge, how often)
  - School logo (PNG/JPG/SVG)
  - Staff list with email addresses
  - Sample student list (for first import)

---

### Phase 2: Onboarding Session (Day 0 — 2-3 hours)

**Goal:** School is fully configured and live by end of session.

**Attendees:** School Admin (required), Principal (recommended), Key Teacher

**Agenda:**

| Time | Activity |
|------|---------|
| 0:00–0:15 | Platform overview demo (3-minute walkthroughs) |
| 0:15–0:45 | Setup Wizard: School Profile, Academic Year, Terms |
| 0:45–1:15 | Setup Wizard: Grades, Sections, Campus |
| 1:15–1:30 | Upload branding, configure communication settings |
| 1:30–1:50 | Fee Categories and Fee Structures setup |
| 1:50–2:10 | Import first batch of students (10–20 sample students) |
| 2:10–2:30 | Invite teachers and set up parent portal access for sample parents |
| 2:30–2:45 | Run readiness check → Go Live |
| 2:45–3:00 | Q&A, next steps, schedule follow-up |

**Deliverables by end of session:**
- [ ] School is in "Live" status
- [ ] At least 1 teacher can log in
- [ ] At least 5 students enrolled with guardian links
- [ ] At least 1 invoice issued to a test student
- [ ] At least 1 parent can log into the portal and see their child's data

---

### Phase 3: First Week (Days 1–7)

**Goal:** Daily operations running smoothly.

**Day 1–2:**
- [ ] Teachers take attendance for their classes (target: all assigned classes)
- [ ] School Admin verifies absence notifications are reaching parents
- [ ] Accountant generates and issues invoices for all enrolled students

**Day 3–5:**
- [ ] School Admin imports all remaining students in bulk
- [ ] All guardians linked to students
- [ ] Parent portal invitations sent to all guardians
- [ ] First fee reminders sent via Communication Hub

**Day 6–7:**
- [ ] Review: attendance submission rate (target: >90% of sessions submitted on time)
- [ ] Review: portal adoption (target: >30% of parent invitations accepted)
- [ ] Address first-week issues and questions

---

### Phase 4: First Month Review (Day 30)

**Goal:** Validate that the platform is delivering value and identify expansion opportunities.

**Review Metrics:**

| Metric | Target | How to Check |
|--------|--------|--------------|
| Attendance submission rate | > 90% | Admin dashboard |
| Parent portal adoption | > 50% | Portal analytics |
| Invoice generation time reduction | Reported by accountant | Interview |
| Fee collection rate vs prior month | Neutral or improved | Finance reports |
| Parent communication quality | Positive feedback | Short survey |
| Support tickets from school | < 5 per week | Support queue |

**Review Meeting Agenda:**
1. Walk through the 30-day metrics dashboard
2. Ask about workflow changes (what's better, what's harder)
3. Identify any missing functionality for their school
4. Discuss expansion: add attendance correction workflow, timetable, or exam schedule
5. Discuss subscription conversion if still on trial

---

## Pilot Support Structure

### Dedicated Support Channel

Create a WhatsApp group for each pilot school with:
- School Admin
- Principal (optional)
- Platform support contact

This is temporary for pilots — production support uses the built-in support ticket system.

### Response SLAs for Pilots

| Issue Severity | Target Response | Target Resolution |
|----------------|----------------|-------------------|
| Critical (cannot access platform) | 30 minutes | 2 hours |
| High (core feature broken) | 2 hours | 8 hours |
| Medium (feature issue, workaround exists) | 4 hours | 24 hours |
| Low (question or UI confusion) | 8 hours | 48 hours |

---

## Common First-Week Issues and Resolutions

| Issue | Likely Cause | Resolution |
|-------|-------------|------------|
| Teacher can't mark attendance | Not assigned to any section | School Admin → Teacher Assignments |
| Parent can't see attendance | Guardian not linked or portal access not enabled | School Admin → Guardians → Enable Portal Access |
| Duplicate students created | Bulk import run twice | Merge or soft-delete duplicates |
| Fee structure not applied to all students | Fee assignment missing | Bulk assign fee structure in student list |
| Invitation email not received | Spam folder or incorrect email | Resend invitation, check email |
| Academic year not active | Year in Draft or Scheduled status | School Admin → Activate Academic Year |
| Sections not visible to teachers | Sections not created for current academic year | Verify academic year on sections |

---

## Data Migration Support

For schools migrating from spreadsheets:

**Student Import Template:**
Provide CSV template with columns:
`student_number, first_name, last_name, date_of_birth, gender, grade_code, section_name, guardian_name, guardian_mobile, guardian_email, guardian_relationship`

**Validation before import:**
- Check for duplicate student numbers
- Validate grade codes match configured grades
- Validate section names match configured sections
- Flag missing mandatory fields

**Post-import checklist:**
- [ ] Verify student count matches school's records
- [ ] Verify grade/section distribution is correct
- [ ] Spot-check 10 random student profiles
- [ ] Verify guardian links on imported students
- [ ] Run duplicate detection and review flagged records

---

## Pilot Feedback Collection

After 30 days, collect structured feedback via a short form:

1. On a scale of 1–10, how much has the platform reduced your administrative workload?
2. Which module do you find most valuable? (Attendance / Fees / Parent Portal / Communication)
3. What is the one thing you wish the platform did that it doesn't currently do?
4. How likely are you to recommend this platform to another school principal? (NPS)
5. What would prevent you from converting to a paid subscription?

Use this feedback to prioritize the product roadmap and identify high-value missing features before the general launch.
