<img src="https://r2cdn.perplexity.ai/pplx-full-logo-primary-dark%402x.png" style="height:64px;margin-right:32px"/>

# continue the sequence m7_14

For M7_14, the right production model is **vehicle -> route -> ordered route stops -> student transport assignment -> optional transport payment linkage -> immutable assignment history**, because the PRD defines transport as a structured operational workflow tied to student records, parent visibility, capacity control, and fee integration.[^1]
This schema avoids creating any separate student identity model and instead anchors transport to SIS students, enrollments, parent-child relationships, and forward-only billing eligibility.[^1]

## 1. Entity list

These are the core entities for M7_14.[^1]

- `vehicles` — tenant-scoped buses or vans with unique identifiers, capacity, and lifecycle state, because vehicles must be uniquely identifiable and capacity-controlled.[^1]
- `drivers` — active or inactive driver records retained historically, because driver records must support status changes without deleting history.[^1]
- `routes` — route master with optional default vehicle and driver, because route ownership and route lifecycle are explicit parts of the module.[^1]
- `stops` — reusable stop master records with address and optional geolocation, because stops are operationally distinct from their placement on a route.[^1]
- `route_stops` — ordered stop sequencing per route with pickup and drop ETA, because route-stop ordering is route-specific by default and must preserve sequence integrity.[^1]
- `student_transport_assignments` — effective-dated student allocation to route and stop by assignment type, because assignment history must never be deleted and only active enrolled students should receive active assignments by default.[^1]
- `vehicle_route_history` — effective-dated route-to-vehicle history, because changing vehicles must preserve historical associations.[^1]
- `route_driver_history` — effective-dated route-to-driver history, because inactive-driver transitions and route responsibility need auditable history.[^1]
- `transport_capacity_overrides` — authorized over-capacity exceptions, because over-capacity attempts must either be blocked or require authorized override.[^1]
- `transport_payment_links` — forward-looking finance eligibility linkage for active assignments, because transport payment rules must not retroactively alter settled records.[^1]
- `transport_assignment_events` — immutable assignment change history for assign, unassign, stop-change, route-change, and closure.[^1]
- `transport_audit_logs` — full audit trail for create, edit, activate, deactivate, assign, unassign, and override actions.[^1]


## 2. Table definitions

I am separating reusable `stops` from `route_stops` because the PRD says stop reuse across routes is policy-based while route sequencing is definitely route-specific.[^1]
I am also using effective-dated history tables for route-vehicle, route-driver, and student assignments because the module explicitly requires preserved history, auditable changes, and forward-only finance linkage.[^1]

