# Card 5 — failed-push-core

**Create:**
- `plan:internal/fabriccore#VerbDirs` -> `type VerbDirs map[string]string`
- `plan:internal/fabriccore#MergeFailedPush` -> `func MergeFailedPush(current fabriclayout.FailedPush, found bool, next fabriclayout.FailedPush, now time.Time) (merged fabriclayout.FailedPush, notify bool)`
- `plan:internal/fabriccore#FailedPushNotice` -> `func FailedPushNotice(f fabriclayout.FailedPush, dirs VerbDirs) fabriclayout.PushNotice`
- `plan:internal/fabriccore#FailedPushWayForward` -> `func FailedPushWayForward(f fabriclayout.FailedPush, dirs VerbDirs) string`
- `plan:internal/fabriccore#PushStep` -> `func PushStep(f fabriclayout.FailedPush, dirs VerbDirs) string`
- `plan:internal/fabriccore#CodeDivergedWayForward` -> `func CodeDivergedWayForward(worktree, branch string, prime bool, dirs VerbDirs, unmade ...string) string`
- `plan:internal/fabriccore#RecordsConflictWayForward` -> `func RecordsConflictWayForward(worktree, branch string, paths, commits []string, pushVerb string) string`
- `plan:internal/fabriccore#RewoundWayForward` -> `func RewoundWayForward(worktree, branch, originHead string, dropped []string, dirs VerbDirs) string`
- `plan:internal/fabriccore#IsOwedClass` -> `func IsOwedClass(class fabriclayout.PushFailureClass) bool`
- `plan:internal/fabriccore#OwedBy` -> `func OwedBy(records []fabriclayout.FailedPush, slug string) []fabriclayout.FailedPush`
- `plan:internal/fabriccore#RefusedPushStands` -> `func RefusedPushStands(records []fabriclayout.FailedPush, recordsBranch, codeBranch string) bool`
- `plan:internal/fabriccore_test#TestFailedPushWayForward` -> `func TestFailedPushWayForward(t *testing.T)`
- `plan:internal/fabriccore_test#TestMergeFailedPush` -> `func TestMergeFailedPush(t *testing.T)`
- `plan:internal/fabriccore_test#TestFailedPushFilters` -> `func TestFailedPushFilters(t *testing.T)`
- `plan:internal/fabricengine#verbDirsOf` -> `func verbDirsOf(worktrees ...string) fabriccore.VerbDirs`

**Edit:**
- `internal/fabricengine#recordPushFailure`
- `internal/fabricengine#FailedPushesOwedBy`
- `internal/fabricengine#FailedPushWayForward`
- `internal/fabricengine#pushStep`
- `internal/fabricengine#recordsConflictWayForward`
- `internal/fabricengine#codeDivergedWayForward`
- `internal/fabricengine#rewoundWayForward`
- `internal/fabricengine#isOwedClass`
- `internal/fabricengine#RefusedPushStands`
- `internal/fabricengine_test#TestFailedPushRecord`
- `internal/fabricengine_test#TestRefusedPushStands`

**Delete:**
- `internal/fabricengine#unionRuns`
- `//internal/fabricengine/wayforward_test.go`

**Extends:**
- `internal/fabriccore/doc.go`

**Uses:**
- `internal/fabricengine#lymVerbDir`

**Intent:**
The failed-push record's rules move to `internal/fabriccore` (new file `failedpush.go`); the record file, its lock and the notifier call stay in the engine.

- `MergeFailedPush` is the merge `recordPushFailure` runs under the record's lock today:
  a first failure starts the count at 1 with `FirstSeen` now;
  a later one keeps `FirstSeen` and `Notified`, bumps the count and unions the runs, existing first;
  `LastSeen` is now.
  `notify` is true only for a record that is neither notified nor of class deferred.
- `FailedPushNotice` builds today's notice line, topic and run from the record and its way forward.
- `FailedPushWayForward`, `PushStep`, `CodeDivergedWayForward`, `RecordsConflictWayForward` and `RewoundWayForward` are today's text functions, word for word.
  Where today's code calls `lymVerbDir(worktree)`, the core reads `dirs[worktree]` and falls back to the worktree itself when absent, as `lymVerbDir` falls back today.
  An empty worktree stays empty.
