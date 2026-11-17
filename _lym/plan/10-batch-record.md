# Card 10 — batch-record

**Create:**
- `internal/weaverengine#recordInputs`
- `internal/weaverengine#endAttempt`
- `internal/weaverengine#recordOutcome`
- `internal/weaverengine#commitRecords`
- `internal/weaverengine#saveRunState`
- `internal/weaverengine#checkCleanTree`

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
- `internal/runplan#AcquireStateMutation`
- `internal/websterengine#Git`

**Intent:** End every attempt the same way, and check a recordable one with the shared post-batch checks (D3, D6, D8, D9, D11).

- `endAttempt` takes the geometry, state, plan, the batch with its record, the attempt, its verdict (card 9) and its parsed report when there is one.
  It runs for every attempt that ends, whatever its verdict, so a failed, lost or needs-parent attempt keeps its journal entry and its commit as a done one does.
- `checkCleanTree` returns the worktree's uncommitted tracked paths through `Git.DirtyPaths`, empty on a clean tree.
  `endAttempt` and the re-entry's stopped-batch check (card 13) both call it, so the two paths cannot drift.
- For a `recordable` verdict `endAttempt` first runs, in order:
  `checkCleanTree`, whose paths make the attempt `failed` naming them, recorded in the attempt's `Uncommitted` and told to the next attempt;
  the report's `head_sha`, resolved to a full SHA through the git surface, equal to HEAD, or HEAD a merge commit whose first parent it is and that `Git.MergeRejection` accepts against the told parent branch (a clean parent merge-in), else `failed`;
  `runplan.PostBatchChecks` with `Batches` the recorded partition mapped onto the current plan (`runplan.MapPartition` with Weaver's re-entry way forward), the report's renames, and `HandBackRemovedTargets` set with a way forward naming the next attempt.
  A `*CardNotDoneError` or `*LaterTargetRemovedError` is `failed` with its reasons, so a batch that removes a symbol a pending card names is refused under Weaver too.
  A pass marks the batch record terminal done with `runplan.Distill`'s digest, its deviations, departures (`runplan.BatchDepartures`) and card trail, and clears `StopRecord`, so a re-entered attempt that ends done leaves no stop for a later re-step to re-enter.
- Then, for every verdict, it records the attempt's outcome and cause, the batch's end commit (HEAD) in `BatchEnds` and `LastEnd`, appends the report's `journal` under `JournalHeading` (card 7) when a report parsed, saves the state through `saveRunState`, and commits through `commitRecords`.
  The journal append runs before the state save, so a crash between them re-appends nothing (the heading key).
- `saveRunState` is Weaver's one save of its state from `Run`.
  With `leaseHeld` unset it first acquires `runplan.AcquireStateMutation` over the scratch directory; with it set its caller already holds that lease, and it skips the acquire, since a second acquire in the same process blocks forever.
  Under the lease it loads the state file, keeps each attempt's and fixer's `Delivered` list as loaded merged with the headings `st` holds, writes the result, and leaves `st` holding the merged lists.
  The decision verb's process records deliveries into the same file (card 21), so a save of `Run`'s in-memory copy never drops a heading another process recorded.
- `commitRecords` calls the told commit seam, which stages `_lym/weaver/` and the plan directory through fabric (card 18), with the message `weaver: batch <label>`.
  A commit failure is returned naming the way forward: the fault is transient, re-step the row.

These unexported helpers are tested through `Run`: card 12's `TestRun` holds their rows.

**ImpactSummary:** Only adds to the new weaverengine package's doc; no existing symbol changes.
