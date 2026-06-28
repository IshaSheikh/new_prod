<!-- Source: PRDs | Last Verified: 2026-06-28 | Owner: Engineering -->
# CI/CD

Continuous integration and deployment pipeline for the platform.

---

## Pipeline Overview

```
Code Push / PR
    ↓
GitHub Actions: CI Pipeline
    ├── Lint + Format Check
    ├── Unit Tests
    ├── Integration Tests (test DB)
    ├── Security Tests (cross-tenant isolation)
    ├── Build (TypeScript compile)
    └── Docker Image Build
          ↓
        PR Merged to main
          ↓
        Deploy to Staging
          ↓
        E2E Tests (Playwright on staging)
          ↓
        Manual Approval (production deploy)
          ↓
        Deploy to Production
          ↓
        Smoke Tests
```

---

## Environments

| Environment | Purpose | Deploy Trigger | URL Pattern |
|-------------|---------|----------------|-------------|
| Local | Development | Manual | localhost |
| Preview | PR review | On PR open | pr-{number}.preview.schoolsaas.com |
| Staging | Integration testing | On merge to main | staging.schoolsaas.com |
| Production | Live traffic | Manual approval | *.schoolsaas.com |

---

## GitHub Actions Workflows

### `ci.yml` — Pull Request Checks

```yaml
name: CI

on:
  pull_request:
    branches: [main, develop]

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      - run: npm ci
      - run: npm run lint
      - run: npm run format:check

  unit-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      - run: npm ci
      - run: npm run test:unit
      - uses: actions/upload-artifact@v4
        with:
          name: coverage-report
          path: coverage/

  integration-tests:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:16
        env:
          POSTGRES_DB: schoolsaas_test
          POSTGRES_USER: testuser
          POSTGRES_PASSWORD: testpass
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
    env:
      DATABASE_URL: postgresql://testuser:testpass@localhost:5432/schoolsaas_test
      JWT_SECRET: test-jwt-secret-not-for-production
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      - run: npm ci
      - run: npm run db:migrate:test
      - run: npm run db:seed:test
      - run: npm run test:integration
      - run: npm run test:security  # Cross-tenant isolation tests

  build:
    runs-on: ubuntu-latest
    needs: [lint, unit-tests, integration-tests]
    steps:
      - uses: actions/checkout@v4
      - run: npm ci
      - run: npm run build
      - name: Build Docker image
        run: docker build -t schoolsaas-api:${{ github.sha }} .
```

### `deploy-staging.yml` — Deploy to Staging

```yaml
name: Deploy to Staging

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Build and push image to ECR
        env:
          AWS_ACCOUNT_ID: ${{ secrets.AWS_ACCOUNT_ID }}
          AWS_REGION: ap-south-1
        run: |
          aws ecr get-login-password --region $AWS_REGION | \
            docker login --username AWS --password-stdin \
            $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com
          docker build -t schoolsaas-api:${{ github.sha }} .
          docker tag schoolsaas-api:${{ github.sha }} \
            $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/schoolsaas-api:staging
          docker push $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/schoolsaas-api:staging
      
      - name: Run database migrations on staging
        run: |
          aws ecs run-task \
            --cluster schoolsaas-staging \
            --task-definition schoolsaas-migrations \
            --overrides '{"containerOverrides":[{"name":"migrate","command":["npm","run","db:migrate"]}]}'
      
      - name: Deploy to ECS staging
        run: |
          aws ecs update-service \
            --cluster schoolsaas-staging \
            --service schoolsaas-api \
            --force-new-deployment
      
      - name: Wait for deployment
        run: |
          aws ecs wait services-stable \
            --cluster schoolsaas-staging \
            --services schoolsaas-api
      
      - name: Run E2E tests
        run: npm run test:e2e -- --base-url=https://staging.schoolsaas.com
      
      - name: Run smoke tests
        run: npm run test:smoke -- --env=staging
```

### `deploy-production.yml` — Deploy to Production

