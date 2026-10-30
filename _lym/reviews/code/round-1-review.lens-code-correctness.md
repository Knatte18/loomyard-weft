```yaml
lens: code-correctness
outcome: reported
```

**Lens: code-correctness.** I ran each changed function through one concrete input.

**Per function, case run, result**
- `fileHeader`, non-beam with an empty brief: returns `[# title, "", rendered, Under, ""]`. Correct. Non-beam with a brief gets the brief and a blank line, with no `grouping` line. Correct. A beam still gets `beam`, a blank line, `Owner:` and a blank line. Correct. The root path is unchanged.
- `renderReadme`, no projects on the board: gives header, `boardIntro`, `## Prime`, meaning line, then `### Tasks` and the rest. Correct. `append(fileHeader, boardIntro...)` copies elements and does not mutate the package variable `boardIntro`. Correct.
- `renderProjectFile`: gives header, intro, `projectsSection`, then `entrySections(project, "##")`. This is the same order and levels as the old `renderBoardFile`. Correct.
- `joinLines`: input ending in `""` gives exactly one trailing newline, same as before. Correct.
- `table`: the extraction from the closure keeps the header, the slug inserted at position 1 and the trailing blank line. The `Prio` header comes only through `columnPriority`. Correct.
- `entrySections`, root with `"###"`: gives `###` for Tasks, Notes and Done, and `####` for the subsections. With `"##"` it gives `##` and `###`, so project pages keep their levels. Correct.
- `entrySections`, root with no done entries: Done is omitted and no extra blank line appears. Correct.
- `childProjects`, root: `Task{}.Slug` is `""`, so the match is `twl.Project == ""`, as before.
- `projectSections`, project with no open members: `_None._` plus a blank line. Correct. Project with a brief: brief shown before the table. Correct.
- `projectSections`, nested: members include the nested project rows, so `P1 (6)` counts `P4` and `P5`. This is consistent with D1's "every kind", and recursion then emits the nested sections after it. Correct.
- `projectSections` heading: `fileLink` applies `cellText` to the title. A `|` in a title would appear in the `##` heading as `\|`. CommonMark renders that as `|`. Correct.

**Findings**

1. **LOW, design.** `render.go:479-497` and `render.go:384`.
   - A project's order key is never applied to the README's root-level project sections.
   - `renderReadme` calls `projectSections(Task{})`, and the zero Task has an empty `Order`.
   - `position` therefore returns `len(project.Order)` = 0 for every slug, and root-level sections sort by handle only.
   - D1 says "each root-level project in the root's order then by handle", and `doc.go` repeats it as "in the root's order key".
   - `Render` takes no root order (`render.go:94-100`), so that order is not reaching the render.
   - The old Projects bullets had the same limit, so this is not a regression, but the new text over-claims.
   - Fix: either pass the root's order into `Render` and `renderReadme`, or reword D1's text in `doc.go` and the `projectSections` comment to "by handle" for root-level projects.

2. **LOW, design.** `render.go:384`, `render.go:379-381` and `render.go:477-481`.
   - An open project whose parent is a done project is never reached.
   - `childProjects` skips done projects, and `projectSections` only recurses from `childProjects`.
   - Its open members therefore appear nowhere in the README: the done parent has no section and Prime does not hold them.
   - The goal is "no entry leaves the README's view".
   - I did not check whether the store allows this state, so this finding depends on it.
   - Fix: if the state is reachable, render a section for an open project under a done parent, for example by walking its open descendants from the root. If it cannot occur, no change is needed.

All other branches I ran were correct.

**Tool calls**
1. `sed -n 478,500p render.go` and `grep -n Order render.go`: my context cut off `childProjects`' `position` body and did not show where a root `Order` could come from.
