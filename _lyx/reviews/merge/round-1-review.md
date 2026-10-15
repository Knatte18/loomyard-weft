---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: scope
    location: internal/shuttleengine/enginedispatch_test.go:507
    summary: no test pins that a strand with no run record is driven through the set's default engine on Send, Interrupt and SessionIdle
    fix: In internal/shuttleengine/enginedispatch_test.go add rows to the dispatch table in TestRunner_EngineSetDispatch, in the same subtest as a_record_with_no_engine_key_is_driven_through_the_default, that build the runner with newFixture and withEngines(rig.engines) but no run record (no withStrand record, only a live strand in rig.reed), call Runner.Send, Runner.Interrupt and Runner.SessionIdle on it, and assert through requireConsulted that engine a (the set's default) was consulted and engine b recorded nothing.
---

### [LOW] Recordless paths are untested against a multi-engine set

**Location:** `internal/shuttleengine/enginedispatch_test.go:507`, `internal/shuttleengine/send.go:116`, `internal/shuttleengine/run.go:1010`

**Issue:** The task's recordless `Send`, `Interrupt` and `SessionIdle` paths merged with the parent's per-run engine resolution. The merged code is correct: `recordlessSendContext` and the recordless branch of `Interrupt` resolve through `engineFor(RunState{})`, so they pick the set's default.
Every recordless test in `run_test.go` runs a single-engine fixture, so a regression to the first or any non-default engine would still pass.
The one default-engine dispatch test, `a_record_with_no_engine_key_is_driven_through_the_default`, builds a fixture with `withStrand`, which writes a run record with no engine key, and exercises only `ReadEvents`, which is not a recordless path.
This is the interaction `PATTERN-test-economy` asks to be pinned (D17: "driven with the runner's default engine").

## Checked and not a defect

- `Runner.Send` and `Runner.Interrupt` both choose the default engine for a recordless strand and skip the pending-send record (`pasteVerified` gates it on `!sc.recordless`), the run-state read (`sessionReadiness` returns the unknown reading for `recordless`) and the retry-landed lookup (`sendVerified`). `SessionIdle` treats a find error as `RunState{}`, so it agrees on the engine.
- `InterruptSettle`, `PaneRunningTurn`, `Inject`, `SessionState` and the other `FindRun` callers still refuse a strand with no record. Their only non-test caller outside shuttle is `tenderengine`, which acts on shuttle runs; creel calls `SessionIdle`, `Send` and `Interrupt` only (`internal/creelcli/delivery.go`). D17 scopes the change to those three.
- `internal/creelcli` imports `internal/shuttleengines` and `shuttleengine` only, and takes the engine set from an injected `enginesFor`, so `PATTERN-shuttle-provider-seam` holds. The set is built per `wire` call rather than cached; that is the task's own cost and not an interaction defect.
- `shuttlefake` lost `ModelSwitchSequenceFn` and gained `EngineSet`; no creel test or `creelfake` code references the removed field.
- The merged `PATTERN.md`, `pattern/PATTERN-pane-text-injection.md` and `pattern/PATTERN-told-geometry.md` keep both sides' text: the record-less send bullet survives beside the parent's removal of `ModelSwitchSequence`, and the bound packages list holds both `shuttleengines` and `creelengine`.
- `test-budget.yaml` holds rows for `internal/creelcli` (4), `internal/creelengine` (11) and `internal/shuttleengines` (3), matching the top-level test functions in the merged tree. `docs/overview.md` lists creel and `shuttleengines` in both the tree and the module descriptions.
- `internal/reedengine` and `internal/shuttlecli` edits do not touch the same calls: reed's change is a doc line, and shuttlecli's `engines` field feeds `NewRunner` consistently with creel's use of it.
