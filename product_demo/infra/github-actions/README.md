# GitHub Actions (CI/CD)

This directory contains the GitHub Actions workflows that power the CI/CD pipeline for the School SaaS Platform.

## Workflows

1. **`ci.yml`**: Runs on pull requests to `main` and `develop`. It handles:
   - Linting and Formatting
   - Unit Tests
   - Integration Tests (spinning up a local test Postgres DB)
   - Security and cross-tenant isolation tests
   - Build compilation
2. **`deploy-staging.yml`**: Triggers on merges to `main`. It handles:
   - Building Docker images and pushing to ECR
   - Running database migrations via ECS task
   - Deploying to ECS staging environment
   - Running E2E tests (Playwright) against the staging environment
3. **`deploy-production.yml`**: Manual trigger workflow for production releases. Handles:
   - Blue/green deployment to ECS production
   - Database migrations
   - Post-deployment smoke tests

## Rules & Standards

For detailed information about our CI/CD strategy, environment variables, and rollback procedures, see the [CI/CD Documentation](../../docs/06-engineering/ci-cd.md).
