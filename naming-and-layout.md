# Rule IV: Consistent Naming and File Layout

Naming MUST follow these conventions across all packages:

- **Files: `camelCase`** by default (e.g., `userRepo.ts`, `authSlice.ts`,
  `useAuth.ts`). A file that exports a single primary non-component symbol
  SHOULD be named for that symbol with a lowercased first letter (`useAuth`
  → `useAuth.ts`).
- **React component files: `PascalCase`.** A file whose primary export is a
  React **Component**, **View**, **Container**, or **Feature** is named in
  `PascalCase`, matching the exported identifier exactly — including the
  layer suffix where one applies (`Button` → `Button.tsx`, `SignInFormView`
  → `SignInFormView.tsx`, `SignInFormContainer` → `SignInFormContainer.tsx`,
  `SignInForm` → `SignInForm.tsx`). Every other file in a feature — hooks,
  providers, and Model modules (slices, thunks, middleware, selectors) —
  stays `camelCase` per the default above.
- **Directories: `kebab-case`** (e.g., `demo-service/`, `test-support/`,
  `user-auth/`).
- **Exception — tool-mandated names.** Files whose names are dictated by
  a tool or ecosystem convention keep those exact names and are NOT
  camelCased: `package.json`, `tsconfig.json`, `*.config.ts`
  (`vite.config.ts`, `vitest.config.ts`), `docker-compose.yml`,
  `nginx.conf`, `README.md`, `.env*`, dotfiles, and framework config
  directories (`.storybook/`). Special colocation directories keep their
  fixed names too: `__tests/`, `__stories/`, `__screenshots__/`.
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
filename of the code under test (`foo.ts` → `foo.test.ts`). Storybook
stories follow the same colocation pattern in a sibling `__stories/`
directory (`foo.tsx` → `__stories/foo.stories.tsx`); stories MUST NOT sit
loose beside the components they cover. Because a test/story mirrors the
source filename, it inherits that file's casing — a `PascalCase` component
file yields a `PascalCase` test and story (`SignInForm.tsx` →
`__tests/SignInForm.test.tsx`, `__stories/SignInForm.stories.tsx`; the
matching visual-regression baseline directory is likewise
`__stories/__screenshots__/SignInForm.stories.tsx/`). Packages MAY document and justify
a different layout in their README, but the default across the monorepo
is the `__tests/` / `__stories/` colocation pattern.

**Rationale**: Predictable layout makes the monorepo navigable without
tooling assistance and prevents per-package drift as the codebase grows.
