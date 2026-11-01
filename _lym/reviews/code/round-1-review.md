---
verdict: APPROVED
---

No finding survives.
The diff makes the change the decision record's Design bullets ask for in `projectBody`.
Behavior checked against the focus questions:

- The own-entries heading appears exactly when `childProjects` returns an open sub-project, `No project` for the zero-Task root and `Own entries` for any other project, each with its meaning line and no entry count.
- A project with no open sub-project takes the early return and renders at `level+1`, as before.
- At depth the cap flattens the own-entries heading and its sections to `######`, as a level-six project's entries were already flattened before the diff; `entrySections` stays at `level+2`, and the deep golden row pins it.
- The README and project-file paths follow the same rule: Prime's children at `###` with `### No project` and entries at `####`, a project file's children at `##` with `## Own entries` and entries at `###`.
- An open project under a done project counts as a root child, so the root gets `No project`.
- Nothing outside `boardengine` reads heading shape; `doc.go`, the file header, the `projectBody` comment and `docs/overview.md` match the implemented behavior.
- Every changed behavior has a golden row in `TestRenderReadmeGolden`, and no test duplicates another's coverage.
- The commit carries no `Pattern-Departure:` trailer and needs none; the diff touches no `PATTERN.md` or `pattern/` file and the directive declares none.
