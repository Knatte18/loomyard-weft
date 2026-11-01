---
round: 1
focus:
  - "`projectBody` in `internal/boardengine/render.go`: does the own-entries heading appear exactly when the project has open sub-projects, with `No project` for the root and `Own entries` for any other project, and does a project with none render byte-for-byte as before?"
  - "Heading depth: the own-entries heading sits at the sub-projects' level and its entry sections one level further down, but `heading` caps at level six. At deep nesting, does the cap make the own-entries heading or its Tasks, Notes and Done sections collide with a sub-project's heading or each other, and does `entrySections` stay consistent at `level+2`?"
  - "README and project-file paths: a project file's header stands as the project heading, so confirm the own-entries heading renders at `##` and its sections at `###` there, and the README Prime case follows the same rule. Check that an open project under a done project, counted as under the root, still gets `No project` when the root has sub-projects."
  - "Heading counts: `No project` and `Own entries` carry no entry count and are excluded from the counted-heading rule. Check that nothing else in the render, such as anchors, links or counts from a done project row, assumed the old shape."
  - "Tests in `internal/boardengine/render_test.go`: do the changed and new cases cover the root and non-root variants, the no-sub-project unchanged case and the depth cap, without duplicating an existing test's coverage?"
  - "Doc comments in `render.go` and `doc.go`, and `docs/overview.md`: do they match the behavior the diff implements and the decision record's Design bullets, and are any other docs describing the board's rendered shape left stale?"
---

Round 1 seed. The answer key is the decision record's Goal, which has no later amendments.
