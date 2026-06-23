# Rule IV: Consistent Naming and File Layout

Naming MUST follow these conventions across all packages:

- Files and directories: `kebab-case` (e.g., `demo-service/`, `user-repo.ts`).
- TypeScript types, interfaces, classes, and React components: `PascalCase`.
- Variables, functions, and methods: `camelCase`.
- Constants exported at module top level and intended as immutable
  configuration: `UPPER_SNAKE_CASE`.
- Workspace package names: `@tms/<kebab-name>`. The on-disk location of
  each package is governed by the **Repository Structure** rule
  (`src/<category>/<name>/`); package names need not encode the category.

Each package MUST keep source under `src/` and emitted output under `dist/`.
Test files MUST sit next to the code they cover, located in a `__tests/`
subdirectory of that code's directory (e.g., `pkg/src/foo.ts` ↔
`pkg/src/__tests/foo.test.ts`). The `__tests/` directory mirrors the
filename of the code under test (`foo.ts` → `foo.test.ts`). Packages MAY
document and justify a different layout in their README, but the default
across the monorepo is the `__tests/` colocation pattern.

**Rationale**: Predictable layout makes the monorepo navigable without
tooling assistance and prevents per-package drift as the codebase grows.
