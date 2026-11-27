# Card 9 — engine-repo-side-names

**Edit:**
- `internal/fabricengine#`

**Intent:** Retire the engine's private repository-side constants `failedPushSideCode`, `failedPushSideRecords` and `failedPushSideBolt`.
Card 8 moved their exported twins to `fabriclayout.RepoCode`, `RepoRecords` and `RepoBolt`, and two names for one value is a name that can go stale (`PATTERN-code-conventions`).
Every engine use, across the coalesce, reconcile, remove, push, spawn, Bolt and destroy files and the package's tests, reads the layout constant instead, and the three private constants go.
`doc.go` is unchanged.
No behavior changes (D5): the values are the same strings.

**ImpactSummary:** One package, about a dozen files of `internal/fabricengine`; a constant-for-constant swap of equal values with no effect outside the package.

**Commit:** `9: engine-repo-side-names`
