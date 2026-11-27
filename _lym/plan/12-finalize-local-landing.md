# Card 12 — finalize-local-landing

**Edit:**
- `internal/landingshed#Deps`
- `internal/landingshed#Finalize.Call`
- `internal/landingshed#Finalize.landingTexts`
- `internal/landingshed#Finalize.pushParent`
- `internal/landingshed#Finalize.pushRecords`
- `internal/landingshed#Finalize.markTaskDone`
- `internal/landingshed#Finalize.ownerChecks`
- `internal/landingshed#Finalize.syncParent`
- `internal/landingshed#Finalize.landUnderLock`
- `internal/landingshed/doc.go`
- `internal/landingshed#TestFinalize_MergeMarkPushOrderAndVerdict`
- `internal/landingshed#TestFinalize_StuckReasons`
- `internal/landingshed#TestFinalize_LockFirstAndFastForward`
- `internal/landingshed#TestFinalize_OwnerChecks`
- `internal/loomcli#landingDeps`
- `internal/testkit/envkit#nilLegal`

**Delete:**
- `internal/landingshed#Finalize.archiveHeldParentMerge`
- `internal/landingshed#Finalize.adopterFirstRepair`
- `internal/landingshed#protectedParentWayForward`
- `internal/landingshed#TestFinalize_TransportPushFailure_ReturnsClassifiedError`
- `internal/loomcli#runRecordsPusher`
- `internal/loomcli#parentMergeArchiver`

**Create:**
- `plan:internal/landingshed#ParentPush` -> `type ParentPush struct`
- `plan:internal/loomcli#landLocally` -> `func landLocally(l *lymcwd.Location, slug string) func(tip string) error`

**Uses:**
- `internal/loomcli/doc.go`
- `internal/landingshed#Finalize.handedOverStop`
- `internal/landingshed#recordingParentMerger`
- `plan:internal/fabriclayout#LandingTrailerKey`
- `plan:internal/fabricengine#AheadByOnly`
- `plan:internal/runpusher#RecordLanding`
- `plan:internal/runpusher#ParentTarget`
- `plan:internal/runpusher#TargetRefused`
- `plan:internal/pushledger#OwedTips`
- `internal/loomcli#TestWire_EverySeamFilled`
- `internal/loomcli#TestWire_DarnRecipe`
- `internal/loomcli#TestWire_SamplerRecipe`
- `internal/loomcli#TestWire_BeamRecipe`
- `internal/loomcli#TestLandingDeps_EveryFieldPopulated`
- `internal/battencli#TestWire_EverySeamFilled`
- `internal/testkit/envkit#TestFullEnv_FillsEverySeam`
- `internal/testkit/envkit#TestNilSeams_ReportsExactPaths`
- `internal/testkit/envkit#TestNilSeams_SkipsNilLegalPaths`

**Intent:** D5 and D6: a local landing merges under the parent lock, records its landing entry, hands its push to the pusher and returns done, so no transient push failure fails it.

`Deps` loses `PushRecords`, `ArchiveParentMerge`, `ParentCommitsOriginLacks` and `OriginParentHoldsWork`, whose only reader was the synchronous parent push, and gains:
`LandLocally func(tip string) error`, which commits the run's status records, records the landing entry on the parent target with the run, `tip`, that records commit, the parent branch and a push-now mark, and starts or wakes the parent target's and the pair's pushers;
`ParentAheadByOwed func() (bool, error)`, whether the local parent is ahead of origin by only tips the parent target records as owed;
and `ParentPushState func() (ParentPush, error)`, whose `ParentPush` reports `Owed` and `Refused` with the refusal's way forward, which card 13 reads.
Each nil seam, as a hand-built `Deps` leaves it, reads as no landing entry and no owed tip, so `envkit#nilLegal` lists the three and drops the four keys of the removed fields.
`landingDeps` wires them through `landLocally`, `fabricengine.AheadByOnly` over `pushledger.OwedTips`, and `runpusher.TargetRefused`, the parent target being `runpusher.ParentTarget`'s.

