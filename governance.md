# Governance

These rules supersede ad-hoc style preferences and individual contributor
habits. When code, review feedback, or tooling configuration conflicts with
any rule, the constitution wins until amended.

**Constitution location**: `.specify/memory/constitution.md`

**Amendment procedure**:

1. Open a PR that modifies `.specify/memory/constitution.md` and the
   affected rule file(s) under `.claude/rules/`.
2. Include a Sync Impact Report (as an HTML comment at the top of
   `.specify/memory/constitution.md`) describing the version bump, changed
   principles, and templates touched.
3. Bump `CONSTITUTION_VERSION` per semantic versioning:
   - **MAJOR**: Removing a principle, redefining a principle in a
     backwards-incompatible way, or changing governance procedure.
   - **MINOR**: Adding a new principle or materially expanding an existing
     one.
   - **PATCH**: Wording clarifications, typo fixes, non-semantic refinements.
4. Update `LAST_AMENDED_DATE` to the merge date (ISO `YYYY-MM-DD`).

**Compliance review**: Reviewers MUST confirm constitutional compliance
before approving a PR. Plans generated via `/speckit-plan` MUST gate on the
Constitution Check section against the rules in `.claude/rules/`; violations
MUST be recorded in the plan's Complexity Tracking table with explicit
justification, or the plan revised to comply.

**Runtime guidance**: Day-to-day implementation guidance lives in
`CLAUDE.md` and the active feature plan under `specs/`. Those documents MUST
NOT contradict these rules; if they appear to, the constitution governs and
the guidance file MUST be corrected.

**Version**: 2.5.0 | **Ratified**: 2026-05-03 | **Last Amended**: 2026-07-05