```sql
CREATE EXTENSION IF NOT EXISTS pgcrypto;
CREATE EXTENSION IF NOT EXISTS citext;

CREATE TYPE vehicle_status AS ENUM (
  'draft', 'active', 'inactive', 'reactivated', 'archived'
);

CREATE TYPE route_status AS ENUM (
  'draft', 'active', 'inactive', 'reactivated', 'archived'
);

CREATE TYPE stop_status AS ENUM (
  'active', 'inactive', 'archived'
);

CREATE TYPE driver_status AS ENUM (
  'active', 'inactive', 'archived'
);

CREATE TYPE assignment_type AS ENUM (
  'pickup_drop', 'pickup_only', 'drop_only'
);

CREATE TYPE transport_assignment_status AS ENUM (
  'draft', 'active', 'suspended', 'reactivated', 'ended', 'archived'
);

CREATE TYPE payment_link_status AS ENUM (
  'pending', 'due', 'paid', 'overdue', 'waived', 'closed'
);

CREATE TYPE override_status AS ENUM (
  'active', 'expired', 'revoked'
);

-- Assumes upstream tables exist:
-- tenants, schools, users, students, student_enrollments, invoices, invoice_line_items

CREATE TABLE vehicles (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID NOT NULL REFERENCES tenants(id),
  school_id UUID NOT NULL REFERENCES schools(id),
  vehicle_code VARCHAR(50) NOT NULL,
  vehicle_number VARCHAR(50) NOT NULL,
  vehicle_type VARCHAR(30) NOT NULL,
  capacity INTEGER NOT NULL,
  status vehicle_status NOT NULL DEFAULT 'draft',
  notes TEXT,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  created_by UUID REFERENCES users(id),
  updated_by UUID REFERENCES users(id),
  CONSTRAINT uq_vehicle_code UNIQUE (tenant_id, school_id, vehicle_code),
  CONSTRAINT uq_vehicle_number UNIQUE (tenant_id, school_id, vehicle_number),
  CONSTRAINT ck_vehicle_capacity CHECK (capacity > 0)
);

CREATE TABLE drivers (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID NOT NULL REFERENCES tenants(id),
  school_id UUID NOT NULL REFERENCES schools(id),
  driver_name VARCHAR(150) NOT NULL,
  mobile VARCHAR(20),
  license_code VARCHAR(50),
  status driver_status NOT NULL DEFAULT 'active',
  notes TEXT,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  created_by UUID REFERENCES users(id),
  updated_by UUID REFERENCES users(id),
  CONSTRAINT uq_driver_license UNIQUE (tenant_id, school_id, license_code)
);

CREATE TABLE routes (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID NOT NULL REFERENCES tenants(id),
  school_id UUID NOT NULL REFERENCES schools(id),
  route_name VARCHAR(120) NOT NULL,
  route_code VARCHAR(50) NOT NULL,
  capacity_limit INTEGER,
  status route_status NOT NULL DEFAULT 'draft',
  default_vehicle_id UUID REFERENCES vehicles(id),
  default_driver_id UUID REFERENCES drivers(id),
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  created_by UUID REFERENCES users(id),
  updated_by UUID REFERENCES users(id),
  CONSTRAINT uq_route_code UNIQUE (tenant_id, school_id, route_code),
  CONSTRAINT ck_route_capacity_limit CHECK (capacity_limit IS NULL OR capacity_limit > 0)
);

CREATE TABLE stops (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID NOT NULL REFERENCES tenants(id),
  school_id UUID NOT NULL REFERENCES schools(id),
  stop_name VARCHAR(120) NOT NULL,
  stop_code VARCHAR(50) NOT NULL,
  address TEXT,
  latitude NUMERIC(10,7),
  longitude NUMERIC(10,7),
  status stop_status NOT NULL DEFAULT 'active',
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  created_by UUID REFERENCES users(id),
  updated_by UUID REFERENCES users(id),
  CONSTRAINT uq_stop_code UNIQUE (tenant_id, school_id, stop_code)
);

CREATE TABLE route_stops (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID NOT NULL REFERENCES tenants(id),
  route_id UUID NOT NULL REFERENCES routes(id) ON DELETE CASCADE,
  stop_id UUID NOT NULL REFERENCES stops(id),
  sequence_no INTEGER NOT NULL,
  pickup_eta TIME,
  drop_eta TIME,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  created_by UUID REFERENCES users(id),
  updated_by UUID REFERENCES users(id),
  CONSTRAINT uq_route_stop UNIQUE (tenant_id, route_id, stop_id),
  CONSTRAINT uq_route_stop_sequence UNIQUE (tenant_id, route_id, sequence_no),
  CONSTRAINT ck_route_stop_sequence CHECK (sequence_no > 0)
);

CREATE TABLE vehicle_route_history (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID NOT NULL REFERENCES tenants(id),
  route_id UUID NOT NULL REFERENCES routes(id) ON DELETE CASCADE,
  vehicle_id UUID NOT NULL REFERENCES vehicles(id),
  effective_from DATE NOT NULL,
  effective_to DATE,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  created_by UUID REFERENCES users(id),
  CONSTRAINT ck_vehicle_route_dates CHECK (
    effective_to IS NULL OR effective_to >= effective_from
  )
);

CREATE TABLE route_driver_history (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID NOT NULL REFERENCES tenants(id),
  route_id UUID NOT NULL REFERENCES routes(id) ON DELETE CASCADE,
  driver_id UUID NOT NULL REFERENCES drivers(id),
  effective_from DATE NOT NULL,
  effective_to DATE,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  created_by UUID REFERENCES users(id),
  CONSTRAINT ck_route_driver_dates CHECK (
    effective_to IS NULL OR effective_to >= effective_from
  )
);

CREATE TABLE student_transport_assignments (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID NOT NULL REFERENCES tenants(id),
  student_id UUID NOT NULL REFERENCES students(id),
  student_enrollment_id UUID NOT NULL REFERENCES student_enrollments(id),
  route_id UUID NOT NULL REFERENCES routes(id),
  stop_id UUID NOT NULL REFERENCES stops(id),
  route_stop_id UUID NOT NULL REFERENCES route_stops(id),
  assignment_type assignment_type NOT NULL DEFAULT 'pickup_drop',
  effective_from DATE NOT NULL,
  effective_to DATE,
  status transport_assignment_status NOT NULL DEFAULT 'draft',
  capacity_override_id UUID,
  finance_enabled BOOLEAN NOT NULL DEFAULT FALSE,
  notes TEXT,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  created_by UUID REFERENCES users(id),
  updated_by UUID REFERENCES users(id),
  CONSTRAINT ck_assignment_dates CHECK (
    effective_to IS NULL OR effective_to >= effective_from
  )
);

CREATE TABLE transport_capacity_overrides (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID NOT NULL REFERENCES tenants(id),
  route_id UUID NOT NULL REFERENCES routes(id),
  student_id UUID REFERENCES students(id),
  approved_by UUID NOT NULL REFERENCES users(id),
  approved_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  reason TEXT NOT NULL,
  override_until DATE,
  status override_status NOT NULL DEFAULT 'active',
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  CONSTRAINT ck_override_until CHECK (
    override_until IS NULL OR override_until >= approved_at::date
  )
);

ALTER TABLE student_transport_assignments
  ADD CONSTRAINT fk_transport_assignment_override
  FOREIGN KEY (capacity_override_id) REFERENCES transport_capacity_overrides(id);

CREATE TABLE transport_payment_links (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID NOT NULL REFERENCES tenants(id),
  student_id UUID NOT NULL REFERENCES students(id),
  student_transport_assignment_id UUID NOT NULL REFERENCES student_transport_assignments(id) ON DELETE CASCADE,
  fee_period VARCHAR(20) NOT NULL,
  amount NUMERIC(12,2) NOT NULL,
  status payment_link_status NOT NULL DEFAULT 'pending',
  due_date DATE NOT NULL,
  paid_at TIMESTAMPTZ,
  finance_invoice_id UUID REFERENCES invoices(id),
  finance_invoice_line_item_id UUID REFERENCES invoice_line_items(id),
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  created_by UUID REFERENCES users(id),
  updated_by UUID REFERENCES users(id),
  CONSTRAINT ck_transport_payment_amount CHECK (amount >= 0),
  CONSTRAINT uq_transport_payment_period UNIQUE (
    tenant_id, student_transport_assignment_id, fee_period
  )
);

CREATE TABLE transport_assignment_events (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID NOT NULL REFERENCES tenants(id),
  student_transport_assignment_id UUID NOT NULL REFERENCES student_transport_assignments(id) ON DELETE CASCADE,
  student_id UUID NOT NULL REFERENCES students(id),
  event_name VARCHAR(80) NOT NULL,
  event_snapshot JSONB NOT NULL DEFAULT '{}'::jsonb,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  created_by UUID REFERENCES users(id)
);

CREATE TABLE transport_audit_logs (
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
```


