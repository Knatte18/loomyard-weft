```yaml
lens: code-affected-set
outcome: reported
```

Lens: code-affected-set. For each changed behaviour, this is what the change touches outside its own lines.

**Per change: the set found**

1. **`Decision record:` line in `renderDesigns` (`render.go:949-952`).**
   - Asserts it: `TestRenderDesignDocGoldens` and the new `TestRenderToDiskLinksOnlyAStoredDecisionRecord` (`render_test.go`). Covered.
   - Other design-doc assertions: no other test asserts a discussed design doc's header. Covered.
   - Prose: the `renderDesigns` doc comment and the `render.go` file header are updated. Covered.
   - Specs, stencils and docs: grep found none describing the design doc header or the Slug-cell link.
2. **`RenderToDisk` rendering a flagged entry without its file as unflagged (`render.go:39-40`, `62-75`).**
   - Callers: `board.go:197`, `replay.go:471` and `ownerset.go:251` all go through `RenderToDisk`. Covered.
   - `ownerset.go:246` calls `Render` directly on the unfiltered entries. Only the file names are used (`paths`), and flags do not change names, so the path list stays correct.
   - Existing tests: only `render_test.go` asserts a `[Sketch](decision-…)` link. The other `Discussed: true` tests (`sketch_test.go`, `labels_test.go`, `layer_test.go`, `sharedbootstrap_test.go`) assert no rendered link. Covered.
   - Docs: the `doc.go` Decision record section states the rule (`doc.go:508-511`). Covered.
3. **`sketchLink` and `recordLinePrefix`.**
   - Used only in `render.go`. Covered.
   - `PATTERN-code-conventions` (one named constant for a name that can go stale): the link text and file name each have one constant or helper. No violation.
4. **PATTERN and commit rules.**
   - Declared PATTERN changes: the diff touches no `PATTERN.md` or `pattern/` file, and the task declares none. Nothing to land.
   - Commit messages: no `Pattern-Departure:` trailer, and none is needed. Covered.
   - `PATTERN-test-speed`: the new test uses `tiergate.Require(t, 0)` and `t.Parallel()`. It spawns nothing: `t.TempDir()`, `os.WriteFile` and `RenderToDisk` only. No violation.

**Findings**

- LOW, `consistency`, `render.go:108`: the new `Render` doc line says every flagged entry "is linked to its decision record, which Render takes to exist". That is correct, but the file header (`render.go:12`) says "A discussed entry's header and Slug cells link its decision record" with no mention of the missing-file rule. `render.go:37-38` and `doc.go:511` do carry it, so this is a minor gap and not a defect.
- No other member of the affected set is left wrong.

**Tool calls**
1. Grep for `Sketch](decision` across the repo. The manifest held only the `render_test.go` hits.
2. Grep for `Discussed: true` in tests outside `render_test.go`, to check whether another test's rendered output depends on the link without a record file.
3. Grep for design doc, design page and Sketch prose in docs and stencils, to find anything describing the design doc header.
4. Grep for decision-record and link wording in `.md` and `.tmpl` files, to check specs and READMEs. It found none that describe the Slug-cell link or the design doc header.
