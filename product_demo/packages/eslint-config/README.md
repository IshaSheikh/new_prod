# ESLint Config Package

This package provides a unified, strict ESLint configuration for the entire School SaaS Platform monorepo.

## Purpose

Enforce code quality, security rules, and style consistency across all TypeScript projects (NestJS, Next.js, and shared packages).

## Usage

In your consuming package, extend the configuration in your `.eslintrc.js`:

```javascript
module.exports = {
  extends: ['@schoolsaas/eslint-config'],
  // Package-specific overrides...
};
```

This configuration integrates rules for:
- TypeScript best practices
- Prettier formatting
- Security checks (e.g., preventing hardcoded secrets)
- React hooks (for frontend projects)
