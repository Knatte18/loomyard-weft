# Card 3 — webster-archive-run-record

**Edit:**
- `internal/websterengine/archive.go`
- `internal/websterengine/archive_test.go`

**Create:**
- `plan:internal/websterengine#ArchiveRunRecord` -> `func ArchiveRunRecord(geom Geometry, dest string) error`

**Intent:**
Add the primitive D8 names: move webster's durable run record into a directory the caller tells it, so the next `lyx webster run` finds no state and starts a fresh run over the live plan only.

`ArchiveRunRecord`:

- takes the run lock non-blocking first and refuses with `ErrRunBusy` (wrapped, naming the way forward: wait for the run to finish, then retry) when a run holds it, then takes the state-mutation lease (`AcquireStateMutation`) for the moves;
- moves every entry of `geom.WebsterDir` (state, reports, outcome, summary, integration report, plan baseline, and any archive siblings) into `dest`, creating `dest`, one `os.Rename` per entry;
- is idempotent for crash resume: an absent `geom.WebsterDir` or an entry already moved is skipped, and an entry present at both source and destination is an error naming both paths and the way forward (remove whichever copy is stale, then re-step);
- clears `geom.PromptsDir` the way the `--fresh` escape does (`clearRenderedPrompts`), since rendered fork prompts are re-renderable and name the retired generation's batches.

`dest` is told and never derived, per the Told-Geometry Invariant; the function knows nothing of rework rounds.

Tests (tier 1, temp directories, no git): a populated webster dir moves whole into `dest` and `LoadState` over `geom.WebsterDir` then returns nil; a second call is a no-op; an absent webster dir is a no-op; an entry present on both sides returns the collision error and moves nothing further; a held run lock returns an error matching `ErrRunBusy`; the prompts dir is gone afterwards.

**ImpactSummary:** additive: one new exported function beside the `--fresh` archive helpers it reuses; no existing caller changes.
