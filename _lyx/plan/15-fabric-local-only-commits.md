# Card 15 — fabric-local-only-commits

**Create:**
- `internal/fabricengine#Fabric.LocalOnlyCommits`
- `internal/fabricengine_test#TestLocalOnlyCommits`

**Intent:**
The read Finalize's adopter-first repair note lists (D6).
`Fabric.LocalOnlyCommits` lists the commits of the local branch its origin tracking ref lacks, newest first.
`TestLocalOnlyCommits` (integration over `hubforge`) covers a branch level with origin, one with two local commits, and one with no tracking ref.

**ImpactSummary:** fabricengine gains one read; no existing caller changes.

**Commit:** 15: fabric: list a branch's commits its origin lacks
