<img src="https://r2cdn.perplexity.ai/pplx-full-logo-primary-dark%402x.png" style="height:64px;margin-right:32px"/>

# proceed with M7_11

For M7_11, I would model admissions as a gated workflow of **inquiry -> applicant -> form and document completion -> evaluation -> decision and offer -> SIS handoff**, because the PRD defines that exact pre-enrollment lifecycle and explicitly says final admitted records must flow into official student and enrollment workflows rather than creating disconnected student records.[^1]
The schema below is production-ready for a multi-tenant K-12 SaaS and is optimized for grade-specific requirements, duplicate detection, offer expiry control, auditability, and enrollment handoff readiness.[^1]

## 1. Entities

These are the core entities I recommend for M7_11.[^1]

- `admission_inquiries` — pre-applicant lead capture with inquiry source and conversion tracking, because an inquiry may exist before a formal applicant record and must remain searchable even if closed or invalid.[^1]
- `applicants` — the canonical pre-enrollment candidate record with target academic year, target grade, duplicate-risk metadata, and lifecycle status.[^1]
- `application_form_versions` — grade-specific form definitions with version retention, because the form version used at submission time must be preserved for auditability.[^1]
- `application_form_submissions` — the applicant’s submitted form data and workflow state such as draft, submitted, returned, resubmitted, and verified.[^1]
- `admission_document_requirements` — grade-specific document checklist definitions, because required documents vary by grade.[^1]
- `applicant_documents` — immutable versioned applicant document uploads with verification outcome and rejection reason.[^1]
- `applicant_duplicate_matches` — duplicate-review records based on phone, email, and date-of-birth similarity, because duplicate detection must run before applicant creation or progression.[^1]
- `applicant_evaluations` and `applicant_evaluation_scores` — interviews, assessments, observations, and manual reviews with auditable outcomes.[^1]
- `admission_decisions` — waitlist, not-selected, recommended-for-offer, and admitted decisions, because evaluation outcomes must be auditable and structured.[^1]
- `admission_offers` and `admission_offer_conditions` — time-bound offers, expiry control, acceptance tracking, and required acceptance conditions.[^1]
- `enrollment_handoffs` — validated handoff into SIS and enrollment workflows, because admitted applicants must convert through official downstream rules.[^1]
- `applicant_status_history` and `admissions_audit_logs` — immutable transition and action tracking, because every workflow transition, offer event, and decision action must be auditable.[^1]


## 2. Table definitions

I am using explicit tables instead of generic polymorphic workflow blobs, because this module has distinct business rules for forms, documents, evaluations, offers, and handoffs that need clean constraints and audit trails.[^1]
I am also using status-driven lifecycle control rather than broad soft deletes, because the PRD says progressed applicants must remain searchable and applicant deletion after workflow progression is prohibited.[^1]

