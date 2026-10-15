# Card 1 — engine-siblings

**Create:**
- `internal/boardengine/siblings.go`
- `internal/boardengine/siblings_test.go`
- `internal/boardengine#Sibling`
- `internal/boardengine#SharedModuleLabels`
- `internal/boardengine#addedSiblings`
- `internal/boardengine#Board.writeReportingSiblings`
- `internal/boardengine#Board.UpsertTaskSiblings`
- `internal/boardengine_test#TestWriteSiblings`

**Edit:**
- `internal/boardengine#Board.UpsertTask`
- `internal/boardengine#Board.SetDeps` -> `func (b *Board) SetDeps(selector any, dependsOn []string) ([]Sibling, error)`
- `internal/boardengine#Board.UpsertTasksBatch` -> `func (b *Board) UpsertTasksBatch(tasks []map[string]any) ([]Sibling, error)`
- `internal/boardengine#Board.MergeTasks` -> `func (b *Board) MergeTasks(removeSlugs []string, upsert map[string]any, setStatus *MergeStatusUpdate) (Task, []Sibling, error)`
- `internal/boardengine_test#TestRunLock`
- `internal/boardengine_test#TestPriorityRoundTrip`
- `internal/boardengine_test#TestEntryDates`
- `internal/boardcli#storeVerbs`
- `internal/boardengine/doc.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/boardengine#Board.boardCriticalSection`
- `internal/boardengine#cloneTasks`
- `internal/boardengine#isDone`
- `internal/boardengine#Vocabulary.IsType`

**Intent:**
The sibling check of decisions D1 and D2, in a new file `internal/boardengine/siblings.go`.

`Sibling` is one reported pair: `Task` (the written task X), `Sibling` (the other task Y), `Deps` (the dependencies that make the pair siblings, each once, in X's `depends_on` order) and `Labels` (the shared module labels, in X's label order), all with JSON tags so a later consumer can print it.

`SharedModuleLabels` is the exported overlap test note `batten-ready-overlap` reuses: the labels `a` and `b` both carry that `v.IsType` does not report as a type label, in `a`'s order, nil when none.
It does no CLI formatting.

`addedSiblings` computes the siblings of one write from the entries as loaded (`before`) and as left by the mutation (`after`), per D1:
- X ranges over the tasks in `after` whose status is not `done`.
- X's baseline is its own `depends_on` in `before`, when it existed there, joined with the `depends_on` of every `before` entry absent from `after`; for a `merge` the absent entries are exactly `remove_slugs`, so folding A into B warns only on dependencies none of them had, and for the other verbs no entry is removed, so the baseline is X's stored list.
- A dependency D is added when X's `depends_on` in `after` holds it and the baseline does not; a D listed twice counts once, and a D whose entry in `after` is `done` is dropped.
- Y ranges over the `after` entries other than X whose status is neither `done` nor `abandoned` and whose `depends_on` holds an added D; an entry the merge removed is not in `after` and is never a sibling.
- The pair is a sibling when `SharedModuleLabels(X, Y, v)` is not empty, with X's labels and status as the write left them.
- A pair reported from both sides, two batch members both newly waiting on a shared D, is reported once: the first orientation in `after` order keeps the entry, and the other side's dependencies join its `Deps` without repeats.
- A write that adds no dependency, such as an upsert without the `depends_on` key, a label-only change, a cleared list or a re-sent list, returns nil.

`Board.writeReportingSiblings` runs `fn` through `boardCriticalSection`, snapshotting `cloneTasks(store.Tasks())` inside the locked mutation before calling `fn` and computing `addedSiblings(snapshot, store.Tasks(), b.vocab)` after it returns without error, so the set is read under the same lock as the write (D2).
A refused write, one `fn` or the later run-lock check rejects, returns its error and no siblings, so a refusal is unchanged; the check never refuses, never changes a stored record and never changes an exit path.

`Board.SetDeps`, `Board.UpsertTasksBatch` and `Board.MergeTasks` run through `writeReportingSiblings` and return the siblings beside their old results.
`Board.UpsertTaskSiblings` is the upsert form returning siblings, and `Board.UpsertTask` keeps its signature and delegates to it, dropping the siblings, so its callers across packages stay unchanged.

