# Incident Runbook

Procedures for responding to platform incidents. All severity levels require a post-incident review within 48 hours.

---

## Severity Definitions

| Severity | Definition | Example |
|----------|-----------|---------|
| **SEV-1** | Complete platform outage or data exposure affecting multiple tenants | All schools unable to access platform; cross-tenant data visible |
| **SEV-2** | Core feature unavailable for one or more tenants | Attendance submission failing; fee payments not processing; portal inaccessible |
| **SEV-3** | Degraded performance or non-critical feature failure | Reports slow to load; notification delivery delayed |
| **SEV-4** | Minor issue with workaround available | Branding not showing correctly; export button not working |

---

## On-Call Rotation

- Primary on-call: rotates weekly
- Escalation contact: platform owner
- All incidents logged in incident tracking system

---

## SEV-1 Response Procedure

**Trigger:** Platform monitoring alert or customer report of complete outage or data exposure.

**Immediate actions (first 15 minutes):**

1. **Acknowledge incident** — respond in monitoring channel: "SEV-1 declared, investigating"
2. **Initial triage:**
   - Check ECS service health: `aws ecs describe-services --cluster schoolsaas-prod`
   - Check RDS connectivity: `pg_isready -h <rds-endpoint>`
   - Check ALB health: verify target group health in AWS console
   - Check Sentry for recent error spikes
3. **If data exposure suspected:**
   - **Immediately put platform in maintenance mode** (disable incoming traffic)
   - Escalate to platform owner within 5 minutes
   - Do NOT investigate scope without senior engineering present
4. **If service outage (no data exposure):**
   - Identify failing component (ECS tasks, RDS, ALB, S3)
   - Assess whether rollback to previous deployment will help

**Communication (within 30 minutes):**
- Post status update to status page: "We are investigating a service disruption. All teams are engaged."
- Notify pilot school contacts if during business hours
- Update every 30 minutes until resolved

**Resolution steps by failure type:**

### ECS Tasks Failing

```bash
# Check task logs
aws ecs describe-tasks --cluster schoolsaas-prod --tasks $(aws ecs list-tasks --cluster schoolsaas-prod --query 'taskArns' --output text)

# View recent logs
aws logs get-log-events \
  --log-group-name /ecs/schoolsaas-api \
  --log-stream-name <latest-stream>

# Force new deployment if tasks are stuck
aws ecs update-service --cluster schoolsaas-prod --service schoolsaas-api --force-new-deployment

# Rollback to previous task definition
aws ecs update-service --cluster schoolsaas-prod --service schoolsaas-api \
  --task-definition schoolsaas-api:<previous-revision>
```

### RDS Connection Issues

```bash
# Check RDS status
aws rds describe-db-instances --db-instance-identifier schoolsaas-prod

# Check connections via CloudWatch
aws cloudwatch get-metric-statistics \
  --namespace AWS/RDS \
  --metric-name DatabaseConnections \
  --dimensions Name=DBInstanceIdentifier,Value=schoolsaas-prod \
  --start-time $(date -u -d '1 hour ago' '+%Y-%m-%dT%H:%M:%SZ') \
  --end-time $(date -u '+%Y-%m-%dT%H:%M:%SZ') \
  --period 300 \
  --statistics Average

# Restart PgBouncer connection pool if stuck connections
aws ecs update-service --cluster schoolsaas-prod --service pgbouncer --force-new-deployment
```

### Migration Failed Mid-Deploy

```bash
# Check migration status
psql $DATABASE_URL -c "SELECT * FROM migrations ORDER BY run_on DESC LIMIT 10;"

# If migration left schema in bad state, assess and decide:
# Option A: Fix the migration and redeploy
# Option B: Roll back the migration (if down() is safe)
# Option C: Apply manual fix SQL (emergency only — must be documented)

# Never use --skip-migrations in production without senior engineering approval
```

---

## SEV-2 Response Procedure

**Trigger:** Core feature reported broken by multiple users or automated monitoring alert.

