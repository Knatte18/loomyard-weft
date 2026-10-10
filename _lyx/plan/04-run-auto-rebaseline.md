# Card 4 — run-auto-rebaseline

**Edit:**
- `internal/websterengine#RunOptions`
- `internal/websterengine#Run`
- `internal/websterengine#finishMasterDone` -> `func finishMasterDone(deps RunDeps, batches []batcher.Batch, outcomePath, summaryPath string, result shuttleengine.Result, freshWarnings []string, rebaselineWarning string, gateNotes *VerifyGateNotes) (RunResult, error)`
- `internal/websterengine#zeroBatchesError` -> `func zeroBatchesError(planDir, step string) error`
- `internal/websterengine/runlevel_test.go`
- `internal/websterengine/doc.go`
- `//contracts/specs/refusal-spec.md`
- `//cmd/lyx/testdata/test-budget.yaml`

**Create:**
- `plan:internal/websterengine#ErrAutoRebaseline` -> `var ErrAutoRebaseline error`
- `plan:internal/websterengine#autoRebaseline` -> `func autoRebaseline(deps RunDeps, plan *planparser.Plan, st *State, sizes batcher.SizeSource) (string, error)`
- `plan:internal/websterengine#appendRebaselineWarning` -> `func appendRebaselineWarning(websterDir, warning string) error`

**Uses:**
- `internal/websterengine#Rebaseline`
- `internal/websterengine#changedPlanFiles`
- `internal/websterengine#MerriamBase`