Callers of the re-signed methods: `storeVerbs` discards the new result for now (card 2 prints it), and `TestRunLock`, `TestPriorityRoundTrip` and `TestEntryDates` take the extra result with `_`.

`TestWriteSiblings`, in a new file `internal/boardengine/siblings_test.go` of package `boardengine_test` with its own header, is one table-driven unit test against a temp board with `SkipGit`, no git spawned, `t.Parallel`, with its own `Config` whose `Labels` hold two module labels (`board`, `cli`) beside `testTypes`.
It is a new top-level test rather than rows of `TestSetDeps`, `TestUpsertTasksBatch` or `TestMergeTasks` because those run on `boardengine.NewStore("")`, and the siblings come back only from the `Board` methods, through `boardCriticalSection`.
Every seeded entry, X, Y, Z, A, B, C, D and E, is a `"kind":"task"` carrying the type label `bug`, plus the module labels a row names, so no seed hits a note refusal.
The one exception is the note N of the note refused row, seeded as a `"kind":"note"` carrying `bug`, which set-deps then gives `depends_on` [D].
Each row seeds entries, runs one write and asserts the exact returned `[]Sibling` and, on success, the stored `depends_on`:
- `set-deps` adding D to X, Y open on D sharing `board`: one sibling {X, Y, [D], [board]}, and X stores [D].
- the same through `UpsertTaskSiblings` with `depends_on` and through `MergeTasks` whose upsert carries `depends_on`; the batch row below covers `UpsertTasksBatch`.
- set-deps adding D to X while Y1 and Y2 both wait on D and share `board` with X: two siblings, {X, Y1} then {X, Y2}, in board order.
- X and Y sharing only the type label `bug`: none.
- Y `done`: none; Y `abandoned`: none; Y running (`running · Webster`): one sibling.
- X written by upsert with status `done` and `depends_on` [D]: none.
- D `done`: none.
- no other task waits on D (also covers Y is X): none.
- D already in X's `depends_on`, set-deps re-sending [D] plus E that no one else waits on: none.
- X stored on [D] and Y on [D, E], set-deps X to [D, E]: one sibling whose `Deps` is [E] only.
- upsert on X without `depends_on` that adds the shared label `board`, X already on D: none.
- set-deps X to []: none.
- upsert setting `depends_on` [D] and labels [bug, cli] on an X stored with [bug, board], Y carrying [bug, cli]: one sibling with `Labels` [cli], the labels as written.
- `UpsertTasksBatch` writing X newly on [D, E] and Z newly on D, Z already stored on E, X and Z sharing `board`: exactly one sibling, {X, Z, [D, E], [board]}, where the X side contributes [D, E] and the Z side [D].
- `MergeTasks` upserting X with `depends_on` [D], Y open on D sharing `board`, with `set_status` marking X `done`: none, since X's status is the one the whole write leaves.
- `MergeTasks` removing Y and upserting X with `depends_on` [D]: none.
- `MergeTasks` removing A, which waited on D, and upserting new B with `depends_on` [D], while C waits on D and shares `board` with B: none.
- set-deps X to [D, D]: one sibling whose `Deps` is [D].
- refused writes, each with a would-be sibling Y open on D sharing `board`: set-deps giving the note N `depends_on` [D] and set-deps naming an unknown dependency, refused inside `fn`, assert a non-nil error and nil siblings; set-deps adding D to an X whose status is `running · Webster`, refused by `checkRunLocks` after `fn` has computed the set, asserts `errors.Is(err, ErrRunLocked)` and nil siblings.
  The error text and the rollback stay pinned by `TestValidateDependencyGraph`, `TestKindDependencyRules`, `TestSetDeps` and `TestRunLock`, so these rows assert neither.

The new top-level test raises `internal/boardengine`'s row in `cmd/lyx/testdata/test-budget.yaml` by one.
`internal/boardengine/doc.go` gains a `# Siblings` section describing the check: the D1 definition, the merge baseline, the module-label overlap through `SharedModuleLabels`, the computation under the write lock, and that it never refuses.

**ImpactSummary:** Three Board methods re-signed with callers in `storeVerbs` and three boardengine tests; `Board.UpsertTask` keeps its signature, so the cross-package callers are untouched; no stored record or refusal changes.

**Commit:** 1: boardengine: report tasks newly waiting on a shared dependency with a shared module label
