# Card 12 — cleanup-removal-split

**Create:**
- `internal/battencli#removeRunRecords`

**Edit:**
- `internal/battencli#cleanupSlug`

**Uses:**
- `internal/battencli#emptyScratch`
- `internal/battencli#TestCleanup`

**Intent:**
Split the per-slug removal out of `cleanupSlug` so the cancel calls the same step under its own held locks (D6), and the two verbs cannot drift on what a run's records are.
`removeRunRecords` takes no lock and makes no judgement: it removes `shedrun.RunDir(location, slug)` when it is on disk, commits `shedrun.RunDirRel(location, slug)` on the records side through `deps.Commit` with `msg`, then empties `shedrun.ScratchDir(location, slug)` beside its lock files through `emptyScratch`.
An already absent run directory still commits, so a removal whose commit failed earlier finishes on a re-run; a removal error, a commit error and an emptying error are returned, each wrapped with what failed.
`cleanupSlug` keeps its lock-taking, its `judgeRunDirectory` judgement, its dry run and its skip reasons unchanged, and for an applied done candidate calls `removeRunRecords` with its existing message `batten cleanup: remove the run directory of <slug>`, mapping a removal or emptying error to its existing skip and a commit error to its existing returned error; the orphan path still calls `emptyScratch` alone.
Cleanup still skips non-done runs.
A pure refactor of cleanup: `TestCleanup` and the other cleanup tests pin the behaviour unchanged.
The cancel's use, the removal of a run that is not done, is reached by card 19's integration test, since card 18's `TestCancelRun` drives the removal through a fake seam.

**ImpactSummary:** `cleanupSlug` is reached only by `cleanup`, behind `Cleanup` and `lyx cleanup`'s batten segment; their observable behaviour is unchanged.
