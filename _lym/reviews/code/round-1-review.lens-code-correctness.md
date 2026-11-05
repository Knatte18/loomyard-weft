```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens for the Sketch diff. Each changed function or branch is listed with the case I ran.

**Per function or branch**
- `withSketchType`, with `types` nil: returns `[sketch]`. With board.yaml listing `sketch`: returns the list unchanged. Correct.
- `withSketchType`, with board.yaml listing `spike`: `validateTypeLetters` refuses at `LoadConfig`, so the hub's whole board config no longer loads. The refusal names the label to rename. This works as designed but is a hub-wide effect (F2).
- `IsSketch` and `RenderCompact` partition: a note labelled `sketch` goes under `sketches:` after the other entries. With no Sketches, no heading is printed. Correct.
- `sketchOf`, with a task labelled `[bug, board, loom]`: gives `[sketch, board, loom]`. With nil `DependsOn`: gives `[]string{}`, and `slices.Equal` treats that as nil. Correct.
- `sketchOf`, with a stale type label missing from the vocabulary: the label is kept as a plain label, and `validateLabels` then refuses. `MakeSketch` errors and the loom side falls back to draft. Acceptable.
- `MakeSketch`, with the entry absent, done or already a Sketch: `noWrite`, returns nil. Correct.
- `MakeSketch`, with a dependent entry: `validateWrite` refuses at store.go:380-389 and the error is returned. Correct.
- `MakeSketch`, on a held task: `checkRunLocks` passes it through `endsAsSketch`. Correct for the run's own call, but see F1.
- `endsAsSketch`, with a held task whose title and body are also edited in the same write: returns true and skips `scopeChanged` entirely (F1).
- `migrateDiscussedStatus`, with a task in `discussed` and no dependents: becomes a Sketch before `loaded` is cloned. `assignHandles` then replaces the stored `T` handle with an `S` one. Dates are not bumped, which is fine. Correct.
- `migrateDiscussedStatus`, with a dependent entry: `validateWrite` refuses, so the task keeps the `discussed` status (F3).
- `migrateDiscussedStatus`, with a project or done task: skipped. Correct.
- `sketchFinish`, with the board refusing the Sketch: warns, then `SetRunStatus` sets draft, which is status-only and passes the lock. Correct.
- `sketchFinish`, with a transient `MakeSketch` error such as a lock or render failure: the same draft fallback runs (F4).
- `newCommitStatusSeam`, in state done with a nil `Finish`: no write, same as before for Finalize recipes. Correct.
- `newCommitStatusSeam`, with `Finish` failing: warns, and the next transition retries. Correct.
- `finishesAsSketch`: the inverse of the Finalize-row check. Correct.
- `typeHeading`, `notesTypeOrder`, `leadingNoteTypes`: the order is sketch, research, then the rest, and an unconfigured `sketch` is skipped. Correct.
- `startguard.go`: now refuses only draft (F3).

**Findings**

- **F1 — MEDIUM, design.** `internal/boardengine/runlock.go:152` and `sketch.go:54-56`. `endsAsSketch` lets a held task through whole whenever the net change is "task to note, `sketch` label, no status". It never checks that title, brief, body, project, issues and slug are unchanged. A hand upsert such as `{slug, title:"X", body:"…", kind:"note", labels:["sketch"], status:null}` on a run-held entry therefore bypasses the scope refusal. The Goal says only the Sketch change should pass. The fix is to make `endsAsSketch` compare `before` with `sketchOf(before, vocab)` outside the run and landing fields, or to run `scopeChanged` against `sketchOf(before)`. Evidence: runlock.go:151-157 versus runlock.go:193-217.

- **F2 — LOW, design.** `internal/boardengine/config.go:191`. A hub whose board.yaml already has a type starting with `S`, such as `spike` or `security`, can no longer load its board config after upgrade. Every `lym board` command then fails until the type is renamed. The refusal names the label to rename, and the summary admits this. Evidence: config.go:186-190 and handles.go:39-42.

- **F3 — MEDIUM, design.** `internal/battencli/startguard.go:99`. The guard now refuses only draft. A legacy `discussed` task is no longer refused, in two cases:
  - Before the next board write migrates it, a hand `lym batten run` can start it.
  - A task that keeps `discussed` as a hand-set word because something depends on it is never migrated, and the guard no longer stops a run on it. The README also no longer has a `Discussed` layer, so it shows in Ready or a letter layer.

  The fix is to keep refusing a task whose status is the legacy `discussed`, or to migrate on the read path as well. Evidence: startguard.go:96-101 and sketch.go:79-87.

- **F4 — LOW, design.** `internal/loomcli/wiring.go:233-235`. Any `MakeSketch` error, including a transient lock-acquire, save or render error, takes the "leave the entry draft" fallback. The comment and doc describe the fallback as the answer to a board refusal. After a post-save render failure the entry is already a Sketch, so `SetRunStatus` then tries to set a status on a note and fails. The retry then finds the entry already a Sketch and passes. Only a refusal should fall back to draft, and other errors should return so the next transition retries. Evidence: wiring.go:226-238 and board.go:160-174.

**Tool calls**
1. Read store.go:335-470: `validateWrite` was not in my context.
2. Read config.go:95-154: `Vocabulary.IsType` was not in my context.
3. Read handles.go:20-80: `validateTypeLetters` and `handleLetter` were not in my context.
