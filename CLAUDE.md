# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

Cadence: a course platform (mini Udemy) on React Router 8 (SSR, framework mode), TypeScript, SQLite via Drizzle, Tailwind 4 + shadcn/ui. It is the exercise repo for the AI Coding Crash Course.

## Commands

- `npm run dev` — dev server at http://localhost:5173
- `npm run typecheck` — runs `react-router typegen` first, so route `+types` are regenerated
- `npm test` — all Vitest tests; one file: `npx vitest run app/services/couponService.test.ts`; one test: add `-t "<name>"`
- `npm run db:migrate`, `npm run db:seed` — apply migrations to `data.db`, reseed it

There is no lint script; Prettier config lives in `.prettierrc`.

## Architecture

Layers, top to bottom:

- **Routes** (`app/routes/`, registered explicitly in `app/routes.ts`): each module owns its `loader`/`action`, validates params and form data with zod via `parseParams`/`parseFormData` (`app/lib/validation.ts`), and dispatches multi-action forms on a discriminated-union `intent` field. Routes under `layout.app.tsx` share the sidebar and the dev UI.
- **Services** (`app/services/*Service.ts`): all DB access. Plain synchronous functions over better-sqlite3 (`.get()`, `.all()`, `.run()`), positional parameters (project convention).
- **DB** (`app/db/`): `schema.ts` holds tables plus TypeScript enums (`UserRole`, `CourseStatus`, …). `app/db/index.ts` opens the connection lazily behind a Proxy so the module stays side-effect free — that is what keeps better-sqlite3 out of the client bundle when a route imports a service. Keep it side-effect free.

Auth is a dev-only cookie session (`app/lib/session.ts`) holding a `userId`; there are no passwords. The dev UI switches users via `api/switch-user` and fakes the visitor's country via `api/set-dev-country` (drives PPP pricing in `app/lib/ppp.ts`). Per-course authorization goes through `getAccess` / `require*` in `app/lib/access.server.ts` — the single place that answers who may see or edit a course.

`request-logger/` and `.sandcastle/` are course tooling, not app code; `request-logger/CLAUDE.md` applies there.
