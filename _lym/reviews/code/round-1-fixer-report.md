# Fixer-report, code round 1

All five findings were coverage gaps and are fixed as proposed, one commit each, in test files only.
Tier 0 passed for the four touched packages (`raddleparser`, `loomshed`, `discussionparser`, `loomcli`); tier 1 passed for `loomcli`.

## Changes

- F1 (`29ad8599`): `internal/raddleparser/proposal_test.go` gains the `UnknownKeyInAPatternCluster` row in `TestParse`, and `TestParse_PatternClusters`, which parses a part's `pattern_clusters` from JSON and compares the values.
- F2 (`1ea42f68`): `internal/loomshed/raddlefile_test.go`, in `MappedPatternChangeIsFiledUnderItsLiveEntry`, now reads the project's record as well as `part-a`'s for the project shape and asserts the live entry's bullet and no `PATTERN-a`.
- F3 (`51683eff`): `internal/loomcli/decision_test.go` gains a `fromFile` row in `TestDecisionVerb_AppendsAndCommits` that withdraws through a `--from` file and asserts `WithdrawnPattern` and `pattern_withdrawn`.
- F4 (`da784027`): `internal/discussionparser/append_test.go` gains two `TestAppendDecision_WithdrawnPatternStrikesBullet` rows (a PATTERN changes section before `## Decisions`, and a bullet on an unterminated last line) and one `TestCheckAddedDecisionsKept` row (a fenced `Withdraws:` line withdraws nothing).
- F5 (`e6cffefa`): `internal/loomcli/raddlecontext_integration_test.go` removes `PATTERN.md` after the main assertions and asserts `raddleBoardContext` returns no error and `LivePattern("PATTERN-live")` is false.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings.

All four forks returned a report.

## Disputed

None.

## Deferred

None.