```yaml
name: Deploy to Production

on:
  workflow_dispatch:
    inputs:
      image_tag:
        description: 'Docker image tag to deploy'
        required: true

jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: production  # Requires manual approval
    steps:
      - name: Run database migrations on production
        run: |
          # Migrations run as separate ECS task before traffic shift
          aws ecs run-task --cluster schoolsaas-prod ...
      
      - name: Deploy to ECS production (blue/green)
        run: |
          aws deploy create-deployment \
            --application-name schoolsaas-api \
            --deployment-group-name production \
            --image-uri ${{ env.FULL_IMAGE_URI }}
      
      - name: Run production smoke tests
        run: npm run test:smoke -- --env=production
```

---

## Infrastructure (AWS)

```
AWS Region: ap-south-1 (Mumbai)

Compute:
  ECS Fargate (API containers)
  Task size: 1 vCPU, 2GB RAM (baseline)
  Auto-scaling: 2-10 tasks based on CPU utilization

Database:
  RDS PostgreSQL 16
  Instance: db.r7g.large (production)
  Multi-AZ: enabled
  Automated backups: 7 days retention
  PgBouncer (transaction mode) for connection pooling

Storage:
  S3 bucket: schoolsaas-files-{env}
  CloudFront distribution for file access
  Lifecycle policy: 
    - Active files: Standard storage
    - Files older than 1 year: S3 Intelligent-Tiering
    - Tenant data after cancellation: retained for 365 days, then deleted

Load Balancing:
  ALB with HTTPS listener
  Host-based routing for tenant domain resolution
  WAF with rate limiting rules

Networking:
  VPC with private subnets for ECS and RDS
  NAT Gateway for outbound access
  Security groups: ECS → RDS only on port 5432
```

---

## Migration Strategy

Migrations are run as a separate ECS task before the new API containers start receiving traffic. This ensures:

1. Schema is updated before code that depends on it goes live
2. Migration failures halt deployment (no partial rollout)
3. Migrations are always forward-only in the deployment pipeline

```bash
# Migration commands
npm run db:migrate        # Apply all pending migrations
npm run db:migrate:down   # Rollback last migration (emergency only)
npm run db:migrate:status # Show migration status
npm run db:seed:prod      # Apply production seed data
```

**Rules:**
- Never run `db:migrate:down` in production without a rollback plan
- Migrations are always additive (add columns, add tables, add indexes)
- Never drop columns in a migration — mark as deprecated and schedule removal
- Index creation uses `CREATE INDEX CONCURRENTLY` to avoid table locks in production

---

## Environment Variables

```bash
# Required for all environments
DATABASE_URL=postgresql://user:pass@host:5432/dbname
JWT_SECRET=<256-bit-random-secret>
JWT_REFRESH_SECRET=<256-bit-random-secret>
JWT_EXPIRY=900          # 15 minutes in seconds
JWT_REFRESH_EXPIRY=2592000  # 30 days in seconds

# AWS
AWS_REGION=ap-south-1
AWS_S3_BUCKET=schoolsaas-files-prod
AWS_SES_FROM_EMAIL=noreply@schoolsaas.com

# Payment Gateway
RAZORPAY_KEY_ID=<key>
RAZORPAY_KEY_SECRET=<secret>
RAZORPAY_WEBHOOK_SECRET=<webhook-secret>

# External Services
SENTRY_DSN=<dsn>
SMS_PROVIDER_API_KEY=<key>
```

Secrets are stored in AWS Secrets Manager and injected as environment variables by ECS task definitions. Never committed to code or `.env` files in version control.

---

## Rollback Procedure

If production deployment fails:

1. **Automatic rollback:** ECS CodeDeploy triggers rollback if health checks fail after deployment
2. **Manual rollback:** Re-deploy the previous image tag via `deploy-production.yml` workflow
3. **Database rollback:** Only needed if migration introduced a bug — use `db:migrate:down` with extreme care; may require maintenance window

**Rollback decision matrix:**

| Issue | Action |
|-------|--------|
| API health checks failing | Auto-rollback via CodeDeploy |
| Bug in new code | Deploy previous image tag |
| Bad migration (additive) | Fix and re-deploy |
| Bad migration (destructive) | Emergency maintenance window + rollback migration |
| Data corruption | Stop traffic, assess, restore from backup |

