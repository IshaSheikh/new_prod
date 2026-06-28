# School SaaS Web App (Next.js)

This is the frontend web application for the School SaaS Platform, built with [Next.js](https://nextjs.org/) App Router and Tailwind CSS.

## Prerequisites

- Node.js 20+

## Local Development Setup

1. Copy the environment variables:
   ```bash
   cp .env.local.example .env.local
   ```
2. Install dependencies:
   ```bash
   npm ci
   ```
3. Start the application:
   ```bash
   npm run dev
   ```

## Architecture & Standards

- **State Management**: We use React Query for server state and Context API for simple client state.
- **Data Fetching**: Server components are preferred for initial data fetching. Client components handle interactive mutations.
- **Styling**: Tailwind CSS with Radix UI primitives. See `ui-contracts` package for shared design tokens.
- **Multi-Tenancy**: The application routes and resolves tenants based on subdomain or custom domain configuration.

## Testing

```bash
# Run unit tests
npm run test

# Run E2E tests (Playwright)
npm run test:e2e
```
