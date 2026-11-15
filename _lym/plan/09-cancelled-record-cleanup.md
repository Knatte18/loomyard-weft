# Card 9 — cancelled-record-cleanup

**Edit:**
- `internal/battencli#judgeStatus` -> `func judgeStatus(data []byte, prefix, slug string) (candidate, tracked, cancelled bool, reason string)`
- `internal/battencli#cleanupDeps`
- `internal/battencli#fabricCleanupDeps`
- `internal/battencli#judgeRunDirectory`
- `internal/battencli#clearDoneRun`
- `internal/battencli#cleanupSlug`
- `internal/battencli#battenCLI.cleanupCmd`
- `internal/battencli#TestCleanup`
- `internal/battencli#TestClearDoneRun_RemovesADoneRunTheRecipeFilterAdmits`
- `internal/pairteardown#Teardown`
- `internal/pairteardown#New`
- `internal/pairteardown#Teardown.Run`
- `internal/pairteardown#fakeSubstrates.teardown`
- `internal/pairteardown#TestTeardown_CallSequence`

**Uses:**
- `internal/battencli#removeRunRecords`
- `internal/pairteardown#ReadBattenRecord`
- `internal/pairteardown/doc.go`

**Intent:** The other paths that remove a cancelled run's pair or a crash's leftover record, per D3.

`lym batten cleanup` only repairs a record a crash left:
- `judgeStatus` reports `cancelled` for a status in `shedengine.StateCancelled`, with candidate false and no reason; every other state is judged as today.
- `cleanupDeps` gains `PairStands func(slug string) (local, onOrigin bool, err error)`, filled in `fabricCleanupDeps` from the fabric topology:
  local is the task worktree or records worktree present here (`PairParts`), and onOrigin is `TaskBranchOnOrigin`, since a kept pair's task branch stays on origin and its record reads `cancelled` on every machine (D1).
  `fabricCleanupDeps` always fills it, so it has no nil branch; every test-built `cleanupDeps` that reaches a cancelled record sets `PairStands`.
- `judgeRunDirectory` asks it for a cancelled record:
  with neither, the record is a candidate, so `--apply` removes it and commits the removal as for a done run;
  with the pair here, it skips with `run is cancelled and its pair stands; way forward: "lym fabric remove <slug>" removes the pair and the record`;
  with only the task branch on origin, it skips with `run is cancelled and its pair stands on another machine; way forward: "lym batten cancel <slug> --remove" on the machine that owns it removes the pair, its task branch and the record`.
  A failed probe skips naming the error.
- `clearDoneRun` ignores the new flag, so `ClearDoneRun` and `ClearMotifRun` never take a cancelled record and never delete its task branch.
- `cleanupCmd`'s Long text names the cancelled case.

`lym fabric remove <slug>` deletes a cancelled run's record in the same step as the pair:
- `Teardown` gains `removeCancelledRecord func(slug string) (bool, error)`, wired in `New`: it reads the batten record through `ReadBattenRecord` and, only when its state is `cancelled`,
  removes the run directory, commits the removal on the prime's records side when the records HEAD tracks it, and spawns the records push.
  A nil field removes nothing; `fakeSubstrates.teardown` sets a recording fake.
- `Run` calls it after a successful removal and before the follow-up read, so `BattenFollowUp` reads no record of a deleted cancelled run and names no `--remove` for it.
  A failure becomes `Result.Warning` naming `lym batten cleanup --apply`, which finishes it, since the pair is already gone.
  `lym batten cancel --remove` removes the pair through `RemovePair`, not `Run`, and deletes the record itself (card 7), so no record is deleted twice.
- Every `Teardown.Run` caller reaches the step, the origin-reconcile teardown (`teardownEnded`) and the beam fork's member removal as well as `lym fabric remove`,
  so each of their removals of a cancelled pair deletes its record too, as D3 asks of every pair removal.

Tests, tier0:
- `TestCleanup` gains rows: a cancelled record with no pair here and no task branch on origin removed; one with its pair here skipped with the `lym fabric remove` reason; one with only its task branch on origin skipped with the `--remove` reason; a failed probe skipped.
- `TestClearDoneRun_RemovesADoneRunTheRecipeFilterAdmits` gains a cancelled row that is left with its branch.
- `TestTeardown_CallSequence` gains rows: a removal over a cancelled record deletes it after the removal and sets no follow-up warning; over a running record it deletes nothing; a failed deletion sets the cleanup warning.

**ImpactSummary:** Cleanup gains one record case and one dependency; the teardown's Run gains one post-removal step, reached by every `Teardown.Run` caller: `lym fabric remove`, the origin-reconcile teardown and the beam fork's member removal.
