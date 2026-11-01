```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens on `projectBody` in `internal/boardengine/render.go`, running one concrete input per branch.

**Branch list**

| Branch | Input I ran | Result |
|---|---|---|
| `len(children) == 0`, root (`render.go:393`) | README, `level=2`, no open project | Correct: `entrySections(root, 3)`, the same as the old `level+1`, so the output is byte-for-byte unchanged. |
| `len(children) == 0`, non-root | Project file, `level=1`, no sub-project | Correct: entries at `##`, as before. |
| `len(children) > 0`, root (`render.go:400-405`) | README, `level=2`, two open projects | Correct: children at `###` through `projectSection(child, 3)`, then `### No project`, a blank line, the meaning line, a blank line, then Tasks at `####` and Running, Ready and Notes subsections at `#####`. Done is at `####`. This matches the golden at `render_test.go:608-612`. |
| `len(children) > 0`, non-root | Project file, `level=1`, one open sub-project | Correct: sub-project at `##`, `## Own entries` with its meaning line, then Tasks at `###` and subsections at `####`. |
| Non-root with children inside a README | Project at `###` with a sub-project | Correct: sub-project at `####`, `#### Own entries`, then entries at `#####` and subsections at `######`. |
| Root test `project.Slug == ""` | Root is the zero `Task`; a real project always has a slug | Correct. `projectSection` uses the same test for Prime. |
| Depth cap | Project at level 5 with a child: child at 6, `heading(6, "Own entries")`, then `entrySections(project, 7)` | Entries cap at six, so the Own entries heading, Tasks, Notes and Done all render at `######`. The same flattening happened before the diff, since a level-6 project's entries were already forced to `######`. The cap gives no wrong result, only a loss of hierarchy, which the docs state ("stop at six"). The goldens at `render_test.go:864-880` pin this, and `entrySections` stays consistent at `level+2`. Not a defect. |
| Open project under a done project | `childProjects(root)` includes it through the `isDone(parent)` clause | Correct: the root has a child, so it gets `No project`. A done project's own file lists its open children directly and gets `Own entries`. Both match the doc. |
| Counted headings | `No project` and `Own entries` are plain `heading()` calls, not `countedHeading()`. `members` in `projectSection` counts entries by project slug and is independent of the new heading. | Correct: the counts are unchanged. |
| Failure paths | None: there is no error return, no I/O, and `children` is never nil-dereferenced. | Nothing to fail. |
| `joinLines` | Trailing blank lines after the last section | Unchanged behavior. |

Findings: none.

Tool calls: none. The manifest already held `render.go`, the diff and the tests.
