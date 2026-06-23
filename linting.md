# Rule II: Linting Is Non-Negotiable

Biome's linter MUST run on every TypeScript/JavaScript file with the
project's shared configuration. Lint errors MUST fail CI. Lint warnings
MUST be triaged before merge — either fixed or explicitly suppressed inline
with a justification comment (`// biome-ignore <rule>: <reason>`). Blanket
suppressions (file-level or directory-level disables, or wide
`overrides`/`includes` exemptions in `biome.json`) require approval in the
PR description.

**Rationale**: Linting catches real bugs (unused vars, accidental promises,
unsafe types) and enforces consistency that formatters cannot. Using Biome
for both formatting and linting collapses two toolchains into one, removing
config drift between them.