```sql
CREATE EXTENSION IF NOT EXISTS pgcrypto;
CREATE EXTENSION IF NOT EXISTS citext;

CREATE TYPE inquiry_status AS ENUM (
  'new', 'contacted', 'qualified', 'invalid', 'converted', 'closed'
);

CREATE TYPE applicant_status AS ENUM (
  'inquiry',
  'applicant',
  'documents_pending',
  'evaluation_ready',
  'under_evaluation',
  'waitlisted',
  'offer_issued',
  'offer_accepted',
  'offer_rejected',
  'expired',
  'admitted',
  'enrolled',
  'withdrawn',
  'closed'
);

CREATE TYPE duplicate_review_status AS ENUM (
  'not_checked', 'flagged', 'pending_review', 'cleared', 'confirmed_duplicate', 'merged'
);

CREATE TYPE form_submission_status AS ENUM (
  'draft', 'submitted', 'returned_for_correction', 'resubmitted', 'verified'
);

CREATE TYPE document_status AS ENUM (
  'uploaded', 'under_review', 'verified', 'rejected', 'reuploaded', 'waived', 'archived'
);

CREATE TYPE evaluation_type AS ENUM (
  'interview', 'assessment', 'observation', 'manual_review'
);

CREATE TYPE evaluation_status AS ENUM (
  'pending', 'scheduled', 'completed', 'waived', 'cancelled'
);

CREATE TYPE decision_status AS ENUM (
  'recommended_for_offer', 'waitlisted', 'not_selected', 'admitted', 'withdrawn'
);

CREATE TYPE offer_status AS ENUM (
  'draft', 'issued', 'accepted', 'rejected', 'expired', 'closed', 'cancelled'
);

CREATE TYPE handoff_status AS ENUM (
  'pending', 'validated', 'completed', 'failed', 'cancelled'
);

-- Assumes upstream master tables exist:
-- tenants, schools, users, academic_years, grade_levels, sections, students, student_enrollments.

CREATE TABLE admission_inquiries (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID NOT NULL REFERENCES tenants(id),
  school_id UUID NOT NULL REFERENCES schools(id),
  academic_year_id UUID NOT NULL REFERENCES academic_years(id),
  target_grade_level_id UUID NOT NULL REFERENCES grade_levels(id),
  student_first_name VARCHAR(100) NOT NULL,
  student_last_name VARCHAR(100) NOT NULL,
  student_date_of_birth DATE,
  parent_name VARCHAR(150) NOT NULL,
  email CITEXT,
  phone VARCHAR(20),
  inquiry_source VARCHAR(100),
  notes TEXT,
  status inquiry_status NOT NULL DEFAULT 'new',
  converted_to_applicant_id UUID,
  deleted_at TIMESTAMPTZ,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  created_by UUID REFERENCES users(id),
  updated_by UUID REFERENCES users(id)
);

CREATE TABLE applicants (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID NOT NULL REFERENCES tenants(id),
  school_id UUID NOT NULL REFERENCES schools(id),
  inquiry_id UUID UNIQUE REFERENCES admission_inquiries(id),
  academic_year_id UUID NOT NULL REFERENCES academic_years(id),
  target_grade_level_id UUID NOT NULL REFERENCES grade_levels(id),
  first_name VARCHAR(100) NOT NULL,
  last_name VARCHAR(100) NOT NULL,
  date_of_birth DATE,
  gender VARCHAR(30),
  parent_name VARCHAR(150) NOT NULL,
  email CITEXT,
  phone VARCHAR(20),
  duplicate_risk_score NUMERIC(5,2),
  duplicate_review_status duplicate_review_status NOT NULL DEFAULT 'not_checked',
  status applicant_status NOT NULL DEFAULT 'applicant',
  current_offer_id UUID,
  admitted_student_id UUID REFERENCES students(id),
  enrolled_student_enrollment_id UUID REFERENCES student_enrollments(id),
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  created_by UUID REFERENCES users(id),
  updated_by UUID REFERENCES users(id)
);

CREATE TABLE application_form_versions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID NOT NULL REFERENCES tenants(id),
  school_id UUID NOT NULL REFERENCES schools(id),
  academic_year_id UUID NOT NULL REFERENCES academic_years(id),
  grade_level_id UUID NOT NULL REFERENCES grade_levels(id),
  version_no INTEGER NOT NULL,
  form_schema JSONB NOT NULL,
  mandatory_fields JSONB NOT NULL DEFAULT '[]'::jsonb,
  status VARCHAR(20) NOT NULL CHECK (status IN ('draft','active','archived')),
  effective_from DATE NOT NULL,
  effective_to DATE,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  created_by UUID REFERENCES users(id),
  updated_by UUID REFERENCES users(id),
  CONSTRAINT uq_form_version UNIQUE (tenant_id, school_id, academic_year_id, grade_level_id, version_no),
  CONSTRAINT ck_form_dates CHECK (effective_to IS NULL OR effective_to >= effective_from)
);

CREATE TABLE application_form_submissions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID NOT NULL REFERENCES tenants(id),
  school_id UUID NOT NULL REFERENCES schools(id),
  applicant_id UUID NOT NULL REFERENCES applicants(id) ON DELETE RESTRICT,
  form_version_id UUID NOT NULL REFERENCES application_form_versions(id),
  form_data JSONB NOT NULL,
  status form_submission_status NOT NULL DEFAULT 'draft',
  submitted_at TIMESTAMPTZ,
  verified_at TIMESTAMPTZ,
  reviewed_by UUID REFERENCES users(id),
  review_notes TEXT,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  created_by UUID REFERENCES users(id),
  updated_by UUID REFERENCES users(id),
  CONSTRAINT uq_applicant_form_submission UNIQUE (tenant_id, applicant_id, form_version_id)
);

CREATE TABLE admission_document_requirements (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID NOT NULL REFERENCES tenants(id),
  school_id UUID NOT NULL REFERENCES schools(id),
  academic_year_id UUID NOT NULL REFERENCES academic_years(id),
  grade_level_id UUID NOT NULL REFERENCES grade_levels(id),
  document_type VARCHAR(80) NOT NULL,
  is_mandatory BOOLEAN NOT NULL DEFAULT TRUE,
  allows_waiver BOOLEAN NOT NULL DEFAULT FALSE,
  allowed_mime_types TEXT[] NOT NULL DEFAULT '{}',
  max_file_size_bytes BIGINT,
  status VARCHAR(20) NOT NULL CHECK (status IN ('active','archived')),
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  created_by UUID REFERENCES users(id),
  updated_by UUID REFERENCES users(id),
  CONSTRAINT uq_document_requirement UNIQUE (tenant_id, school_id, academic_year_id, grade_level_id, document_type)
);

CREATE TABLE applicant_documents (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID NOT NULL REFERENCES tenants(id),
  school_id UUID NOT NULL REFERENCES schools(id),
  applicant_id UUID NOT NULL REFERENCES applicants(id) ON DELETE RESTRICT,
  requirement_id UUID REFERENCES admission_document_requirements(id),
  document_type VARCHAR(80) NOT NULL,
  storage_key TEXT NOT NULL,
  original_file_name VARCHAR(255) NOT NULL,
  mime_type VARCHAR(120) NOT NULL,
  file_size_bytes BIGINT NOT NULL,
  version_no INTEGER NOT NULL,
  status document_status NOT NULL DEFAULT 'uploaded',
  review_reason TEXT,
  uploaded_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  reviewed_at TIMESTAMPTZ,
  reviewed_by UUID REFERENCES users(id),
  waived_by UUID REFERENCES users(id),
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  created_by UUID REFERENCES users(id),
  updated_by UUID REFERENCES users(id),
  CONSTRAINT uq_applicant_document_version UNIQUE (tenant_id, applicant_id, document_type, version_no),
  CONSTRAINT ck_document_file_size CHECK (file_size_bytes > 0)
);

CREATE TABLE applicant_duplicate_matches (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID NOT NULL REFERENCES tenants(id),
  applicant_id UUID NOT NULL REFERENCES applicants(id) ON DELETE CASCADE,
  matched_applicant_id UUID REFERENCES applicants(id),
  matched_inquiry_id UUID REFERENCES admission_inquiries(id),
  match_basis TEXT[] NOT NULL DEFAULT '{}',
  risk_score NUMERIC(5,2) NOT NULL,
  review_status duplicate_review_status NOT NULL DEFAULT 'flagged',
  reviewed_by UUID REFERENCES users(id),
  reviewed_at TIMESTAMPTZ,
  review_notes TEXT,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  created_by UUID REFERENCES users(id),
  CONSTRAINT ck_duplicate_target CHECK (
    (matched_applicant_id IS NOT NULL AND matched_inquiry_id IS NULL) OR
    (matched_applicant_id IS NULL AND matched_inquiry_id IS NOT NULL)
  )
);

CREATE TABLE applicant_evaluations (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID NOT NULL REFERENCES tenants(id),
  school_id UUID NOT NULL REFERENCES schools(id),
  applicant_id UUID NOT NULL REFERENCES applicants(id) ON DELETE RESTRICT,
  evaluation_type evaluation_type NOT NULL,
  evaluator_user_id UUID REFERENCES users(id),
  scheduled_at TIMESTAMPTZ,
  completed_at TIMESTAMPTZ,
  status evaluation_status NOT NULL DEFAULT 'pending',
  total_score NUMERIC(8,2),
  recommendation decision_status,
  remarks TEXT,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  created_by UUID REFERENCES users(id),
  updated_by UUID REFERENCES users(id)
);

CREATE TABLE applicant_evaluation_scores (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID NOT NULL REFERENCES tenants(id),
  evaluation_id UUID NOT NULL REFERENCES applicant_evaluations(id) ON DELETE CASCADE,
  criterion_code VARCHAR(80) NOT NULL,
  criterion_label VARCHAR(120) NOT NULL,
  max_score NUMERIC(8,2),
  awarded_score NUMERIC(8,2),
  remarks TEXT,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  created_by UUID REFERENCES users(id),
  updated_by UUID REFERENCES users(id),
  CONSTRAINT uq_evaluation_criterion UNIQUE (tenant_id, evaluation_id, criterion_code)
);

CREATE TABLE admission_decisions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID NOT NULL REFERENCES tenants(id),
  school_id UUID NOT NULL REFERENCES schools(id),
  applicant_id UUID NOT NULL REFERENCES applicants(id) ON DELETE RESTRICT,
  decision_status decision_status NOT NULL,
  decision_reason TEXT,
  decided_by UUID NOT NULL REFERENCES users(id),
  decided_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE TABLE admission_offers (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID NOT NULL REFERENCES tenants(id),
  school_id UUID NOT NULL REFERENCES schools(id),
  applicant_id UUID NOT NULL REFERENCES applicants(id) ON DELETE RESTRICT,
  offer_no VARCHAR(50) NOT NULL,
  status offer_status NOT NULL DEFAULT 'draft',
  offer_text TEXT,
  issued_at TIMESTAMPTZ,
  expires_at TIMESTAMPTZ,
  accepted_at TIMESTAMPTZ,
  rejected_at TIMESTAMPTZ,
  closed_at TIMESTAMPTZ,
  issued_by UUID REFERENCES users(id),
  response_notes TEXT,
  offer_snapshot_json JSONB NOT NULL DEFAULT '{}'::jsonb,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  created_by UUID REFERENCES users(id),
  updated_by UUID REFERENCES users(id),
  CONSTRAINT uq_offer_no UNIQUE (tenant_id, school_id, offer_no),
  CONSTRAINT ck_offer_dates CHECK (
    (expires_at IS NULL OR issued_at IS NULL OR expires_at > issued_at)
  )
);

CREATE TABLE admission_offer_conditions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID NOT NULL REFERENCES tenants(id),
  offer_id UUID NOT NULL REFERENCES admission_offers(id) ON DELETE CASCADE,
  condition_code VARCHAR(80) NOT NULL,
  condition_label VARCHAR(150) NOT NULL,
  is_mandatory BOOLEAN NOT NULL DEFAULT TRUE,
  satisfied_at TIMESTAMPTZ,
  satisfied_by UUID REFERENCES users(id),
  notes TEXT,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  created_by UUID REFERENCES users(id),
  updated_by UUID REFERENCES users(id),
  CONSTRAINT uq_offer_condition UNIQUE (tenant_id, offer_id, condition_code)
);

CREATE TABLE enrollment_handoffs (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID NOT NULL REFERENCES tenants(id),
  school_id UUID NOT NULL REFERENCES schools(id),
  applicant_id UUID NOT NULL UNIQUE REFERENCES applicants(id) ON DELETE RESTRICT,
  target_academic_year_id UUID NOT NULL REFERENCES academic_years(id),
  target_grade_level_id UUID NOT NULL REFERENCES grade_levels(id),
  target_section_id UUID REFERENCES sections(id),
  status handoff_status NOT NULL DEFAULT 'pending',
  validation_errors_json JSONB NOT NULL DEFAULT '{}'::jsonb,
  student_id UUID REFERENCES students(id),
  student_enrollment_id UUID REFERENCES student_enrollments(id),
  handed_off_at TIMESTAMPTZ,
  completed_at TIMESTAMPTZ,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  created_by UUID REFERENCES users(id),
  updated_by UUID REFERENCES users(id)
);

CREATE TABLE applicant_status_history (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID NOT NULL REFERENCES tenants(id),
  applicant_id UUID NOT NULL REFERENCES applicants(id) ON DELETE CASCADE,
  from_status applicant_status,
  to_status applicant_status NOT NULL,
  changed_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  changed_by UUID REFERENCES users(id),
  reason TEXT,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE TABLE admissions_audit_logs (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID NOT NULL REFERENCES tenants(id),
  school_id UUID NOT NULL REFERENCES schools(id),
  actor_user_id UUID REFERENCES users(id),
  entity_name VARCHAR(80) NOT NULL,
  entity_id UUID NOT NULL,
  action_name VARCHAR(80) NOT NULL,
  old_value JSONB,
  new_value JSONB,
  ip_address INET,
  user_agent TEXT,
  request_id UUID,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

ALTER TABLE applicants
  ADD CONSTRAINT fk_applicants_current_offer
  FOREIGN KEY (current_offer_id) REFERENCES admission_offers(id);
```


