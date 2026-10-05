# Fixer report — discussion round 1

All edits are in `_lyx/discussion/decision-record.md`; `support-log.md` needed none.

## Changed

- **F1 (D5):** Each bullet clause now says what the finding names.
  Webster-Review names the covering test, the uncovered behavior, or the non-gap finding.
  Plan-Review names the covering test, or the public-surface test for a per-symbol test, and a vague "seems redundant" is stated not to count.
- **F2 (D11):** The Rationale states that a file-level comment excuses only tests touching the named state, and a test touching none calls `t.Parallel`.
- **F3:** The Acceptance bullet lists the full set of removed phrases: "not a substitute for a bundled test", "independently reviewable/testable unit" and "the bundle-your-own-test rule", with the Plan-Review rubric added to the files checked.
  D3 now quotes the old granularity phrase as the file carries it.
- **F4:** Scope moves the PATTERN work to one "Already done" line and keeps `refusal-spec.md` as Plan's item.
  D8 states the done wording as fact, drops the re-audit invitation, and names the `refusal-spec.md` alignment as Plan's half.
  The `docs/` link-integrity and overview-format Constraints are removed, since nothing Plan does adds a link under `docs/` or a PATTERN line.

## Deferred

None.
