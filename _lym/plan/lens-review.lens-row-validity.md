```yaml
lens: row-validity
outcome: reported
```

Row-validity lens: whether each planned test row reaches its branch, would fail if the code were wrong, and is needed.

**Findings**

1. **MEDIUM — Card 6, `TestRemove_ArchiveOrder` row 3 (retry on "missing necessary objects"): the row sleeps out a production delay.**
   - The hook refuses the first push attempt, so `PushWithRetry` sleeps for real before the second attempt.
   - `tagAndPushArchiveTip` passes `nil` as the sleep (`internal/fabricengine/archive.go:131`), and the first delay in `archiveTagPushDelays` is 2 s (`archive.go:22-23`).
   - This breaks `PATTERN-test-speed`, "No test in any tier waits out a production duration".
   - Fix: card 6 makes the delays or the sleep injectable, the same way it adds `archiveTagPushHook`, and the row passes a short one.

2. **MEDIUM — Card 6, `TestRemove_ArchiveOrder` row 2 (diverged origin tip, then "a later Add reads the pair not live"): the assertion cannot fail.**
   - The row removes with `remote` true, so the removal deletes origin's weft branch (`removeWeftWorktree`, `internal/fabricengine/weftwiring.go:222-249`).
   - A later `Add` then finds no origin tip and forks whether or not the second archive tag exists (`probeWeftLeftover`, `remoteleftover.go:88-89`).
   - The tag over the diverged tip is what D4's criterion is about, and the row never depends on it.
   - Fix: remove with `remote` false, so the diverged branch stays on origin and only the second tag makes `Add` read the pair not live.

3. **MEDIUM — Card 9, the new rows in `TestReconcileOrigin_Judge`: the existing helper cannot drive two of them.**
   - The package helper `reconcile` always passes `fabricengine.OriginSeams{}` and fails the test on any error (`internal/fabricengine/originreconcile_integration_test.go:96-101`).
   - So the owned-elsewhere row cannot give the `OwnerOf` seam or an origin Bolt ledger. Those come from the `publishBoard` and `originBoard` fixtures (`originreconcileended_integration_test.go:58,68`) and the moves test's `OwnerOf` setup (around `originreconcilemoves_integration_test.go:318`), none of which the card lists.
   - The "without the option" row expects an error, which the helper turns into a test failure.
   - Fix: list those fixtures under `**Uses:**`, and have both rows call `fabricengine.ReconcileOrigin` directly.

4. **MEDIUM — Card 12, `TestBattenIntegration_GoneTaskWorktree` Land row: the task branch is gone only if the test lands it first, and the row does not say so.**
   - The row expects the stop to name "the task branch at its parent".
   - `Remove` deletes the local task branch only when its work is on another ref or landed on the parent (`deleteWarpBranch`, `remove.go:662-688`).
   - It deletes origin's task branch only with `remote` set and the work landed (`deleteTaskBranchOnOrigin`, `remove.go:694-731`).
   - Without both, `TaskBranchGone` reads false and the restore step never appears.
   - Fix: the row squash-lands the task work on the parent and removes with `remote` true; `squashLand` and `pushedPair` already do this (`remove_remotetask_integration_test.go`).

5. **MEDIUM — Card 12, integration row "A re-adoption failing on an unreachable origin": `Add` is never reached.**
   - With the origin URL pointed at a missing path, the first reconcile's `ls-remote` fails before `Add` runs.
   - That failure is not a transport timeout, so `ReconcileOrigin` returns it as an error (doc at `originreconcile.go:700`), and card 10's step 1 refuses the pass.
   - D2's criterion is a re-adoption whose `Add` fails on an unreachable origin. That needs origin to answer the reconcile and then fail `Add`'s probes, for example through a hook or seam between the two.
   - Without that, the row only repeats card 10's reconcile-error refusal.

6. **MEDIUM — Card 13, the `TestBattenFollowUp` subtest that drives `Teardown.Run`: the fake it needs is not on the card.**
   - The subtest uses a `battenRecord` fake on the package's `fakeSubstrates`, but `internal/pairteardown#fakeSubstrates` is not under `**Edit:**`.
   - The field is new, so the fake must change.
   - A test of `Run` filling `Result.Warning` is a different behavior from the `BattenFollowUp` wording table. It fits better as a row of the existing tests that drive `Teardown.Run` over `fakeSubstrates` in `internal/pairteardown/teardown_test.go` than as a subtest of the wording table.

7. **LOW — Card 12, the tier-0 rows assert "`CommitStatus` called once with `failed`", but `newFakeReceiver` leaves `CommitStatus` unset.**
   - `newFakeReceiver` sets only `StatusPath`, `LockPath` and `StatusLockPath` (`internal/battencli/run_test.go:48-51`).
   - Each row must install a recording `CommitStatus`. Otherwise the "nil commits nothing" branch passes silently and the assertion has nothing to count.
   - The card should say the rows set it.

8. **LOW — Card 12, Land rows at tier 0 depend on a real path lookup.**
   - The `*ErrRemoteLeftover` way forward fills `<records repo>` from `fabricengine.RecordsRepoRoot` over the hand-built location (`HubPath` a temp dir, `RepoName` "example").
   - If `RecordsRepoRoot` resolves the prime from disk, these tier-0 rows error instead of reaching the text.
   - Fix: a `RecordsRoot` seam on `readoptSeams`.

9. **LOW — Test economy, rows that repeat a covered behavior:**
   - Card 7, row "No hook: the branch at the read tip is deleted": `TestRemove_RemoteTrueDeletesWeftBranchOnRemote` already covers it (`removeremote_integration_test.go:27`). Turn it into an assertion there, that the delete is leased.
   - Card 10, integration rows "prime with tracked dirt" and "stale registration pruned": these repeat card 4's `TestAdd_AdoptOnly` rows, which drive the same `Add` path. The batten-level value is only the prune call, which the tier-0 row already orders.
   - Card 4, row "adopted local warp branch under a stale tracking ref": card 10's D8 task-branch integration row repeats it.

**Tool calls**
- Grepped the gitrepo, tendercli and pairteardown test and type names, because the context had only the subagents' summaries.
- Grepped the lock package's lock kind, `newFakeReceiver`'s fields and the reconcile test helper's signature, because the context had no bodies of these.