## 3. Constraints and indexes

The key constraints are one inquiry converting to only one applicant, no workflow progression without required form and document completion, one active offer per applicant by default, versioned documents, and immutable historical audit.[^1]
The main query patterns are pipeline dashboards by academic year and grade, duplicate review queues, missing-documents queues, evaluation workload, expiring offers, and pending SIS handoff validation.[^1]

**Important constraints**[^1]

- `admission_inquiries.inquiry_id -> applicants.inquiry_id` is one-to-one, because one inquiry may be converted into one applicant only.[^1]
- Add a trigger that blocks applicant progression to `evaluation_ready` or `under_evaluation` until the latest form submission is verified and all mandatory documents are verified or explicitly waived.[^1]
- Add a trigger that blocks offer issuance unless the applicant is in an eligible evaluation state, because offers may only be issued to eligible applicants.[^1]
- Add a partial unique index for one active offer per applicant, because one active offer per applicant is the default rule unless multi-round policy is introduced later.[^1]
- Revoke `DELETE` on `applicants`, `application_form_submissions`, `applicant_documents`, `admission_decisions`, `admission_offers`, `enrollment_handoffs`, `applicant_status_history`, and `admissions_audit_logs`, because progressed admissions history must remain reconstructable.[^1]

```sql
CREATE INDEX idx_inquiries_pipeline
ON admission_inquiries (tenant_id, school_id, academic_year_id, target_grade_level_id, status, created_at DESC);

CREATE INDEX idx_inquiries_contact_lookup
ON admission_inquiries (tenant_id, email, phone, student_date_of_birth);

CREATE INDEX idx_applicants_pipeline
ON applicants (tenant_id, school_id, academic_year_id, target_grade_level_id, status, created_at DESC);

CREATE INDEX idx_applicants_duplicate_review
ON applicants (tenant_id, duplicate_review_status, duplicate_risk_score DESC);

CREATE INDEX idx_form_submissions_status
ON application_form_submissions (tenant_id, applicant_id, status, submitted_at DESC);

CREATE INDEX idx_doc_requirements_grade
ON admission_document_requirements (tenant_id, school_id, academic_year_id, grade_level_id, status);

CREATE INDEX idx_applicant_documents_status
ON applicant_documents (tenant_id, applicant_id, status, document_type, version_no DESC);

CREATE INDEX idx_duplicate_matches_review
ON applicant_duplicate_matches (tenant_id, review_status, risk_score DESC, created_at DESC);

CREATE INDEX idx_evaluations_worklist
ON applicant_evaluations (tenant_id, school_id, status, evaluator_user_id, scheduled_at);

CREATE INDEX idx_decisions_applicant
ON admission_decisions (tenant_id, applicant_id, decided_at DESC);

CREATE UNIQUE INDEX uq_active_offer_per_applicant
ON admission_offers (tenant_id, applicant_id)
WHERE status IN ('issued');

CREATE INDEX idx_offers_expiry
ON admission_offers (tenant_id, school_id, status, expires_at);

CREATE INDEX idx_handoffs_status
ON enrollment_handoffs (tenant_id, school_id, status, created_at DESC);

CREATE INDEX idx_status_history_applicant
ON applicant_status_history (tenant_id, applicant_id, changed_at DESC);

CREATE INDEX idx_admissions_audit_entity
ON admissions_audit_logs (tenant_id, school_id, entity_name, entity_id, created_at DESC);
```


