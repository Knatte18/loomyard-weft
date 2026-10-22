# Card 3 — fabric-parent-slug

**Create:**
- `internal/fabricengine#ParentSlug`
- `internal/fabricengine_test#TestParentSlug`

**Uses:**
- `internal/fabricengine#ReadOriginFor`

**Intent:**
The effective owner's parent resolver, shared by every owner read (D2).
`fabricengine.ParentSlug` returns the slug of the pair whose branch slug's origin record names as its parent branch, found among the hub's warp worktrees by branch, and `""` for the prime's branch, a slug with no local pair and a pair with no origin record; a read error is returned unchanged, a propagated raw I/O error.
`TestParentSlug` (integration over `hubforge`) covers a pair forked from the prime, a pair forked from another pair, a slug with no pair and a pair with no origin record.

**ImpactSummary:** fabricengine gains one read helper; no existing caller changes.

**Commit:** 3: fabric: resolve a task's parent pair slug from its origin record
