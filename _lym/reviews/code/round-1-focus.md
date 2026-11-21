---
round: 1
focus:
  - "Leftover coverage-map triage lines: grep the tree outside `_lym` for the removed concept (a triage line under a failed test, the map-based 'reaches none of this run's changes' wording) in docs, specs, stencils, skills and `contracts/`; the remaining `triage` hits include the flaky-failure triage (`AppendIntegrationTriage`, `triageWarnings`), which is a different feature, so settle for each hit which one it is."
  - "Removed code's consumers: `internal/gateslot/triage.go`, `internal/gatecli/triage.go`, the `runlog.go` and `counters.go` changes and the `loomshed`, `runplan`, `weaverengine` and `websterengine` edits; settle that no field, record shape or exported symbol that another binary or a persisted verify-failure record reads was dropped incompatibly, and that nothing is left unused or dangling."
  - "Scope: settle that the reach note, the reach turn and the mutation step are untouched in the diff, and that no unrelated behavior changed in `failurereport.go`, `run.go` and `verifygate.go`."
  - "Tests deleted with the feature: settle that each removed test covered only triage behavior, and that the surviving tests (`TestRenderFailureReport`, `TestVerifyFailureFindings`, `TestRenderVerifyGateFindings`, `TestVerifyGate_FailedEvaluationWritesReport`) still check what their names and doc comments state and that no remaining assertion was loosened."
  - "Doc comments in the touched `doc.go` files and in functions whose bodies lost triage handling: settle that none still describes triage lines."
---

The decision record's Goal is the whole answer key: remove triage lines and every caller and consumer, keep the reach note, reach turn and mutation step, and drop docs text that describes triage lines.