## 4. Sample records and query patterns

These samples reflect the PRD’s main workflow of inquiry capture, applicant creation, grade-specific form submission, document collection, offer issuance, and enrollment handoff.[^1]

```sql
INSERT INTO admission_inquiries (
  id, tenant_id, school_id, academic_year_id, target_grade_level_id,
  student_first_name, student_last_name, student_date_of_birth,
  parent_name, email, phone, inquiry_source, status, created_at, updated_at
) VALUES (
  'b1000000-0000-0000-0000-000000000001',
  '10000000-0000-0000-0000-000000000001',
  '20000000-0000-0000-0000-000000000001',
  '30000000-0000-0000-0000-000000000001',
  '40000000-0000-0000-0000-000000000005',
  'Aarav', 'Sharma', '2019-05-10',
  'Meera Sharma', 'meera@example.com', '+919900000001', 'website_form', 'qualified', now(), now()
);

INSERT INTO applicants (
  id, tenant_id, school_id, inquiry_id, academic_year_id, target_grade_level_id,
  first_name, last_name, date_of_birth, parent_name, email, phone,
  duplicate_risk_score, duplicate_review_status, status, created_at, updated_at
) VALUES (
  'b1000000-0000-0000-0000-000000000002',
  '10000000-0000-0000-0000-000000000001',
  '20000000-0000-0000-0000-000000000001',
  'b1000000-0000-0000-0000-000000000001',
  '30000000-0000-0000-0000-000000000001',
  '40000000-0000-0000-0000-000000000005',
  'Aarav', 'Sharma', '2019-05-10',
  'Meera Sharma', 'meera@example.com', '+919900000001',
  0.00, 'cleared', 'documents_pending', now(), now()
);

INSERT INTO application_form_versions (
  id, tenant_id, school_id, academic_year_id, grade_level_id, version_no,
  form_schema, mandatory_fields, status, effective_from, created_at, updated_at
) VALUES (
  'b1000000-0000-0000-0000-000000000003',
  '10000000-0000-0000-0000-000000000001',
  '20000000-0000-0000-0000-000000000001',
  '30000000-0000-0000-0000-000000000001',
  '40000000-0000-0000-0000-000000000005',
  1,
  '{"fields":["address","previous_school","blood_group"]}'::jsonb,
  '["address","previous_school"]'::jsonb,
  'active',
  '2026-04-01',
  now(), now()
);

INSERT INTO application_form_submissions (
  id, tenant_id, school_id, applicant_id, form_version_id, form_data,
  status, submitted_at, created_at, updated_at
) VALUES (
  'b1000000-0000-0000-0000-000000000004',
  '10000000-0000-0000-0000-000000000001',
  '20000000-0000-0000-0000-000000000001',
  'b1000000-0000-0000-0000-000000000002',
  'b1000000-0000-0000-0000-000000000003',
  '{"address":"Anna Nagar","previous_school":"Little Steps"}'::jsonb,
  'submitted',
  now(), now(), now()
);

INSERT INTO admission_offers (
  id, tenant_id, school_id, applicant_id, offer_no, status,
  issued_at, expires_at, offer_snapshot_json, created_at, updated_at
) VALUES (
  'b1000000-0000-0000-0000-000000000005',
  '10000000-0000-0000-0000-000000000001',
  '20000000-0000-0000-0000-000000000001',
  'b1000000-0000-0000-0000-000000000002',
  'OFF-2026-0001',
  'issued',
  now(),
  now() + interval '7 days',
  '{"grade":"Grade 1","academic_year":"2026-27"}'::jsonb,
  now(), now()
);
```

