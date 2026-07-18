# Repository Structure

All source code in this monorepo lives under `src/`. The first level
under `src/` is restricted to the following eight canonical directories.
Adding a ninth top-level category requires a constitution amendment.

| Category        | Path              | Contains                                                    |
|-----------------|-------------------|-------------------------------------------------------------|
| Apps            | `src/apps/`       | Deployable frontends — anything an end user sees (websites, web apps, admin UIs). |
| Services        | `src/services/`   | Deployable backends — anything that exposes a reachable API, internal or external. |
| Libraries       | `src/libs/`       | Shared, framework-agnostic TypeScript code with no UI and no DB I/O. |
| Repositories    | `src/repositories/` | All database-touching code: schemas, migrations, query layers, ORMs. |
| Components      | `src/components/` | Reusable UI components (presentational building blocks).    |
| Features        | `src/features/`   | Pre-built, reusable end-to-end feature slices that an app can drop in to gain a complete capability (composing components, repositories, and services). |
| Infra           | `src/infra/`      | Reusable infrastructure code (Terraform modules, CDK constructs, shared cloud configuration) consumed by apps and services. No application logic. |
| Styles          | `src/styles/`     | Shared frontend styling tooling and configuration (e.g., Tailwind config, design tokens, shared PostCSS/CSS setup) consumed by component libraries, features, and apps. No component markup or application logic. |

**Package location rule**: Every workspace package MUST live exactly one
level deep inside one of the eight categories — `src/<category>/<name>/`.
Packages MUST NOT live directly under `src/` (depth-one), and MUST NOT
nest further (depth-three) without an amendment justifying the
exception.

**Category-level package exception — `features`**: The `features`
category is exempt from the depth rule above: it is itself a single
workspace package (`@tms/features`) rooted at `src/features/`, rather
than a directory of `src/features/<name>/` packages. Each individual
feature is a **subpath export** of that one package
(`@tms/features/<feature>`), with its source under
`src/features/src/<feature>/` (e.g. `src/features/src/auth/` →
`@tms/features/auth`). This keeps the entire feature library installable
and versioned as one unit while still letting apps import one feature at
a time. No other category may adopt this category-level-package shape
without its own amendment; the default for every other category remains
`src/<category>/<name>/`.

**Cross-category dependency direction** (recommended, not yet
mandatory): `apps` → `features` → (`components`, `services`,
`repositories`) → `libs`. `components` and `features` MAY also depend
on `styles`. `libs` and `styles` MUST NOT import from any other
category. Cycles between categories MUST be avoided; reviewers SHOULD
flag them.

**Workspace glob**: This taxonomy implies a pnpm workspace pattern of
`src/*/*` (each direct child of a category directory is a workspace
package), plus the depth-one `src/features` entry for the category-level
`@tms/features` package described in the exception above. The
`pnpm-workspace.yaml` MUST reflect both patterns:

```yaml
packages:
  - "src/*/*"
  - "src/features"
```

**E2E packages**: As specified in Rule VI, an E2E package mirrors
its target inside the same category — `src/apps/<app>-e2e/` for a
frontend, `src/services/<svc>-e2e/` for a backend. E2E packages do NOT
warrant their own top-level category.

**Boundary heuristics** (resolve "where does this go?" questions):

- Touches the database directly (raw SQL, Prisma client, Mongo driver,
  etc.)? → `repositories/`.
- Pure logic, no UI, no DB, no HTTP server? → `libs/`.
- A reusable button, form, modal, or layout primitive? → `components/`.
- A drop-in capability that wires UI + data + actions together (e.g.,
  "user onboarding flow", "billing portal")? → `features/`.
- An HTTP/RPC server or worker that gets deployed? → `services/`.
- A user-facing app shell that gets deployed? → `apps/`.
- A reusable Terraform module, CDK construct, or shared cloud configuration
  consumed by multiple apps or services? → `infra/`. Package-specific
  Terraform roots (used by exactly one app or service) belong in that
  package's own `infra/` subdirectory, NOT here.
- Frontend styling tooling or configuration (a Tailwind config, a design
  token set, a shared PostCSS setup) meant to be consumed by multiple
  component libraries, features, or apps? → `styles/`. Package-specific
  styling that only one app or component uses stays inside that
  package, NOT here.
