# Card 5 — pairteardown-read-run

**Create:**
- `plan:internal/pairteardown#ReadRun` -> `func ReadRun(prime *lymcwd.Location, slug string) (runfsm.Records, runfsm.Inputs, error)`
- `plan:internal/pairteardown#runReader` -> `type runReader struct`
- `plan:internal/pairteardown#runReader.read` -> `func (r runReader) read(slug string) (runfsm.Records, runfsm.Inputs, error)`
- `plan:internal/pairteardown#TestReadRun` -> `func TestReadRun(t *testing.T)`
- `plan:internal/pairteardown#TestReadRun_ReadsTheSameStateWithoutTheCache` -> `func TestReadRun_ReadsTheSameStateWithoutTheCache(t *testing.T)`

**Extends:**
- `internal/pairteardown/doc.go`

**Uses:**
- `internal/pairteardown#ReadBattenRecord`
- `internal/pairteardown#OwnerPauseHeld`
- `plan:internal/fabricengine#OriginParentHoldsTaskWork`
- `plan:internal/fabricengine#OriginRunStatus`
- `internal/battenshed#FinalizePastMerge`
- `plan:internal/runfsm#Records`
- `plan:internal/runfsm#Inputs`

**Intent:** The one reader of a run's FSM inputs, which `battencli`, `loomcli` and `tendercli` all import; it sits in `pairteardown` because `tendercli` cannot import `battencli`, which imports it, and `pairteardown` already holds the batten-record and owner-pause readers.
`ReadRun` wires a `runReader` over the hub of prime and returns its read; `runReader` holds one function field per substrate, so `TestReadRun` drives every branch with fakes.

Records, tracked only:
- the board entry from the local board: `Released` for a task whose status is empty (`boardengine.IsEmptyStatus`), `BoardDone` for `done`;
- batten's record from the prime's `shedrun.StatusFile`, read tolerantly, with `BattenStarted` true when its row is not one of `battenrecipe.RowsBeforeRunShed()`;
- `TaskBranchOnOrigin`, read through the fabric topology only when the batten record's state is `cancelled`, since only that rule reads it;
- the child record from the task worktree when the pair stands here, else `fabricengine.OriginRunStatus`;
  `ChildLanding` when the child is at `loomshed.NameFinalize` or `done`, or batten is at `battenrecipe.NameLand` or `battenrecipe.NameWorktreeTeardown`.

Inputs, without `State`, which `runfsm.Judge` fills:
- `OwnedElsewhere`, `Owner` and `AdoptSubject` from the local Bolt's owners through `hubgeom.ParentOf` and `machinename.Name`, as `boardengine.OwnedElsewhere` judges;
- `PauseCovers` from this machine's pause record coverage of the slug; an adopt pause is such an entry, so it waits the run and `lym tender resume <slug>` lifts it;
- `Held` false: D7's Bolt holds fill it in a later part (Shared Decisions);
- `PastParentMerge` true when batten is at `battenrecipe.NameLand` or `battenrecipe.NameWorktreeTeardown`, whose Finalize already merged and pushed the parent, as `childReachedParent` counts them;
  otherwise from `battenshed.FinalizePastMerge` over the child record, its parent read through `OriginParentHoldsTaskWork`,
  or through `ParentHoldsTaskWork` when `fabricengine.EnvSyncOptions().SkipPush` is set, since the push skip pushes nothing to origin;
- `PushOwedOffline` and `CodeTipMissing` false: their sources arrive with the later parts.
Every read error is returned naming the read and the slug.

`TestReadRun` is tier0, over fakes: rows for each record source present and absent, the child read from origin when no pair stands here,
the task branch read only for a cancelled record, the parent read from origin and, under the push skip, locally,
and `PastParentMerge` true for batten at Land and at Worktree-Teardown with no parent read.

`TestReadRun_ReadsTheSameStateWithoutTheCache` is tier1, over a `hubforge` hub with a pair whose run is seeded running and a pause entry for it:
it reads `ReadRun` and `runfsm.Derive`, removes the prime's and the pair's `.lym` directories, and reads the same state again, so a reader that consulted a cache fails it (D3's criterion).
The pause coverage is the one input it reads from `.lym`, and the test asserts only the derived state, which reads no input.

`doc.go` gains a paragraph naming `ReadRun` as the run FSM's input reader.

**ImpactSummary:** New reader only; its callers arrive on later cards, and the doc gains one paragraph.
