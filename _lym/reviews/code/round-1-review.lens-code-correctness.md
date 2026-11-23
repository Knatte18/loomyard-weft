```yaml
lens: code-correctness
outcome: reported
```

Lens: code-correctness. I traced each function and branch the diff adds or changes with a concrete input.

**Functions and branches, with the case I ran:**
- `validateWrite` with the note refusal removed (`store.go:503`):
  - upsert of note `n` with `depends_on:[dep]`, where `dep` is an open task: it passes the dangling, isolated, note-target and cycle checks. Correct.
  - note `n` with `depends_on:[idea]`, where `idea` is a note: refused by the note-target loop (`store.go:508-512`). This is correct for the code as written; whether it should be refused is the scope lens's question.
  - demotion of task `x` with `depends_on:[dep]` to a note: passes, and the dependents check finds nothing. Correct.
  - with `DependsOn` empty, every loop is skipped. Correct.
- `ListTasksBrief` `Pending` (`store.go:870,900`):
  - note `waits` with `depends_on:[open, landed]`: `Pending` gives `[open]`. Correct.
  - entry with `depends_on` nil: `Pending` gives nil and `omitempty` drops the key. Correct.
  - entry whose dependency slug is no longer on the board (left by `lym board remove`): `Pending` gives nil, because `met` treats a missing slug as met (`dependency.go:51`). The README shows the same dependency in After. Inconsistent; see finding 1.
- `RenderCompact` and `afterText` (`text.go`):
  - no entry pending: `afterWidth` is 0, so the column is left out. Correct.
  - one note pending `[a,b]`: the cell reads `after a,b`, padded, and non-waiting rows get a blank padded cell before `[status]`. Correct.
  - Sketches have `depends_on` cleared, so they add no width. Correct.
- `boardView.noteTable` (`render.go:697`):
  - a section with no waiting note: one Note column, `priority:true`, the same as the old literal. Correct.
  - a section with one note on an open task: Note and After columns, and `afterCell` gives the handle. Correct.
  - an empty `notes` slice: `ContainsFunc` is false, so it returns the plain table. Correct.
- `entrySections` (`render.go:681`) builds the table per section from `byType[typeLabel]`, the same slice that `section` receives. Correct.
- `addedSiblings` (`siblings.go`):
  - promote of note X (`depends_on:[D]`) while task Y waits on D: X was not a task in before, so its baseline is nil, `added` is `[D]` and the pair is reported. Correct per its doc.
  - `set-deps` on note N: N is skipped as x. Correct.
  - note Y waiting on D while task X gains D: Y is skipped as y. Correct.
  - merge removing a note M (`depends_on:[D]`) into task X: M's dependencies no longer join `removedDeps`, so D counts as added and a sibling is reported. This matches the new doc ("held as tasks"). Correct.
- `promoteTo` (doc change only) and `PromoteWithDeps`:
  - nil keeps `[c]`; `[d]` replaces it; `[]` clears it. The `TestPromote` rows assert all three. Correct.
- `ComputeLayers` is unchanged, and a note with an unmet dependency now gets a letter layer. The summary states this, and it is not an error.

**Findings**

1. LOW · design · `internal/boardengine/store.go:900`: `list`'s `pending` (and so the `list --text` after column) and the README After column disagree on a dependency whose entry is no longer on the board.
   - `Pending` counts a missing slug as met (`internal/boardengine/dependency.go:51`, `case !found || isDone(entry): return true`).
   - The README's `unmetDependencies` counts it as unmet (`internal/boardengine/render.go:750`, `if _, found := b.bySlug[dep]; found &amp;&amp; b.dependencies.met(dep)`).
   - `RemoveTask` strips nothing from other entries' `depends_on` (`internal/boardengine/store.go:777-784`), so `lym board remove` of a dependency leaves the README showing it after the slug, while `list --text` shows no after.
   - The board entry asks `list --text` to show a note's dependencies "as they show a task's" in the README.
   - Fix: make `Pending` and `unmetDependencies` judge a missing slug the same way, or strip removed slugs on remove so the case cannot arise.

**Tool calls**
- `sed store.go:840-915` and grep for `ComputeLayers`: my context lacked the body of `ListTasksBrief` and where layers are computed, which I needed to run the `Pending` and layer cases.
- `sed layer.go:1-120`: my context lacked `ComputeLayers`, needed to check how a note's new `depends_on` changes its layer and the cycle walk.
