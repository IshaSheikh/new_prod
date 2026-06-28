<!-- Source: PRDs | Last Verified: 2026-06-28 | Owner: Engineering -->
# Documentation Status

This document tracks the current state of the platform's documentation, identifying gaps and upcoming priorities.

## Priority Backlog

1. **[P0] M7.17 (Analytics) PRD:** Currently missing entirely. A placeholder PRD has been created, but it needs product requirements.
2. **[P1] Architecture Validation:** Verify that module boundaries accurately reflect current implementation, especially around cross-module data dependencies.
3. **[P2] UX Documentation:** Wireframes and user journeys need to be expanded for the newly added M7.5 (Timetable) module.
4. **[P3] API Documentation:** Comprehensive OpenAPI spec generation setup is required.

## Module Documentation Completeness Matrix

| Module | PRD | DB Schema | APIs | State Machines | Test Strategy | Status |
|--------|-----|-----------|------|----------------|---------------|--------|
| M7.1 Platform Core | ✅ | ✅ | ✅ | ✅ | ✅ | Complete |
| M7.2 Onboarding | ✅ | ✅ | ✅ | ✅ | ✅ | Complete |
| M7.3 RBAC | ✅ | ✅ | ✅ | ✅ | ✅ | Complete |
| M7.4 SIS | ✅ | ✅ | ✅ | ✅ | ✅ | Complete |
| M7.5 Timetable | ✅ | ✅ | ✅ | ❌ | ❌ | In Progress |
| M7.6 Attendance | ✅ | ✅ | ✅ | ✅ | ✅ | Complete |
| M7.7 Gradebook | ✅ | ✅ | ✅ | ✅ | ❌ | In Progress |
| M7.8 Fees | ✅ | ✅ | ✅ | ✅ | ✅ | Complete |
| M7.9 Portal | ✅ | ✅ | ✅ | ❌ | ❌ | In Progress |
| M7.10 Communication | ✅ | ✅ | ✅ | ❌ | ❌ | In Progress |
| M7.11 Admissions | ✅ | ✅ | ❌ | ❌ | ❌ | Planned |
| M7.12 Extracurricular | ✅ | ❌ | ❌ | ❌ | ❌ | Planned |
| M7.13 Behavior | ✅ | ❌ | ❌ | ❌ | ❌ | Planned |
| M7.14 Transport | ✅ | ✅ | ❌ | ❌ | ❌ | Planned |
| M7.15 Library | ✅ | ✅ | ❌ | ❌ | ❌ | Planned |
| M7.16 HR & Payroll | ✅ | ✅ | ❌ | ❌ | ❌ | Planned |
| M7.17 Analytics | ❌ | ✅ | ❌ | ❌ | ❌ | Missing PRD |

*Note: DB Schema indicates presence of DDL in `PRDs/data7_*.md`. APIs indicate documentation in `04-api`.*
