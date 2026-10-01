# Card 63 — lyx-state-run-dirs

**Edit:**
- `internal/websterengine/audit.go`
- `internal/websterengine/doc.go`
- `internal/websterengine/auditseverity_test.go`

**Intent:**
Fix PR #321 round-4 finding F.
A Master write under `.lyx` outside webster's `ScratchDir` (another module's lock or pause flag, a reed launch script) is git-ignored, so `ClassifyViolation` classes it policy, although `.lyx` is lyx's own never-tracked run state and a write there can corrupt it.

In `ClassifyViolation`, `runDirs` gains `filepath.Dir(geom.ScratchDir)`, mirroring `filepath.Dir(geom.WebsterDir)`.
The doc comment names it.
`checkSuspectPaths` already reports such a path unverifiable, so its way forward is the `--fresh` route, consistent with the scratch directory.

`doc.go`'s audit paragraph lists the `.lyx` state directory among the correctness places.

`auditseverity_test.go` gains rows: a Master write to `<.lyx>/reed/launch/<guid>` and to another module's lock under `.lyx` → correctness;
a git-ignored build artifact in the task worktree outside `_lyx` and `.lyx` → still policy.

**ImpactSummary:** A Master write anywhere under the run's `.lyx` state directory is a correctness finding instead of a policy warning.
