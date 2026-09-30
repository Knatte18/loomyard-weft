# Card 6 — websterengine-rebaseline

**Edit:**
- `internal/websterengine/fingerprint.go`
- `internal/websterengine/runlevel_test.go`

**Create:**
- `internal/websterengine/fingerprint_rebaseline_test.go`

**Intent:**
Export the sanctioned plan-rewrite re-baseline D6 asks for, so the rework row can extend an approved plan without Webster refusing it with `ErrFingerprintMismatch`.
Add `RebaselinePlanFingerprint(geom Geometry) error` to `fingerprint.go`:
it holds the lease from `AcquireStateMutation(geom.ScratchDir)` across `LoadState`, `restampFingerprint(st, geom.PlanDir)` and `SaveState`, releasing it before returning.
A nil state (no Webster run has started) is a no-op with a nil error, since there is no recorded fingerprint to desync.
Its doc comment names the rule it serves — every sanctioned plan rewrite is re-baselined immediately by its writer — and that it runs outside `Run`, which is why it takes the lease itself.
Run entry gains no rework special case.
`fingerprint_rebaseline_test.go`: an existing `state.json` with a stale fingerprint is restamped to the plan's current digest, its batch records survive untouched, and an absent `state.json` stays absent.
Add a `runlevel_test.go` case for the acceptance criterion:
seed what a finished run over a two-card plan leaves: a state recording batches 1–2 complete, a done `outcome.yaml` and a `summary.md` in the webster directory, and an integration report in the reports directory;
then append a third card, call `RebaselinePlanFingerprint`, and call `Run` with the test's fake Master starter.
The run returns no `ErrFingerprintMismatch`, the stale outcome and summary are archived and the stale integration report removed before Master spawns, the rendered Master prompt carries batches 1–2 as complete and only batch 3 as pending, and the integration prompt is written because the plan carries a `## verify:` section.

**ImpactSummary:** Adds one exported function beside restampAndSaveFingerprint; no existing caller changes.
