# Rule VII: Containerized Deployment Surface

Every workspace package under `src/apps/` and `src/services/` MUST be
fully containerized. The repo as a whole MUST be bootable with a single
command.

**Per-package Dockerfile**:

- Each app and service MUST publish a `Dockerfile` at its package root
  (e.g., `src/apps/<app>/Dockerfile`, `src/services/<svc>/Dockerfile`).
  No app or service is exempt; "compose-only" packages are not
  permitted.
- The image MUST be self-contained: running it MUST start the
  application with no host-side setup other than environment variables
  documented in the package's README.
- Dockerfiles SHOULD use multi-stage builds for TypeScript packages
  (build stage runs `pnpm install` + `pnpm build`; runtime stage
  copies only the production artifacts and a pruned `node_modules`).
- Final images MUST NOT include the `.git` directory, dev-only
  dependencies, test fixtures, source maps for unrelated packages, or
  Storybook output. Use `.dockerignore` to enforce this at build time.
- Final images MUST NOT bake `.env` files into the image. Runtime
  configuration is supplied via the container runtime's environment
  (compose `environment:` / `env_file:`, Kubernetes Secrets, etc.) and
  parsed by the package's `env.ts` per Rule VIII.

**Per-package `docker-compose.yml`**:

- Each app under `src/apps/` and each service under `src/services/`
  MUST ship a `docker-compose.yml` at its package root (e.g.,
  `src/services/<svc>/docker-compose.yml`). Its `services:` block
  MUST contain exactly the entry (or entries) the package owns —
  the deployable itself plus any sidecars or workers the package
  alone is responsible for. Inline definitions in the repo-root
  compose file are NOT permitted (see "Repo-root `docker-compose.yml`"
  below).
- Supporting infrastructure (databases, message queues, MSW mock
  servers, object storage) MUST be declared in the per-package
  `docker-compose.yml` of the package that owns it. The natural
  owner is the package whose code defines the schema, contract, or
  client for that infrastructure (e.g., the `postgres` data store
  belongs in `src/repositories/main-postgres/docker-compose.yml`,
  NOT in the consuming service). When ownership is genuinely
  ambiguous, place the entry in the consuming app's or service's
  compose file and document the choice in that package's README.
- When a Dockerfile copies workspace-level files (lockfile,
  `pnpm-workspace.yaml`, sibling package manifests), the per-package
  compose file's `build.context` MUST resolve to the repo root via
  a relative path (e.g., `../../..` from a `src/<category>/<name>/`
  package). The `dockerfile:` field is given relative to that
  context.
- Volumes, networks, and other top-level compose resources owned by
  the package MAY be declared in its compose file; they are merged
  into the project at root-include time. Do NOT redeclare the same
  named volume in multiple per-package files.

**Repo-root `docker-compose.yml`** (the index):

- A single `docker-compose.yml` at the repository root is the
  canonical entry point for the local stack. It MUST contain only
  `include:` directives (and optional comments) — every per-package
  compose file MUST be referenced by its repo-relative path.
  Inline `services:`, `volumes:`, or `networks:` blocks in the root
  file are NOT permitted; if a service or resource is needed, it
  MUST live in the per-package compose file of the owning package.
- The `include:` list MUST cover every package under `src/apps/`
  and `src/services/`, plus every package that owns supporting
  infrastructure. Adding a new app, service, or infrastructure
  component to the monorepo REQUIRES (a) creating its per-package
  `docker-compose.yml` and (b) appending one `include:` line in
  the root file, in the same PR.
- Running `docker compose up` from the repo root MUST bring the
  entire merged stack online — every app and every service. No
  package may be silently omitted; if a package is intentionally
  not bootable in the default profile (e.g., a heavy worker), it
  MUST be declared in a named compose `profile` inside its
  per-package compose file, and that profile MUST be documented in
  the repo README.
- Inter-service dependencies MUST be expressed inside the
  per-package compose files via `depends_on` (with
  `condition: service_healthy` where a healthcheck exists).
  External orchestration scripts MUST NOT be required to bring the
  local stack up. Cross-package `depends_on` references work because
  `include:` merges all services into one compose project — service
  names from other packages are reachable by name in the merged
  graph.

**Health and readiness**:

- Each app and service SHOULD expose a documented health endpoint
  (e.g., `GET /healthz`) and the corresponding compose service entry
  SHOULD declare a matching `healthcheck`. Healthchecks unblock the
  E2E target contract in Rule VI and make startup ordering
  deterministic.

**E2E alignment**: The local Docker target referenced by Rule VI
(the `E2E_TARGET` contract) is THIS compose stack. CI's pre-merge E2E
job runs against `docker compose up`-equivalent infrastructure; a
developer running E2E locally uses the same stack.

**Rationale**: Constraining all deployable units to one container
runtime and one merged compose stack makes "boot the whole system"
a single command for every contributor — no per-app scripts, no
missing dependencies, no drift between what CI runs and what a
laptop runs. The per-package + root-`include:` layout keeps
ownership local (a package's compose entry, healthcheck, and
supporting infrastructure live next to its code) while the root
file remains a single discoverable bootstrap. It also keeps E2E
configuration trivial: the same merged compose stack is both the
local dev environment and the local E2E target.
