# React Component Architecture (C-M-V-C-F)

All React code in this monorepo follows the **C-M-V-C-F** pattern:

```
Components → ( Model → View → Container ) → Feature ↺
```

It is ordinary **MVC** with a **C**omponent layer bolted on the front
and a **F**eature layer on the back. The cycle is closed: a Feature is
consumed like any other component, so Features can compose into pages
that are themselves consumed by larger Features. Each layer consumes
only the layer(s) below it, in the direction shown.

## The five layers

### 1. Components (C)

Pure, presentational React elements. They consume **simple types**
(strings, numbers, booleans, plain callbacks) and produce visual output.
They have **no** connection to any store, network, or other outside
dependency — they are pure and reusable in any context.

- Named for **what they are**: a button is `Button`, a dialog is
  `Dialog`.
- Shared components live in the dedicated component library
  (`src/components/`). Needing a component inside a feature is **rare**;
  Views cover almost every case. Only when a feature needs a bespoke
  component too specific to belong in the shared library may a
  `components/` directory exist inside that feature (see §"Feature
  folder layout").

### 2. Model (M)

The data representation backing everything. In React this is the
**Redux store** and everything that makes it stateful: slices, thunks,
middleware, actions, and selectors.

- The Model is the **single source of truth** for state. State that
  outlives a single render belongs in the Model, **not** in ad-hoc
  `useState`/context inside a Container.
- The Model is consumed **directly** by Views, Containers, and Features
  (via selectors / typed hooks) — as needed, wherever needed.
- Features that use Redux Toolkit / RTK Query MUST follow `rtk-query.md`
  for the Model's internal layout (`model/slices|thunks|middleware|selectors`
  plus a sibling `api/` directory), for not exporting a configured store,
  and for mounting every slice/middleware/api dynamically.

### 3. View (V)

Presentational components that are **not** pure. A View integrates with
the Model, consumes **business types**, and organizes base Components,
raw HTML elements, and other Views into a larger visual presentation.

- Named for what they represent, suffixed `View` (identifier
  `SystemDialogView`, file `system-dialog-view.tsx`).
- Views implement **presentation logic only** — never business logic.
- A View reads the Model **directly** for the state it needs. It does
  **not** receive Model state prop-drilled through a Container.

### 4. Container (C)

Compositional components that take **little to no props** and arrange one
or more Views into a meaningful whole. Containers implement **business
logic** and tell their Views how to respond to user behavior.

- Named for what they represent, suffixed `Container` (identifier
  `SystemDialogContainer`, file `system-dialog-container.tsx`).
- The typical Container↔View relationship is the Container passing
  **callbacks** to the View for user actions.

### 5. Feature (F)

The crowning result of C-M-V-C-F: a complete, user-facing capability
(e.g. an address picker, an auth surface). A Feature is essentially a
**pure, self-contained, drop-in** unit.

- Named for **what it is**: `AddressPicker` (`address-picker.tsx`). A
  feature package MAY export several feature components.
- Features consume **simple types** (numbers, strings — usually ids or
  similar), **never** large complex business objects. This is what keeps
  them drop-in.
- A Feature is consumed like a regular Component, closing the cycle.

## The rules that keep it honest (NON-NEGOTIABLE)

1. **Views MUST NOT consume Containers.** A Container using a View that
   in turn uses another Container creates confusing cycles. If you feel
   you need it, restructure. Views only reach for Components, other
   Views, and the Model.
2. **Containers MUST NOT prop-drill the Model into Views.** A View gets
   the Model it needs from the Model directly (selectors / hooks).
   Passing Model state Container→View contradicts MVC and is forbidden.
   Passing **callbacks** Container→View is expected and correct.
3. **The core MVC contract always applies.** C- and -F only extend it;
   they never relax it.
4. **Avoid prop-drilling callbacks through nested Views.** When a View
   nests many layers of Views, do not thread callbacks down by hand. Use
   a **React Context** for that subtree — this is the preferred method.
5. **Consumption direction is fixed:** Components → Views → Containers →
   Features, with the Model consumed directly by Views, Containers, and
   Features. Do not invert it.

## File and identifier naming

Component **identifiers** are `PascalCase` and carry the layer suffix
where one applies: `SignInFormView`, `SignInFormContainer`, `SignInForm`.
Per the monorepo file rule (`naming-and-layout.md`, Rule IV — **component
files are PascalCase, all other files are camelCase, directories are
kebab-case**), the **file** that exports a Component, View, Container, or
Feature is named in **PascalCase matching that identifier exactly**:

