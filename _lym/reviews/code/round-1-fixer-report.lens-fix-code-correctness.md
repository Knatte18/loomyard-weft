```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the fix's code changes for correctness through the fix-code-correctness lens.

**Findings**

1. **LOW** — `TestRunNoticeTags` (`internal/beamcli/dobbydeps_test.go`) and `TestRunNotifyTags` (`internal/battencli/dobbynotice_test.go`) pass with the defect in place.
   - They pin only the extracted helpers `runNoticeTags` and `runNotifyTags`, and the review's gap is the call sites.
   - The `runNotices.Post` closure in `hubDobbyDeps` could go back to inline or wrong tags, or `notifyRunParent` could pass `PostTags{}`, and both tests would stay green.
   - Evidence: neither test calls the closure or `notifyRunParent`. Each asserts a constant-returning function against its own literal.
   - F3's callers `NotifyParent` and `NotifyRunFrom` are pinned end to end by `TestCreelNoticeTags/NotifyParentMailIsDroppedWithThePairsSlug`. The beamcli and battencli call sites are not.

Everything else I traced holds:
- **`dropRunMail`:** it behaves as the old closure did. A client error returns 0 with the error, and `DropRun` returns the count of dropped messages. `InstallRunMailDropper` still passes it to `pairteardown.InstallRunMailDropper` with the same type.
- **`startCreelDaemon` busy receiver:** the added `SetIdle(receiver, false)` is local to the orchcli test helper, and the beamcli copy is separate. The tier 1 orchcli run was green after the change.
- **`TestCreelNoticeTags` subtests:**
  - Superseded subtest: it fails if the notifier drops `Topic`, because the first mail would stay unsuperseded.
  - Dropped-with-run subtest: it fails if `Run` is dropped, since `dropRunMail` would return 0.
  - `NotifyParent` subtest: it fails if `Run` is empty, since `dropRunMail("task1")` would return 0.
  - Together they cover the `Topic` and `Run` mapping in the client and in the daemon's `server.go` dispatch.
- **F5:** `notices.Post == nil` is a correct addition to the seam check.
- **F7:** the new `failedpush_test.go` row has an empty `CodeWorktree` and expects no run, matching `pairSlug`. The doc's "a records record never marks the prime" matches `pushclassified.go:269`, where `Prime` is set only for the code side.
- **F1 and F6:** these are text moves only.

Tool calls made: none; my context already held every changed file and the gate results.
