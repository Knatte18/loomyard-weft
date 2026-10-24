# Fixer-report — code round 1

## Changes

- **F1** (fixed with my own change): `lyx loom decision add --by parent` now compares the caller to the nearest live orch up the beam chain, the destination of notices.
  - `internal/orchcli/notify.go` gains `NearestLiveParent`, which `internal/loomcli/decision.go` uses as its parent resolver.
  - A test pair in `internal/orchcli/notify_integration_test.go` pins the live-beam and stopped-beam answers.
  - The review also proposed moving the Discussion-Write parent-review addressee (`wiring.go:390`).
    I did not make that change: the nearest-live answer names the prime with an empty `Worktree`, which `newParentReviewConfig` reads as "no parent" and treats as always live.
    `internal/hubgeom/doc.go` instead states that prompts' parent names and the parent review keep the recorded parent.
- **F2** (fixed with my own change): in `internal/fabricengine/add.go`, an adopted weft with an origin record keeps its parent fields and gains the beam mark in the origin commit when `opts.Beam` is set and the record is unmarked.
  - The review proposed rewriting the record with the options' `ParentBranch` and `ParentWorktree` too; I did not, because that overwrites the recorded parent a task moved between machines keeps.
  - `TestAdd_AdoptedWeftTakesTheBeamMarkOnly` fails with the old condition.
- **F3** (as proposed): `internal/boardengine/rename_test.go` says `grouping` and `grouping-two` throughout.
- **F4** (as proposed): `docs/overview.md` names the run's parent worktree's reed session for `--window`.
- Self-check follow-ups, in the last commit:
  - `internal/fabricengine/doc.go`, the `Add` doc comment, and the guard texts (`internal/loomcli/decision.go` header and help, `docs/overview.md`) follow F2 and F1.
  - `internal/battenshed/doc.go` follows F4.

## Self-check

- fix-consistency, MEDIUM, `fabricengine/doc.go` adopted-record sentence: fixed as proposed.
- fix-consistency, MEDIUM, four stale "recorded parent" guard texts: fixed as proposed.
- fix-consistency, LOW, `Add` doc comment omits the beam mark: fixed as proposed.
- fix-consistency, LOW, `decision.go:564` refusal wording: disputed, `every fix worse`, see Disputed.
- fix-affected-set, LOW, `battenshed/doc.go:59` stale reed session: fixed as proposed.
- fix-code-correctness, LOW, a corrupt origin record now aborts the adopt-path Add: disputed, `every fix worse`, see Disputed.
- fix-code-correctness, LOW, no test pins the guard's resolver wiring: deferred, see Deferred.
- fix-proposal: no findings.

## Disputed

- Ground `every fix worse`, fix-consistency LOW on `decision.go:564`: the premise holds, but the text is correct as written.
  "The run's parent could not be resolved" and "ask %s" name the nearest live orch, which is the one that may now record the call; the prompts' parent name is a separate, documented staleness.
- Ground `every fix worse`, fix-code-correctness LOW on the corrupt record: the premise holds, but the old behaviour left an unparsed record in place and a read-failing record would be overwritten on a stat error.
  The refusal `read adopted origin record: ...` stops before any mutation, so no other change keeps the recorded parent safe.

## Deferred

- fix-code-correctness LOW, no test for `decision.go` resolver wiring: the line sits in the cobra constructor, where the deps struct is built from the live hub and reed, and no seam exists to substitute `OrchLive`.
  A test needs a new seam in the constructor, which is a change beyond this round's findings.