`landingTexts` adds the trailer `Lym-Landing: <slug>` to the landing commit's message, so `fabricengine.LandingCommits` finds the landing on the parent (card 10).
`pushParent` keeps `handedOverStop` before the landing and pushes nothing.
With `PushSkipped` it commits the status records and makes the `done` board write itself, as today, and writes no landing entry.
Otherwise it calls `LandLocally` with the parent head the merge left and returns done: no `done` write while the entry stands (the pusher makes it, card 11), no synchronous records push, no parent push.
`pushRecords` commits the status file and pushes nothing.
`archiveHeldParentMerge`, `adopterFirstRepair` and `protectedParentWayForward` go with the push; the pusher's drop (card 11) takes over the adopter-first case and its refused-push record carries the protected-branch way forward.
`landingDeps` drops `runRecordsPusher` and `parentMergeArchiver`, their seams gone.
The docs that describe the push change with it: `Finalize.Call`'s doc drops the parent push, its protected-branch way forward and "the push is a no-op";
`markTaskDone`'s doc says the write is made only with pushes off; `internal/landingshed/doc.go` says Finalize lands on a parent ahead of origin by owed landing tips and hands its push to the parent target;
and the file comments of `finalize.go`, which says it pushes the parent branch, and `finalize_owner.go`, which words the repair note, drop those clauses.

Every Finalize pass that finds the local parent holding the task's work while origin's parent lacks it records the landing entry, the resume included:
`landUnderLock`, finding the parent already holding the work, reaches the same `pushParent`, whose `LandLocally` replaces the run's landing with the parent's head, so a Finalize killed between the merge and the record still leaves the push owed.

The owner checks wait for origin (D6): the run-scope reconcile's unreachable report in `ownerChecks`, for both `gateUnlocked` and `landUnderLock`, returns a `git-transport` transient error in place of the owner halt,
so the Go driver waits it out and the landing merges once origin answers.
The parent-owner read keeps today's halt, since it reads the local board the reconcile left and touches no network.
`boardengine.ErrOwnerOriginUnreachable` is unchanged, so the catch-up and `lym adopt` keep their behaviour.
`syncParent`'s fetched tips read that fails on transport returns a `git-transport` transient error instead of a plain one.
`syncParent` reads a local parent ahead of origin as level when `ParentAheadByOwed` reads true, so the landing adds its own entry on it;
any other ahead parent keeps today's Stuck reason, a diverged parent too, and a lagging one is fast-forwarded as today.

Tests, tier0 rows, adding about a second to one tier0 run.
`TestFinalize_MergeMarkPushOrderAndVerdict`: the order is merge, then `LandLocally` with the landing commit, then done, with no `push-records`, no parent push and no `mark-done`;
the merge message carries the `Lym-Landing` trailer;
a row where every push would fail as owed still returns done, since nothing pushes;
a `PushSkipped` row commits and makes the `done` write with no `LandLocally`;
the rows of the protected-branch, archive and adopter-first push failures go with their code, and so does `TestFinalize_TransportPushFailure_ReturnsClassifiedError`.
`TestFinalize_LockFirstAndFastForward`: a resume whose parent already holds the work, with origin lacking it, calls `LandLocally` with the parent head;
a parent ahead by owed tips lands with its own entry, and one that `ParentAheadByOwed` reads false for, as with an unrecorded commit below an owed tip, is Stuck as today; the re-step row that pushed the parent before the records goes.
`TestFinalize_OwnerChecks`: an unreachable reconcile returns an error whose transient class is `git-transport` and halts nothing, in the lock-free pass and under the lock, and a transport failure of the fetched tips read does the same.
`TestFinalize_StuckReasons` drops its push-failure rows.

**ImpactSummary:** Removes Finalize's synchronous pushes and four seams across `landingshed`, its `loomcli` wiring and `envkit`, and adds a trailer to the landing commit; every other row of Finalize keeps its steps.