**Query patterns**[^1]

1. Inquiry-to-applicant funnel by academic year and grade, because admissions reporting must support inquiry, applicant, grade, status, and date-based filters.[^1]
```sql
SELECT academic_year_id, target_grade_level_id, status, count(*) AS total
FROM applicants
WHERE tenant_id = $1
GROUP BY academic_year_id, target_grade_level_id, status
ORDER BY academic_year_id, target_grade_level_id, status;
```

2. Duplicate-review queue, because duplicate detection must be reviewed before progression when strict duplicate control is enabled.[^1]
```sql
SELECT a.id, a.first_name, a.last_name, a.phone, a.email, a.date_of_birth,
       dm.risk_score, dm.match_basis, dm.review_status
FROM applicants a
JOIN applicant_duplicate_matches dm
  ON dm.applicant_id = a.id AND dm.tenant_id = a.tenant_id
WHERE a.tenant_id = $1
  AND dm.review_status IN ('flagged', 'pending_review')
ORDER BY dm.risk_score DESC, a.created_at DESC;
```

3. Missing mandatory documents before evaluation, because evaluation cannot begin until mandatory documents are submitted or waived.[^1]
```sql
SELECT a.id AS applicant_id, r.document_type
FROM applicants a
JOIN admission_document_requirements r
  ON r.tenant_id = a.tenant_id
 AND r.school_id = a.school_id
 AND r.academic_year_id = a.academic_year_id
 AND r.grade_level_id = a.target_grade_level_id
LEFT JOIN applicant_documents d
  ON d.tenant_id = a.tenant_id
 AND d.applicant_id = a.id
 AND d.document_type = r.document_type
 AND d.status IN ('verified', 'waived')
WHERE a.tenant_id = $1
  AND r.is_mandatory = TRUE
  AND d.id IS NULL;
```

