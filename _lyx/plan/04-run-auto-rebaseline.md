# Card 4 — run-auto-rebaseline

**Edit:**
- `internal/websterengine#RunOptions`
- `internal/websterengine#Run`
- `internal/websterengine#finishMasterDone`
- `internal/websterengine/runlevel_test.go`
- `internal/websterengine/doc.go`
- `//contracts/specs/refusal-spec.md`
- `//cmd/lyx/testdata/test-budget.yaml`

**Create:**
- `plan:internal/websterengine#ErrAutoRebaseline` -> `var ErrAutoRebaseline error`
- `plan:internal/websterengine#autoRebaseline` -> `func autoRebaseline(deps RunDeps, plan *planparser.Plan, st *State, sizes batcher.SizeSource) (string, error)`

**Uses:**
- `internal/websterengine#Rebaseline`
- `internal/websterengine#changedPlanFiles`

**Intent:** Let the Webster row rebaseline a changed plan on entry instead of failing on the fingerprint (D1).
`RunOptions` gains `AutoRebaseline`, which `lyx webster run` never sets.
In `Run`'s fingerprint-mismatch case, when it is set and `Fresh` is not, `autoRebaseline` runs before anything else: it reads `changedPlanFiles`, takes every changed card file's number as the named cards, and calls `Rebaseline` with the plan, `deps.Batcher`, the disk sizes, Merriam's base, the loaded state and `Step` set to `deps.reentryStep()`, through the same inputs `lyx webster rebaseline` builds.
An accepted rebaseline is saved with `SaveState` before the batch loop starts, so a crash after it re-enters on a matching fingerprint; the batches are then `ExecutionBatches` over the restamped state.
The warning it returns names the rebaseline and the accepted cards, is logged at Warn, and joins the run's entry warnings: a done outcome carries it on `RunResult.Warnings` and in the summary's warnings, and `finishMasterDone` opens a stuck outcome's reason with it.
Every error the rebaseline or its save returns is wrapped in `ErrAutoRebaseline`, with the refusal's or the transient's own text, and a transient's text says to re-step the row.
Any mismatch with `AutoRebaseline` unset keeps today's `ErrFingerprintMismatch` text, and `Fresh` keeps its archive path.
A nil plan or state stays a returned error.
Bound: the auto path accepts exactly what an operator naming every changed card with `--card` gets; a done batch's card edit, a begun batch's card-set change, a failed batch with uncheckable findings and an overview change outside its Card Index are refused with the rebaseline's own step-keyed text, and a card of an in-flight batch is accepted as an amendment, recorded unrendered.
The auto path syncs nothing to the fabric itself: the restamped state reaches it with the run's next sync.
The refusal-spec gains the row `Webster row: auto-rebaseline refused`, whose way forward is the rebaseline's own text re-stepping the row, and the row `Webster row: auto-rebaseline transient`, re-stepping the row.
`doc.go`'s crash/resume section states the auto-rebaseline beside the `--fresh` escape.
A new `TestRun_AutoRebaseline` table in `runlevel_test.go` drives the acceptance rows: a follow-up card added after the last begun batch continues over the kept batches with the new card batched, the restamped state on disk before the first batch starts and the warning on the result; a done card's edit returns `ErrAutoRebaseline` with a text naming re-stepping and neither `lyx webster run` nor `lyx webster rebaseline`; a begun batch's card-set change, a failed batch with uncheckable findings, an overview change outside the Card Index and a save failure each return `ErrAutoRebaseline`; the same edits with the option unset return `ErrFingerprintMismatch`; an accepted rebaseline followed by Master's stuck gives a reason opening with the warning; and after a simulated crash mid-batch an in-flight card's edit is recorded as an unrendered amendment.
The websterengine row of `test-budget.yaml` rises by one for the new test.

**ImpactSummary:** Run's fingerprint case gains one branch behind a new option; every existing caller leaves the option unset and sees no change.