## 3. Constraints

The core rules are unique tenant-scoped identifiers, route-stop sequence integrity, active-only assignment targets, one active assignment per student per assignment type by default, capacity enforcement, and no retroactive finance mutation.[^1]

- Every transport-owned table has `tenant_id`, because all vehicles, routes, stops, assignments, and payment records must be tenant-scoped and inaccessible across tenants.[^1]
- `vehicles.vehicle_code` and `vehicles.vehicle_number` are unique within school context, because vehicles must be uniquely identifiable within the tenant.[^1]
- `route_stops` enforces unique `(tenant_id, route_id, sequence_no)` so route order stays deterministic and searchable.[^1]
- A trigger should block `student_transport_assignments` if the referenced student is not actively enrolled unless future assignment policy is explicitly enabled, because only active enrolled students may receive active transport assignments by default.[^1]
- A trigger should block assignment to inactive route or inactive stop, because transport assignments cannot reference inactive route or stop records.[^1]
- A trigger should verify `route_stop_id` belongs to the same `route_id` and same `stop_id`, because invalid route-stop references are a stated data-quality risk.[^1]
- Add a partial unique index for one active assignment per student per assignment type, because duplicate active transport assignments are explicitly disallowed by default.[^1]
- Capacity enforcement should count only active assignments for the route, because utilization and available capacity must use active records only.[^1]
- Over-capacity assignment should be blocked unless a valid `transport_capacity_overrides` row is attached, because the PRD permits authorized overrides.[^1]
- Route or stop deactivation should block new assignments and raise review flags for existing active assignments, because inactive routes and stops cannot receive new student assignments.[^1]
- Finance triggers should create forward-looking `transport_payment_links` only for future periods and must never rewrite settled invoices or paid records, because historical finance records cannot be retroactively altered.[^1]


