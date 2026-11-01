```yaml
lens: fix-consistency
outcome: reported
```

Task: check the F1 change set for consistency with its surroundings and for twin sites left unchanged.

**Findings**

1. **MEDIUM** — `task.go:477` (`ApplyPatch`) — the twin of the `NewTask` check was left as it was.
   - `ApplyPatch` still calls only `validateSlugLength`. The reserved-slug check was added to `NewTask` and `Rename` but not here.
   - A patch such as `{"slug":"root"}` on a project entry therefore yields a project with slug `root`, and the collision in F1 recurs.
   - The code (`task.go:477`) shows that `ApplyPatch` validates the slug length the same way `NewTask` does, so the reserved check belongs beside it.
   - The fixer may have skipped it because an already-stored `root` entry would then fail every later patch. If so, a check applied only when the patch changes the slug (`result.Slug != existing.Slug`) avoids that.
   - Either way, the asymmetry between the three slug entry points is undocumented.

2. **LOW** — `task_test.go` (the new `NewTask` subtest) — it is inconsistent with the sibling rows. The new case sits in a subtest list while its `Rename` twin is a table row, and no `ApplyPatch` case exists (`task_test.go:448` has the length twin). This follows from finding 1; fixing that finding means adding the matching patch case.

3. **LOW** — `render.go:3` and `doc.go:173` — both comments say "the root to project-root.md" and are still correct. Neither says the `root` slug is reserved, so a reader of the render docs does not learn why a project cannot be named `root`. Optional.

**Checked, consistent**
- `doc.go:338` now says `project-root.md` is the root project's.
- `docs/overview.md:365` matches.
- `rootProjectFile` is derived from `RootSubject`.
- The error text names the reserved slug as the review's `fix` asked.

**Further tool calls**
1. `grep` over `project-root`, `reserved` in `boardengine/*.go` and `docs/overview.md`, plus `validateSlugLength` and `validateSlugNotReserved` callers: my context did not show other doc statements of the root file, nor every caller of the slug validators.
