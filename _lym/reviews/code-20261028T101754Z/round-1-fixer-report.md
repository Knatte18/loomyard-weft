# Fixer-report, round 1

## Changes

Commits, in order: `5e938cc6` (F5), `8f0a498f` (F1), `cdf8d8fb` (F8), `37c59730` (F2 F3 F4 F6 F7, one commit because the edits shared files), and a last commit for the self-check follow-ups to F6 and F7.

- F1: `internal/fabriccli/hubpull_integration_test.go` has `TestRunHubPull_WaitsForAHeldParentLock`.
  It holds the prime branch's parent lock as another owner, waits on the store for the pull's queued ticket, and asserts the ticket's run id is `fabric-pull`.
  One row releases the holder and asserts the pull finishes with the prime moved.
  The other row cancels the context and asserts only the prime side stopped while the board and the beam still ran.
  `export_test.go` gained `RunHubPullForTest` to pass the context.
  Fixed differently: the run id is read from the waiter ticket, not the lock record, because the record is only observable in the instant between grant and release.
- F2: `TestPullHub_StoppedSideLeavesTheOthersPulled` has a pushed, moved beam after the dirty prime, asserted moved with its file present, beside the no-upstream beam.
- F3: `pullPairSide` returns `up_to_date` with a git-skipped detail when `SkipGit` is set; `TestPullHub_HonorsSkipGitWithEverySideUpToDate` covers it.
- F4: `localBeamLocations` is now `localBeamCandidates`, returning per-slug results; `PullHub` reports a failed slug as a stopped beam side and goes on.
  `TestPullHub_UnreadableBeamRecordStopsOnlyThatBeam` has a corrupt record beside a healthy moved beam.
- F5: the original wording of the two help bullets is restored.
- F6: the moved-sides test has a second beam and a non-beam task pair, asserting beam order and the task pair's absence.
  New rows cover a dirty-board skip stop and a pair whose records origin is unreachable, with the exact detail pinned.
  Not added: a beam owned by another machine, because `Origin` carries no machine field to vary; and a `PullHub` list-error row, because the worktree list of a hub that resolved cannot be made to fail without breaking the fixture.
- F7: fixed differently.
  The proposed bounded poll is still a real-time wait that passes vacuously for a slow ignoring pull (the self-check lenses said so).
  `TestBoltPull_HoldsTheBoardWriteLockWhileItPulls` holds the push lock, which `Bolt.Pull` takes after the write lock, and polls the write lock with a try-acquire until it is observed held.
  It then asserts HEAD unchanged, releases the push lock, and asserts the pull finishes and the write lock is free.
- F8: the `Bolt.Pull` and `PullHub` comments state the skip contract inline.
- Gate repair: the comment above the push lock in `TestBoltPull_HoldsTheBoardWriteLockWhileItPulls` was a fixed-column wrap; it now breaks at the sentence boundary.
- Gate repair: with the original bullets restored (F5), the `pull` Long text with its `--hub` paragraph came to 2184 bytes against the 2000-byte cap of `TestCLIForm_EveryCommand`.
  The Long text now ends with one `--hub` line, and the lock, wait and stop semantics moved into the `--hub` flag description.
- Self-check follow-ups: `fabricengine/doc.go` states the SkipGit and unreadable-record cases and drops the "pull-first path" wording; the `localBeamCandidates` comment says an unreadable record is reported as a beam side.

## Disputed

None.

## Deferred

None.

## Self-check

- fix-code-correctness, MEDIUM, F7 still a fixed real-time wait: fixed with an own change (see F7).
- fix-code-correctness, LOW, unreadable non-beam record labelled a beam: fixed by a comment on `localBeamCandidates`; the label cannot be avoided because the mark is unreadable.
- fix-code-correctness, LOW, F6 detail not pinned: fixed, the exact detail is asserted.
- fix-code-correctness, LOW, worktree order assumed by the tests: no change; the hub pull's contract is `git worktree list` order, which git fixes by name.
- fix-proposal, MEDIUM, F7: same as above, fixed.
- fix-proposal, LOW, module doc stale for SkipGit and unreadable record: fixed in `doc.go`.
- fix-proposal, LOW, mislabelled non-beam pair: same as above, fixed by the comment.
- fix-proposal, LOW, SkipGit reports `up_to_date`: no change; it matches Bolt, and the detail says git was skipped.
- fix-affected-set, LOW, `doc.go` "pull-first path": fixed.
- fix-affected-set, LOW, `doc.go` missing SkipGit and unreadable record: fixed.
- fix-consistency: no findings.
