```yaml
lens: code-correctness
outcome: reported
```

Task: check the code-correctness of the `render.go` changes by running concrete cases through them.

**Per function or branch, case run, result**
- `Render`, board with one project `slug: root`: the loop at `render.go:107-111` writes `project-root.md` after the root's file, so the real project's file overwrites the root's. **Incorrect** (F1).
- `Render`, board with no projects: the map holds `README.md` and `project-root.md`, and the loop adds nothing. Correct.
- `renderReadme`, empty board: header, intro, then the root section at `##` with Tasks `(0)`, Ready `_None._` and Notes `(0)`. Correct.
- `renderReadme`, top-level projects P1 and P2 plus a project under a done project: `childProjects(rootProject)` returns all three by handle, each a `##` section, the root's last. Each project appears once, as a section and never as a row. Correct.
- `projectSection(rootProject, 2)`, member count: only entries with `Project == ""` and kind other than project, not done, are counted. Open projects are excluded and done entries are skipped. A fixture of one ready task and one done project gives 1, matching the golden. Correct.
- `projectSection` for a non-root project: the count still includes sub-projects, as before. Correct.
- `projectBody(rootProject, level)`: it returns `entrySections(root, level+1)` directly, with no `Own entries` heading, even when open projects exist. Correct.
- `entrySections(rootProject, ...)`: `own` takes `Project == ""`. Tasks and notes land in their sections. A done project goes to Done with a link to `projectFileName(slug)`. Open projects are not listed. A project that shares the root's slug match is not duplicated. Correct.
- `entrySections`, done project under no other: it appears once, as a Done row in the root section and in `project-root.md`, and is not rendered as a project section. Correct.
- `upLink(rootProject)`: `bySlug[""]` finds nothing, so it returns `[Board](README.md)`. Correct.
- `upLink`, top-level project: same `README.md` link. Correct.
- `upLink`, project whose parent is not on the board: `README.md`. Correct.
- `upLink`, project under a done parent: it links the done parent's file. The code is unchanged from before. Correct.
- `projectLink(rootProject)`: it links `project-root.md` through `rootProjectFile`, text `P0 Root: entries under no project`. Correct.
- `projectFacts(rootProject)`: returns `Owner <prime>`. Correct. With no prime owner it returns `Owner none recorded`, which matches the golden. Correct.
- `fileHeader(rootProject)`: the brief is empty and `Beam` is false, so it returns six lines with no beam word. Correct.
- `P0` handle: `ParseHandle` rejects a leading zero (`handles.go:53`), and assigned numbers start at 1, so no board entry can take `P0`. Correct.
- Heading depth: with five nested projects, P5 lands at level 6 and its own entries are capped. The golden pins this. I did not read `heading`'s cap itself. Correct per the golden.
- `isBoardOverview` (`replay.go:66-73`): it claims `project-root.md` through the prefix, so the replayer treats it as an overview file. Correct.

**Findings**

- **F1, MEDIUM, design.** `render.go:105` and `render.go:107-111`; `task.go:353-362`.
  - **Defect:** `Render` writes the root's file as `project-root.md` and then lets any project overwrite that key. `projectFileName("root")` equals `rootProjectFile`, so a project with slug `root` replaces the root's file.
  - **Visible effects:** the root section's heading link then points at that project's file. `Render` returns no error and `README.md` shows no sign of it.
  - **Slug validation:** `validateSlugLength` is the only slug check I found in `NewTask` (`task.go:357`). A grep of `boardengine` and `boardcli` found no reserved-slug check. I did not read the whole create path, so a check elsewhere is possible.
  - **Fix:** refuse a project with slug `root` at slug validation, or have `Render` return an error when a project's file name equals `rootProjectFile`.

**Tool calls**
1. Read `handles.go:15-85`: to confirm `P0` cannot be assigned to a board entry.
2. Read `render.go:350-363`: to see the rest of `fileHeader`, which the manifest lacked.
