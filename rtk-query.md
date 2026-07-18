# RTK Query in Features

This rule governs how RTK Query (and the Redux Toolkit Model behind it)
is structured inside a feature package. It refines the Model ("M") layer
of the C-M-V-C-F pattern (see `react-cmvcf.md`) for features that use
Redux Toolkit / RTK Query.

## 1. Folder structure

A feature's Redux Toolkit code is split across two top-level directories:

```
src/
  model/
    slices/         # one file per slice (createSlice)
    thunks/         # one file per thunk group
    middleware/     # one file per middleware
    selectors/      # one file per selector group
    hooks.ts        # typed useSelector / useDispatch (+ RootState/Dispatch types)
  api/
    <feature>Api.ts # the RTK Query api(s) — at the TOP LEVEL of api/
    graphqlBaseQuery.ts  # transport / baseQuery
    graphql/        # hand-authored .graphql documents (queries, mutations)
    generated/      # codegen or otherwise stubbed output (typed endpoints/hooks)
```

- **`model/`** holds the state machinery, one concern per subfolder:
  `slices/`, `thunks/`, `middleware/`, `selectors/`. Each subfolder
  contains all files of that kind for the feature. Typed react-redux
  hooks and the feature's `RootState`/`Dispatch` types live at
  `model/hooks.ts`.
- **`api/`** holds the RTK Query api(s) plus anything generated or
  stubbed. **The api object(s) live at the top level of `api/`** so they
  are easy to find. Generated code (`generated/`) and the GraphQL
  documents (`graphql/`) are nested inside `api/`, not scattered
  elsewhere in the package.
- Selectors MUST be defined in `model/selectors/` (typically via the
  slice's `selectSlice`/`selectors`), NOT inline in `hooks.ts` or in
  Views/Containers.

## 2. Features MUST NOT export a configured store

A feature MUST NOT ship or export a pre-configured store (no
`createFeatureStore`, no exported `store.ts`). A feature owns *pieces*,
not an application's store.

- Export only the things a host needs to configure a store or use the
  individual slices: the api(s), the slice(s), the middleware
  factory/instance, the thunks, and the selectors.
- If the feature provides a drop-in React provider, that provider MAY
  assemble a store **internally** (kept private to the provider) — but it
  MUST do so by dynamically mounting the exported pieces (see §3), and it
  MUST NOT export that store or a store factory.
- Rationale: an application typically has ONE root store that many
  features contribute to. A feature that exports its own configured store
  cannot be composed into that root store and fights the host for
  ownership.
- The canonical host store is the shared **`@tms/ts/rtk-q`** subpath of
  the `@tms/ts` lib (`src/libs/ts/`, source at `src/libs/ts/src/rtk-q/`):
  a singleton built on an empty `combineSlices()`
  root plus a `createDynamicMiddleware()` instance. Features inject into
  it — most conveniently via its `useDynamic({ slices, apis, middlewares })`
  hook (or the lower-level `slice.injectInto(rootReducer)` / `mountApi(api)`);
  apps get store integration from a single import. Prefer it over a
  bespoke per-app `configureStore`.

## 3. All RTK Query components MUST be mounted dynamically

To keep features drop-in and code-splittable, every Redux Toolkit piece a
feature contributes MUST support **dynamic injection** into a host store
(Redux Toolkit "code splitting"). Nothing may be baked in statically as
the only way to use it.

- **Slices** — created with `createSlice` and mounted via
  `slice.injectInto(rootReducer)` (or `rootReducer.inject(slice)`), NOT
  hard-wired into a static `configureStore({ reducer: { … } })` map.
- **Middleware** — added via a `createDynamicMiddleware()` instance's
  `addMiddleware(...)`, NOT hard-listed in `configureStore`'s
  `middleware` array. Middleware SHOULD be exported as a factory
  (`createXMiddleware(options)`) or a plain middleware so the host can add
  it to its own dynamic-middleware instance.
- **API endpoints** — declared with `api.injectEndpoints({ … })` against
  an empty base `createApi({ …, endpoints: () => ({}) })`, so endpoints
  code-split off the base and the api can be injected into any store.
  Generated RTK Query code (e.g. `@graphql-codegen/typescript-rtk-query`)
  already emits `injectEndpoints`; keep the base api separate from the
  injected endpoints.
- Selectors and typed hooks MUST be typed against the feature's OWN
  contributed state (e.g. `WithSlice<typeof api> & WithSlice<typeof
  slice>`), never against a concrete host `RootState`, so the feature
  composes into a store of unknown shape.
- When injecting into a store built with `combineSlices()`, inject
  reducers **before** `configureStore(rootReducer)` (or dispatch an init
  action after) so the store's initial state already carries the slice —
  avoiding a first-render "slice missing" gap. Prefer the slice's
  injection-aware selectors (`selectSlice`), which fall back to the
  slice's initial state when it is not yet mounted.

## Canonical example

`src/features/src/auth/` (the `@tms/features/auth` subpath) implements all
three rules:

- **`api/authApi.ts`** — empty base `createApi`; **`api/generated/api.ts`**
  injects the endpoints/hooks; **`api/graphql/`** holds the documents;
  **`api/graphqlBaseQuery.ts`** is the transport.
- **`model/slices/authSlice.ts`**, **`model/thunks/authThunks.ts`**,
  **`model/middleware/sessionInvalidMiddleware.ts`**,
  **`model/selectors/authSelectors.ts`**, **`model/hooks.ts`**.
- **No `store.ts`.** `index.ts` exports the pieces (`authApi`,
  `authSlice`, `createSessionInvalidMiddleware`, thunks, selectors).
- **`authProvider.tsx`** injects those pieces into the shared `@tms/ts/rtk-q`
  store via its `useDynamic({ slices, apis, middlewares })` hook, then
  provides that store to its subtree.

## Rationale

Splitting the Model into `slices/`/`thunks/`/`middleware/`/`selectors/`
and isolating the api under `api/` makes any RTK feature navigable on
sight and keeps generated code quarantined from hand-written logic.
Forbidding an exported store and requiring dynamic mounting are what make
a feature genuinely drop-in: it can be injected into a host application's
single root store — lazily, alongside any number of other features —
instead of forcing its own store or a static wire-up. This is the Redux
Toolkit code-splitting model applied as a package boundary.
