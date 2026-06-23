# Rule I: Automated Formatting Is the Source of Truth

All committed source code MUST be formatted by the project's configured
formatter (Biome for TypeScript, JavaScript, JSX/TSX, JSON, and CSS).
Manual formatting decisions (spacing, line breaks, quote style, trailing
commas) are NOT permitted; the formatter's output is authoritative. Code that
does not match formatter output MUST be rejected at review and MUST fail CI.

**Rationale**: Eliminates style debates in review, keeps diffs focused on
behavior changes, and removes a class of bikeshedding from the workflow.
