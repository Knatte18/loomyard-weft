# Card 16 — retire-fresh-after-reset

**Edit:**
- `internal/websterengine#freshRestartSteps`
- `internal/websterengine#planBaselineWayForward`
- `internal/websterengine#RestorePlan`
- `internal/websterengine#Rebaseline`
- `internal/websterengine#rebaselineBatches`
- `internal/websterengine#deleteReferencedWayForward`
- `internal/websterengine#resetToStartSteps`
- `internal/websterengine#fingerprintMismatchWayForward`
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
- `internal/webstercli#websterCLI.runCmd`
- `internal/webstercli/verbs_test.go`
- `internal/webstercli/cli_test.go`
- `contracts/specs/refusal-spec.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Delete:**
- `internal/websterengine#stepRunFresh`

**Create:**
- `cmd/lyx/websterretiredsteps_test.go`

**Uses:**
- `internal/websterengine#stepResetToStart`
- `cmd/lyx/norecoverydeny_test.go`

**Intent:** No webster way forward names `lyx webster run --fresh` as the step after a reset, now that `reset --to start` archives the run record (card 13) and a plain run starts over (D3's text half).
Every "1) `lyx webster reset --to start`; 2) `lyx webster run --fresh`" becomes "1) `lyx webster reset --to start`; 2) `lyx webster run`":
- `freshRestartSteps` and `planBaselineWayForward` become a function of the re-entry step, and their callers `Rebaseline`, `rebaselineBatches`, `deleteReferencedWayForward` and `RestorePlan` pass the plain `lyx webster run`;
- `fingerprintMismatchWayForward` takes the re-entry step; of its three call sites, `PlanEditError` and `batchCardEditError` (both in `fingerprint.go`, a target of this card) pass the plain `lyx webster run`, and `Run` passes `deps.reentryStep()`, so a refusal surfaced through the Webster producer keeps ending in the row's re-entry step;
  `PlanEditError`'s signature is unchanged, so its callers (`BeginBatch`, `RecordBatch`, `PersistRecoveryTerminal` and `webstercli`'s validate) need no edit;
- `pendingPathsWayForward`'s reset route ends in its `reentry` argument;
- `freshPendingDrop`'s and `headBeforeEveryStart`'s texts for their `Run` caller only (card 12 gave both a caller's re-run steps; the reset caller's texts name no reset and stay), `AcceptPendingAudit`'s unverifiable clause, `AcceptBatchFabricReference`'s remaining start-route refusals and `RecoverSpawnOrAttach`'s needs-fresh text call `resetToStartSteps` with `lyx webster run`;
- `stepRunFresh` is deleted once no caller remains;
- comments that state the reset route ends in `run --fresh` (`RunDeps.ReentryStep`, `BatchState.Uncheckable`, `restampFingerprint` in `fingerprint.go`, the package doc) say it ends in a plain run;
- the accept-audit, rebaseline and reset long help say `lyx webster run` after a reset;
- `runCmd`'s long help, which says the fingerprint-mismatch refusal is "a message naming run --fresh", says the refusal names its way forward (the reset-to-start route, or rebaseline for an edit card 21 accepts), keeping its sentence on what `--fresh` itself does.
Unchanged: `run --fresh` itself and its own refusals' re-run steps that follow no reset (`rm <path>`, then `lyx webster run --fresh`), and `sequence.go`'s order refusal for a run that has not begun a batch.
`websterretiredsteps_test.go` pins the absence: it scans the non-test Go string literals under `internal/webster*`, `contracts/stencils/webster/` and `contracts/specs/` with the token matcher `norecoverydeny_test.go` already declares, and fails on any `git reset --keep`, and on any line where `lyx webster reset` precedes `lyx webster run --fresh`; a self-test plants both shapes and a lone `lyx webster run --fresh` that passes.
The scan and its self-test are two top-level tests, mirroring `norecoverydeny_test.go`'s pair, so `test-budget.yaml` raises `cmd/lyx` by two.
Grep the tree for `lyx webster run --fresh`, `2) lyx webster run --fresh` and a bare `--fresh` in test assertions; every test asserting the old text joins this card (the files listed), updated to the plain run.
That includes `TestRebaselineCmd_Refusals` in `webstercli/cli_test.go`, whose "removed card of a begun batch" and "named card of a done batch" rows assert `--fresh` and now assert the reset-then-plain-run route; card 21 later rewrites the done-batch row again for its follow-up-card way forward.

`refusal-spec.md`, by row, each rewritten so its reset-then-fresh pair reads `1) lyx webster reset --to start; 2) lyx webster run` and nothing else in it changes:
- `rebaseline card set changed`, `rebaseline: begun card of a done batch`, `rebaseline: failed batch recovery cannot check`, `rebaseline overview changed`;
- `plan fingerprint mismatch` (its pair and its "take the `--fresh` route" clause, which becomes "take the reset-to-start route") and `plan changed before the verb's own rewrite`;
- `restore-plan: copy missing`;
- `audit: fork-state-write` (its pair, and its "refuses toward the `--fresh` route" clause, which becomes "refuses toward the reset-to-start route");
- `audit: parent-write, tracked, ...` and `audit: fabric-reference`: their "refuses toward the `--fresh` route" clause, the same way;
- `accept-audit --batch: not acceptable` (the pair only; card 15 has already rewritten its `git reset --keep` clause);
- `card not done: delete still referenced by a later card` and `recover-batch: delete still referenced by a later card`;
- `run-exit audit correctness`: its pair becomes `1) lyx webster reset --to start; 2)` the re-entry step, as `pendingPathsWayForward` now ends;
- `recover-batch: findings recovery cannot check` (the pair; card 15 has already replaced its `git reset --keep`) and `accept-audit: finding not checkable`;
- `fresh over a differing suspect path`, `fresh: HEAD not at the start commit` and `fresh: recorded starts diverge` (the pair; card 13 has already rewritten the last row's resolving clause).
Unchanged: `batch order: dependency on a later batch`, `fresh over a contract file last written by a fork` and `fresh over a differing plan path`, whose `run --fresh` follows no reset.
Order with the other cards writing these rows: this card runs after cards 13 and 15 and before cards 18 and 21, so card 18's trim of `rebaseline card set changed` and card 21's rewrites of `rebaseline: begun card of a done batch`, `rebaseline overview changed`, `plan fingerprint mismatch` and `plan changed before the verb's own rewrite` start from this card's text.

**ImpactSummary:** Rewords every reset-then-fresh way forward across webster's verbs and help; fingerprintMismatchWayForward gains a re-entry parameter passed by its three call sites (PlanEditError and batchCardEditError in fingerprint.go, Run), while PlanEditError keeps its signature, so its four callers in begin-batch, record-batch, recovery persist and webstercli's validate see only the reworded text.