**Response within 30 minutes:**

1. **Identify affected scope:** Which tenant(s)? Which feature? Which operations?
2. **Check recent deployments:** Did a deployment happen in the last 2 hours?
3. **Check error rates in Sentry:** Look for new error patterns
4. **Assess workaround:** Can affected schools work around the issue temporarily?
5. **If deployment-related:** Rollback the deployment
6. **If data-related:** Identify and scope the corrupted data

**Communication:**
- Post to status page: "We are aware of issues affecting [feature]. We are working on a fix."
- Contact affected pilot schools directly if business hours

---

## SEV-3 / SEV-4 Response Procedure

**Response within 2 hours (SEV-3) or next business day (SEV-4):**

1. Log the issue in the issue tracker with full reproduction steps
2. Assign to the on-call engineer for investigation
3. Post status update if users may notice the degradation
4. Fix in the next scheduled deployment unless the issue worsens

---

## Security Incident Response

If a security incident is suspected (unauthorized access, data exposure, suspected breach):

1. **STOP** — do not attempt to investigate alone
2. **Escalate immediately** to platform owner
3. **Preserve evidence** — take screenshots of error logs, do not clear logs
4. **Assess scope:**
   - Is it a single tenant or multiple tenants?
   - What data may have been exposed?
   - Was it caused by a code bug, configuration error, or external attack?
5. **Isolate:** If multi-tenant exposure, put platform in maintenance mode
6. **Notify affected tenants** within 72 hours per data protection requirements
7. **Conduct thorough post-incident review** within 24 hours

---

## Useful Diagnostic Queries

### Check audit logs for unexpected access

```sql
-- Recent audit events by tenant
SELECT created_at, actor_user_id, action, entity_table, entity_id
FROM audit_logs
WHERE tenant_id = '<tenant-id>'
  AND created_at > now() - interval '1 hour'
ORDER BY created_at DESC;

-- Cross-tenant access attempts (should be zero)
SELECT a.actor_user_id, a.action, a.entity_id,
       m.tenant_id AS actor_tenant,
       a.tenant_id AS affected_tenant
FROM audit_logs a
JOIN user_tenant_memberships m
  ON m.user_id = a.actor_user_id
WHERE a.tenant_id != m.tenant_id
  AND a.created_at > now() - interval '24 hours';
```

### Check for failed payments needing attention

```sql
SELECT id, student_id, amount, status, created_at, failure_reason
FROM payment_transactions
WHERE status IN ('failed', 'expired', 'refund_failed')
  AND created_at > now() - interval '24 hours'
ORDER BY created_at DESC;
```

### Check RLS effectiveness

```sql
-- As the app_school_user role, verify this returns ONLY the configured tenant's data
SET app.tenant_id = '<test-tenant-id>';
SELECT count(*) FROM students;  -- Should only count test tenant's students
RESET app.tenant_id;
SELECT count(*) FROM students;  -- Should return 0 (no tenant set = no data)
```

### Check unreconciled payments

```sql
SELECT count(*), sum(p.amount)
FROM payment_transactions p
WHERE p.status = 'success'
  AND NOT EXISTS (
    SELECT 1 FROM reconciliation_items ri
    WHERE ri.payment_transaction_id = p.id
      AND ri.match_status = 'matched'
  )
  AND p.created_at > now() - interval '3 days';
```

---

## Post-Incident Review Template

Complete within 48 hours of incident resolution:

```
## Incident Report: [Brief Description]

**Date/Time:** 
**Duration:**
**Severity:** SEV-[1/2/3/4]
**Affected:**  [Tenants/Features/Users]

### Timeline
- HH:MM — Event 1
- HH:MM — Event 2
- HH:MM — Incident resolved

### Root Cause
[What caused the incident?]

### Impact
[What was the user-visible impact?]

### Resolution
[What steps resolved the incident?]

### What Went Well
- 

### What Could Be Improved
- 

### Action Items
| Action | Owner | Due Date |
|--------|-------|----------|
|        |       |          |
```
