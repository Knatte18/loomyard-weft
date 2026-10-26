# Round 1 fixer-report

## Changes

- F1 (MEDIUM): `internal/discussionparser/doc.go:11` now says Validate refuses an `adds` or `changes` bullet whose rule text, the entry text without a trailing `(test)` or background link, would run past the rule-text cap. Fixed as proposed, commit a65b081f.
- F2 (LOW): the `Validate` doc comment at `internal/discussionparser/validate.go:61` now says "whose rule text is longer than a PATTERN.md entry's rule text may be". Fixed as proposed, commit 87ee175f.

Both changes are comment-only, so no test was added or run.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings.

No fork returned without a report.

## Disputed

None.

## Deferred

None.