| Layer     | Identifier              | File                          |
|-----------|-------------------------|-------------------------------|
| Component | `Button`                | `Button.tsx`                  |
| View      | `SignInFormView`        | `SignInFormView.tsx`          |
| Container | `SignInFormContainer`   | `SignInFormContainer.tsx`     |
| Feature   | `SignInForm`            | `SignInForm.tsx`              |

Every other file in the feature is camelCase, per Rule IV — hooks
and providers (`useAuth` → `useAuth.ts`, `AuthProvider` →
`authProvider.tsx`) and Model data modules alike (`authSlice.ts`,
`authThunks.ts`, `sessionInvalidMiddleware.ts`). Only the **directories**
are kebab-case (`views/`, `containers/`, `test-support/`).

## Test and story colocation

Tests and stories are colocated with the code they cover, each in its own
dedicated directory (never loose beside the source, never in a separate
top-level tree):

- **Unit / integration tests** live in a `__tests/` directory and mirror
  the filename of the code under test (`SignInFormView.tsx` →
  `__tests/SignInFormView.test.tsx`); see `naming-and-layout.md`.
- **Storybook stories** live in a `__stories/` directory, named
  `<Component>.stories.tsx` (e.g. `__stories/SignInForm.stories.tsx`).
  Visual-regression baselines land under
  `__stories/__screenshots__/` alongside them.

Files under `__tests/` and `__stories/` are exempt from the layering
consumption rules — they may import whatever layer they exercise.

## Feature folder layout

Within a feature package, organize by layer:

Component/View/Container/Feature files PascalCase; every other file
camelCase; all directories kebab-case (Rule IV):

```
src/
  model/          # the Model — see rtk-query.md for RTK features:
    slices/       #   createSlice files
    thunks/       #   thunk groups
    middleware/   #   middleware
    selectors/    #   selector groups
    hooks.ts      #   typed useSelector/useDispatch + RootState/Dispatch
  api/            # RTK Query api(s) at top level, + graphql/ + generated/
  views/          # XxxView.tsx        (presentation)
  containers/     # XxxContainer.tsx   (business logic)
  components/     # OPTIONAL — only feature-specific bespoke components
  __tests/        # xxx.test.ts(x)  — unit/integration, colocated
  __stories/      # xxx.stories.tsx — Storybook, colocated
  XxxFeature.tsx  # the drop-in Feature component(s) at the package root
  index.ts        # the package's public surface (pieces, NOT a store)
```

(A feature that does not use Redux Toolkit keeps a simpler `model/`; the
subfolder split above applies to RTK features per `rtk-query.md`.)

`model/`, `views/`, and `containers/` directories are the default
organization. A `components/` directory appears only for the rare
feature-local Component that is not generic enough for
`src/components/`.

## Canonical example

`src/features/src/auth/` (the `@tms/features/auth` subpath) is the
reference implementation:

- **Model** — `src/model/` (RTK layout per `rtk-query.md`):
  `slices/authSlice.ts` holds the session state (`status` + `account`);
  `thunks/authThunks.ts` holds `signUp`/`signIn`/`logOut`/`refreshSession`;
  `middleware/sessionInvalidMiddleware.ts`; `selectors/authSelectors.ts`;
  `hooks.ts` exposes the typed hooks. The RTK Query api lives under
  `src/api/`. Session state lives here, not in React state — and the
  feature exports no store (its provider mounts the pieces dynamically).
- **Views** — `src/views/SignInFormView.tsx`, `SignUpFormView.tsx`:
  pure presentation over `@tms/components-react` primitives, receiving a
  single `onSubmit` callback.
- **Containers** — `src/containers/SignInFormContainer.tsx`,
  `SignUpFormContainer.tsx`: drive the Model via `useAuth()` and own the
  on-success behavior.
- **Features** — `src/SignInForm.tsx`, `src/SignUpForm.tsx`: the drop-in
  `SignInForm`/`SignUpForm` exported from `index.ts`.
- **Tests / stories** — `src/__tests/*.test.ts(x)` and
  `src/__stories/*.stories.tsx`, colocated in their dedicated
  directories.

## Rationale

C-M-V-C-F gives every React package the same predictable shape. Pinning
the Model as the single source of truth (and forbidding Model
prop-drilling) keeps state flow legible and stops "which copy of this
data is authoritative?" bugs. Separating presentation (Views) from
business logic (Containers) means visual changes and behavioral changes
touch different files, keeping diffs and reviews focused. Making
Features pure, simple-typed drop-ins means a completed capability can be
reused across pages — and re-composed into larger Features — without
dragging its internals along. Standardizing the layering across the
monorepo makes any React package navigable on sight, exactly as the
Repository Structure and Naming rules do for the tree as a whole.
