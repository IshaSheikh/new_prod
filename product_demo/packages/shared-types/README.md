# Shared Types Package

This package contains shared TypeScript interfaces, types, and enums used across the API, Web, and Mobile (via generation) applications.

## Purpose

By keeping shared types in a single package, we ensure that API responses, request payloads, and domain enums remain strictly synchronized between the frontend and backend.

## Usage

In any monorepo package (e.g., `apps/web` or `apps/api`), add the dependency:

```json
"dependencies": {
  "@schoolsaas/shared-types": "workspace:*"
}
```

Then import the types:

```typescript
import { UserRole, AttendanceStatus } from '@schoolsaas/shared-types';
```

## Maintenance

When updating or adding types here, ensure you run the build step to compile TypeScript declarations, so other packages pick up the changes.
