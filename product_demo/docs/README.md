# Platform Documentation

Welcome to the School SaaS Platform documentation. This directory contains the complete technical and product documentation, organized by phase and domain. 

## Single Source of Truth

The Product Requirements Documents (PRDs) located in the `PRDs/` folder are the single source of truth for all business logic, features, and requirements. All documentation in this folder is derived from and must remain aligned with the PRDs.

---

## Directory Structure

* **`00-vision/`** — High-level product strategy, thesis, and target customer personas.
* **`01-domain/`** — Core business domain rules, state machines, and vocabulary.
* **`02-architecture/`** — System context, module boundaries, and architectural decision records (ADRs).
* **`03-data/`** — Database schemas, entity relationship diagrams (ERDs), RLS policies, and seed data. Detailed DDLs are in `PRDs/data7_*.md`.
* **`04-api/`** — API contracts, authentication flows, and standardized error responses.
* **`05-ux/`** — Information architecture, user journeys, and wireframes.
* **`06-engineering/`** — Coding standards, test strategies, CI/CD pipelines, and AI coding rules.
* **`07-release/`** — QA checklists, incident runbooks, and pilot onboarding guides.

---

## Traceability Matrix (Docs to PRDs)

This matrix maps which PRD modules inform which technical documents:

| Technical Document | Primary PRD Sources | Description |
|--------------------|---------------------|-------------|
| `01-domain/business-rules.md` | PRD_M7_1 - PRD_M7_10 | Core business constraints |
| `01-domain/fee-rules.md` | PRD_M7_8 | Financial transaction rules |
| `01-domain/state-machines.md` | All PRDs | Lifecycle definitions |
| `02-architecture/multi-tenant-strategy.md` | PRD_M7_1 | Tenant isolation architecture |
| `02-architecture/module-boundaries.md` | All PRDs | Inter-module communication |
| `03-data/postgres-schema.md` | PRD_M7_1, M7_2, M7_4 | Core tables and relationships |
| `03-data/rls-policies.md` | PRD_M7_1, PRD_M7_3 | Access control policies |
| `04-api/auth-flows.md` | PRD_M7_1, PRD_M7_3, PRD_M7_9 | Authentication implementation |
| `05-ux/user-journeys.md` | All PRDs | Core workflows |
| `07-release/qa-checklists.md` | All PRDs | Pre-launch validation |

---

## Reading Guide

**For New Engineers:**
1. Start with `00-vision/product-thesis.md` and `01-domain/glossary.md`
2. Read `02-architecture/system-context.md` and `module-boundaries.md`
3. Read `06-engineering/coding-standards.md` and `06-engineering/rules.md`
4. Dive into the specific `PRDs/` for the module you are building

**For Product Managers:**
1. Focus on `PRDs/` as your primary workspace
2. Review `05-ux/user-journeys.md` and `07-release/pilot-onboarding.md`

**For QA and Ops:**
1. Read `07-release/qa-checklists.md` and `07-release/incident-runbook.md`

---

## Documentation Status

For an inventory of missing documentation and upcoming priorities, please refer to [documentation-status.md](./documentation-status.md).
