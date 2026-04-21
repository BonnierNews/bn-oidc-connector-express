# AGENTS.md

Centralized guidelines for AI coding agents working in this repository.

## Project Overview

A **reusable TypeScript/Express middleware library** for consuming OIDC authentication headers set by Bonnier News Fastly Compute. It parses the `x-bnlogin-user` header, validates and decodes JWT claims, and attaches them to the Express request object via `req.oidc`.

Published as `@bonniernews/bn-oidc-connector-express` — a dual ESM/CJS package.

## AGENTS.md maintenance

- **Scope:** put content in the narrowest file where it applies. Only create a new AGENTS.md when a subdirectory has enough distinct content to warrant it.
- **When to update:** when the file is no longer accurate — a rule changed, structure moved, a new cross-cutting decision landed. Not for routine bug fixes or feature work.
- **Size:** keep under ~200 lines — they load into context every session.

## Key Commands

```bash
# Run all tests (node built-in test runner)
npm test

# Run tests with coverage
npm run coverage

# Type check
npm run typecheck

# Lint (ESLint + typecheck)
npm run lint

# Auto-fix lint issues
npm run format

# Build the library (tsup, ESM + CJS)
npm run build
```

---

## Architecture Overview

### Public API

Entry point: `index.ts` re-exports:
- `auth(options?)` — Express Router factory; mounts `oidcContext` and `idToken` middleware
- `isAuthenticated` — middleware that throws `UnauthenticatedError` if not logged in
- `isEntitled` — middleware for entitlement checks
- `AuthOptions` — TypeScript type for options

### Core Modules

- **`lib/auth.ts`** — `auth()` factory; wires up middleware chain
- **`lib/middleware/oidc-context.ts`** — attaches `req.oidc` stub to every request
- **`lib/middleware/id-token.ts`** — reads the user header, decodes JWT, populates `req.oidc`
- **`lib/middleware/is-authenticated.ts`** — guards requiring `req.oidc.isAuthenticated`
- **`lib/middleware/is-entitled.ts`** — guards requiring specific entitlements in `req.oidc`
- **`lib/utils/claims.ts`** — claim extraction helpers
- **`lib/errors.ts`** — custom error classes (`UnauthenticatedError`, etc.)
- **`lib/types/index.ts`** — shared TypeScript types

### Key Design Pattern

The library consumes headers injected by the upstream Fastly OIDC edge layer — it does **not** implement OAuth flows itself. The primary header is `x-bnlogin-user` (configurable via `AuthOptions.headers.user`).

---

## Testing Approach

Tests use **Node.js built-in test runner** (`node --test`) with BDD-style syntax from `@bonniernews/node-test-bdd` (Feature/Scenario/Given/When/Then globals).

- **`test/feature/`** — integration-style feature tests using `supertest` against a real Express app
- **`test/unit/`** — isolated unit tests for utilities and helpers
- **`test/helpers/`** — shared test setup (app factory, cookie helpers)
- **`test/setup.ts`** — global test bootstrap (imported via `--import`)

Assertions use **chai** (`expect`). HTTP assertions use **supertest**. External HTTP is mocked with **nock**.

---

## Coding Conventions

### Do

- Write TypeScript — all source files in `lib/` and `index.ts`
- Use ESM syntax (`import`/`export`) — the package is `"type": "module"`
- Use guard clauses and early returns over nested conditionals
- Add new dependencies via `npm install` — keep `package-lock.json` in the same commit
- Use `supertest` + a real Express app for feature tests, not mocked middleware chains
- Write BDD feature tests for middleware behaviour, unit tests for pure functions

### Don't

- Never read, cat, or access `.env` files — they contain secrets
- Never commit directly to `main` — always work on a feature branch
- Don't create abstractions preemptively — write inline code first, abstract only when there's demonstrated need
- Don't add helpers or utilities for one-time operations
- Don't over-engineer — make the minimal change needed to achieve the goal
- Don't add features, refactoring, or "improvements" beyond what was asked
- Don't introduce deeply nested if/else chains
- Don't skip writing tests — every change should have corresponding test coverage
- Don't make changes without understanding the existing code first
- Don't introduce dead code — remove unused imports, types, and functions
- Don't consider a task complete without running `npm test` and confirming tests pass
