# UI Contracts Package

This package contains shared design tokens, Tailwind configuration, and base React UI components used across the web-based applications of the School SaaS Platform.

## Purpose

Centralizing the UI contracts ensures a consistent look and feel (branding, spacing, typography) across all frontend properties.

## Contents

- **Design Tokens**: Standardized colors, spacing, and typography.
- **Tailwind Config**: A shared base configuration for Tailwind CSS.
- **Base Components**: Low-level, highly reusable React components (e.g., Buttons, Inputs, Modals) built using Radix UI primitives and Tailwind.

## Usage

In your consuming app (e.g., `apps/web`):

1. Extend the Tailwind config in `tailwind.config.js`:
   ```javascript
   const sharedConfig = require('@schoolsaas/ui-contracts/tailwind.config.js');
   module.exports = {
     presets: [sharedConfig],
     // App specific overrides...
   };
   ```
2. Import components:
   ```typescript
   import { Button } from '@schoolsaas/ui-contracts';
   ```
