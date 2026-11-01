# Card 10 — Fabric lists the task branches of a slug prefix

**Create:**
- `internal/fabricengine#TaskBranchSlugs`
- `internal/fabricengine_test#TestTaskBranchSlugs`

**Extends:**
- `internal/fabricengine/doc.go`

**Intent:**
D6 numbers each adopt fix task past every `<prefix>-t<N>` slug a task branch fabric named still holds, local or on origin, since a torn-down pair keeps its task branch after `lym batten cleanup --apply` removes its run directory.
Only `fabricengine` runs git on warp (PATTERN-fabric-git), so the listing lives here.

`TaskBranchSlugs` returns, sorted and without duplicates, the slugs whose task branch, the hub's fabric `BranchPrefix` followed by the slug, exists in the prime's warp repository as a local branch or as an `origin` remote-tracking branch, for every slug that starts with prefix.
It reads the fabric config through `LoadConfig(ConfigDirs(prime))` and the refs in-process through the package's git read seam, never `git` on the command line and never the network: the remote-tracking refs are the ones the last fetch left, which adopt's origin reconcile refreshes before the check runs.
An empty result is no error.

`TestTaskBranchSlugs` (tier1, package `fabricengine_test`, hermetic git as the package's other integration git tests set it up): with local branches for `adopt-fix-x-t1` and `other`, and an origin remote-tracking branch for `adopt-fix-x-t3`, prefix `adopt-fix-x-t` returns `adopt-fix-x-t1` and `adopt-fix-x-t3`; a prefix nothing matches returns none.

`doc.go` gains the line naming `TaskBranchSlugs` and its read-only, offline listing.

**ImpactSummary:** one new read-only function in fabricengine; no existing caller changes.

**Commit:** 10: fabric: list the task branches of a slug prefix, local and on origin
