# Copilot / AI agent instructions for vivasan-backend

This repository is a Medusa v2-based backend. The notes below capture the minimal, actionable knowledge an AI coding assistant needs to be productive.

- **Project type:** Medusa backend (see `package.json` dependencies: `@medusajs/medusa`, `@medusajs/framework`).
- **Node / package manager:** `node` engine set to `22` and `pnpm@9.10.0` (see `package.json`).
- **Primary scripts:**
  - `pnpm dev` -> `medusa develop` (local development)
  - `pnpm start` -> `medusa start` (production start)
  - `pnpm build` -> `medusa build`
  - `pnpm seed` -> runs `src/scripts/seed.ts`
  - Tests: `test:integration:http`, `test:integration:modules`, `test:unit` (these set `TEST_TYPE` and run `jest` with `--experimental-vm-modules`).

Architecture & conventions (what to know)
- Medusa framework + plugins: configuration is in `medusa-config.ts`. Key env vars used:
  - `DATABASE_URL`, `REDIS_URL`, `WORKER_MODE`, `BACKEND_URL`, `STORE_CORS`, `ADMIN_CORS`, `AUTH_CORS`, `JWT_SECRET`, `COOKIE_SECRET`.
  - Modules enabled: `cache-redis`, `event-bus-redis`, `workflow-engine-redis` (these expect `REDIS_URL`).
- HTTP endpoints: custom route files live under `src/api/{admin,store}/custom/route.ts` and export HTTP-method named handlers (e.g. `export async function GET(req, res)`). Use `MedusaRequest`/`MedusaResponse` from `@medusajs/framework/http` — follow existing pattern in `src/api/*/custom/route.ts`.
- Admin frontend files exist under `src/admin/` (vite + TS). Be careful when editing admin code to keep `vite`/`tsconfig` expectations.
- Scripts and seeds: `src/scripts/seed.ts` is the canonical seeding script referenced by `package.json`.

Testing & CI
- Integration tests use `@medusajs/test-utils` and the `medusaIntegrationTestRunner` helper (see `integration-tests/http/health.spec.ts`). Tests set `TEST_TYPE` to choose suites.
- `integration-tests/setup.js` clears MikroORM `MetadataStorage` before tests. Don't remove this file — it's required to avoid metadata leakage between test runs.
- Jest runs with SWC (`@swc/jest`) and uses `--experimental-vm-modules` in package scripts. When adding tests, mirror the existing environment variable approach (`TEST_TYPE`).

Patterns & pitfalls
- Request handlers export named functions per HTTP verb (e.g., `GET`) rather than express-style router definitions. When adding routes, follow that module signature so the Medusa framework picks them up.
- Configuration is environment-driven. Prefer reading/adding keys in `medusa-config.ts` rather than hardcoding secrets. If adding a new plugin, register it in `modules` in `medusa-config.ts` and surface options from env vars.
- Background workers: `workerMode` is read from `WORKER_MODE` and can be `shared`, `worker`, or `server`. When changing background processing or event-bus code, validate `WORKER_MODE` behavior.

Files to inspect for examples
- `medusa-config.ts` — environment-driven configuration and enabled modules.
- `src/api/admin/custom/route.ts` and `src/api/store/custom/route.ts` — canonical custom endpoints.
- `src/scripts/seed.ts` — seeding pattern (script is invoked by `pnpm seed`).
- `integration-tests/*` — integration test structure and `medusaIntegrationTestRunner` usage.

How to make safe changes (quick checklist for AI edits)
- Prefer additive changes: add new route files or new modules rather than refactoring core behavior.
- When editing config, add matching `process.env` keys and default values only when safe.
- If touching tests, run the relevant `npm`/`pnpm` script locally (see scripts above). Use `--runInBand` for deterministic behavior as in existing `package.json` scripts.

If unsure, ask the maintainer these quick questions:
- Which environment (postgres/sqlite) should new migrations target? Check `DATABASE_URL` in the environment.
- Is the project running in `worker` mode in your deployment? (affects event bus / workflows)

End of file — open an issue or ask for clarification if anything here is unclear.
