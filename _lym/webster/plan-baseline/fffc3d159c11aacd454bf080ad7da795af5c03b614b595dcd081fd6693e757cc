# Card 4 — protected-push-class

**Create:**
- `plan:internal/fabricengine#PushFailureProtected` -> `const PushFailureProtected PushFailureClass = "protected"`
- `plan:internal/fabricengine#protectedBranchMarkers` -> `var protectedBranchMarkers = []string{"protected branch", "repository rule violations"}`
- `plan:internal/fabricengine#ProtectedBranchRefusal` -> `func ProtectedBranchRefusal(err error) bool`
- `plan:internal/fabricengine#IsPushProtected` -> `func IsPushProtected(err error) bool`

**Edit:**
- `internal/gitrepo#mapPlainPushError`
- `internal/fabricengine#rejectionRetryKind`
- `internal/fabricengine#pushJob.classifyFailure`
- `internal/fabricengine#FailedPushWayForward`
- `internal/landingshed#Finalize.pushParent`
- `internal/fabricengine_test#TestPushFailureClasses`
- `internal/fabricengine_test#TestFailedPushRecord`
- `internal/fabricengine_test#TestIsPushOwed`

**Delete:**
- `internal/landingshed#protectedBranchRefusal`
- `internal/landingshed#protectedBranchMarkers`

**Extends:**
- `internal/fabricengine/doc.go`
- `internal/gitrepo/doc.go`

**Uses:**
- `internal/fabricengine#oneRepositoryAt`
- `internal/landingshed/doc.go`
- `internal/landingshed#TestFinalize_MergeMarkPushOrderAndVerdict`
- `internal/landingshed#TestFinalize_PullRequest`

**Intent:** D2 and D7: the protected-branch match moves from `landingshed` into fabric's push classification, and a push origin refuses as a protected branch records a failed push of the new class `protected`.

- `mapPlainPushError` keeps git's rejection line in the `ErrPushRejected` error and appends the push's `remote:` lines after it,
  so a hook's own refusal text, such as GitHub's `GH006: Protected branch update failed`, reaches the classifier.
  Assertions that read the rejection line through `strings.Contains`, such as `TestPushFailureClasses`' `(pre-receive hook declined)` checks, keep passing.
- `ProtectedBranchRefusal(err)` is the moved `protectedBranchRefusal`: it lowercases `err.Error()` and matches `protectedBranchMarkers`, the two markers `landingshed` holds today, unchanged.
  GitHub's rejection line `(protected branch hook declined)` matches it as it does today.
- `rejectionRetryKind` stops on a refusal `ProtectedBranchRefusal` matches, so a protected refusal is not retried through the rejection backoff.
- `pushJob.classifyFailure` checks it ahead of the auth and rejection cases, so a refusal it matches records `PushFailureProtected` with git's error,
  but only for a push of a one-repository prime's branch: `oneRepositoryAt(j.path)` holds and `j.path` is that repository's main worktree.
  Every other refusal `ProtectedBranchRefusal` matches, a task branch's or a two-repository weft's, keeps the class it gets today,
  so `IsPushProtected` and card 11's `Protected` read true only for the prime, and a run anywhere else halts at its transition as today.
  `FailedPushesOwedBy` and `IsPushOwed` do not read the class as owed: neither lists it, and this card leaves both as they are.
- `IsPushProtected(err)` is true when `err` holds at least one `*PushFailedError` and every one has class `PushFailureProtected`, read through `pushFailuresOf` as `IsPushOwed` reads them. Card 11's transition seam reads it.
- `FailedPushWayForward` gains the `PushFailureProtected` case: the branch refuses direct pushes; way forward: allow direct pushes to `<branch>` for the account lym pushes as, then run the record's push step, after which the next push carries what is unpushed.
- `Finalize.pushParent` calls `fabricengine.ProtectedBranchRefusal` where it called the deleted helper, so its stuck and cancelled reasons and their tests are unchanged.
  The two `TestFinalize_*` tests listed under Uses keep passing as they stand: their refusals match the same markers.

Tests:
- `TestPushFailureClasses` (tier1) gains the row `AProtectedBranchRefusalIsRecordedProtectedAndIsNotOwed`:
  the fixture's code prime commits `.lym-anchor` at its root, so `oneRepositoryAt` reads it as a one-repository prime,
  and its bare origin carries a `pre-receive` hook that prints `GH006: Protected branch update failed for refs/heads/<branch>.` and exits 1, which git relays as a `remote:` line;
  the push returns a `*PushFailedError` of class `protected` after one attempt, which `IsPushProtected` reads true and `IsPushOwed` false, and the record's way forward names allowing direct pushes.
  It also gains the row `AProtectedRefusalOutsideAOneRepositoryPrimeKeepsTodaysClass`: the fixture's two-repository weft prime refused by the same hook records the class it gets today, not `protected`, and `IsPushProtected` reads false.
  Card 14's `TestPrimePushGuard` pins the same bound for a one-repository task branch.
- `TestFailedPushRecord` (tier0) gains a row asserting `FailedPushWayForward` of a `protected` record names the branch and allowing direct pushes,
  and its row `OwedByListsTheRunsOwedClassesNewestFirst` gains a `protected` record naming the run, which `FailedPushesOwedBy` leaves out.
- `TestIsPushOwed` (tier0) gains a `wantProtected` column asserted through `IsPushProtected` on every row, and rows `Protected` (not owed, protected) and `ProtectedJoinedWithTransport`, kept for its `wantProtected` false.

**ImpactSummary:** Every fabric push classifies one more class and a rejected push's error carries the remote's lines; landingshed's one caller of the moved predicate keeps its behavior, and the two deleted landingshed members have no other caller.
