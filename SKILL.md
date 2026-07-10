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
- Prefer monorepos with `apps/*`, `services/*`, and `packages/*` when the product has multiple surfaces.
- Keep shared contracts in packages such as `shared`, `db`, `api-client`, `config`, `runtime`, `storage`, or domain-specific packages.
- Favor explicit boundaries. If a package boundary checker exists, keep imports within it and update the checker when introducing a legitimate new boundary.
- Use TypeScript types as architecture. Derive runtime-facing types from schemas where practical, and avoid `any` unless bridging untyped external input.

## Backend Style

- Use Elysia or Hono naturally for APIs; preserve whichever the project already uses.
- Prefer services with dependency-injected repositories/adapters over direct hard-coded infrastructure calls.
- Keep repositories responsible for persistence and state transitions. Keep services responsible for permissions, auth, orchestration, validation, and business flow.
- Model domain errors explicitly with stable error codes/classes or discriminated results. Return typed service errors where the existing project does that; throw only for truly exceptional or infrastructure-level failures.
- Put auth, membership, and permission checks close to service entry points.
- Use Drizzle for Postgres schemas and queries when already present. Keep enum/string literal sources of truth shared with API/frontend types when feasible.
- Use Zod, Elysia `t`, or existing schema tooling for request/response/domain validation. Derive types from schemas instead of duplicating shapes.
- Prefer small pure helpers for normalization, slug validation, retry policy, path rendering, cache policy, and similar domain rules.
- Make time, randomness, storage roots, sessions, providers, and external clients injectable in testable code.

## Frontend Style

- Use React + Vite as the default web stack unless the repo says otherwise.
- Use Tailwind, shadcn-style components, Radix/Base UI patterns, and lucide-react icons when present.
- Build practical product UI first: dashboards, consoles, deploy tools, upload flows, admin views, and dense operational screens should be compact, direct, and usable.
- Prefer feature folders and hooks for client behavior: `features/*`, `shared/*`, `components/ui/*`, `pages/*`, or the local equivalent.
- Use typed API clients and React Query/Zustand only when the project has already introduced them or the workflow clearly benefits.
- Include loading, empty, error, read-only, permission, and pending states for user-facing workflows.
- Preserve i18n/theme support if present. Do not introduce visible instructional copy unless the UI needs it.
- For UI changes, verify responsive behavior and text overflow in compact layouts.

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
- Keep comments for architectural invariants, state machines, safety constraints, and non-obvious lifecycle behavior. Avoid narrating simple code.
- Do not collapse clear domain packages into one large utility module. The user's code tends to name concepts directly: `task-engine`, `workflow-engine`, `deploy-core`, `storage`, `runtime`, `api-client`, `repository`, `service`, `model`.
- Prefer deterministic IDs, slug normalization, safe path handling, explicit cache headers, and stable error response codes in deployment/storage features.

## New Project Bias

- Start with a small but real vertical slice rather than a hollow scaffold.
- If building a full-stack product, prefer:
  - `apps/web` or `apps/client` for the frontend.
  - `services/api` or `apps/server` for the API.
  - `services/worker` when background work or provider polling exists.
  - `packages/shared` for schemas and pure domain types.
  - `packages/db` for Drizzle schema and database helpers.
  - `packages/api-client` for typed client access.
- Add scripts for `dev`, `build`, `typecheck`, `test`, and `lint` early.
- Use Docker Compose for local Postgres when the app needs persistence.

## Ask When Unclear

Ask the user when a choice changes product direction, package manager, framework, desktop shell technology, database, auth model, deployment target, or visual style. Otherwise, make a conservative choice consistent with the current repo.
