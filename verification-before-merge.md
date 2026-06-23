# Rule V: Verification Before Merge

The following checks MUST pass on every pull request, blocking merge on
failure:

1. `pnpm -r run format:check` (Biome formatter clean).
2. `pnpm -r run lint` (no Biome lint errors; warnings triaged).
3. `pnpm -r run typecheck` (strict TypeScript compilation).
4. `pnpm -r run test:unit` (unit tests, including all Storybook visual
   modes per Rule VI; coverage thresholds enforced — see Rule VI's
   Coverage threshold sub-section).
5. `pnpm -r run test:integration` (Vitest integration tests with MSW
   mocks, plus Storybook interaction tests; coverage thresholds enforced
   — see Rule VI's Coverage threshold sub-section).
6. `pnpm -r run test:e2e` (Playwright suite executed against the local
   Docker build of each app; see Rule VI for the `E2E_TARGET` contract).
7. `pnpm -r run env:check` (per-package environment-variable validator
   — see Rule VIII). It MUST assert, for every package that owns
   an `env.ts`: (a) the key-set declared in the Zod schema equals the
   key-set listed in the package's `.env.example`, (b) the
   `.env.example` file exists and is tracked by git, and (c) the
   package's `.env` file is NOT tracked by git. Any drift fails CI.

**In addition**, a Husky-managed git hook MUST run a fast subset of these
checks locally before changes are committed:

- The `husky` package MUST be installed at the workspace root and
  initialized via a `prepare` script (`"prepare": "husky"`) so that hooks
  install automatically after `pnpm install`.
- A `pre-commit` hook MUST execute `pnpm -r run format:check`,
  `pnpm -r run lint`, `pnpm -r run typecheck`, `pnpm -r run test:unit`,
  and `pnpm -r run env:check`. Integration and E2E tests are
  intentionally excluded from the pre-commit hook because their
  runtime makes them unsuitable for an interactive commit gate; CI is
  responsible for those layers (items 5–6 above). `env:check` is
  cheap and is included locally because env-schema drift is one of
  the most common causes of "works on my machine" startup failures.
- The hook MAY use a staged-files runner (e.g., `lint-staged`) to scope
  formatter and linter work to changed files, but typecheck, unit
  tests, and `env:check` MUST run against the full workspace to catch
  cross-package regressions.
- Bypassing the hook or CI (`--no-verify`, force-merge, skipping required
  checks) requires explicit approval recorded in the PR description.

The local hook does NOT replace CI; both layers are mandatory.

**Rationale**: Style/quality checks only protect the codebase if they are
executed automatically and consistently. CI is the merge gate, but the
Husky hook catches violations before a contributor pushes — saving a CI
round-trip and keeping the main branch green.
