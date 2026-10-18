---
verdict: APPROVED
---

## Measure

Board entry `wait-world-lock-grace-flake` (issue #716): give the wait-world fixture the production lock grace, and check the integration tier's other fixtures for short grace values that make verify nondeterministic.
The decision record has no `Added after Discussion` entries, so the board body is the measure unamended.
Its one open risk (the verifytree and githubclient bounds stay real-time) names no parent correction.

## Diff against the entry

PR 725 at `a85e9c0f5` against `main`, two commits, five files:

- `internal/shedverbs/loop.go`: drops `LoopSpec.LockGrace` and its zero-selects-default branch, so `runLoop` waits `loopLockGrace` for the loop lock.
  The wait-world fixture's 20 ms grace was already removed by `743c11f0d`, which is on `main`; removing the field left without a setter makes the production grace the only one, the entry's first point.
  No `LockGrace` reference remains at the PR head.
- `internal/preflightshed/preflight_integration_test.go`, `internal/lock/lock_test.go`, `internal/verifytree/verify_integration_test.go`, `internal/githubclient/githubclient_test.go`: each short bound on a success path is raised or replaced by the default, and the held-lock row of `TestAcquireWriteLockWithin` keeps its 50 ms expiry.
  This is the entry's second point, the audit of other fixtures.
  `lock_test.go` and `githubclient_test.go` are untagged, but `lyx gate test --tags integration ./...` runs untagged tests too, so they sit in the tier the entry names.

Nothing in the diff goes beyond the two points, and neither point is left out.

## Overlap

- Peers (`own_row` 22): `config-migrate-hub-layer` (row 0), `tmux-conf-concurrent-boot-flake` (row 17), `timing-report-darn` (row 0), `batten-start-speed` (row 10), `creel-hold-log-spam` (row 17), `orch-rules-files` (row 7).
  None of their changed paths shares a file or package with this diff.
  `batten-start-speed` and `orch-rules-files` have no changed paths yet; neither one's decision record or plan touches `shedverbs`' loop lock, the `lock` package's acquire, preflight's lock wait, verifytree's slot gate or githubclient's token timeout.
- Parent commits: `lyx loom peers` lists none.
- Board: `tmux-conf-concurrent-boot-flake` and `darn-verify-tier-unrunnable` concern other flakes (reedengine's tmux boot, darn's tier refusal) and do not overlap this entry.

No overlap, so no landing order applies.

## Superseded

None: no other board entry is made obsolete by this run.
