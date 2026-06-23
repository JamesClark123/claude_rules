# Rule VIII: Environment Variable Discipline (NON-NEGOTIABLE)

Every workspace package that reads any environment variable MUST follow
the discipline below. A package that genuinely consumes no env vars MAY
omit `env.ts` and the `.env*` files; the moment it adds its first
`process.env.<X>` access, all four sub-rules below apply.

**1. Per-package `.env` for local development**:

- Each package that consumes env vars MUST own a `.env` file at its
  package root (e.g., `src/services/<svc>/.env`,
  `src/apps/<app>/.env`). Loading is package-local; a package's own
  `.env` is authoritative for its variables. Walking up the directory
  tree to discover env files MUST NOT be relied on — if a package
  needs a value, that value MUST appear in the package's own `.env`.
- The `.env` file MUST NOT be committed. The repo-root `.gitignore`
  already excludes `.env` and `.env.local`; per-package overrides MUST
  NOT re-include them.
- Loaders: TypeScript / Node packages MUST load `.env` via `dotenv`
  (or `dotenvx`) at process start, before `env.ts` parses the
  resulting `process.env`. Vite/Next-style apps MAY use the
  framework's native env loader, which serves the same role; the
  loader choice MUST be documented in the package README.

**2. Central Zod-parsed `env.ts` per package**:

- Each package that consumes env vars MUST expose exactly one
  `src/env.ts` (or `env.ts` at package root for non-`src/` layouts)
  that:
  - declares a Zod schema covering every variable the package reads,
  - calls `schema.parse(process.env)` (or `safeParse` with an explicit
    failure path) at module load time,
  - exports a single typed `env` object — no other shape and no
    additional barrel re-exports.
- All consumer code in the package MUST import the parsed values from
  `env.ts`. **Direct reads of `process.env.<X>` outside `env.ts` are
  forbidden** and MUST be flagged in review. The only sanctioned
  exception is the `dotenv` loader call itself (typically the very
  first import in the package's entrypoint).
- Parse failure MUST throw at startup with a human-readable error
  naming the offending key(s). Silent fallbacks to defaults for
  missing required variables are NOT permitted; defaults are
  permitted only where Zod's `.default()` makes the variable
  semantically optional.
- The schema's variable names MUST follow the `UPPER_SNAKE_CASE`
  convention from Rule IV (env vars are constants).

**3. Committed `.env.example`**:

- Each package that has a `.env` MUST also ship a `.env.example` file
  at the same path (e.g., `src/services/<svc>/.env.example`).
  `.env.example` MUST be tracked by git.
- `.env.example` documents every variable the package's `env.ts`
  schema parses. Values MUST be **safe placeholders** — never real
  secrets, real production URLs, or real credentials. Suggested
  forms: `<placeholder>`, `replace-me`, `localhost:5432`,
  `change-me-in-prod`.
- Comments in `.env.example` (lines starting with `#`) SHOULD be used
  to explain non-obvious variables, link to where they are issued,
  or note format constraints (e.g., `# Postgres connection URL
  including ssl=require for prod`).

**4. `.env` ↔ `.env.example` sync (enforced)**:

- The set of variable **keys** in `.env.example` MUST exactly equal
  the set of keys parsed by the package's `env.ts` Zod schema. Adding
  a key to the schema without updating `.env.example` (or vice versa)
  is a constitutional violation.
- Renaming a variable counts as both an add and a remove and MUST
  update `.env.example` in the same PR.
- A workspace `env:check` script (Rule V gate item 7) MUST
  enforce this equality programmatically by introspecting each
  package's `env.ts` schema (e.g., via `schema.shape` for Zod
  objects) and diffing against the parsed keys of its
  `.env.example`. The check MUST also verify that `.env.example` is
  tracked by git and that `.env` is NOT.
- Local `.env` files MAY contain extra keys beyond the schema (a
  developer might keep notes, scratch values, or feature-flag toggles
  there), but the schema and the example MUST stay in lockstep. Only
  schema/example are the contract.

**Rationale**: Centralizing env access through a single Zod-parsed
module turns a class of runtime errors ("undefined is not a string",
"DATABASE_URL is unparseable") into startup-time failures with
actionable messages, and gives the type system real knowledge of what
configuration the package needs. Committing `.env.example` makes the
contract discoverable for new contributors without leaking secrets.
Enforcing sync via `env:check` prevents the most common drift mode in
practice — someone adds a new variable to the schema, ships it, and
nobody else's `.env` knows about it until production breaks.
