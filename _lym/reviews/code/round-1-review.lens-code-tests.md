```yaml
lens: code-tests
outcome: reported
```

Lens: code-tests. For each behaviour the diff adds or changes, I looked for the test row that would fail if it broke.

Per behaviour:

1. **Sync fails after the records commit landed, then the push fails.** The verb refuses with the sync refusal, and the envelope carries `push_error`. An owed failed-push record is written and the transition commit stays on the records HEAD.
   Pinned by `TestResetCmd_RecordsTheRunTransitionAtTheNewTip/SyncFailingAfterTheCommitLeavesAnOwedRecord`, at `internal/webstercli/reset_test.go:189` and `:221-253`.
   Without the fix no push runs, so `push_error` is nil, no owed record exists, and the `:232` assertion fails.
2. **Sync fails after the commit landed, then the push succeeds.** The verb refuses without `push_error`, and the commit reaches origin.
   No test. The new row always sets `unreachable: true`, and the `ls-remote` origin check at `:235-241` runs only for reachable rows.
3. **Sync fails after the commit landed, and the push finds a hand-over.** The refusal carries the hand-over in `push_error`, and no warning is logged.
   No test. Nothing in `internal/webstercli` tests asserts `ErrHandedOver` or "handed over".
4. **Sync fails before any commit landed.** The verb refuses with no push.
   No reset row reaches it. Its code is unchanged apart from the new `!committed` guard; the `:232` check holds `push_error` absent only on reachable rows.
5. **Success path after the extraction into `pushResetRecords`:**
   - An unreachable origin puts `push_error` in the envelope and writes an owed record: pinned by `UnreachableRecordsOriginLeavesAnOwedRecord` (`:188`, `:232`).
   - A reachable origin pushes the commit: pinned by `Start` and `BatchStartWithNothingToClear` (`:235-241`).
   - A hand-over fails the verb: no test, and none existed at the review base either. This branch was moved, not added.
   - A nil `pushRecords`, `SkipGit` or `SkipPush` skips the push: `SkipGit` is covered only indirectly. `TestResetCmd` builds its fixture with `SkipGit` (`reset_test.go:264`), where `fabricSync` returns `(false, nil)` and no push runs. No row asserts that `SkipPush` skips the push.
   - The warning log is human-read output, so it needs no test.

Findings:

- **MEDIUM · design · `internal/webstercli/reset_test.go:189`.** Two behaviours the diff adds on the sync-failure path have no row: a push that succeeds (refusal without `push_error`, commit on origin), and a push that finds a hand-over (reported in `push_error` beside the sync refusal).
  - Evidence: `reset.go:200-205` adds both outcomes. The only new row is `unreachable: true` (`reset_test.go:189`), and nothing in `internal/webstercli` tests asserts `ErrHandedOver` (grep).
  - Fix: add a reachable-origin row with `syncFailsAfterCommit`, asserting the refusal, no `push_error`, and origin's records branch at HEAD.
  - Caveat: the detached push that `commit` spawns (`internal/fabricengine/commit.go`, `spawnDetachedPushFn`) may land the same commit, so the row has to be checked to fail without the fix.
- **LOW · design · `internal/webstercli/reset.go:196-198`.** The `!committed` branch has no row: a sync that fails before any commit landed must refuse with no push and no `push_error`.
  - Evidence: no reset test refuses on a sync failure except the new row (grep for "fabric sync failed" in `internal/webstercli/*_test.go`). That row reaches only `committed == true`.

Tool calls:
1. Grepped the `internal/webstercli` tests for hand-over, skip and `pushRecords` coverage, the fabricengine test hooks for `spawnDetachedPushFn`, and read the head of `newResetFixture`. My context did not show which tests reach these branches.
2. Grepped the tests for "fabric sync failed", `push_error` and `ErrHandedOver`, to check whether any row reaches the not-committed and hand-over branches.
