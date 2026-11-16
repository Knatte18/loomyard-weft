```yaml
lens: code-correctness
outcome: reported
```

Each function / branch the diff adds or changes — case run — result:

- `runplan.ResolveAtDispatch` (`internal/runplan/conflict.go:169`): nil `Index` → `errNoIndex` before any restamp; correct. `ValidateDispatch` error + restamp error → `resolveErr` returned, restamp error not; correct. No blocking → advisories; correct. Blocking, none begun → `plan_defect`; delete-before-reference only → `plan_order`; correct. `WayForward` set from `planConflictWayForward(batchNumber)` → `Error()` renders the same text as before; correct.
- `runplan.PostBatchChecks` (`internal/runplan/postbatch.go:1379`): `CompletedCards` now computed after the re-sign restamp instead of by the caller — it reads only `st.Batches` Terminal flags, which the restamp does not touch → same result; correct. `deltaErr` quarry-unavailable → no hand-back, could-not-run warning; correct. `HandBackRemovedTargets` with removed targets → sets `LaterTargetsReturned`, returns error with caller way forward; correct. Restamp failure after bind error → bind error wins; correct. Blocking drift → `RecordWarning` → `recordBatchWarning(st, bs, …)` as before; correct.
- webster `postBatchInputs` wrapper (`internal/websterengine/recordbatch.go:4016`): `Label` = `CardID` of first card = record-batch's `polledID` (`%02d-%s` of `BatchIdentity`) and recovery's old label; `Number` from `BatchIdentity` of the batch `findBatch` matched = requested number; recovery passes `handBack=false` as before; correct.
- `failCardNotDone` (`recordbatch.go`): `errors.As` against `*runplan.CardNotDoneError` copies exported flags; fallback trims `ErrCardNotDone` alias prefix, which is the same sentinel the bind-blocking wrap uses; correct.
- `runplan.CardNotDoneFor(0, nil, findings)` in `export_test.go`: no handles logged, same as old zero `postBatchInputs`; correct.
- `runplan.MapPartition` / `FindBatch`: refusal text byte-identical to old webster text given the passed constants; correct.
- `runplan.CheckRecoveredSuspects`: plan reason ends in `planPathClause("\"lym webster recover-batch NN\"")`, same as old inline; correct.
- `runplan.RestampFingerprint` / `moveBegunCardHashes`: `before` nil → no move; store failure → `st` unchanged; correct.
- webster `RemoveRecoveryStrands` (`internal/websterengine/strand.go`): nil `st` → nil; order batch strands then verify fixers then plan fixers as before; correct.
- webster `DispatchScope` wrapper: nil `st` → `nil, nil` before dereferencing `&st.State`; correct.
- `runplan.IsLastBatch` empty batches → false; correct.
- `runplan.AwaitBatch` unknown batch → same `findBatchWayForward` text; correct.
- Embedded `runplan.State` in webster `State` through `internal/state` unknown-key keeper: `declaredFields` walks embedded structs via `reflect.VisibleFields` (`internal/state/keep.go:186-200`), so promoted keys count as known; correct.
- `runplan.PlanEditError` (`internal/runplan/fingerprint.go:650`): mismatch message changed — wrong, see finding.
- `runplan.BatchCardEditError`: format `"%w: %s; %s"` matches old; correct.

Findings:

1. **LOW, design** — `internal/runplan/fingerprint.go:650` `PlanEditError`: webster's fingerprint-mismatch refusal text now reads "the plan changed since the run state was created" where it read "the plan changed since state.json was created" — an unannounced change to webster's refusal text against D11's "Webster's behaviour stays as it is" (the change summary declares only the `webster:`→`runplan:` sentinel prefix change), and webster's own `Run` entry check still says "state.json", so begin-batch/record-batch/recover-batch/validate and `run` now word the same refusal differently. Evidence: `git show 194650f0:internal/websterengine/fingerprint.go:142` (old text), `internal/runplan/fingerprint.go:163` (new), `internal/websterengine/runlevel.go:575` (unchanged "state.json" text); callers `beginbatch.go:362`, `recordbatch.go:447`, `recoverbatch.go:720`, `webstercli/validate.go:214`.

Tool calls:
1. `git show` of old `fingerprint.go`, `conflictrecord.go`, `strand.go`, `PlanEditError` callers, `internal/state` function list — to compare old/new refusal text and JSON key handling, which context lacked.
2. `git show` of old `batchCardEditError` tail and old verifyparse/verifygatereport function lists plus grep for the "state.json was created" text — to confirm the wording drift and renamed-function parity.
