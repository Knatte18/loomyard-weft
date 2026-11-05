```yaml
lens: fix-code-correctness
outcome: reported
```

Lens fix-code-correctness: I ran each changed function and branch with a concrete input and checked the callers and the added tests.

**Findings**

1. **LOW** — `TestRunLock_SketchExemptionIsTheWholeNetChange`, row `NoteWithoutTheSketchLabelIsRefused` (`internal/boardengine/sketch_test.go`).
   - The row passes with the defect in place, so it pins nothing new.
   - With the old `endsAsSketch`, a note without the `sketch` label fails `IsSketch`, so `checkRunLocks` falls through to `scopeChanged`.
   - `scopeChanged` sees `Kind` change from task to note and refuses it.
   - Only the title and body rows fail on the old code, which is the F1 gap. The label row is harmless as a regression pin, but F6's claim that all its rows fail before the fix is wrong for this one.

**Traces with no defect found**

- **`endsAsSketch`**
  - Input: before is a task with labels `[bug, board]`, status running, recipe motif. now is the `MakeSketch` output.
  - `sketchOf(before, vocab)` gives labels `[sketch, board]`, status nil, empty `DependsOn`, and recipe, isolated and `PullRequest` cleared.
  - Every field `scopeChanged` compares matches `now`, so it passes.
  - A title or body edit alongside the change returns true from `scopeChanged`, so it is refused.
- **`checkRunLocks` callers**
  - `board.go` passes `b.vocab`, the same value `store.vocab` points to.
  - Both `replay.go` calls pass `*r.store.vocab`, which `Apply` always sets. The tier-0 and tier-1 gates passed, so no test builds a `replayRun` with a nil vocab.
  - `runlock_test.go` and the `export_test.go` var were updated, and the packages compile.
- **`MakeSketch` and `sketchFinish`**
  - `validateWrite` failures are wrapped with `%w: %w`, so `errors.Is(err, ErrSketchRefused)` holds. The error still names the dependent entry and `set-deps`.
  - A lock, save or render failure is not wrapped, so `sketchFinish` returns it and the seam retries.
  - The retry sees an already-Sketch entry, which `MakeSketch` treats as a no-write success, so a post-save render failure converges.
- **Start guard**
  - A task with status `discussed` and no status file returns that status, so the refusal text names "discussed". The restored test row asserts this.
- **Migration test**
  - Migration runs before each write's mutation, so `talked` becomes a Sketch at the `needed` write.
  - `needed` is refused as a Sketch because `needs` depends on it, so it keeps status `discussed`.
  - Both are asserted, and the tier-0 gate passed.
- **Integration row `RefusedSketchLeavesTheEntryDraft`**
  - It is the only row whose `MakeSketch` returns `ErrSketchRefused`. It fails if the fallback is removed, since the entry would stay at the running status.
  - Both wirings ran and passed in tier 1.
- **Replay row `HeldTaskBecomingASketchIsNotRefused`**
  - It asserts `Refused` is empty, and the entry ends as a status-less note.
  - A wrong exemption would refuse it, so it guards the replay claim in `doc.go`.

**Tool calls:** none. My context already held every changed file and the gate output.
