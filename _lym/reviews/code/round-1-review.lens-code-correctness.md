```yaml
lens: code-correctness
outcome: reported
```

**List — function/branch, case run, correct or not**

1. `Store.ListTasksBrief` parent-beam switch (`internal/boardengine/store.go:768-780`):
   - No seam, entry with no project → `ParentBeam` gives `""` → switch picks `RootSubject`. Correct.
   - No seam, member of an open beam → `"beam"`. Correct.
   - Seam answers `("", nil)` → `root`. Correct.
   - Seam answers `("beam", errSeam)` → `parentErr != nil` → `""` (omitted key). Correct, matches `parentbeam_test.go:142-152`.
   - Seam answers `("", err)` → `""`. Correct (root never stands in for an unread parent).
   - Note / done entry under the prime → `root`. Correct per D8 "every entry".
2. `get` handler + `outputGetTask` (`internal/boardcli/cli.go:907-913`, `1374-1383`):
   - Found task under prime → `ParentBeamOf` → `""` → `parent_beam: "root"`. Correct.
   - Under a beam → slug. Correct.
   - `ParentBeamOf` error → handler returns `outputError` before `outputGetTask`. Correct, not swallowed.
   - Not found → `outputGetTask(out, nil, "")` → key omitted. Correct.
3. `withRootProjectRow` (`internal/boardcli/text.go:91-99`):
   - Empty listing → `at = -1` → `at = 0` → single P0 row. Correct, no panic.
   - Open task + open note → P0 between them (matches `verbs_test.go:347-350`). Correct.
   - Sketch note only → P0 before it; `RenderCompact` moves the sketch to the end anyway. Correct.
   - Open tasks + done tasks, no open project/note → `IndexFunc` finds none → P0 appended after the done tasks. **Wrong place** (F1).
   - Open tasks + done task + done project → P0 lands between the done task and the done project. **Wrong place** (F1).
   - `--sort age` / `--sort revised` → `sortByDate` runs first (`cli.go:951`), then P0 inserted before whichever project/note sorts first by date: arbitrary position; the row's zero dates would put it last under the sort's own rule. **Questionable** (F2).
   - `slices.Insert` aliasing: the slice is freshly built by `ListTasksBrief`; no shared backing. Correct.
4. `RootProjectBrief` (`internal/boardengine/render.go:188-190`): handle P0, slug root, title from `rootProject`, kind project, no dates → `FormatAge` blank. Correct.
5. `Store.Find` (`store.go:912-931`) inherits the mapping: find JSON now carries `root`; `find --text` adds no P0 row (no `withRootProjectRow` on that path, `cli.go:1198-1206`). Consistent.
6. `list` with `--label` → no P0 row (`cli.go:961-964`). Correct.

**Findings**

- **F1** — severity LOW, class design, location `internal/boardcli/text.go:91-99` — With no open project and no open note, the P0 row is placed among or after done entries instead of ahead of them, where README order puts projects; insertion point is the first entry whose kind is project or note, so open tasks + done tasks put P0 last (after done tasks), and open tasks + done task + done project put it between the two done entries. Evidence: `RenderOrder` puts every done entry after every open entry regardless of kind (`internal/boardengine/layer.go:192-223`), while `IndexFunc` ignores done status (`text.go:92-94`); the doc comment's claim "so that in README order it leads the projects" (`text.go:90`) is then false. Fix: insert before the first entry that is a project, a note, or done.
- **F2** — severity LOW, class design, location `internal/boardcli/cli.go:951-964` — Under `--sort age`/`--sort revised`, the P0 row is inserted before whichever project/note sorts first by date, a position with no meaning in a date-sorted listing; since `sortByDate` runs before `withRootProjectRow` and the row has no dates, the sort's own undated-last rule (`cli.go:1186-1191`) would put it at the end. Evidence: `cli.go:951` sorts, then `cli.go:962-963` inserts. Fix: under `--sort`, append the row last (undated-last rule) or omit it as for `--label`.

**Tool calls:** none.
