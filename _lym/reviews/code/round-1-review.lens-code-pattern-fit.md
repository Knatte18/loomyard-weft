```yaml
lens: code-pattern-fit
outcome: reported
```

**Lens: code-pattern-fit.** I checked each rule against the changed files, from the inherited context only.

**Rules checked**

- **`PATTERN-test-speed`: held.** The diff edits rows in `layer_test.go` and `render_test.go` and adds no test function. Both files keep their existing tier tags and `t.Parallel` structure. The new `ComputeLayers` row and the `RenderOrder` row extend existing tables.
- **Declared PATTERN changes landed: held.** The constraints declare no entry add, change or remove, and the diff touches neither `PATTERN.md` nor `pattern/`. No new cross-cutting invariant is introduced.
- **Commit messages and departures: held.** One task commit (`14a1601b`) carries no `Pattern-Departure:` trailer, and none is needed because no listed entry is departed from. The merge commit is a plain merge of the parent tip.
- **CLAUDE.md "Docs land in the same commit": held.** `doc.go`'s README part is updated in the same commit as the code.
- **CLAUDE.md "Markdown: semantic line breaks": held.** The new `doc.go` lines break one sentence per line, with a semicolon break in the draft sentence.
- **CLAUDE.md "Notes go in git" and "Worktrees": held.** The diff touches only the six `internal/boardengine` files and adds no memory file. It stays out of `docs/research` and the link allowlist, as the decision record's last entry requires.
- **CLAUDE.md "no board entry body/scope edit": held.** The diff has no `_board` change.
- **Comment conventions: held.**
  - `isDraft`'s doc comment opens with its own name.
  - The comments on `draftLayer`, `ComputeLayers` and the `RenderOrder` bucket line are self-contained.
  - There are no end-of-line comments.
  - Line breaks and wrapping are not judged, per the rubric.
- **Constraint "layer computed at read time, never stored": held.** `draftLayer` is only returned from `ComputeLayers` and ordered in `RenderOrder`. Nothing is persisted.
- **Terminology and naming: held.** The `draft` check reuses `StatusDraft`. `entryCell` now calls `isDraft` instead of its inline check, so there is one source for the test.

**Findings**

- **F-A, LOW, `consistency`.** The `standingCell` doc comment (`internal/boardengine/render.go:442`) still lists only "Done, the run's place, Ready, Note, or the name of its dependency layer". A draft member now reads `Draft` through the `layerSection` fall-through, so the comment no longer covers the case this change adds. Evidence: `render.go:442-455` against the `draftLayer` case at `render.go:474`.
- **F-B, NIT, `consistency`.** `internal/boardengine/doc.go` still says "Ready holds the open tasks with no open dependency". That is no longer exact, because an open draft task with no open dependency now sits under Draft. The adjacent new Draft sentence states the exception, so the Ready sentence could say "open, released tasks". Evidence: the Ready sentence in the `doc.go` diff hunk, below the new Draft lines. The same sentence form appears at `layer.go:29` ("An open task with no open dependency is readyLayer"). Only the earlier draft sentence in that doc comment makes it read correctly.

**Tool calls:** none. The manifest already held the diff, `layer.go`, the `render.go` excerpts and the PATTERN text.
