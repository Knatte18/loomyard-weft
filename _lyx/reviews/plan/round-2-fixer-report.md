# Fixer report — plan round 2

Both findings are fixed.
`lyx loom validate-plan` over the edited plan returns `ok: true` with informational `create-new-unit` findings only.

## Fixed

- **F1 (BLOCKING): card 16's hold ends before the round commit lands.**
  Rewrote card 16.
  `statusCommitPathspec` now holds back the reviews root and the loom durable directory for as long as a rejection is pending (`loomengine.LoomRejectionPath` exists, or its stat fails other than not-exist).
  It no longer reads the highest round's class, so the window from card 5's class write to its round commit is covered.
  `ReworkArchiveInFlight` and the card's `generation.go`/`generation_test.go` targets are dropped, which leaves the card local to loomcli.
  The Intent names the failure the old predicate allowed: a classed record reaches HEAD, `committedRejections` counts it, and `finish(true)` clears the rejection with no round commit.
  The bound and the ImpactSummary are restated.
  The tests now cover a pending rejection over a classed but uncommitted round, both in the table and in the real-git integration test.
  In the overview, `committed-round-carries-a-class` and `one-commit-per-round` are reworded to the pending-rejection hold, as is card 5's restatement of the classed-only rule.
- **F2 (MEDIUM): verify does not run planglyph's integration tests.**
  Added `./internal/planglyph/...` to the overview's `go test -tags integration` line, and to the `go vet -tags smoke` line for consistency.

## Deferred

None.

## Regression check against round 1

Round 1's F2–F4 fixes (`CheckFirstCard` in planparser, the class condition in the generation-round rule, card 10's bound) are intact.
Round 1's F1 fix (card 16) was partial and is this round's F1.
