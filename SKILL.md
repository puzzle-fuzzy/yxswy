---
name: yxswy
description: Apply the user's personal TypeScript full-stack engineering style. Use when creating, editing, reviewing, or explaining TypeScript/JavaScript projects, especially Bun or pnpm monorepos, React/Vite frontends, Elysia or Hono APIs, Drizzle/Zod domain code, deployment tools, Electron apps, or Tauri/Rust desktop shells for this user.
---

# yxswy

## Overview

Use this skill to align implementation choices with the user's existing coding habits. Prefer local project conventions first; use these preferences when starting new code, filling gaps, or choosing between reasonable options.

The user is a TypeScript full-stack engineer. They sometimes use Tauri/Rust and Electron; treat Rust as familiar but not the user's deepest area unless the repo clearly asks for it.

## Core Defaults

- Inspect the repository before changing code. Identify package manager, workspace layout, scripts, linting, typecheck, test runner, and naming conventions.
- Prefer the repo's current package manager. For greenfield personal projects, Bun and pnpm are both natural; Bun is common for newer service-heavy work, pnpm is common in larger workspace setups.
- Prefer monorepos with `apps/*`, `services/*`, and `packages/*` when the product has multiple surfaces. If there is only one frontend app and one backend API, keep both under `apps/*` (for example `apps/web` and `apps/api`). Use `services/*` when there are multiple backend services or multiple frontend apps and the backend should stand apart.
- Keep shared contracts in packages such as `shared`, `db`, `api-client`, `config`, `runtime`, `storage`, or domain-specific packages.
- Favor explicit boundaries. If a package boundary checker exists, keep imports within it and update the checker when introducing a legitimate new boundary.
- Preserve shared engineering config packages when present, such as workspace `eslint-config`, `tsconfig`, Tailwind config, Biome, oxlint, lint-staged, Husky, or Turbo pipeline settings.
- Use TypeScript types as architecture. Derive runtime-facing types from schemas where practical, and avoid `any` unless bridging untyped external input.
- Prefer pure domain or policy packages for reusable rules. Keep them free of DB, provider, storage, app, and other IO imports; inject adapters at the app/service layer.

## Backend Style

- Use Elysia or Hono naturally for APIs; preserve whichever the project already uses.
- Preserve end-to-end TypeScript contracts between frontend and backend. For Elysia APIs, prefer `@elysia/eden`; for Hono APIs, prefer `hono/client` or the repo's equivalent typed client.
- Prefer app factories for testability. Avoid top-level `listen` or provider startup in modules that tests import.
- Register middleware intentionally: OpenAPI/docs, logging, request ID, security headers, rate limit, CORS, static assets, error handling, auth, then business routes, unless the repo has a different established order.
- Prefer services with dependency-injected repositories/adapters over direct hard-coded infrastructure calls.
- Keep repositories responsible for persistence and state transitions. Keep services responsible for permissions, auth, orchestration, validation, and business flow.
- Model domain errors explicitly with stable error codes/classes or discriminated results. Return typed service errors where the existing project does that; throw only for truly exceptional or infrastructure-level failures.
- Put auth, membership, and permission checks close to service entry points.
- Fail fast for missing or unsafe production environment variables. In development, warnings are acceptable when the project already uses that style.
- Add security headers, strict CORS, request IDs, access logs, and metrics around public APIs when building production-facing services.
- Use Drizzle for Postgres schemas and queries when already present. Keep enum/string literal sources of truth shared with API/frontend types when feasible.
- For database-backed products, prefer soft deletion, audit columns, partial unique indexes, idempotency keys, explicit status fields, and indexes matching the main query paths.
- Use Zod, Elysia `t`, or existing schema tooling for request/response/domain validation. Derive types from schemas instead of duplicating shapes.
- Prefer small pure helpers for normalization, slug validation, retry policy, path rendering, cache policy, and similar domain rules.
- Make time, randomness, storage roots, sessions, providers, and external clients injectable in testable code.

## Frontend Style

- Choose frontend framework by project complexity. Use Vue 3 + Vite for simpler projects, admin/H5 surfaces, and straightforward CRUD or interaction flows. Use React + Vite for more complex products because its ecosystem is more complete for rich state, UI composition, dashboards, typed clients, and advanced product workflows.
- Use Tailwind, shadcn-style components, Radix/Base UI patterns, and lucide-react icons when present.
- Build practical product UI first: dashboards, consoles, deploy tools, upload flows, admin views, and dense operational screens should be compact, direct, and usable.
- Prefer feature folders and hooks for client behavior: `features/*`, `shared/*`, `components/ui/*`, `pages/*`, or the local equivalent.
- Use typed API clients and React Query/Zustand only when the project has already introduced them or the workflow clearly benefits.
- For H5, WeChat, screen, or event-interaction projects, isolate SDK/auth/pay/websocket/device-detection logic into packages or composables/hooks instead of burying it inside pages.
- Include loading, empty, error, read-only, permission, and pending states for user-facing workflows.
- Preserve i18n/theme support if present. Do not introduce visible instructional copy unless the UI needs it.
- For UI changes, verify responsive behavior and text overflow in compact layouts.

