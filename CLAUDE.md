# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository layout

pnpm workspace monorepo (`pnpm@11.9.0`, workspaces `apps/*` and `packages/*`). Early scaffold stage: all three packages contain only generated boilerplate.

- `apps/api` — NestJS 12 backend (package name `api`)
- `apps/web` — Angular 22 frontend (package name `web`)
- `packages/contracts` — `@expense/contracts`, shared DTOs and Zod schemas for both apps. ESM package built with `tsc -b` (composite) into `dist/`. Both apps depend on it via `"@expense/contracts": "workspace:*"`; build it before the consuming app.

`tsconfig.base.json` holds shared strict compiler options; `packages/contracts` extends it. The apps currently use their own standalone tsconfigs and do **not** extend the base.

## Commands

From the repo root (each runs recursively across workspaces that define the script):

```bash
pnpm install
pnpm build
pnpm test
pnpm lint        # only apps/api defines lint today
```

Target one package with `pnpm --filter <api|web|@expense/contracts> <script>`.

### API (`apps/api`)

```bash
pnpm --filter api start:dev                 # watch mode, port $PORT or 3000
pnpm --filter api test                      # vitest, *.spec.ts
pnpm --filter api test:e2e                  # vitest with vitest.config.e2e.ts, *.e2e-spec.ts under test/
pnpm --filter api lint                      # oxlint --type-aware (not ESLint)
pnpm --filter api format                    # prettier
pnpm --filter api exec vitest run src/app.controller.spec.ts   # single test file
pnpm --filter api exec vitest run -t "name of test"            # single test by name
```

### Web (`apps/web`)

```bash
pnpm --filter web start                     # ng serve, http://localhost:4200
pnpm --filter web build
pnpm --filter web test                      # ng test (Angular unit-test builder on Vitest + jsdom)
pnpm --filter web exec ng test --include src/app/app.spec.ts   # single spec
pnpm --filter web exec ng generate component <name>            # components default to SCSS
```

## API conventions

- ESM package (`"type": "module"`, `module: nodenext`): relative imports **must** use the `.js` extension (e.g. `import { AppService } from './app.service.js'`). `main.ts` uses top-level `await`.
- Vitest runs with `globals: true` (`vitest/globals` is in tsconfig `types`), so `describe`/`it`/`expect` need no imports. Path aliases from tsconfig are resolved via `resolve.tsconfigPaths`.
- oxlint enforces `typescript/no-floating-promises` as an error; `no-explicit-any` is off.
- `strictPropertyInitialization` is disabled (common for Nest DTOs/entities).

## Web conventions

- Standalone components with signals; app bootstrapped from `src/app/app.config.ts` (`provideRouter`, routes in `app.routes.ts`). Component selector prefix is `app`.
- Component files use the newer naming without a `.component` suffix (`app.ts`, `app.html`, `app.scss`, `app.spec.ts`).
- Production build budgets: initial bundle warns at 500kB / errors at 1MB; per-component styles warn at 4kB / error at 8kB.
