---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: scope
    location: internal/shuttleengine/enginedispatch_test.go:559
    summary: No test pins the recordless Send and Interrupt answer when the default engine is unregistered, nor that a bad claude.yaml leaves them working
    fix: In internal/shuttleengine/enginedispatch_test.go, add a row to the table-driven test that holds "a_record_naming_an_unregistered_engine_refuses_every_operation_and_never_blocks_a_stop" (or a sibling subtest in the same test function) that builds a runner whose EngineSet default name is not registered, sets rig.reed.StatusQueue to liveStrandStatus(true) with no run record for the guid, and asserts Runner.Send and Runner.Interrupt both return an error satisfying errors.Is(err, ErrEngineNotRegistered) with no SendKeyCalls, SendTextCalls or PasteSubmitCalls. In internal/shuttleengines/shuttleengines_test.go, add one assertion that an engine set built over a Dirs whose claude.yaml holds an invalid key still answers a recordless Runner.Send and Runner.Interrupt without a config error, since D2 says engine config loads only in Prepare, OpenLaunch and PromptCacheLifetime.
---

### [LOW] Recordless send and interrupt: engine resolution and config independence are unpinned

**Location:** internal/shuttleengine/send.go:113 (`recordlessSendContext`), internal/shuttleengine/run.go:1018 (`Runner.Interrupt`)
**Issue:** The parent's recordless `Send` and `Interrupt` resolve the default engine through `engineFor(RunState{})` for a live reed strand no run record names.
This task's D2 moves `claude.yaml` loading into the engine's `Prepare`, `OpenLaunch` and `PromptCacheLifetime`, and D9 makes the strand's `Provider` tag opaque to reed.
I traced the interaction and found it correct: the send, readiness and interrupt paths read no engine config (`loadConfigOrRefuse` is reached only from `claudeengine/command.go` and `claudeengine.go:107`), so a bad `claude.yaml` cannot refuse a recordless send or interrupt.
`PromptCacheLifetime`'s only non-test caller is `loomcli/wiring.go`.
Only one engine is registered, so the default engine and the strand's `Provider` tag cannot disagree today.
The behaviors that are not covered are the two merged-code outcomes, an unregistered default engine on a recordless strand (the `ErrEngineNotRegistered` answer after the liveness check) and an invalid `claude.yaml` beside a recordless send.
The existing recordless tests in `run_test.go` use fake engines whose default is registered, and the `enginedispatch_test.go` unregistered-engine test seeds a run record.
This is a coverage gap in the interaction, not a defect in the merged code.

## Rejected

- A recordless strand is always driven by the default engine, not by the strand's `Provider` tag: with one registered engine the two agree, and the decision record keeps the tag opaque to reed and sets no consumer for it in the send path, so this is the parent's design and not an interaction defect.
- `StrandStatus` has one production construction site (`reedengine/lifecycle.go`) and the parent's `PanePID` addition and this task's provider and scrub changes touch disjoint fields; no merged call site uses a removed symbol (`CleanClaudeEnv` survives only in docs and research notes, not Go code).
- `RelevantPath` is set at both `loomcli/wiring.go` sites the parent changed, in `PlanTable`, and in `envkit.FullEnv`; this task's wiring edits add no third construction site of the env.
- The merged `test-budget.yaml` rows match the merged top-level test counts for the overlapping packages (each differs from a plain `func Test` count only by `TestMain`), and the budget test passed in verify.
- This task's new code follows the parent's changed pattern files: `shuttleengines` and `creelengine` both appear in the told-geometry list, `shuttleengines` and `shellguard` import no `lyxcwd`, and the task adds no stencil-prose test (`PATTERN-test-economy`'s new rule) and no temp or journal write the creel patterns govern.
- `PATTERN.md` and `docs/overview.md` merged without overlapping edits to the same lines; the parent's `creel` entries and the task's entries read correctly side by side.
