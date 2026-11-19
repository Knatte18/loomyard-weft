```yaml
lens: fix-code-correctness
outcome: reported
```

Task: fix-code-correctness lens over the F2/F3/F4/F6/F7/F8 change set in `tmuxkit`.

I ran each changed function by hand on one concrete input and found two problems.

**MEDIUM: the rename row of `TestRunCheckingGitEntries` can pass with the create-only filter removed.** The changed text is the `RunMovingTheEntryAwayTakesNoSnapshotForTheMove` row and its `len(snapshotted)` check in `gitentry_test.go`.
- With the `event.Op != "create"` filter removed, the rename of `.git` yields a snapshot first, and the run's `&lt;-snapshotted` consumes that one.
- The run then returns at once and `stop()` closes the watchers.
- The later create event for the real `.git` is dropped by that close. This is the same in-flight drop as F5, since `fswatch.forward` returns on `done` over a pending event.
- If it is dropped, `remaining` is 0 and the row passes with the defect in place, so the pin on the create-only filter depends on timing.
- The run should wait for the snapshot of the create by checking the received dir and counting.
- A deterministic alternative: have the run receive snapshots until it has seen the one taken after its own create, then assert the total count was 1.

**LOW: the `!attributed` finding text is now wrong on Linux.** The changed text is `describeGitEntry`'s `case !sighting.attributed`, unchanged at `gitentry.go:156-157`.
- `snapshotProcesses` now also returns an unattributed sighting on Linux, when `Pids()` is nil or does not list this binary's own pid.
- The line still says "the processes running then cannot be read on this platform". The package doc and `PATTERN-test-isolation.md` were updated to the wider bound, and this message was not.
- `TestCheckNewGitEntries` pins the old wording through the `cannot be read on this platform` assertion.

Everything else I stepped through held:
- `checkNewGitEntries` reports each sighting and fails on any attributed `ownInDir` one. A seen-but-already-removed entry skips the removal branch correctly.
- `classifyProcesses` returns `ownInDir` as intended, and `descendsFrom` returns true when `pid == self` even if self is missing from the table.
- The looping-chain row terminates through the `seen` guard.
- The long-argv rows compute `["` plus 198 characters and `…`. A byte cut gives 99 `é`, so the multibyte row fails with the defect.
- The missing-directory row skips the failed `fswatch.Watch` and still sees the other directory.
- The caller `runCheckingGitEntries(os.Stderr, shared, snapshotProcesses, m.Run)` in `tmuxkit.go` still matches the signature on both the Windows and non-Windows paths.
- The import lists of `tmuxkit_test.go` and `gitentry_test.go`, and the build tag header of `tmuxkit_snapshot_other_test.go`, are correct. This matches `go vet` and the tier0 and tier1 gates passing.

Tool calls: none; the context already held every changed file.
