# Rule VI: Multi-Level Testing Discipline (NON-NEGOTIABLE)

Automated testing is mandatory. Every feature MUST be covered at the
appropriate layers, and the codebase as a whole MUST maintain test suites
at all three of the following levels:

**1. Unit tests** — exercise a single function, module, or component in
isolation.

- Functional code: Vitest.
- Visual code (UI components): Storybook, executed in test mode (e.g., the
  Vitest Storybook addon for headless runs). Each component story MUST
  cover all five visual-testing modes:
  - **Snapshot** — guards serialized output against unintended structural
    change.
  - **Visual regression** — guards rendered pixels against unintended
    visual change.
  - **Accessibility** — guards against WCAG / axe violations.
  - **Interaction** — guards user-driven behaviors (clicks, keyboard,
    focus) at the component level.
  - **Render** — guards that the component mounts under representative
    prop and state combinations without error.

**2. Integration tests** — exercise the seam between two or more units, or
between application code and its dependencies.

- Functional code: Vitest.
- Visual code: Storybook, **interaction mode only**. The other four visual
  modes (snapshot, visual regression, accessibility, render) belong at the
  unit layer and MUST NOT be duplicated at the integration layer.
- All HTTP / network boundaries crossed during integration tests MUST be
  mocked with MSW (`msw`). Ad-hoc `fetch`/SDK mocks that bypass MSW are
  NOT permitted; this keeps the integration tier deterministic and
  guarantees a single source of truth for request/response fixtures.

**3. End-to-end (E2E) tests** — drive a deployed application as a user
would. The tool differs by package category:

- **Apps** (`src/apps/`): Tool is **Playwright**. A different framework MAY
  be used only when Playwright cannot reasonably target the platform (e.g.,
  a native runtime it does not support); deviations MUST be justified in the
  package's README and in the PR introducing them.
- **Services** (`src/services/`): Tool is **supertest**. Because services
  expose HTTP/RPC APIs rather than a user-facing UI, supertest drives the
  deployed service's HTTP surface directly. Playwright MUST NOT be
  introduced for service E2E tests. E2E testing is only required for
  services that expose an HTTP/RPC API surface — services whose sole
  external interface is a non-HTTP protocol (e.g., a raw PostgreSQL
  database, a message broker) are exempt and MUST NOT be given a
  `<svc>-e2e` package. The exemption MUST be documented in the service's
  README.

- E2E tests live in a dedicated workspace package per target, named
  `<target>-e2e` and located as a sibling of the target inside the same
  category directory (see Repository Structure rule). For a frontend app at
  `src/apps/<app>/`, the E2E package is `src/apps/<app>-e2e/`. For a
  service at `src/services/<svc>/`, the E2E package is
  `src/services/<svc>-e2e/`. E2E tests MUST NOT be colocated inside the
  application package and MUST NOT live outside their target's category.
- Both Playwright configs (apps) and supertest suites (services) MUST
  accept a runtime target via the `E2E_TARGET` environment variable (or a
  documented equivalent), selecting between (a) a locally-built Docker image
  of the target and (b) a remote deployment URL. The same test suite MUST
  run unchanged against both targets.
- CI MUST run the E2E suite against the local Docker build on every PR.
  Running against remote (staging / production) targets is permitted on
  demand and via scheduled jobs but MUST NOT replace the local-Docker
  PR run.

**4. Stress tests** — validate that a service sustains expected throughput
and latency under load.

- Tool: **artillery** (`artillery` npm package). Alternative load-testing
  tools MUST NOT be introduced without a constitution amendment.
- Stress tests apply only to services that expose an HTTP/RPC API (the
  same set that requires E2E testing). App packages (`src/apps/`) and all
  other categories are exempt, as are services whose sole external interface
  is a non-HTTP protocol.
- Stress test scenarios (artillery YAML config files) MUST live inside the
  service's `<svc>-e2e` package alongside the supertest E2E suite — NOT
  inside the service package itself.
- The artillery config MUST read `E2E_TARGET` (or the same documented
  equivalent used by the supertest suite) to point at the correct base URL,
  so scenarios run unchanged against both the local Docker build and remote
  deployments.
- Stress tests are exposed via a `test:stress` script in the `<svc>-e2e`
  package. They MUST NOT be run on every PR (execution time and infra cost
  make them unsuitable as a commit gate); instead they MUST be runnable on
  demand and MAY be scheduled (e.g., nightly against staging).
- Stress tests are intentionally OUT of scope for the 90% coverage
  threshold. They are evaluated by latency percentiles and error-rate
  targets documented in the artillery config, not by code coverage.

**Visual-testing layer matrix** (Storybook):

| Mode              | Unit stage | Integration stage |
|-------------------|:----------:|:-----------------:|
| Snapshot          | ✅         | —                 |
| Visual regression | ✅         | —                 |
| Accessibility     | ✅         | —                 |
| Interaction       | ✅         | ✅                |
| Render            | ✅         | —                 |

**Mocking discipline**: Unit and integration tests MUST NOT make real
network requests. Use MSW for HTTP boundaries; for non-HTTP boundaries
(databases, queues), document the test pattern in the package's README.
E2E tests, by contrast, MAY (and typically WILL) hit real services —
that is the layer's purpose.

**Coverage threshold** (NON-NEGOTIABLE):

- Every package's Vitest unit suite (`test:unit`) AND Vitest integration
  suite (`test:integration`) MUST achieve **at least 90%** coverage on
  each of the four standard metrics: **lines, branches, functions, and
  statements**. A package whose suites collectively fall below 90% on
  any one metric MUST fail CI.
- The coverage provider MUST be `@vitest/coverage-v8`. Alternative
  providers (`@vitest/coverage-istanbul`, etc.) MUST NOT be introduced
  without a constitution amendment.
- Each package's `vitest.config.ts` MUST configure coverage explicitly:
  ```ts
  coverage: {
    provider: 'v8',
    thresholds: { lines: 90, branches: 90, functions: 90, statements: 90 },
  }
  ```
  The 90 values are floors, not ceilings; packages that have already
  surpassed them SHOULD raise their per-package thresholds to lock in
  the higher bar.
- The threshold applies per package, not as a workspace-wide average.
  A high-coverage package MUST NOT compensate for a low-coverage one.
- Generated code, type-only files (`*.d.ts`), build outputs (`dist/`),
  Storybook configuration (`*.stories.{ts,tsx}` are tested via the
  visual-mode pipeline, not coverage-counted twice), and E2E packages
  (`*-e2e/`) MAY be excluded from coverage measurement; exclusions MUST
  be declared in the per-package `coverage.exclude` array and MUST be
  narrow.
- Reviewers MUST examine the coverage **delta** on a PR, not just the
  total. Adding untested code is not made acceptable by an inherited
  cushion above 90%; PRs that lower coverage on touched files SHOULD be
  sent back even when the package as a whole still passes.
- E2E (Playwright) coverage is intentionally OUT of scope for this
  threshold. E2E suites are evaluated by behavior, not by coverage
  percentage.

**Rationale**: The three layers each catch a different class of bug.
Units catch logic regressions cheaply and run in seconds; integration
tests catch wiring mistakes between components and their collaborators;
E2E catches environment- and contract-level breakage that only surfaces
in a deployed system. Standardizing on Vitest + Storybook + MSW +
Playwright keeps onboarding cheap and prevents each package from
inventing its own testing stack. The 90% coverage floor — measured by
`@vitest/coverage-v8` — turns "we have tests" into a verifiable
property, prevents silent erosion of the test suite as the codebase
grows, and makes coverage regressions visible at PR time rather than
quarter-end.
