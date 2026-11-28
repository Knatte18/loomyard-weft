# Card 10 — origin-ended-core

**Create:**
- `plan:internal/fabriccore#SeamTeardown` -> `const SeamTeardown = "teardown"`
- `plan:internal/fabriccore#EndedFacts` -> `type EndedFacts struct`
- `plan:internal/fabriccore#EndedTeardowns` -> `func EndedTeardowns(snapshot Snapshot) Answer`
- `plan:internal/fabriccore#EndedEntries` -> `func EndedEntries(snapshot Snapshot) []fabriclayout.EntryOutcome`
- `plan:internal/fabriccore#MergeEndedEntries` -> `func MergeEndedEntries(classified, ended []fabriclayout.EntryOutcome) []fabriclayout.EntryOutcome`
- `plan:internal/fabriccore#WithoutRemovedPairs` -> `func WithoutRemovedPairs(branches []fabriclayout.BranchOutcome, entries []fabriclayout.EntryOutcome) []fabriclayout.BranchOutcome`
- `plan:internal/fabriccore#StalePushFailures` -> `func StalePushFailures(records []fabriclayout.FailedPush, local map[string]bool) []fabriclayout.FailedPush`
- `plan:internal/fabriccore_test#TestEndedTeardowns` -> `func TestEndedTeardowns(t *testing.T)`

**Edit:**
- `internal/fabriccore#Snapshot`
- `internal/fabricengine#ReconcileOrigin`
- `internal/fabricengine#reconcileEnded`
- `internal/fabricengine#recordedRuns`
- `internal/fabricengine#clearStalePushFailures`

**Delete:**
- `internal/fabricengine#mergeEndedEntries`
- `internal/fabricengine#withoutRemovedPairs`

**Extends:**
- `internal/fabriccore/doc.go`
- `internal/fabricengine/doc.go`

**Uses:**
- `internal/fabricengine#runRule`
- `internal/fabricengine#OriginSeams`
- `plan:internal/fabricengine#RunRecordPaths`
- `plan:internal/fabricengine#StatusCodec`

**Intent:**
The removal of ended runs becomes a core rule whose teardown is a seam-call action, and `originreconcileended.go` stops importing `internal/shedrun` (D3, D13).

- `recordedRuns` lists the run directories through `RunRecordPaths.RunsRoot` and `Reserved`, in place of the shedrun calls.
  It reads each status file's bytes at `RunRecordPaths.StatusFile` and asks `StatusCodec.Recorded`, in place of `readRunStatusRows`, so fabric decodes no shed record (D13):
  an absent file is not recorded, and a verdict error fails the reconcile wrapped as "decode <path>".
  The verdict parses shed's whole `Status`, not fabric's narrow `runStatusRows`, so its error text is the codec's,
  and a record whose field only shed's `Status` declares has the wrong type now fails where the narrow decode read it recorded; that judgment is the codec's, as D13 hands it to shed.
  Card 9's pre-lock refusal guarantees both seams are installed when it runs.
  `readRunStatusRows` stays for its callers in `replay.go`, the replay concern's.
- `EndedFacts` (in `Snapshot.Ended`) holds, per ended slug in slug order, its how, whether this machine holds a part of its pair, whether an earlier snapshot tag exists, and the scope; and, once run, each teardown's report: tags, kept, busy and on what, error, mutations.
- `EndedTeardowns` answers one `ActionSeamCall` with `SeamTeardown` and the slug in `Ref`, for the first entry not yet torn down that has a local part, outside a judge-only scope, not the scope's own run, and with a teardown seam installed.
  The list ends at the seam call, since the next answer reads its report (D3); a failing teardown stops no other entry.
  With nothing left it answers done.
- `EndedEntries` is `reconcileEnded`'s per-entry outcome over those facts: ended with its how and local flag; busy with `busyWayForward`'s line; an error on the entry; removed; or untouched with no teardown.
  `busyWayForward`'s two lines move with it word for word.
- `MergeEndedEntries` and `WithoutRemovedPairs` are today's helpers word for word.
- `StalePushFailures` returns the records whose branch is not local, `local` keyed by repository path and branch, which `clearStalePushFailures` reads and then clears.
- The engine's runner for `SeamTeardown` calls `seams.Teardown` with today's `EndedTeardown`, appends the report's mutations to the reconcile's record and adds the report to the snapshot.
  `EndedTeardowns` runs in its own `runRule`, after `OriginJudge`'s has returned and released the remote-refs locks, with a nil lock, as `reconcileEnded` runs today;
  the reads of local parts and snapshot tags stay engine reads, that rule's `PhasePreLock` reads.
  The printed mutations of a reconcile are unchanged (D4, D5).

Tests (the commit message lists them):
- `TestEndedTeardowns` (core, tier0) is the acceptance row of a seam call whose report feeds the next answer:
  two ended local entries answer the first teardown, then, given its busy report, the second, and `EndedEntries` reads busy then removed;
  a failing first teardown still answers the second;
  judge-only, the scope's own run, no local part and no seam answer no call and report the entry ended untouched;
  `MergeEndedEntries` sorts and overrides classified entries; `WithoutRemovedPairs` drops a removed pair's branches; `StalePushFailures` keeps only branches not local.
  It adds about 1 second to one tier0 run.
- `TestReconcileOrigin_Ended` passes unchanged here: it pins the ended set the board's replay reads, and moves to `internal/boardengine` in card 12.
  Its rows whose assertion is only the teardown hand-off or its report (busy, failing teardown, judge-only, run scope, earlier snapshot tag) are deleted in card 12 as held by `TestEndedTeardowns`.

**ImpactSummary:** Private reconcile helpers move to the core; originreconcileended.go drops its shedrun import; ReconcileOrigin's report and the teardown seam's contract are unchanged.