4. Expiring offers, because offer expiry must be tracked and expired offers cannot be accepted unless reissued.[^1]
```sql
SELECT id, applicant_id, offer_no, expires_at
FROM admission_offers
WHERE tenant_id = $1
  AND status = 'issued'
  AND expires_at <= now() + interval '48 hours'
ORDER BY expires_at;
```

5. Pending SIS handoffs, because admitted applicants must be validated against downstream academic-year, grade, and enrollment rules before conversion completes.[^1]
```sql
SELECT eh.id, eh.applicant_id, eh.status, eh.validation_errors_json
FROM enrollment_handoffs eh
WHERE eh.tenant_id = $1
  AND eh.status IN ('pending', 'failed')
ORDER BY eh.created_at DESC;
```


## 5. Migration sequence and security-sensitive columns

The safest build order is inquiry and applicant master first, then form and document configuration, then evaluation and decisions, then offers, then handoff and audit, because the PRD makes controlled progression and auditability the core of the module.[^1]
The most sensitive data here is applicant identity, guardian contact data, uploaded documents, duplicate-review evidence, offer snapshots, and handoff validation details, so these columns should be role-protected and encrypted or tightly masked where appropriate.[^1]

**Migration sequence**[^1]

- Step 1: Create enums and foundational tables `admission_inquiries` and `applicants`, then back-link inquiry conversion.[^1]
- Step 2: Create `application_form_versions`, `application_form_submissions`, and `admission_document_requirements`, because grade-specific requirements are a gating rule.[^1]
- Step 3: Create `applicant_documents` and duplicate-review table `applicant_duplicate_matches`, then add progression triggers.[^1]
- Step 4: Create `applicant_evaluations`, `applicant_evaluation_scores`, and `admission_decisions`.[^1]
- Step 5: Create `admission_offers` and `admission_offer_conditions`, then add active-offer and expiry rules.[^1]
- Step 6: Create `enrollment_handoffs`, `applicant_status_history`, and `admissions_audit_logs`, then add RLS and immutable-history protections.[^1]

