# Card 5 — pairteardown-read-run

**Create:**
- `internal/pairteardown#ReadRun`
- `internal/pairteardown#runShedRow`
- `internal/pairteardown#runReader`
- `internal/pairteardown#runReader.read`
- `internal/pairteardown#TestReadRun`
- `internal/pairteardown_test#TestReadRun_ReadsTheSameStateWithoutTheCache`

**Edit:**
- `internal/pairteardown#landRow`
- `internal/pairteardown#worktreeTeardownRow`

**Extends:**
- `internal/pairteardown/doc.go`

**Uses:**
- `internal/pairteardown#ReadBattenRecord`
- `internal/pairteardown#OwnerPauseHeld`
- `internal/fabricengine#OriginParentHoldsTaskWork`
- `internal/fabricengine#OriginRunStatus`
- `internal/fabricengine#FetchedTaskBranchOnOrigin`
- `internal/battenshed#FinalizePastMerge`
- `internal/runfsm#Records`
- `internal/runfsm#Inputs`

**Intent:** The one reader of a run's FSM inputs, which `battencli`, `loomcli` and `tendercli` all import; it sits in `pairteardown` because `tendercli` cannot import `battencli`, which imports it, and `pairteardown` already holds the batten-record and owner-pause readers.
`ReadRun` wires a `runReader` over the hub of prime and returns its read; `runReader` holds one function field per substrate, so `TestReadRun` drives every branch with fakes.
`pairteardown` cannot import `battenrecipe`, as `followup.go`'s row-name const block records: the recipe package reaches a package whose tests import this one.
So `ReadRun` names no `battenrecipe` symbol; it reads batten's rows through `runShedRow`, declared in that const block beside `landRow` and `worktreeTeardownRow`,
whose doc comment then says `ReadRun` reads them too.

Records, tracked only:
- the board entry from the local board: `Released` for a task whose status is empty (`boardengine.IsEmptyStatus`), `BoardDone` for `done`;
- batten's record from the prime's `shedrun.StatusFile`, read tolerantly, with `BattenStarted` true when its row is `runShedRow`, `landRow` or `worktreeTeardownRow`, the rows from Run-Shed on;
- `TaskBranchOnOrigin`, read through `fabricengine.FetchedTaskBranchOnOrigin` only when the batten record's state is `cancelled`, since only that rule reads it;
  that read uses the refs the last fetch left, so the dobby, which runs `ReadRun` for every listed run on each tick, makes no network call for a kept cancel and still reads it offline;
- the child record from the task worktree when the pair stands here, else `fabricengine.OriginRunStatus`;
  `ChildLanding` when the child is at `loomshed.NameFinalize` or `done`, or batten is at `landRow` or `worktreeTeardownRow`.

Inputs, without `State`, which `runfsm.Judge` fills:
- `OwnedElsewhere`, `Owner` and `AdoptSubject` from the local Bolt's owners through `hubgeom.ParentOf` and `machinename.Name`, as `boardengine.OwnedElsewhere` judges;
- `PauseCovers` from this machine's pause record coverage of the slug; an adopt pause is such an entry, so it waits the run and `lym tender resume <slug>` lifts it;
- `Held` false: D7's Bolt holds fill it in a later part (Shared Decisions);
- `PastParentMerge` true when batten is at `landRow` or `worktreeTeardownRow`, whose Finalize already merged and pushed the parent, as `childReachedParent` counts them;
  otherwise from `battenshed.FinalizePastMerge` over the child record, its parent read through `OriginParentHoldsTaskWork`,
  or through `ParentHoldsTaskWork` when `fabricengine.EnvSyncOptions().SkipPush` is set, since the push skip pushes nothing to origin,
  and its `restepped` seam through `sheddriver.ReadSettleRecords` over `shedrun.SettleRecords(task, shedrun.SelfRunID)` then `sheddriver.RestepRecorded`, as `childRestepped` does today;
  both seams need the task worktree, so when no pair stands here and the child was read through `fabricengine.OriginRunStatus`, `PastParentMerge` is false with neither seam called:
  the act rule hands a run owned elsewhere over before the input matters, and for any other run false only withholds the act a hold would otherwise yield;
- `PushOwedOffline` and `CodeTipMissing` false: their sources arrive with the later parts.
Every read error is returned naming the read and the slug.

`TestReadRun` is tier0, over fakes: rows for each record source present and absent, the child read from origin when no pair stands here,
the task branch read only for a cancelled record, the parent read from origin and, under the push skip, locally,
and `PastParentMerge` true for batten at Land and at Worktree-Teardown with no parent read;
a child at Finalize, running, read from origin with no pair here reads `PastParentMerge` false with no error and calls neither seam.
`TestReadRun` sits in the internal package `pairteardown`, since it drives the unexported `runReader`.
A row pins the local row names against the recipe, as `TestBattenFollowUp` pins the other two:
`runShedRow` equals `battenrecipe.NameRunShed`, and the started set equals the batten recipe's rows, parsed from `recipes.BattenRecipe` through `shedbuild.Parse`, less `battenrecipe.RowsBeforeRunShed()`.
Only the test imports `battenrecipe`, which closes no cycle, since the cycle runs through a production import of it.

`TestReadRun_ReadsTheSameStateWithoutTheCache` is tier1, over a `hubforge` hub with a pair whose run is seeded running and a pause entry for it:
it reads `ReadRun` and `runfsm.Derive`, removes the prime's and the pair's `.lym` directories, and reads the same state again, so a reader that consulted a cache fails it (D3's criterion).
The pause coverage is the one input it reads from `.lym`, and the test asserts only the derived state, which reads no input.
It sits in the external test package `pairteardown_test`, as `teardown_integration_test.go` does, and calls the exported `pairteardown.ReadRun`,
since `hubforge` imports `pairteardown` and an internal-package file importing `hubforge` is an import cycle.

`doc.go` gains a paragraph naming `ReadRun` as the run FSM's input reader.

**ImpactSummary:** New reader only; its callers arrive on later cards, and the doc gains one paragraph.