## 4. Indexes

These indexes match the likely hot paths for transport dashboards, student assignment search, route utilization, stop-level counts, parent views, and finance linkage.[^1]

```sql
CREATE INDEX idx_vehicles_school_status
ON vehicles (tenant_id, school_id, status, vehicle_code);

CREATE INDEX idx_routes_school_status
ON routes (tenant_id, school_id, status, route_name);

CREATE INDEX idx_stops_school_status
ON stops (tenant_id, school_id, status, stop_name);

CREATE INDEX idx_route_stops_route_sequence
ON route_stops (tenant_id, route_id, sequence_no);

CREATE INDEX idx_vehicle_route_history_active
ON vehicle_route_history (tenant_id, route_id, effective_from, effective_to);

CREATE INDEX idx_route_driver_history_active
ON route_driver_history (tenant_id, route_id, effective_from, effective_to);

CREATE INDEX idx_transport_assignments_student
ON student_transport_assignments (tenant_id, student_id, status, effective_from DESC);

CREATE INDEX idx_transport_assignments_route
ON student_transport_assignments (tenant_id, route_id, stop_id, status, effective_from DESC);

CREATE INDEX idx_transport_assignments_enrollment
ON student_transport_assignments (tenant_id, student_enrollment_id, status);

CREATE UNIQUE INDEX uq_active_transport_assignment
ON student_transport_assignments (tenant_id, student_id, assignment_type)
WHERE status IN ('active', 'reactivated') AND effective_to IS NULL;

CREATE INDEX idx_transport_payment_links_student_period
ON transport_payment_links (tenant_id, student_id, fee_period, status);

CREATE INDEX idx_transport_payment_links_assignment
ON transport_payment_links (tenant_id, student_transport_assignment_id, status);

CREATE INDEX idx_transport_assignment_events_assignment
ON transport_assignment_events (tenant_id, student_transport_assignment_id, created_at DESC);

CREATE INDEX idx_transport_audit_logs_entity
ON transport_audit_logs (tenant_id, school_id, entity_name, entity_id, created_at DESC);
```


## 5. Sample records

These examples show one active vehicle, one route, one route stop, one active student assignment, and one transport-linked future fee obligation.[^1]