**Security-sensitive columns**[^1]

- `admission_inquiries.email`, `admission_inquiries.phone`, `applicants.email`, `applicants.phone`, `parent_name`, and `date_of_birth` are direct PII and should be masked in non-operational views.[^1]
- `application_form_submissions.form_data` may contain high-risk family, address, medical, and prior-school data and should be field-level protected.[^1]
- `applicant_documents.storage_key`, `original_file_name`, and file metadata identify sensitive uploaded documents and should never be exposed as raw storage paths.[^1]
- `applicant_duplicate_matches.match_basis`, `review_notes`, and `risk_score` can reveal internal fraud or duplication analysis and should be limited to admissions roles.[^1]
- `admission_offers.offer_snapshot_json` and `enrollment_handoffs.validation_errors_json` may expose decision rationale or downstream rule failures and should be visible only to authorized staff.[^1]
- `admissions_audit_logs.old_value`, `new_value`, `ip_address`, and `user_agent` are sensitive operational audit fields and should be retained with least-privilege access.[^1]

The next logical module is M7_12, unless you want me to normalize M7_11 against M7_1 and M7_9 so applicant-to-parent and final portal visibility line up cleanly.[^1]
<span style="display:none">[^2][^3][^4][^5]</span>

<div align="center">⁂</div>

[^1]: PRD_M7_11.txt

[^2]: PRD_M7_16.txt

[^3]: PRD_M7_8.txt

[^4]: PRD_M7_12.txt

[^5]: PRD_M7_13.txt

