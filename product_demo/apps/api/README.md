# School SaaS API (NestJS)

This is the core backend API for the School SaaS Platform, built with [NestJS](https://nestjs.com/).

## Prerequisites

- Node.js 20+
- PostgreSQL 16+
- Docker (for local database and Redis)

## Local Development Setup

1. Copy the environment variables:
   ```bash
   cp .env.example .env
   ```
2. Start the local database:
   ```bash
   docker-compose up -d db
   ```
3. Run migrations and seed data:
   ```bash
   npm run db:migrate
   npm run db:seed
   ```
4. Start the application:
   ```bash
   npm run start:dev
   ```

## Architecture & Standards

- **Module Structure**: Code is organized into business modules in `src/modules/` (e.g., `attendance`, `finance`).
- **Data Access**: We use TypeORM with strict row-level security (RLS) enforcement. See [Coding Standards](../../docs/06-engineering/coding-standards.md) for repository guidelines.
- **Authentication**: JWT-based with refresh tokens. See [Auth Flows](../../docs/04-api/auth-flows.md).
- **Error Handling**: Follow the standard error format in [Error Contracts](../../docs/04-api/error-contracts.md).

## Testing

```bash
# Unit tests
npm run test

# Integration tests
npm run test:e2e
```

Please refer to the [Test Strategy](../../docs/06-engineering/test-strategy.md) for coverage requirements.
