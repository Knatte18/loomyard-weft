Lens fix-affected-set: no findings.

- F3: no stale "worktree's root" or "worktree root" wording remains in `internal/heddlecli` or `internal/heddleengine`; `internal/heddleengine/render.go:37` keeps "root" for the no-anchor case only. A repo-wide grep for render/root phrasing outside `_lym` and `.lym` found no other stale doc or help text.
- F2: unchecked. The added row covers the `wireJunctionSet` anchor argument through `WireJunctions`, the only call the review named.
- F4: unchecked. The new subtest covers both file targets at the anchor and the `show` verb; the review named only `verbs.go:295` and `:335`.
- F1: disputed by the fixer, so not checked.

Tool calls:
1. grep for the "root" wording in `internal/heddlecli` and `internal/heddleengine`: the context did not hold the other files in those packages.
2. grep for render/root phrasing repo-wide: the stale-text set is under one key, so the finding's own search was rerun.
