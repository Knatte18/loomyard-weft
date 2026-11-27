```yaml
lens: code-correctness
outcome: reported
```

Lens code-correctness: I ran each function and branch the diff adds or changes on concrete inputs.

**Cases run**

- `checkOfferedRecipe`, through `UpsertTask` (`store.go:714`): the snapshot is `s.tasks`, which still holds the old entry.
  - A new entry naming `"recipe":"arras"`: `prior` is nil, so the write is refused. Correct.
  - An entry holding `darn` rewritten to arras: `prior.Recipe` is `darn`, so the write is refused. Correct.
  - An entry already holding arras: the write passes. Correct.
  - Another recipe or an empty one: no effect. Correct.
- `checkOfferedRecipe`, through `UpsertTasksBatch` (`store.go:1100-1133`): an existing entry patched to arras in a batch payload. **Wrong**: the write is not refused. See finding 1.
- `checkOfferedRecipe`, through `MergeTasks` (`store.go:1294-1321`): `incoming` is `ApplyPatch(projected[foundIdx])`, and `projected` still holds the old entry when `validateWrite` runs. A merge upsert naming arras is refused, and so is one onto a removed slug, where `prior` is nil. Correct.
- `checkOfferedRecipe` through `SetStatus`, `SetDeps` and `promoteTo`: each validates against `s.tasks`, which holds the old entry. An arras entry keeps taking status and dependency writes, which is the stated intent. Correct.
- `checkOfferedRecipe` through replay:
  - `addEntry` (`replay.go:673`): a local add carrying arras is refused and recorded as refused.
  - `changeEntry` (`replay.go:740`): origin's arras entry is `prior`, so the change passes.
  - Both match the refusal rule. Correct.
- `checkOfferedRecipe` through `MakeSketch` and `repointDependents`: `sketchOf` clears the recipe, and a dependent keeps its own recipe, so neither is affected. Correct.
- `DobbyRuns`, the child-recipe read (`dobbyreads.go:230-234`):
  - No record: empty recipe, so `Motif` is false. Correct.
  - Record `motif`: `Motif` is true. Correct.
  - A read failing for a reason other than not-exist: the error ends the whole `DobbyRuns` call, so no run of any slug gets a view that tick. This is the same handling as the sibling reads (status mtime, cancel marker, driver lock), so it is consistent and not filed.
- `rememberRuns`, the new switch (`landnotice.go:141-148`):
  - A remembered motif run that landed, with `land_notice` off, on or never read: the Sketch line is set, owed and kept. Correct.
  - A motif run past Land that did not land and was never read: `continue`, so it stays undecided. Correct.
  - A motif run that did not land, read off: no line, so it is forgotten. Correct.
  - A motif run first seen past Land: not remembered, so no line. Correct.
  - A memory holding `ClearOwed`: deleted before the switch. Correct.
  - A memory an older binary left owing the plain landed line: `note.owed()` skips the switch, so that run posts the plain landed line and no Sketch line. This is a one-time upgrade edge and not filed.
- `sketchNoticeLine`: renders the prefix, the slug twice and the `lym board get` command. Correct.
- Clearing timing: `ClearMotifRun` runs only from the raddle start guard (`startguard.go:159`) and `beam create` (`create.go:211`). Both need the Sketch promoted first, which the Sketch line itself leads to. The line is fixed in the state record once decided, so clearing the record afterwards does not lose it. Correct.

**Findings**

1. **BLOCKING · design · `internal/boardengine/store.go:553-563`, as called from `UpsertTasksBatch` at `store.go:1132`.** A batch upsert that gives an existing entry the arras recipe is not refused.
   - `UpsertTasksBatch` writes the patched entry into `snapshot[foundIdx]` (`store.go:1109-1113`) before calling `validateWrite(snapshot, incoming)`.
   - `snapshotIndex[incoming.Slug]` is therefore the incoming entry itself. `prior.Recipe == arras` holds, and `checkOfferedRecipe` returns nil.
   - A new entry in a batch is appended to `snapshot` (`store.go:1119`), so it passes the same way.
   - So every batch write naming `"recipe":"arras"`, new entry or existing, takes the recipe. This breaks the entry's "A board write naming `recipe: arras` is refused".
   - Fix: judge `prior` against the pre-batch `s.tasks`, not the projected snapshot. Add a batch row to `TestUpsertRefusesTheArrasRecipe`, which today covers `UpsertTask` only.

2. **LOW · design · `internal/boardengine/store.go:558`.** The "already holds the recipe" exemption keys on the slug's current recipe, so an entry demoted to a note keeps its arras recipe and still takes arras writes. Strictly, that matches the doc text ("An entry that already holds the arras recipe keeps taking writes"). No defect beyond what the rule states; I'm listing it so the handler can judge whether a held arras entry that is no longer running should keep the exemption.

**Tool calls**

1. `dobbyreads.go` 140–300, `ClearMotifRun` and its callers, `childrecipe.go`: to see how a recipe read error is handled and when the record is cleared. The manifest held only the diff hunk.
2. `LandOutcomeOf`, the rename span, `UpsertTasksBatch` 1100–1140 and `MergeTasks` 1290–1325: to see which snapshot each write path hands `validateWrite`.
3. The rename span, the `UpsertTask` span and the head of `UpsertTasksBatch`: to confirm how the batch snapshot is built.