```sql
INSERT INTO vehicles (
  id, tenant_id, school_id, vehicle_code, vehicle_number, vehicle_type,
  capacity, status, created_at, updated_at
) VALUES (
  'e1000000-0000-0000-0000-000000000001',
  '10000000-0000-0000-0000-000000000001',
  '20000000-0000-0000-0000-000000000001',
  'BUS-01',
  'TN-10-AB-1234',
  'bus',
  40,
  'active',
  now(), now()
);

INSERT INTO routes (
  id, tenant_id, school_id, route_name, route_code, capacity_limit, status,
  default_vehicle_id, created_at, updated_at
) VALUES (
  'e1000000-0000-0000-0000-000000000002',
  '10000000-0000-0000-0000-000000000001',
  '20000000-0000-0000-0000-000000000001',
  'North Zone Morning',
  'R-NZ-01',
  40,
  'active',
  'e1000000-0000-0000-0000-000000000001',
  now(), now()
);

INSERT INTO stops (
  id, tenant_id, school_id, stop_name, stop_code, address, status,
  created_at, updated_at
) VALUES (
  'e1000000-0000-0000-0000-000000000003',
  '10000000-0000-0000-0000-000000000001',
  '20000000-0000-0000-0000-000000000001',
  'Anna Nagar Main',
  'STOP-ANN-01',
  'Anna Nagar Main Road',
  'active',
  now(), now()
);

INSERT INTO route_stops (
  id, tenant_id, route_id, stop_id, sequence_no, pickup_eta, drop_eta,
  created_at, updated_at
) VALUES (
  'e1000000-0000-0000-0000-000000000004',
  '10000000-0000-0000-0000-000000000001',
  'e1000000-0000-0000-0000-000000000002',
  'e1000000-0000-0000-0000-000000000003',
  1,
  '07:10',
  '16:20',
  now(), now()
);

INSERT INTO student_transport_assignments (
  id, tenant_id, student_id, student_enrollment_id, route_id, stop_id, route_stop_id,
  assignment_type, effective_from, status, finance_enabled, created_at, updated_at
) VALUES (
  'e1000000-0000-0000-0000-000000000005',
  '10000000-0000-0000-0000-000000000001',
  '70000000-0000-0000-0000-000000000101',
  '80000000-0000-0000-0000-000000000101',
  'e1000000-0000-0000-0000-000000000002',
  'e1000000-0000-0000-0000-000000000003',
  'e1000000-0000-0000-0000-000000000004',
  'pickup_drop',
  '2026-06-15',
  'active',
  TRUE,
  now(), now()
);

INSERT INTO transport_payment_links (
  id, tenant_id, student_id, student_transport_assignment_id, fee_period,
  amount, status, due_date, created_at, updated_at
) VALUES (
  'e1000000-0000-0000-0000-000000000006',
  '10000000-0000-0000-0000-000000000001',
  '70000000-0000-0000-0000-000000000101',
  'e1000000-0000-0000-0000-000000000005',
  '2026-07',
  1800.00,
  'due',
  '2026-07-10',
  now(), now()
);
```


## 6. Query patterns

These are the main operational queries the PRD implies.[^1]

1. Route utilization and over-capacity monitoring, because principals and admins need route utilization, vehicle capacity, and active assignment metrics.[^1]
```sql
SELECT
  r.id,
  r.route_name,
  COALESCE(r.capacity_limit, v.capacity) AS effective_capacity,
  count(sta.id) FILTER (
    WHERE sta.status IN ('active', 'reactivated')
      AND (sta.effective_to IS NULL OR sta.effective_to >= current_date)
  ) AS active_students
FROM routes r
LEFT JOIN vehicles v
  ON v.id = r.default_vehicle_id
LEFT JOIN student_transport_assignments sta
  ON sta.route_id = r.id
 AND sta.tenant_id = r.tenant_id
WHERE r.tenant_id = $1
  AND r.status IN ('active', 'reactivated')
GROUP BY r.id, r.route_name, r.capacity_limit, v.capacity
ORDER BY r.route_name;
```

2. Student transport assignment current and history view.[^1]
```sql
SELECT
  sta.id,
  sta.assignment_type,
  sta.status,
  sta.effective_from,
  sta.effective_to,
  r.route_name,
  s.stop_name
FROM student_transport_assignments sta
JOIN routes r ON r.id = sta.route_id
JOIN stops s ON s.id = sta.stop_id
WHERE sta.tenant_id = $1
  AND sta.student_id = $2
ORDER BY sta.effective_from DESC, sta.created_at DESC;
```

