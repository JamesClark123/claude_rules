# Development Workflow

- Formatter, linter, type-check, and test scripts MUST be exposed with
  uniform names so they are callable workspace-wide via
  `pnpm -r run <script>`:
  - `format`, `format:check`, `lint`, `typecheck`.
  - `test:unit` (Vitest unit tests + Storybook visual modes; runs with
    `@vitest/coverage-v8` and the 90% thresholds from Rule VI).
  - `test:integration` (Vitest integration tests with MSW + Storybook
    interaction tests; runs with `@vitest/coverage-v8` and the 90%
    thresholds from Rule VI). Packages with no integration scope
    MAY expose a no-op script.
  - `test:e2e` (Playwright). Only the `<app>-e2e` packages need to
    implement this; other packages MAY omit it.
  - `test` SHOULD be defined as an alias running `test:unit` (the fast
    layer used by the Husky hook).
  - `env:check` (per-package environment-variable schema/example
    validator; see Rule VIII). Every package that ships an
    `env.ts` MUST implement this script. Packages with no env
    consumption MAY expose a no-op script so `pnpm -r run env:check`
    succeeds workspace-wide.
- E2E packages MUST be a sibling workspace package named `<app>-e2e`
  inside the same category as the target (e.g.,
  `src/apps/web-e2e/` for `src/apps/web/`,
  `src/services/demo-service-e2e/` for `src/services/demo-service/`).
  They MUST read `E2E_TARGET` (or a documented project-specific
  equivalent) to switch between local-Docker and remote-deployment
  targets without code changes.
- Generated, vendored, or third-party files (e.g., `dist/`, build outputs,
  `node_modules/`) MUST be excluded from formatting and linting via
  Biome's `files.includes` / `files.experimentalScannerIgnores` (or the
  equivalent ignore configuration in the active `biome.json` schema).
  The same files MUST be excluded from coverage measurement via
  `coverage.exclude` in `vitest.config.ts`.
- New packages added under `src/` MUST inherit the workspace-wide tooling
  configuration — including the `env.ts` + `.env.example` scaffold from
  Rule VIII if they read any environment variable. Packages that
  opt out of any check MUST document the reason in their README and
  link the exception from the constitution.
- Code review MUST verify that any inline suppressions (`biome-ignore`,
  `@ts-expect-error`, `any`) carry the required justification comment,
  and that no `process.env.<X>` reads exist outside `env.ts`. PRs that
  introduce unjustified suppressions or env-bypass reads MUST be sent
  back for revision.