## Workers And Providers

- For async work, prefer explicit task records with `type`, `domain`, `status`, `priority`, `attempts`, `maxAttempts`, `nextRunAt`, `lockedBy`, `lockedUntil`, and structured input/output/error JSON when persistence is involved.
- Use claim/heartbeat/sweep/cancel semantics for workers. Recover queued or non-terminal tasks on startup when the product needs durability.
- Route provider calls through registries/runners instead of scattering provider-specific logic through services.
- Add provider health/degradation policy when model calls can fail repeatedly. Keep the policy pure and let DB/app layers persist or enforce it.
- Use SSE, database notifications, or an event bus for live status updates when a task lifecycle is visible to the frontend.

## Production Readiness

- Treat configuration as code. Prefer typed env packages or config modules with client/server separation, production fail-fast checks, safe development defaults, and no secret leakage into frontend bundles.
- Keep database migrations explicit. Separate `generate`, `migrate`, `push`, `studio`, and test DB scripts; do not run destructive migrations or reset data without explicit user approval.
- Design auth for revocation and auditing. Prefer httpOnly cookies or scoped API keys, hash stored secrets, track last-used metadata, support logout/revocation, and enforce quota/rate-limit where external usage can cost money.
- Make public APIs observable. Include request IDs or trace IDs, structured logs, health endpoints, useful metrics, and consistent error bodies with stable codes.
- Keep upload, file, and static-serving paths hardened: sanitize path components, cap request body size, set MIME/content type intentionally, and isolate storage behind adapters.
- Keep OpenAPI/docs useful but avoid exposing internal route/schema details in production unless intentionally configured.
- Add a top-level `verify` or equivalent script when a repo grows: boundary checks, typecheck, lint, unit tests, and focused integration tests should be easy to run together.
- Favor graceful degradation over silent failure for external systems: retries with backoff, circuit/degradation state, user-facing recovery guidance, and diagnostics that can be copied into support/debug flows.

## Desktop Style

- For Electron, keep `main`, `preload`, and `renderer` separated. Use `contextBridge`, typed window bridges, `contextIsolation: true`, and IPC methods that mirror the typed client surface.
- Keep native-only capabilities in an explicit bridge namespace, such as file picking, upload-by-path, notifications, auth expiration, server configuration, and external links.
- Handle lifecycle details carefully: single-instance lock, close-to-tray, dev-server retry, parent-process death, and broken pipe guards where relevant.
- For Tauri, do not overbuild Rust business logic unless the repo already does. It is acceptable to keep Rust as a thin shell and put most product logic in TypeScript.
- When adding Rust, keep commands small, typed, and easy to audit.

## Testing And Validation

- Add focused tests for behavior changes, especially service permissions, state transitions, repository edge cases, upload/deploy flows, retry/backoff, and shared domain rules.
- Prefer the existing runner: `bun test`, Vitest, Playwright, or repo scripts. Do not invent a separate test stack.
- For monorepos, run the narrow package test first, then `typecheck`, `lint`, or the repo's `verify`/`check` script if available.
- Keep type-only tests (`*.test-d.ts`) when the contract itself is the behavior.
- Preserve and extend existing helpers/fixtures instead of creating parallel test harnesses.

## Code Shape

- Match local formatting: semicolons, quote style, trailing commas, and comment language vary across the user's projects.
- Use `unknown` plus narrowing at boundaries; use `as const`, `satisfies`, discriminated unions, and literal arrays for stable domain vocabularies.
- Prefer `undefined` for absent in-memory results and `null` where API/database shape already uses it.
- Keep comments for architectural invariants, state machines, safety constraints, and non-obvious lifecycle behavior. Detailed Chinese comments are natural when documenting design intent. Avoid narrating simple code.
- Do not collapse clear domain packages into one large utility module. The user's code tends to name concepts directly: `task-engine`, `workflow-engine`, `deploy-core`, `storage`, `runtime`, `api-client`, `repository`, `service`, `model`.
- Prefer deterministic IDs, slug normalization, safe path handling, explicit cache headers, and stable error response codes in deployment/storage features.

## New Project Bias

- Start with a small but real vertical slice rather than a hollow scaffold.
- If building a full-stack product, prefer:
  - `apps/web` or `apps/client` for the frontend.
  - `apps/api` or `apps/server` for the backend when there is only one backend service.
  - `services/api`, `services/worker`, or other `services/*` folders when there are multiple backend services or the workspace has multiple frontend apps.
  - `services/worker` when background work or provider polling exists.
  - `packages/shared` for schemas and pure domain types.
  - `packages/db` for Drizzle schema and database helpers.
  - `packages/api-client` for typed client access.
- Add scripts for `dev`, `build`, `typecheck`, `test`, and `lint` early.
- Use Docker Compose for local Postgres when the app needs persistence.

## Ask When Unclear

Ask the user when a choice changes product direction, package manager, framework, desktop shell technology, database, auth model, deployment target, or visual style. Otherwise, make a conservative choice consistent with the current repo.
