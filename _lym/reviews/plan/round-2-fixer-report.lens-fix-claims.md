```yaml
lens: fix-claims
outcome: reported
```

Result: one MEDIUM finding; every other claim the fix added or rewrote holds at HEAD.

**MEDIUM: card 12 says no helper fabric keeps is copied, but several are**
- **Changed text:** `_lym/plan/12-board-test-moves.md`, "Support for the moved tests", first bullet: "no fixture type, notifier fake or second copy of a helper fabric keeps is created in `boardengine_test`".
- **Why it is false:**
  - The card creates `boardengine_test#requireEntry`, but fabric keeps its own `requireEntry` at `internal/fabricengine/originreconcilemoves_integration_test.go:98`. The card's Delete list does not remove it, and that file's kept `TestReconcileOrigin_Moves` rows still call it.
  - The new `pushPrimeRecords`, `commitOnOrigin` and `endEntryOnOrigin` restate `recordsFixture.primeWeft` and `recordsFixture.originCommit` (`internal/fabricengine/replay_integration_test.go:84`, `:95`) and `recordsFixture.endEntryOnOrigin` (`internal/fabricengine/pushclasses_integration_test.go:163`). Fabric keeps all three for its own rows.
- **What fixes it:** the sentence needs narrowing to say that the git-using helpers fabric keeps are restated as plain functions, since a kit may not import `gitkit` (`internal/testkit/enforcement_test.go`, `kitDeniedImports`). Otherwise the copies need handling some other way.

**Claims that hold:**
- **Card 7, the leased reset's refusal:**
  - `UpdateRemoteBranchLeased` wraps a hook refusal as a plain git error (`internal/gitrepo/push.go:402`), and `updateRemoteBranch` wraps it again with no `ErrPushRejected`.
  - `rejectionRetryKind` treats it as named and not retryable, so `settleRejection` returns it unjoined.
  - `classifyFailure` matches no class for it, and `pushClassified` records it as `other` (`internal/fabricengine/pushclassified.go:184`).
  - `protectedParentWayForward` is set in `pushJob.record`.
- **Card 7, the two in-package tests:** all three subtests call `CoalescePushRequest`, and neither test builds a hub.
- **Card 3, the package-`fabricengine` search hits:**
  - `spawn_test.go` only reaches `SpawnDetachedPush` gating, and `spawn_linux_test.go` reaches `startDetachedPush`.
  - `teardownlock_test.go` calls `DetachedPush` on its paused exit (`teardownlock.go:99`).
  - `coalesce_test.go`, `commit_partial_integration_test.go` and `export_test.go` call none of the push or reconcile verbs.
- **Card 12, the rest:**
  - `requireTrackingRefPrunedWithUpstreamSet`'s only callers are the two moved `OwedUnrecorded` rows.
  - The helper callers in other concerns' files are the three named.
  - `tagsOf`, `landOn`, `originHasBranch` and `boardFileOf` sit in other concerns' files.
  - With no notifier, `recordPushFailure` writes the record and mails nothing (`internal/fabricengine/failedpush.go:99`), and nothing in `internal/boardengine` installs one.
  - The prime's `_lym` resolves into its records worktree as a junction (`internal/fabricengine/junction.go:28`), so writing at shed's path for the prime and committing that file there holds.
  - `heddleengine` does not import `boardengine`.

**Tool calls:**
1. Package clause and verb-call count of each package-`fabricengine` search hit, `heddleengine`'s board imports, fabric's `requireEntry`, and `_lym` junction code. My context had the search's file list, not the packages or call counts.
2. The push-verb calls in `spawn_test.go`, `spawn_linux_test.go` and `teardownlock_test.go`, and `DetachedPush`'s paused exit. My context did not show those files.
