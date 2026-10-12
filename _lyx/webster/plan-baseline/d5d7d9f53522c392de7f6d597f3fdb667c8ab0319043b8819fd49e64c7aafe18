# Card 27 — lock-wait-pause-after

**Create:**
- `internal/loomcli#TestParentLockDeps_PauseAfterTheWaitingRowEndsTheWait`

**Edit:**
- `internal/loomcli#parentLockDeps`
- `internal/landingshed#ParentLockDeps`
- `internal/landingshed#parentGuard.ensure`
- `internal/landingshed#parentGuard.pauseRequested`
- `internal/landingshed#TestParentGuard_EnsureReasonsAndErrors`
- `internal/loomcli#loomVerbTexts`
- `contracts/specs/refusal-spec.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:**
Let `lyx loom pause --after <row>` end a parent lock wait at that row (D8), since the bare pause that used to end it is gone: the only mid-row halt left for a loom run.
`parentLockDeps`' `PauseRequested` seam reads the run's status under its lock and reports true when `pause_after` names the run's current producer, which is the lock-taking row waiting in the queue, instead of reading `pause_requested`.
The wait then ends through the existing path: `Acquire` returns `ErrPaused` with the ticket removed, and `parentGuard.ensure` returns the halt reason; the halt leaves `pause_after` recorded, and `lyx loom goto` clears both conditions as it always does, so `goto` then `resume` moves the run.
So `--after` no longer means only a halt after the named row finishes: naming a row waiting for the parent lock, it ends that wait mid-row. A pause alone never ends the wait (card 7).

`ParentLockDeps.PauseRequested`'s and `parentGuard.pauseRequested`'s docs say the seam reports a `--after` naming the waiting row; `ensure`'s reason reads `halted while waiting for the parent lock on "<branch>" held by <holder>, as "lyx loom pause --after <row>" asked; way forward: run "lyx loom goto --to <row>", then "lyx loom resume"`.
`loomVerbTexts`' pause help names the mid-row case.
`refusal-spec.md`'s `parent lock wait paused` row stays, its trigger reworded to "`lyx loom pause --after <row>` names a lock-taking landing row waiting in the parent lock's queue".

Reachability: a run running at a lock-taking row (Publish, PR-Gate or Finalize) whose parent lock another live run holds, with `pause_after` naming that row, reaches the halt at the wait's next check.

Tests: `TestParentGuard_EnsureReasonsAndErrors`' paused row asserts the new reason; the new loomcli test writes a status running at `Publish` and asserts the seam reports false with no condition, false with `pause_after` naming another row, true with it naming `Publish`, and false with only `pause_requested` set; the loomcli budget row grows by one.

**ImpactSummary:** loom's one wiring of the lock-wait pause seam reads a different status field; landingshed's text and docs follow, and batten's own lock wait is untouched.
