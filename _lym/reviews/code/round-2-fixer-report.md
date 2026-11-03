# Round 2 fixer-report

## Changes

- F1 (LOW), fixed as proposed: `Env.PreflightStencils` in `internal/shedrecipe/recipe.go` now states that it checks only the Darn-Write writer table, since the other seat tables are built from call-time values Preflight does not have. Commit 70327c4a.
- F2 (LOW), fixed as proposed: the `Table.CheckMarkers` refusal in `internal/seatengine/prompt.go` names both causes, stencils newer than the lym binary or a blank value, and a way forward for each. The `CheckMarkers` comment, `internal/seatengine/doc.go` and the matching substrings in `TestTable_CheckMarkers` are updated. Commit 7853ef2b.
- Self-check follow-up to F2: the twin comments in `internal/loomcli/darnwiring.go` and `internal/preflightshed/doc.go` now name the blank-value cause too. Commit after 7853ef2b.

Gate: `lym gate test` over `./internal/seatengine`, `./internal/shedrecipe`, `./internal/preflightshed` and `./internal/loomcli` passed, failed 0.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: LOW, `preflightshed/doc.go:19-20` stale single cause — fixed as proposed.
- fix-consistency: LOW, `darnwiring.go:224` stale single cause — fixed as proposed.
- fix-proposal: LOW, `preflightshed/doc.go` — duplicate, merged, fixed.
- fix-proposal: LOW, `darnwiring.go:224` — duplicate, merged, fixed.
- fix-affected-set: LOW, `preflightshed/doc.go` — duplicate, merged, fixed.
- fix-affected-set: LOW, `darnwiring.go:224` — duplicate, merged, fixed.

All four forks returned a report.

## Disputed

None.

## Deferred

None.