3. Parent child transport visibility, because parent access must follow relationship-based access and linked-child filtering.[^1]
```sql
SELECT
  sta.id,
  r.route_name,
  s.stop_name,
  rs.pickup_eta,
  rs.drop_eta,
  v.vehicle_number
FROM parent_student_links psl
JOIN student_transport_assignments sta
  ON sta.student_id = psl.student_id
 AND sta.tenant_id = psl.tenant_id
JOIN routes r ON r.id = sta.route_id
JOIN stops s ON s.id = sta.stop_id
JOIN route_stops rs ON rs.id = sta.route_stop_id
LEFT JOIN routes rt ON rt.id = sta.route_id
LEFT JOIN vehicles v ON v.id = rt.default_vehicle_id
WHERE psl.tenant_id = $1
  AND psl.parent_user_id = $2
  AND psl.status = 'active'
  AND sta.status IN ('active', 'reactivated');
```

4. Stop assignment counts for route review, because stop-level visibility supports route administration and dispute handling.[^1]
```sql
SELECT
  rs.sequence_no,
  s.stop_name,
  count(sta.id) FILTER (
    WHERE sta.status IN ('active', 'reactivated')
      AND (sta.effective_to IS NULL OR sta.effective_to >= current_date)
  ) AS active_assignments
FROM route_stops rs
JOIN stops s ON s.id = rs.stop_id
LEFT JOIN student_transport_assignments sta
  ON sta.route_stop_id = rs.id
 AND sta.tenant_id = rs.tenant_id
WHERE rs.tenant_id = $1
  AND rs.route_id = $2
GROUP BY rs.sequence_no, s.stop_name
ORDER BY rs.sequence_no;
```

5. Finance-linked transport dues, because accountants need to view transport-linked payment obligations where finance integration is enabled.[^1]
```sql
SELECT
  tpl.student_id,
  tpl.fee_period,
  tpl.amount,
  tpl.status,
  tpl.due_date
FROM transport_payment_links tpl
WHERE tpl.tenant_id = $1
  AND tpl.status IN ('pending', 'due', 'overdue')
ORDER BY tpl.due_date, tpl.student_id;
```


## 7. Migration sequence

The safest order is vehicle, driver, route, and stop masters first, then route history and route-stop mapping, then student assignments, then overrides and finance links, then eventing and audit.[^1]

1. Create enums and master tables `vehicles`, `drivers`, `routes`, and `stops`.[^1]
2. Create `route_stops`, `vehicle_route_history`, and `route_driver_history`.[^1]
3. Create `student_transport_assignments` with effective-dated validation and active-assignment constraints.[^1]
4. Create `transport_capacity_overrides` and wire the optional override foreign key.[^1]
5. Create `transport_payment_links`, `transport_assignment_events`, and `transport_audit_logs`.[^1]
6. Add triggers for active-student validation, active-route-stop validation, route-stop consistency, capacity enforcement, safe deactivation, forward-only payment-link creation, and tenant RLS policies.[^1]

## 8. Security-sensitive columns

The most sensitive transport data is child movement allocation, parent visibility, driver contact details, and finance-linked transport obligations, because the PRD highlights safety, privacy, and relationship-based portal access as central risks.[^1]

- `drivers.mobile` and `license_code` are sensitive personal fields and should be visible only to authorized operational staff.[^1]
- `stops.address`, `latitude`, and `longitude` can reveal child pickup locations and should be restricted in parent/student APIs to only the linked child’s active assignment context.[^1]
- `student_transport_assignments.notes` may contain operational or safety context and should be least-privilege only.[^1]
- `transport_payment_links.amount`, `status`, and finance references should follow finance-role and parent-portal policies rather than broad transport-role visibility.[^1]
- `transport_assignment_events.event_snapshot` and `transport_audit_logs.old_value`, `new_value`, `ip_address`, and `user_agent` are sensitive audit fields and should be protected accordingly.[^1]

Next in sequence is M7_15, which should naturally reuse the same student, parent-link, and immutable history patterns.[^1]

<div align="center">⁂</div>

[^1]: PRD_M7_14_15.txt