**Intent:** Let the Webster row rebaseline a changed plan on entry instead of failing on the fingerprint (D1).
`RunOptions` gains `AutoRebaseline`, which `lyx webster run` never sets.
In `Run`'s fingerprint-mismatch case, when it is set and `Fresh` is not, `autoRebaseline` runs before anything else: it reads `changedPlanFiles`, takes every changed card file's number as the named cards, and calls `Rebaseline` with the plan, `deps.Batcher`, the disk sizes, Merriam's base, the loaded state and `Step` set to `deps.reentryStep()`, through the same inputs `lyx webster rebaseline` builds.
An accepted rebaseline is saved with `SaveState` before the batch loop starts, so a crash after it re-enters on a matching fingerprint; the batches are then `ExecutionBatches` over the restamped state.
The warning it returns names the rebaseline and the accepted cards, is logged at Warn, and joins the run's entry warnings.
`Run` hands it to `finishMasterDone` as `rebaselineWarning`, empty when no rebaseline ran, beside the entry warnings it already passes.
A done outcome carries it on `RunResult.Warnings`, and `finishMasterDone` writes it into `summary.md` through `appendRebaselineWarning`, a `Plan rebaselined` section appended beside the audit-warning section, so the summary the step's Done points at names it.
`finishMasterDone` opens a stuck outcome's reason with it after the verify-gate demotion, so a demoted reason carries it too, and every error `Run` returns after an accepted rebaseline is wrapped to open with the warning, keeping `errors.Is` on the original, so a returned error the adapter maps to Stuck names it as well.
Every error the auto-rebaseline or its save returns is wrapped in `ErrAutoRebaseline`: a refusal keeps the rebaseline's own step-keyed text, `Rebaseline`'s transients already carry the keyed re-step clause (card 3), and `autoRebaseline` adds that the step is transient and to re-step the row to a `SaveState` failure, to a `changedPlanFiles` read failure, which carry no way forward of their own, and to a `MerriamBase` failure, whose own `re-run the verb` clause names no step the row can take.
On a failed save `Run` returns before it starts Master or any batch, so it never continues on the unsaved restamp.
After an accepted rebaseline, `Run`'s own plan refusals that follow it, the plan-validation refusal, the zero-batch refusal and the quarry transient, name fixing the plan and re-stepping the row in place of `lyx webster rebaseline --card NN` and `lyx webster run`, and are wrapped in `ErrAutoRebaseline`, so a Plan-loop edit that breaks validation blocks the row as Stuck rather than failing it hard; `zeroBatchesError` takes the step, empty keeping today's text, and its two calls in `Run` pass the empty step unless a rebaseline was accepted.
The record licenses this widening twice: D1 says that on the auto path each refusal says to fix the plan and re-step the Webster row, never to run `lyx webster rebaseline` by hand, and its acceptance row says no auto-path refusal text contains `lyx webster rebaseline`, which these three texts do today once a rebaseline has put the run on the auto path; and D1 says a stuck outcome's reason opens with the rebaseline warning, which the error prefix carries to a returned error the adapter maps to Stuck.
Bound: the widening reaches only the run in which an auto-rebaseline was accepted; with none accepted, every later text and every disposition of `Run` is today's, and an error not wrapped in `ErrAutoRebaseline` or a sentinel the adapter already maps is still returned and still fails the row.
Any mismatch with `AutoRebaseline` unset keeps today's `ErrFingerprintMismatch` text, and `Fresh` keeps its archive path whether `AutoRebaseline` is set or not.
A nil plan or state stays a returned error.
Bound: the auto path accepts exactly what an operator naming every changed card with `--card` gets; a done batch's card edit, a begun batch's card-set change, a failed batch with uncheckable findings and an overview change outside its Card Index are refused with the rebaseline's own step-keyed text, and a card of an in-flight batch is accepted as an amendment, recorded unrendered.
The auto path syncs nothing to the fabric itself: the restamped state reaches it with the run's next sync.
The refusal-spec gains the row `Webster row: auto-rebaseline refused`, whose way forward is the rebaseline's own text re-stepping the row, and the row `Webster row: auto-rebaseline transient`, re-stepping the row.
`doc.go`'s crash/resume section states the auto-rebaseline beside the `--fresh` escape, and its summary passage names the `Plan rebaselined` section.
A new `TestRun_AutoRebaseline` table in `runlevel_test.go` drives the acceptance rows: a follow-up card added after the last begun batch continues over the kept batches with the new card batched, the restamped state on disk before the first batch starts, the warning on the result and in `summary.md`'s `Plan rebaselined` section; a done card's edit, a begun batch's card-set change, a failed batch with uncheckable findings, an overview change outside the Card Index and a follow-up card ordered before a card it uses each return `ErrAutoRebaseline` with a text naming re-stepping the row; a `SaveState` failure, forced by making the webster dir read-only (`0o555`) while its `plan-baseline` directory stays writable, as `runlevel_test.go` already does to fail a write there, so the restamp stores its copies and the state write fails, returns `ErrAutoRebaseline` with the re-step clause and the fake starter sees no `Start`; a restamp failure, forced by a file in place of the plan-baseline directory, and a `MerriamBase` failure, forced by a directory in place of the worktree's `CLAUDE.md`, each return `ErrAutoRebaseline` with the re-step clause; every `ErrAutoRebaseline` row asserts the text contains neither `lyx webster run` nor `lyx webster rebaseline`; an accepted rebaseline followed by Master's stuck, and one followed by Master's done demoted by a failed verify gate, each give a reason opening with the warning; after a simulated crash mid-batch an in-flight card's edit is recorded as an unrendered amendment, which a following `RecoverSpawnOrAttach` over that state renders in its recovery prompt; `Fresh` set beside `AutoRebaseline` over a changed plan archives the record as `Fresh` alone does and runs no rebaseline; an accepted rebaseline followed by pending audit findings returns an error opening with the warning that still matches `ErrPendingAuditFindings`; an accepted rebaseline followed by a blocking plan-validation finding returns `ErrAutoRebaseline` naming re-stepping the row and neither manual verb; and the accepted row asserts exactly one captured Warn line naming the rebaseline.
The log capture swaps the logger's process-global output, so `TestRun_AutoRebaseline` and its rows run without `t.Parallel` under a comment naming that output, as `TestRun_FreshOverPendingFindings` does.
`TestRun_RefusesBeforeSpawn` already pins that a mismatch with the option unset refuses with the fingerprint text, so the new table repeats no such row.
The websterengine row of `test-budget.yaml` rises by one for the new test.

**ImpactSummary:** Run's fingerprint case gains one branch behind a new option; finishMasterDone and zeroBatchesError gain a parameter whose only caller is Run, on this card; every existing caller of Run leaves the option unset and sees no change.
