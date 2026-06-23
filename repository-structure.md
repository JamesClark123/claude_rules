# Repository Structure

All source code in this monorepo lives under `src/`. The first level
under `src/` is restricted to the following six canonical directories.
Adding a seventh top-level category requires a constitution amendment.

| Category        | Path              | Contains                                                    |
|-----------------|-------------------|-------------------------------------------------------------|
| Apps            | `src/apps/`       | Deployable frontends — anything an end user sees (websites, web apps, admin UIs). |
| Services        | `src/services/`   | Deployable backends — anything that exposes a reachable API, internal or external. |
| Libraries       | `src/libs/`       | Shared, framework-agnostic TypeScript code with no UI and no DB I/O. |
| Repositories    | `src/repositories/` | All database-touching code: schemas, migrations, query layers, ORMs. |
| Components      | `src/components/` | Reusable UI components (presentational building blocks).    |
| Features        | `src/features/`   | Pre-built, reusable end-to-end feature slices that an app can drop in to gain a complete capability (composing components, repositories, and services). |

**Package location rule**: Every workspace package MUST live exactly one
level deep inside one of the six categories — `src/<category>/<name>/`.
Packages MUST NOT live directly under `src/` (depth-one), and MUST NOT
nest further (depth-three) without an amendment justifying the
exception.

**Cross-category dependency direction** (recommended, not yet
mandatory): `apps` → `features` → (`components`, `services`,
`repositories`) → `libs`. `libs` MUST NOT import from any other
category. Cycles between categories MUST be avoided; reviewers SHOULD
flag them.

**Workspace glob**: This taxonomy implies a pnpm workspace pattern of
`src/*/*` (each direct child of a category directory is a workspace
package). The `pnpm-workspace.yaml` MUST reflect that pattern.

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
