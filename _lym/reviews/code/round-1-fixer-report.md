# Round 1 fixer-report

## Changes

- **F1** fixed as proposed (`b2e38f76`): `internal/commentlint/linelength.go` tests the counted text against the new `anyLevelHeadingPattern` (`^#{1,6}(\s|$)`); `headingPattern` is unchanged. Added the `a level-two heading is exempt` row to `TestFindNewLongLines`.
- **F2** fixed as proposed (`b2e38f76`): `findNewLongLines` counts each trimmed base comment literal in a `map[string]int` and skips a new line only while its count is above zero. Added rows for a second copy of a long line (flagged at line 4) and a long line moved within the file (not flagged).
- **F3** fixed as proposed (`b2e38f76`): added `TestFindFileFindingsOrdersByLine` in `lint_test.go`, with a long line before a wrapped break and the reverse.
- **F4** fixed as proposed (`327b7285`): `internal/gateslot/template.yaml` line 17 names both rules.
- **F5** fixed as proposed (`8372eac7`): `internal/websterengine/doc.go` says "the comment line lint".

F1, F2 and F3 share files, so they sit in one commit rather than one each.
The gate then flagged the `findNewLongLines` doc comment in `linelength.go` for a break after a comma that does not open a clause.
It is repaired to one sentence per line, in a follow-up commit.
`lym gate test ./internal/commentlint ./internal/gateslot ./internal/websterengine` passed at tier 0 before the commits.

## Self-check

Every fork returned a report; each said "no findings".

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.
