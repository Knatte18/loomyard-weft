---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed-differently
    reason: "The Dropped blockers were not moved above the early return in pushParent. A new step, payOwedSteps, reads every carried Dropped commit and every owed hand-over in settleLandings, so it records the refusal with no landing left, in a pass and in SettleRefused alike. Moving the blockers would not have retried the reset or the hand-over."
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed-differently
    reason: "Fabric.PushBranch moved into export_test.go instead of being deleted with its callers retargeted, as PATTERN-test-only-code allows for a method. PushWarpRebaseFreeAt, whose only production caller was PushBranch, moved with it, and its own test file was removed."
  - id: F5
    disposition: fixed-differently
    reason: "The proposed line offered the trail setting or a hand pull request as alternatives. Setting trail lands nothing for the refused landing, so a reader taking that branch and then resetting would drop the only copy on the parent. The line now requires the hand pull request and names trail for later landings."
  - id: F6
    disposition: fixed
  - id: F7
    disposition: fixed
  - id: F8
    disposition: fixed
  - id: F9
    disposition: fixed
  - id: F10
    disposition: fixed
  - id: F11
    disposition: fixed
  - id: F12
    disposition: fixed
  - id: F13
    disposition: fixed
  - id: F14
    disposition: fixed-differently
    reason: "PushPairAnchored reads the branch's landing commits itself instead of taking them from the parent pusher, because the `lym fabric push` a beam record names would otherwise replace the record with one that names no landing."
  - id: F15
    disposition: fixed
  - id: F16
    disposition: fixed
  - id: F17
    disposition: fixed
  - id: F18
    disposition: fixed
---

# Fixer report, code review round 1

## Changes

- F1: a landing settled on origin for a run owned elsewhere now clears the parent's refused-push record.
- F2: the ledger entry gained `HandOvers`. `payOwedSteps` makes owed resets and hand-overs again on every pass and in `SettleRefused`, and records a failed one as `held_back`.
- F3: the prime's tick starts the pusher of each beam pair with parent work that does not read refused.
- F4: the two dead methods are gone from production code, as the dispositions describe.
- F5: a protected-branch refusal of a landing push carries its own way forward.
- F6: a landing commit the parent no longer holds is not pushed. It is refused, naming `git merge --no-ff <commit>` and the parent's push.
- F7: a landing recorded again keeps holding the run's earlier unpushed records commit.
- F8: a pause lift decides from `RunCommitsAhead`. `PairPushDeferred` and its test are gone.
- F9: `lym webster reset` pushes the pair's records before it returns, waiting for a running push, and reports a failure in `push_error`. Both PATTERN entries say so.
- F10: a diverged prime holding an owed tip is judged as diverged.
- F11: `TestParentPush` has the card's rows, plus rows for F1, F2 and F6.
- F12: tests for the pushes-off landing, `lym fabric sync-status`, the wired refusal read and a pair's network wait.
- F13: tests for a rejected capped push, the two reset refusals and the configured timing wiring.
- F14: a diverged parent push names its landing commits, with the way forward of its own kind of worktree.
- F15: the two teardown sentinels and their branches are gone.
- F16: the reflog revisions go to `git rev-list` on stdin.
- F17: the stale docs, comments and help texts are corrected. `AddSaysItPushesDetached` is renamed and pins the new wording.
- F18: the three caches are listed.

Removed checks are recorded as `Loosened-Check` trailers in the F15 commit and in the commit "tests: record the checks removed with their dead code".

## Comment repair

The round's gate rejected comment lines it had added that wrapped mid-sentence or reached 120 characters, and ran no test.
The comments it listed, in `pull.go`, `pushclassified.go`, `landingdeps.go`, `parentpush_integration_test.go`, `ledger.go`, `parent.go` and `targetreads.go`, are rewritten as one sentence per line.
No code changed.
The tier 0 gate of the edited packages passed afterwards.

## Vocabulary repair

The round's next verify failed `TestEnforcement_FabricVocabulary` in `internal/lymcwd`:
the F10 test row in `internal/battencli/landpull_integration_test.go` matched a refusal text that named one of the fabric's two sides.
It now matches another part of the same message, `remote diverged and local`, which only that refusal carries.
The tier 1 gate of `internal/lymcwd` and `internal/battencli` passed afterwards.

## Verification

Every group was committed after a green gate on tier 0 or tier 1 of its packages. Tests of higher tiers were skipped by those runs and were not run.

One guard fails and is not from this change set: `TestTestOnlyCode_EveryProductionFunctionIsReachableFromAMain` reports `internal/shedcli.names` (`internal/shedcli/table.go:111`). It arrived with the merge from main, and no finding names it, so it is unchanged here.

## Self-check

- fix-consistency 1, LOW (held-entry descriptions omit owed hand-overs): fixed.
- fix-consistency 2, LOW (the push summary did not list an owed hand-over as held): fixed.
- fix-consistency 3, LOW (`beamcli` doc on the tick's pusher restarts): fixed.
- fix-consistency 4, LOW (`webstercli` doc omits the reset's push): fixed.
- fix-proposal 1, MEDIUM (F5's way forward offered `trail` as an alternative): fixed.
- fix-affected-set 1, MEDIUM (an owed step with no record was never retried): fixed; a parent target now reads as ahead while it owes one.
- fix-affected-set 2, LOW (the same push-summary gap as fix-consistency 2): fixed with it.
- fix-affected-set 3, LOW (a hand push of a beam named no landing and lost the protected-branch line): fixed.
- fix-code-correctness 1, MEDIUM (the same retry gap as fix-affected-set 1): fixed with it.
- fix-code-correctness 2, LOW (a busy push lock left the reset's push unrecorded): fixed.

## Disputed

None.

## Deferred

None.

## Rulings

None.
