# Rule III: Type Safety (NON-NEGOTIABLE)

All TypeScript packages MUST compile under `"strict": true` (which implies
`strictNullChecks`, `noImplicitAny`, and the rest of the strict family).
`any` is prohibited except at well-documented external boundaries (e.g.,
parsing untyped third-party payloads), and each use MUST carry an inline
comment naming the boundary. `// @ts-ignore` is forbidden; use
`// @ts-expect-error -- <reason>` so that the suppression fails loudly when
the underlying issue is fixed.

**Rationale**: A monorepo accumulates implicit contracts between packages;
strict typing turns those contracts into enforced ones and prevents silent
drift.
