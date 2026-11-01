```yaml
lens: code-affected-set
outcome: reported
```

Task: lens code-affected-set for the Recipe-cell/profile-bullet diff.

**Per change, the set found:**
- Removal of the `profile: <name>` bullet in `entryCell` (`render.go`): callers are `entryCell` in the Notes, Done, wait and Running tables, all in `render.go`. The grep for `profile: ` and `profile bullet` finds no other renderer, doc, stencil, spec or fixture that describes the bullet. `doc.go` is updated. Covered.
- `recipeCell` now appends ` · profile`: its only callers are `render.go:471` (the wait table: Ready, layers, Draft) and `render.go:493` (Running). Both pass the whole `Task`, so the profile reaches every Recipe cell. `doc.go:205` matches the new behavior. Covered.
- Test doc comment on `TestRenderReadmeGolden` (`render_test.go:267`) still says "loom for an empty recipe". This was stale before the diff, because the code uses `tapestry`. The comment also does not mention the profile in the Recipe cell, which the golden row now pins at line 333. Not covered; see F1.
- PATTERN.md and `pattern/`: the grep for README and board entries finds no entry that governs the board README rendering. The diff declares no PATTERN changes and carries no `Pattern-Departure:` trailer. Nothing to land.
- Commit message: it matches the diff.

**Findings:**
- F1, LOW, consistency, `internal/boardengine/render_test.go:267`: the test's doc comment says "loom for an empty recipe" (the code shows `tapestry`) and does not say that the Recipe cell now carries the run profile. The diff changes the assertion at line 333 but leaves the comment describing the old behavior. Evidence: `render_test.go:252` (fixture with `Profile: "docs"`), `render_test.go:267`, `render.go:193-194`.

**Tool calls:**
1. Grep for `profile: `, `profile bullet`, `Recipe column` and `Recipe cell` across the repo (excluding `.git`, `.lym`, reviews), to find other docs, stencils or specs describing the bullet.
2. Grep for "loom for" in `render_test.go` and for README mentions in `PATTERN.md` and `pattern/`, to confirm the stale test comment and that no PATTERN entry applies.
