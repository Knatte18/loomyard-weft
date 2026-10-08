# Card 14 — retire-fresh-after-reset

**Edit:**
- `internal/websterengine#freshRestartSteps`
- `internal/websterengine#planBaselineWayForward`
- `internal/websterengine#RestorePlan`
- `internal/websterengine#Rebaseline`
- `internal/websterengine#rebaselineBatches`
- `internal/websterengine#deleteReferencedWayForward`
- `internal/websterengine#resetToStartSteps`
- `internal/websterengine#fingerprintMismatchWayForward`
- `internal/websterengine#BeginBatch`
- `internal/websterengine#Run`
- `internal/websterengine#RunDeps`
- `internal/websterengine#pendingPathsWayForward`
- `internal/websterengine#freshPendingDrop`
- `internal/websterengine#headBeforeEveryStart`
- `internal/websterengine#AcceptPendingAudit`
- `internal/websterengine#AcceptBatchFabricReference`
- `internal/websterengine#RecoverSpawnOrAttach`
- `internal/websterengine#BatchState`
- `internal/websterengine/fingerprint.go`
- `internal/websterengine/doc.go`
- `internal/websterengine/auditledger_test.go`
- `internal/websterengine/planbaseline_test.go`
- `internal/websterengine/findingtext_test.go`
- `internal/websterengine/suspect_test.go`
- `internal/websterengine/rebaseline_test.go`
- `internal/websterengine/runlevel_test.go`
- `internal/websterengine/recoverbatch_test.go`
- `internal/websterengine/recordbatch_test.go`
- `internal/websterengine/beginbatch_test.go`
- `internal/webstercli#websterCLI.acceptAuditCmd`
- `internal/webstercli#websterCLI.rebaselineCmd`
- `internal/webstercli#websterCLI.resetCmd`
- `internal/webstercli/verbs_test.go`
- `contracts/specs/refusal-spec.md`

**Delete:**
- `internal/websterengine#stepRunFresh`

**Create:**
- `cmd/lyx/websterretiredsteps_test.go`

**Uses:**
- `internal/websterengine#stepResetToStart`
- `cmd/lyx/norecoverydeny_test.go`

**Intent:** No webster way forward names `lyx webster run --fresh` as the step after a reset, now that `reset --to start` archives the run record (card 12) and a plain run starts over (D3's text half).
Every "1) `lyx webster reset --to start`; 2) `lyx webster run --fresh`" becomes "1) `lyx webster reset --to start`; 2) `lyx webster run`":
- `freshRestartSteps` and `planBaselineWayForward` become a function of the re-entry step, and their callers `Rebaseline`, `rebaselineBatches`, `deleteReferencedWayForward` and `RestorePlan` pass the plain `lyx webster run`;
- `fingerprintMismatchWayForward` takes the re-entry step, `BeginBatch` passes `lyx webster run` and `Run` passes `deps.reentryStep()`, so a refusal surfaced through the Webster producer keeps ending in the row's re-entry step;
- `pendingPathsWayForward`'s reset route ends in its `reentry` argument;
- `freshPendingDrop`'s and `headBeforeEveryStart`'s `run --fresh` refusals, `AcceptPendingAudit`'s unverifiable clause, `AcceptBatchFabricReference`'s remaining start-route refusals and `RecoverSpawnOrAttach`'s needs-fresh text call `resetToStartSteps` with `lyx webster run`;
- `stepRunFresh` is deleted once no caller remains;
- comments that state the reset route ends in `run --fresh` (`RunDeps.ReentryStep`, `BatchState.Uncheckable`, `restampFingerprint` in `fingerprint.go`, the package doc) say it ends in a plain run;
- the accept-audit, rebaseline and reset long help say `lyx webster run` after a reset.
Unchanged: `run --fresh` itself and its own refusals' re-run steps that follow no reset (`rm <path>`, then `lyx webster run --fresh`), and `sequence.go`'s order refusal for a run that has not begun a batch.
`websterretiredsteps_test.go` pins the absence: it scans the non-test Go string literals under `internal/webster*`, `contracts/stencils/webster/` and `contracts/specs/` with the token matcher `norecoverydeny_test.go` already declares, and fails on any `git reset --keep`, and on any line where `lyx webster reset` precedes `lyx webster run --fresh`; a self-test plants both shapes and a lone `lyx webster run --fresh` that passes.
Grep the tree for `lyx webster run --fresh` and `2) lyx webster run --fresh`; every test asserting the old text joins this card (the files listed), updated to the plain run, and `refusal-spec.md`'s webster rows naming the pair are rewritten.

**ImpactSummary:** Rewords every reset-then-fresh way forward across webster's verbs and help; fingerprintMismatchWayForward gains a re-entry parameter its two callers pass.
