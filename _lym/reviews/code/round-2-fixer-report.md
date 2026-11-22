---
dispositions:
  - id: F26
    disposition: fixed-differently
    reason: "The proposal widened Rewrite to take the told worktrees and passed boardDir for the carry; the walk now tells Rewrite the worktrees only (configengine.HubWorktree list), and the carry left Rewrite for a per-machine step (see F5) that takes boardDir and the same list, so Rewrite needs no boardDir."
  - id: F2
    disposition: fixed
  - id: F27
    disposition: fixed
  - id: F28
    disposition: fixed
  - id: F5
    disposition: fixed-differently
    reason: "The proposal asked for a source of each live worktree's board entry run_settings beam keys plus the retired keys, run from the per-machine part of the reconcile; that is done. The proposal's gate, a build stamp or a marker under .lym, is a marker under the Bolt checkout's untracked .lym/config, carried-beam-per-task-settings, written after a successful board read."
  - id: F13
    disposition: fixed
---

## Changes

Commit `0c0fb756a` holds F26, F2, F5 and F13, which share files.

- F26: `configengine.HubWorktree` (name, anchor, main) is the geometry the walk tells a reinterpretation.
  `configreg.Reinterpretation.Rewrite` takes `(existing, worktrees)`; `gateslot.RetireHardenLiveCopies` and the hubreconcile test rewrite follow.
  `hubreconcile` fills the list from the code worktrees the walk holds (`hubWorktrees`).
  `dobbyengine.settingPlaces`, its `os.ReadDir`, `.git` probe, offset join and the `lymdirs` import are gone, and so is the told-geometry exception in `seam_enforcement_test.go`.
  `TestReinterpretPerTaskSettings_ToldWorktrees` pins a worktree whose anchor differs from the prime's, a decoy at the prime's offset and a weft sibling holding `.git`.
- F2: `beamsetting.Module` replaces the literal in `dobbyengine.ConfigSpec`, the configreg beam row and `beamcli.mergeParent`.
- F5: the carry left the hub-once reinterpretation.
  `dobbyengine.CarryBeamTaskSettings(boardDir, worktrees)` runs from `hubreconcile.carryBeamTaskSettings`, at the end of a walk that did not skip, guarded by an untracked per-machine marker.
  Its source is `boardengine.BeamSettingsByEntry`, which replaces `LegacyBeamSettings`.
  It reads each entry's beam run settings and the retired keys while they remain, a run setting winning, and reports a retired `off` apart.
  A board that cannot be read leaves the marker unwritten, so the next walk retries.
  Tests: `TestCarryBeamTaskSettings` has a row that saves the board before the carry, `TestCarryBeamTaskSettings_RunsOncePerMachine` and `_UnreadableBoardLeavesNoMarker` in hubreconcile, and `TestBeamSettingsByEntry`.
- F13: `foldLegacyBeamFields` returns early for a project, so its `pull_request` and `notify_on_land` stay among its kept keys and survive a save.
  The `Load` comment, the fold comment and the `boardengine` doc paragraph match; the store test row and a post-save carry row pin it.

Commit `f05f76228` — F27: the keep marker above `TestLookup_BeamCarriesThePerTaskSettingsReinterpretation`.
Commit `61e78208f` — F28: the duplicate row of `TestPrepareRunSettings` is deleted.

## Self-check

- fix-code-correctness, MEDIUM: the carry flagged and skipped a run-settings `pull_request: off`, which is a current value — fixed in `dobbyengine: fix self-check` (the flag applies to `entry.RetiredOff` only; a `task-run-off` row pins that it carries).
- fix-consistency, LOW: the `ParseBoardTasks` comment still said each entry's keys are folded — fixed in `boardengine: fix self-check`.
- fix-proposal: no findings.
- fix-affected-set: no findings.

## Gate findings

The gate flagged six comment lines over 120 characters or wrapped mid-sentence, in `legacybeam_test.go`, `reinterpret.go`,
`reinterpret_test.go` and `hubreconcile.go`.
Each is now split one sentence per line; no code changed.
A second gate pass flagged the `flagRetiredValues` comment, which the first split had broken mid-sentence;
it is now two whole sentences.

A verify failure then flagged the fabric-vocabulary scan: `reinterpret_test.go` named a sibling directory `beam-a-weft`.
It is now `beam-a-records`.

## Disputed

None.

## Deferred

None.

## Rulings

None.
