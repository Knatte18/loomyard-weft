```yaml
lens: code-scope
outcome: reported
```

Task: the code-scope lens, which traces the diff's changes to the board entry's goal points.

**Decisions** (taken from the board entry under `## Goal`; the record has no `Added after Discussion` entries):
- **G1 (Seen):** `lym board list --text` showed a landed task of a beam project under P0, not under its beam.
  - Built by: none. The diff leaves `internal/boardcli` untouched.
- **G2 (Expected):** a done task stays listed under its own project until it is pruned.
  - Built by: C1, but only on the `board-data.js`/`board.html` surface.

**Changes:**
- **C1:** `internal/boardengine/view.go` `viewProjects` now appends every done project after the open walk, each with `Parent` from `projectHandle`. The `BoardView.Projects` and `viewProjects` doc comments are updated to match.
  - Traces to: G2, on the viewer surface.
- **C2:** `internal/boardengine/view_test.go` adds fixture T11, a done member of done project P3, plus its placement row. The project-list subtest is renamed and now expects P3.
  - Traces to: forced by C1.

**Findings:**

1. **MEDIUM · scope · `internal/boardcli/text.go:106` (`withRootProjectRow`), left untouched**
   - **Summary:** the entry names `lym board list --text` as the surface where the defect was seen, but the diff changes only the viewer projection, so the `--text` listing still shows a landed beam task below the P0 row.
   - **Evidence:**
     - `withRootProjectRow` inserts P0 "before the first project, note or done entry" (`text.go:104-113`).
     - `RenderOrder` sorts every done entry last, whatever its project (`layer.go:218-223`).
     - So in `list --text`, every done task, a beam project's landed task included, sits after the P0 row and is never grouped with its project. That matches the Goal's "Seen" line, and nothing in the diff changes it.
     - `_lym/landing/summary.md:29-31` drops `--text` on the writer's own reading ("that listing is flat… left as it is").
     - The decision record has no `Added after Discussion` entry that narrows the Goal to the viewer. Only the entry's title says "Board view".
   - **Fix:** either make the `--text` listing keep a done task with its project until prune, or ask the parent to record the viewer-only narrowing with `lym loom decision add` and leave `--text` as it is.

No other untraced change: C1 and C2 both trace.

Outside this lens: `board.html` `projectSelect` (`board.html:589`) now offers done projects in the project dropdown, and `checkProjectField` refuses them (`project.go:97-98`). That belongs to the affected-set and correctness lenses.

Tool calls: none.
