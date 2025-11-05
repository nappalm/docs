# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

BuildFast is an AI-driven React boilerplate for rapid development of SPAs using React, React Router Dom, Chakra UI, React Query, Axios, Supabase, and Stripe integration. The project follows a "Screaming Architecture" pattern organized by domain features rather than technical types.

## Development Commands

### Core Development
- `npm install` - Install dependencies
- `npm run dev` - Start development server (runs on http://localhost:5173)
- `npm run build` - Build for production (also generates sitemap)
- `npm run preview` - Preview production build

### Code Quality
- `npm run lint` - Run ESLint
- `npm run format` - Format code with Prettier

### Feature Generation
- `npm run plop` - Generate new feature scaffolding using Plop templates
  - Creates self-contained feature modules in `src/features/`
  - Includes components, hooks, pages, router, and services

### Git Hooks
- Husky is configured with commit-msg hook that enforces conventional commits (@commitlint/config-conventional)

## Architecture

### Screaming Architecture Principles

The codebase is organized by **domain features** (business capabilities) rather than technical file types. Each feature is a self-contained vertical slice.

#### Directory Structure

**`src/app/`** - Global application configuration
- `main.tsx` - Application entry point
- `providers.tsx` - Global provider composition (ChakraProvider → ReactQueryClient → AuthProvider → RouterProvider)
- `router.tsx` - Main router with protected/public route configuration

**`src/features/`** - Domain-driven feature modules (auth, home, settings)
Each feature contains:
- `components/` - Feature-specific UI components
- `hooks/` - Custom hooks for orchestration
- `pages/` - Route-level page components
- `router/` - Feature routing configuration
  - `paths.ts` - Route path constants
  - `router.tsx` - React Router configuration
- `services/` - API communication functions
- `utils/` - Feature-specific utilities
- `index.ts` - **Public API** - Only exports what other features can consume

**`src/lib/`** - Third-party library configurations
- `axios/` - Axios client setup
- `chakra-ui/` - Custom Chakra theme and component overrides
- `react-query/` - QueryClient configuration (refetchOnWindowFocus: false)
- `stripe/` - Stripe integration setup
- `supabase/` - Supabase client and database migrations

**`src/shared/`** - Cross-cutting reusable code
- `components/` - Generic UI components (BaseLayout, SettingsLayout)
- `hooks/` - Generic hooks not tied to features
- `services/` - Shared API utilities
- `utils/` - Generic helper functions

### Feature Module Rules

1. **Public API via index.ts**: Features only expose what's necessary:
   - Data hooks (e.g., `useAuthSession`, `useProfile`)
   - Route definitions and paths (e.g., `authRoutes`, `AUTH_PATHS`)
   - Public components/widgets

2. **Private Implementation**: Internal components, services, and hooks should NOT be exported from `index.ts`

3. **Unidirectional Dependencies**:
   - Features can import from `@shared` (using path alias `@/*`)
   - `@shared` should not depend on feature internals
   - **Exception**: Layouts in `@shared` can use feature public APIs (e.g., `useAuthSession` from auth feature)

### Router Architecture

The app uses nested routing with protection layers:
- **Public routes**: Auth-related routes (`/auth/*`)
- **Protected routes**: Wrapped in `<ProtectedRoute>` component
  - Main app routes under `<BaseLayout>`
  - Settings routes under `<SettingsLayout>`

All features export route configurations consumed by `src/app/router.tsx`.

### Authentication Flow

- Uses Supabase authentication
- `AuthProvider` wraps the app in `src/app/providers.tsx`
- `ProtectedRoute` component guards authenticated routes
- Auth hooks: `useAuthSession`, `useProfile` from `features/auth`

### Supabase Integration

- Client configured in `src/lib/supabase/supabase-client.ts`
- Requires environment variables:
  - `VITE_APP_SUPABASE_URL`
  - `VITE_APP_SUPABASE_ANON_KEY`
  - `VITE_APP_SUPABASE_DELETE_ACCOUNT_URL`
- Database migrations located in `src/lib/supabase/migrations/`
- Type-safe database types via `database.types.ts`

### Stripe Integration

- Configured in `src/lib/stripe/`
- Requires `VITE_APP_STRIPE_PUBLISHABLE_KEY` environment variable
- Uses @stripe/stripe-js and @stripe/react-stripe-js

## Environment Configuration

Create `.env` file in project root with variables from `.env.development` or `.env.production`:
- Supabase credentials (URL, anon key, delete account URL)
- Stripe publishable key
- Locale settings for date-time and currency

## Path Aliases

TypeScript and Vite configured with `@/*` alias pointing to `./src/*`
- Import example: `import { supabaseClient } from "@/lib/supabase"`

## Technology Stack

- **React 18** with TypeScript
- **Vite** - Build tool and dev server
- **React Router Dom v6** - Routing
- **Chakra UI** - Component library with custom theme
- **React Query (TanStack Query)** - Server state management
- **Axios** - HTTP client
- **Supabase** - Backend and authentication
- **Stripe** - Payment processing
- **React Hook Form + Yup** - Form handling and validation
- **Vitest** - Unit testing (configured in vite.config.js)
- **Husky + ESLint + Prettier + Commitlint** - Code quality tooling

## Key Implementation Patterns

### Creating New Features

Use Plop generator: `npm run plop`
- Generates feature structure in `src/features/{{kebabCase name}}/`
- Follow existing feature structure (auth, home, settings as reference)
- Export only public API through `index.ts`

### React Query Configuration

Default configuration disables refetch on window focus. Located in `src/lib/react-query/query-client.tsx`.

### Component Development

- Use Chakra UI components with custom theme from `src/lib/chakra-ui/`
- Shared layouts: `BaseLayout` for main app, `SettingsLayout` for settings
- Feature-specific components stay within feature directory

### Routing

- Define paths in feature's `router/paths.ts`
- Export routes from feature's `router/router.tsx`
- Import and compose in `src/app/router.tsx`