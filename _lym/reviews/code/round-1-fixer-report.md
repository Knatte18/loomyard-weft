---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed-differently
    reason: "Proposed a wantAdmits column; the Admit assertion compares admissions to wantPushes instead, which is equal on every row and needs no second literal per row. The Run-Shed row also changed after the self-check, from Run-Shed paused then running to Run-Shed running then Seed-Child running, since the paused marker state masked the producer conjunct."
  - id: F5
    disposition: fixed
  - id: F6
    disposition: fixed-differently
    reason: "The Deps.Push and package docs are reworded as proposed. The correction to _lym/landing/summary.md is not made: the fixer never touches _lym, so the writer's change description still says internal/statuscommit is untouched."
  - id: F7
    disposition: fixed
  - id: F4
    disposition: ruling
    reason: "The parent's decision 'Only an uninterrupted start skips the push' keeps the code as it is: a start row resumed after a pause or halt pushes. The target already holds it, so nothing changed and the ruling was not flagged."
---

## Changes

- F1: `seedFreshStatus` in `internal/battencli/arm.go` seeds the absent status and, once it seeded one, calls the run's commit seam for (Worktree-Create, running) after the status lock is released; both pre-run hooks use it.
  `battenCLI.commitStatus` holds the seam and `wire.go` shares that instance with the shed.
  `TestFreshStartCommitsWorktreeCreateThroughTheSeam` drives both hooks: Worktree-Create pushes, the following Seed-Child commits without a push.
- F2: `commitStatusMarker` carries `push_pending`; the core's after-commit callback writes it true for a made commit, the seam clears it once the core returns nil, and the start branch needs it false.
  `TestNewCommitStatusSeam_RetriesAFailedStartChainPush` covers a failed first push.
- F3: three rows and the admission assertion added to `TestNewCommitStatusSeam`.
- F5: the start-row exception added to `PATTERN-pair-remote` in `PATTERN.md` and to `pattern/PATTERN-pair-remote.md`.
- F6: `statuscommit` docs reworded to state a deferring caller passes a no-op Push.
- F7: seam doc reads "commits without a push".
- Self-check changes: a failed seed commit maps to `KindBootstrap` in the step path through `seedCommitFailure`; the Run-Shed row replaced; the test, seam and `PATTERN-run-fsm` docs updated for the push-pending lifecycle; the deferring-Push sentence kept once.

## Self-check

- fix-code-correctness, MEDIUM: the Run-Shed paused row did not pin the marker-producer conjunct — fixed as proposed (row replaced).
- fix-consistency, LOW: `TestNewCommitStatusSeam` doc stale for the new rows and assertion — fixed.
- fix-consistency, LOW: seam doc omits the push-pending clear — fixed.
- fix-consistency, LOW: `PATTERN-run-fsm` understates the exception — fixed.
- fix-proposal, MEDIUM: same as the code-correctness row finding — merged, fixed.
- fix-proposal, LOW: step path reports `KindUnseeded` for a failed seed commit — fixed with `seedCommitFailure`.
- fix-proposal, LOW: deferring-Push sentence stated twice in `statuscommit` — fixed.
- fix-affected-set: no findings.

- Gate repair: four comments in `arm.go`, `commitstatus.go` and `statuscommit.go` re-broken one sentence per line, with no wording change.

- Gate repair, reach: `TestFreshStartSeedCommitFailure` added in `predrive_test.go`; it reaches `seedCommitFailure.Unwrap` through the run hook and pins the step hook's bootstrap kind.

## Disputed

None.

## Deferred

- The `_lym/landing/summary.md` corrections (the "pushes once, at Worktree-Create" claim now holds; the "internal/statuscommit is untouched" statement is now false) are left to the loop that owns `_lym`.

## Rulings

- F4: the parent ruled the start-row exemption covers an uninterrupted start only; the code, `ResumedStartRowPushes` and the PATTERN wording already hold that, so nothing changed and the parent was not told.
