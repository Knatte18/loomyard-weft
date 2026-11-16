# Card 10 — batch-record

**Create:**
- `plan:internal/weaverengine#recordInputs` -> `type recordInputs struct`
- `plan:internal/weaverengine#endAttempt` -> `func endAttempt(in recordInputs) (recordOutcome, error)`
- `plan:internal/weaverengine#recordOutcome` -> `type recordOutcome struct`
- `plan:internal/weaverengine#commitRecords` -> `func commitRecords(commit func(msg string) error, label, reentryStep string) error`
- `plan:internal/weaverengine#TestEndAttempt` -> `func TestEndAttempt(t *testing.T)`

**Extends:**
- `internal/weaverengine/doc.go`

**Uses:**
- `internal/runplan#PostBatchChecks`
- `internal/runplan#PostBatchInputs`
- `internal/runplan#MapPartition`
- `internal/runplan#BatchDepartures`
- `internal/runplan#Distill`
- `internal/runplan#LaterTargetRemovedError`
- `internal/runplan#CardNotDoneError`
- `internal/websterengine#Git`

**Intent:** End every attempt the same way, and check a recordable one with the shared post-batch checks (D3, D6, D8, D9, D11).

- `endAttempt` takes the geometry, state, plan, the batch with its record, the attempt, its verdict (card 9) and its parsed report when there is one.
  It runs for every attempt that ends, whatever its verdict, so a failed, lost or needs-parent attempt keeps its journal entry and its commit as a done one does.
- For a `recordable` verdict it first runs, in order:
  the clean-tree check (`DirtyPaths` empty, else `failed` naming the uncommitted paths, which it records in the attempt's `Uncommitted` and the next attempt is told);
  the report's `head_sha`, resolved to a full SHA through the git surface, equal to HEAD, or HEAD a merge commit whose first parent it is and that `Git.MergeRejection` accepts against the told parent branch (a clean parent merge-in), else `failed`;
  `runplan.PostBatchChecks` with `Batches` the recorded partition mapped onto the current plan (`runplan.MapPartition` with Weaver's re-entry way forward), the report's renames, and `HandBackRemovedTargets` set with a way forward naming the next attempt.
  A `*CardNotDoneError` or `*LaterTargetRemovedError` is `failed` with its reasons, so a batch that removes a symbol a pending card names is refused under Weaver too.
  A pass marks the batch record terminal done with `runplan.Distill`'s digest, its deviations, departures (`runplan.BatchDepartures`) and card trail.
- Then, for every verdict, it records the attempt's outcome and cause, the batch's end commit (HEAD) in `BatchEnds` and `LastEnd`, appends the report's `journal` under `JournalHeading` (card 7) when a report parsed, saves the state under the state-mutation lease, and commits through `commitRecords`.
  The journal append runs before the state save, so a crash between them re-appends nothing (the heading key).
- `commitRecords` calls the told commit seam, which stages `_lym/weaver/` and the plan directory through fabric (card 18), with the message `weaver: batch <label>`.
  A commit failure is returned naming the way forward: the fault is transient, re-step the row.

`TestEndAttempt` is tier0, over a fake `Git`, a fake code index and a recording commit seam:
a clean `OK` attempt records done, appends its journal, records the end commit and commits `_lym/weaver` with the plan;
a `FAILED` attempt appends its journal entry, records its outcome and end commit and commits;
a dirty tree fails naming the paths and records them as `Uncommitted`; a head mismatch fails; an abbreviated `head_sha` and a clean parent merge on top of it are accepted;
a done-check finding fails with its reason; a removed later target fails;
recording the same attempt twice appends one journal entry; a commit failure is returned naming the re-step.

**ImpactSummary:** Only adds to the new weaverengine package's doc; no existing symbol changes.
