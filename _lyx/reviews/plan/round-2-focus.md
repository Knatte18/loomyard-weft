---
round: 2
focus:
  - "Cards that add a required top-level marker such as `{{.edit_directive}}` to a stencil (`stencil.Fill` in `internal/stencil/stencil.go` refuses a template whose marker has no value and drops a value whose marker the template lacks): does each such card name every test in its targeted packages that hand-builds that stencil's text or its values map, for example by `stencil.Fill`, `parentdirective.MarkerName` or a `{{.parent_directive}}` fixture string, and is each planned assertion reachable against what that test renders?"
  - "The pair call in `hubreconcile` (matching `opts.Pair` against a `CodeWorktree.Path` built from `fabricengine.List`, whose `parseWorktreePorcelain` stores git's porcelain path as printed): do the cards that build the listing, call it and pass `opts.Pair` (from `fabricengine.WorktreePath`, a `filepath.Join` result) state one normalized path form on both sides of the comparison, and does a planned test pin that equality for a pair?"
---

Focus targets the two gating design questions of the closing round, restated from their sites.
