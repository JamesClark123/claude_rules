# Tooling Standards

The following tools are the canonical implementations of the project rules.
Substitutions require a constitution amendment (see `.specify/memory/constitution.md`
Governance section):

- **Package manager**: pnpm with workspaces. Lockfile (`pnpm-lock.yaml`) MUST
  be committed.
- **Formatter**: Biome, configured by a single `biome.json` at the
  workspace root. Per-package overrides are discouraged; use them only when
  a package's ecosystem demands it (e.g., a generated file pattern that
  must be excluded).
- **Linter**: Biome (same `biome.json`). The formatter and linter MUST be
  configured as a single Biome installation — separate `prettier` and
  `eslint` toolchains MUST NOT be reintroduced without an amendment.
- **Type checker**: TypeScript with `"strict": true`. Each package MUST
  have a `tsconfig.json` (it MAY extend a shared base).
- **Unit / integration test runner**: Vitest. A shared base config MAY
  live at the workspace root; per-package `vitest.config.ts` files MUST
  extend it rather than diverge.
- **Coverage tool**: `@vitest/coverage-v8`. Each package's
  `vitest.config.ts` MUST configure it with `provider: 'v8'` and the
  90% per-metric thresholds (lines, branches, functions, statements)
  required by Rule VI's Coverage threshold sub-section.
  Reintroducing `@vitest/coverage-istanbul` or any alternative provider
  requires an amendment.
- **Component workshop and visual testing**: Storybook, with stories
  colocated next to their components. The Storybook test addon (or
  equivalent Vitest integration) MUST run stories headlessly in CI to
  cover all five visual-testing modes specified in Rule VI.
- **HTTP mocking for tests**: MSW (`msw`). A shared `handlers/` module
  MAY be exposed from a workspace package (e.g., `@tms/test-fixtures`)
  for reuse across packages.
- **E2E framework**: Playwright. Each app's E2E package
  (`src/<category>/<app>-e2e/`, where `<category>` is `apps` or
  `services`) MUST contain its own `playwright.config.ts` honoring the
  `E2E_TARGET` contract.
- **Schema-validated environment variables**: **Zod**, used inside each
  package's `env.ts` to parse `process.env` (Rule VIII). The
  loader for `.env` files MUST be **`dotenv`** (or `dotenvx`) for
  Node/TypeScript runtimes, or the framework-native loader for
  Vite/Next.js apps. Alternative validation libraries (`yup`, `joi`,
  `valibot`, hand-rolled checks) MUST NOT be introduced without an
  amendment. Sharing Zod env schemas across packages via a workspace
  library (e.g., `@tms/env-shared`) is permitted and encouraged when
  the same variables genuinely repeat.
- **Container runtime**: Docker. The local stack is composed from a
  single repo-root `docker-compose.yml` that uses `include:`
  directives to merge per-package `docker-compose.yml` files. As
  mandated by Rule VII, every package under `src/apps/` and
  `src/services/` MUST publish a `Dockerfile` AND a per-package
  `docker-compose.yml` declaring its service, and the root file
  MUST `include:` it. Inline `services:`/`volumes:`/`networks:`
  blocks in the root file are not permitted. The same merged
  compose stack serves as both the local dev environment and the
  local-target E2E surface for Rule VI.
- **Git hook manager**: Husky, installed at the workspace root with hooks
  checked into `.husky/` and bootstrapped by the `prepare` script (see
  Rule V).
- **Editor configuration**: An `.editorconfig` at the repo root MUST
  define charset (utf-8), end-of-line (lf), final-newline (true), and
  indentation defaults consistent with Biome's configuration.
- **Line endings**: LF only. CRLF MUST NOT enter the repository.
