---
verdict: APPROVED
rationale: "Both round-2 findings are fixed in the current plan. Card 16 now holds back the reviews root and loom durable directory for as long as the untracked pending-rejection record exists, which covers card 5's class write through its round commit and is tested over a classed but uncommitted round. The verify integration and smoke lines now include ./internal/planglyph/... Round 1's other fixes still hold, and D1-D15 each map to a card."
---

# Plan-Review round 2: judge verdict

The round-2 report has no `## Focus departures` section, so there is no departure to ratify or reject.

## The report's findings

Both findings were correct when the report was written, and neither falls under the rubric's Do-not-flag list.
The plan now on disk carries the round-2 fixer's edits, and both fixes hold.

- **F1 (card 16's hold ends before the round commit) is fixed.**
  Card 16 no longer reads the highest round's class.
  `statusCommitPathspec` returns only the status file and the drive reports while `loomengine.LoomRejectionPath` exists, or while a stat of it fails for any reason other than not-exist.
  I checked the marker against the code.
  `LoomRejectionPath` (`internal/loomengine/config.go:224`) sits under `LoomScratchDir` and is never tracked, so no status commit can land it, and stat sees exactly the working-tree state.
  `prRework.Call` removes it only through `finish`, which runs after `deps.Commit` (`internal/loomshed/prrework.go:133-136`), so the hold spans the archive, the session, the class write and the round commit.
  A `Commit` seam error or a crash after the class write now leaves `_lyx/loom/` and `_lyx/reviews/` out of every persist's status commit.
  `committedRejections` then finds no classed record at HEAD, and the resume re-enters the round instead of taking `finish(true)`.
  The table test covers a pending rejection over both a classless round and a classed but uncommitted one.
  The real-git integration test asserts that HEAD gains no `round-<N>/` file while the rejection is pending.
  The overview's `committed-round-carries-a-class` and `one-commit-per-round` decisions, and card 5's two references to card 16, now describe the pending-rejection hold.
- **F2 (verify omits planglyph's integration tests) is fixed.**
  The `go test -tags integration` and `go vet -tags smoke` lines in `## verify:` now list `./internal/planglyph/...`, which carries `drift_integration_test.go`, `delta_integration_test.go` and `donecheck_integration_test.go`.

## Regression and fidelity check

- Round 1's other fixes are intact:
  - Card 7 still puts `CheckFirstCard` in `internal/planparser/firstcard.go` and adds `planglyph.ValidateRework`.
  - The overview's `generation-round-rule` still carries the class condition and names card 9 as authoritative.
  - Card 10's recipe comment still states D7's full bound.
- Every Decision D1–D15 in `_lyx/discussion/decision-record.md` is carried by a card, as the cards' own D-citations show.
  Card 16 is licensed by D14's one-commit and crash-resume requirements.
  I found no card that adds scope the record does not license.
- Card 16's Intent states its bound: the hold needs a pending rejection and can only defer the sweep, never drop anything from it.
  That answers the attack-surface question for the new guard.

## Observation (not a finding)

Card 16's bound says only PR-Rework clears the pending rejection.
`lyx loom approve` also removes it (`internal/loomcli/approve.go:183`).
This does not weaken the hold: `approveVerb` refuses unless the run is halted at PR-Gate, and at that point no archive step has started.
An operator `goto` away from PR-Rework mid-round leaves the hold in place until the next PR-Gate approval or rejection.
That defers the sweep and drops nothing, which matches the card's own bound.
