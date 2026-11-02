# Code-writing seats load the base skills before their language skills

The landing merge fixer, spawned by Parent-Align and Finalize to fix a merge or test failure, loaded only the hub's language skills from `language.yaml`.
It edits code and tests, yet it never loaded `scribe:prose`, `scribe:code-quality` or `scribe:testing`, unlike every other code-writing seat.

## What changes

- **Landing merge fixer** (`internal/landingshed/fixer.go`): `FixerTable` now sets the seat's skills to `fixerSkills` (`scribe:prose`, `scribe:code-quality`, `scribe:testing`) followed by `in.LanguageSkills`, merged through `languageconfig.AppendSkills` so a language skill that repeats a base skill is dropped.
  `TestFixerTable` asserts the full list in both modes, base skills first.
- **Conflict session** (`internal/mergeresolve/spec.go`): `conflictSkills` gains `scribe:testing`, so a session resolving a conflict in a test file loads the testing skill.
  The package doc and the assertions in `mergeresolve_test.go` and landingshed's two conflict-session integration tests follow.
- **Sampler agent** (`internal/samplerengine/spec.go`): the sampler writes and commits code but loaded only `scribe:prose` and `scribe:conversation`; it now loads `scribe:code-quality` and `scribe:testing` after those.

## Seats checked for the same gap

Seats that edit code or tests, and their skills after this change:

- darn writer (`darnSkills`), webster master, recovery strand and verify fixer (`roleSkills`), burler fix and review seats (`burlerSkills`), drift verify (`verifySkills`): already prose, code-quality and testing, then language skills. Unchanged.
- landing merge fixer: fixed above.
- merge conflict session: fixed above.
- sampler agent: fixed above.

Seats checked that write no code, left unchanged:

- landing fix review (`fixReviewTable`): reads the fixer's commits and writes a verdict report only.
- webster plan fixer (`roleSkills`, no language skills) and the plan and parts writers (`planSkills`): edit plan documents.
- drift baseline, describe, discussion and discussion advisor, PR reviewer, treadle judge and targeting, friction, bouncer seed and judge, orch, dobby tidy: write prose or records, not code.
- tackler, shed arbiter and loom driver: load no skills by design, their prompt being the whole procedure.
- MultiLLM seats and generic `seatengine` tables: take their skills from the recipe row, so no code default applies.

## Design decisions

- The sampler fix goes beyond the two seats the board entry names; the entry asks every writer seat to be checked in the same pass, and the sampler is the one other seat with the gap.
  It does not gain the hub's language skills, which would need wiring the sampler's callers do not carry today.
