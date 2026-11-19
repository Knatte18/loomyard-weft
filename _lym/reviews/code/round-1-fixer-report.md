---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed
  - id: F5
    disposition: fixed
  - id: F6
    disposition: fixed
  - id: F7
    disposition: fixed
---

## Changes

Each of F1 to F6 is one commit; the gate (`lym gate test` tier 0 over the touched packages, tier 1 over `fabricengine`, `loomcli` and `fabriccli`) passed with every changed and added test running.

- **F1** (9dce3b70): restored `Inputs.PushOwedWayForward`, `Verdict.PushOwedWayForward`, the copy in `Act`, the clearing in `Judge`'s `PushOwed` aside and the `Refusal` branch that names it.
  Restored `inputs.PushOwedWayForward = fabricengine.FailedPushWayForward(owed[0])` in `pairteardown/runread.go` and the `TestReadRun` `wantIn`.
  Added a `TestAct` row for a set way forward, using a records-side record's `lym fabric push` in the code worktree in place of the removed `git push` example.
  `pattern/PATTERN-run-fsm.md` now says the refusal names the owed record's own way forward, as `FailedPushWayForward` gives it.
- **F2** (0588c7b6): `t.Parallel()` after `tiergate.Require` in `TestCommand_CloneEngineRefusalReachesEnvelopeUnchanged`.
- **F3** (8e128bbd): `ensureBoardWorktree` reads `refs/remotes/origin/<warpBranch>`, treats `ErrRefNotFound` as absent, and adds the `--track` worktree when only origin holds the branch; the doc sentence is back.
  Added `TestCloneHub_BoltCheckoutAdoptsARemoteOnlyBoardBranch`, a weft remote whose HEAD is not the warp's branch, asserting the Bolt checkout holds origin's board commit and tracks `origin/main`.
- **F4** (3aaf4f3c): restored the `records cherry-pick in progress commits the transition` subtest, `startStoppedCherryPickInRecords`, `runGitAllowingFailure` and the `os/exec` import, as at 7346d6a8; the one-repository wait rows stay deleted.
- **F5** (72446422): the warn row asserts `loading the orch thresholds for the overview failed`; the no-warn row still rejects any `WARN`.
  The commit's `Loosened-Check` trailer records that the assertion is restored, not loosened.
- **F6** (d62f6fb7): the `MergeStateActive` paragraph in `internal/fabricengine/doc.go` states that a weft cherry-pick, revert or rebase does not count, and that fabric's own replay cherry-picks under the write lock the commit waits on.
- **F7**: `_lym/landing/summary.md` Blast radius now names fabric, lymcwd resolution, the statuscommit transition commit (loom and batten), landingshed Finalize and the runfsm act refusal.
  The file lives under `_lym`, which this round never commits, so the edit is on disk only.

- **Gate repair:** the doc comment of `TestCloneHub_BoltCheckoutAdoptsARemoteOnlyBoardBranch` is rewrapped at sentence boundaries, one sentence per line, each under 120 characters.

- **Reach notes:** the second gate run listed changed functions with the tests that reach them, and named no function without a reaching test.
  The code is unchanged by this repair, and no test was added.

## Self-check

All four forks returned a report, saved beside this file as `round-1-fixer-report.lens-<lens>.md`.

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.

## Rulings

None.