- `IsOwedClass` is today's `isOwedClass`; `OwedBy` is `FailedPushesOwedBy`'s filter and newest-first sort; `RefusedPushStands` is `RefusedPushStands`' loop over the records.

The engine keeps every exported signature.
`verbDirsOf` maps each non-empty worktree to `lymVerbDir` of it, the one disk read the texts need, so the snapshot fact is read before the rule.
`recordPushFailure` calls `MergeFailedPush` inside `state.UpdateJSON` and posts `FailedPushNotice` through the notifier when `notify` holds; a notifier error or false answer leaves `Notified` false as today.
`FailedPushWayForward`, `pushStep`, `recordsConflictWayForward`, `codeDivergedWayForward` and `rewoundWayForward` become one-line calls of the core functions with `verbDirsOf` of the worktrees they name,
so their callers in `pull.go`, `bolt.go`, `landing.go` and `anchorstop.go`, other concerns' files, change nothing (D9).
`isOwedClass` calls `IsOwedClass`; `FailedPushesOwedBy` and `RefusedPushStands` read the records and the branches, then call the core filters.
`unionRuns` goes with the merge.

Tests (the commit message lists them, per the overview's Shared Decisions):
- `TestFailedPushWayForward` (core, tier0) takes every `WayForward_*` row of `TestFailedPushWayForward`'s engine twin and of `TestFailedPushRecord`, and the two `pushStep` rows of `wayforward_test.go`, with `VerbDirs` set as a row needs.
  `wayforward_test.go` is deleted in this card, since its rows are held here.
- `TestMergeFailedPush` (core, tier0): first failure; a later failure keeps first-seen and notified and bumps the count; runs union in order; deferred never notifies; notified never notifies again;
  and the `FailedPushNotice` rows taken from `TestFailedPushRecord`'s `NoticeTags_*` rows: a pair code branch's run is the worktree's base name, a records branch's the code worktree's,
  and the prime code branch, a records branch with no paired code worktree and a Bolt branch carry no run; each row asserts today's line and topic.
- `TestFailedPushFilters` (core, tier0): owed classes and `OwedBy`'s filter and order; `RefusedPushStands` over no record, a refused record of the records branch, of the code branch, an owed record and another branch's record.
- `TestFailedPushRecord` keeps the rows that need the record file or the notifier:
  `FirstFailureMailsOnceAndLaterOnesOnlyUpdate`, `NotPostedLeavesNotifiedUnsetSoTheNextFailureMails`, `NotifierErrorLeavesTheRecordWrittenAndUnnotified`, `ClearRemovesTheRecordAndASecondClearIsNil`, `ListOverAnAbsentDirectoryIsEmptyAndSortsByRepoAndBranch`, `UnknownKeySurvivesAnUpdate`,
  and the anchored `WayForward_*` rows plus `InvalidAnchorMarkerFallsBackToTheRoot`, which pin `verbDirsOf`'s read of the anchor marker.
  It deletes the other `WayForward_*` rows, `DeferredWritesTheRecordAndNeverMails`, `RunsRoundTripAndUnionAcrossFailuresOfOneBranch`, `OwedByListsTheRunsOwedClassesNewestFirst` and the `NoticeTags_*` rows, held by the core rows above.
  Its `StopWayForward_*` rows stay for card 9, and the `WeftDirtyStop`/`WeftConflictStop` rows belong to the pull concern and stay.
- `TestRefusedPushStands` keeps `NotFastForwardRecordOfThePairsCodeBranch`, which reads both branches of a real pair, and deletes its other rows, held by `TestFailedPushFilters`.

The three core tests add about 1 second to one tier0 run.

**ImpactSummary:** Rules move behind unchanged exported signatures; the engine's text helpers become delegations, so the other concerns' callers and every caller outside fabric are untouched.
