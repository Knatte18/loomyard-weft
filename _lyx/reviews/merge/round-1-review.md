---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: internal/shuttleengine/send.go:113
    summary: "recordless Send and Interrupt always use the default engine, though lyx shuttle open now opens a session on the alias's engine and stamps it as the strand's provider tag"
    fix: "In `recordlessSendContext` (send.go) and the recordless branch of `Interrupt` (run.go, `state = RunState{}`), resolve the engine from the live strand's provider tag instead of an empty `RunState`. That needs reed's `StrandStatus` (internal/reedengine/lifecycle.go) to expose `Provider`, and `requireLiveStrand` to return the matching strand. Pass `RunState{Engine: strand.Provider}` to `engineFor`, with an empty tag still meaning the default. Add a row to the `Send_and_Interrupt_on_a_live_strand_with_no_record_consult_the_default` test in enginedispatch_test.go for a strand tagged with the non-default engine. If the plan defers this while one engine is registered, state that in the `recordlessSendContext` doc and in shuttleengine's doc.go."
---

### [MEDIUM] Recordless Send and Interrupt ignore the engine the opened session runs on

**Location:** `internal/shuttleengine/send.go:113` (`recordlessSendContext`), `internal/shuttleengine/run.go:1023` (`Interrupt`), against `internal/shuttleengine/open.go:53` and `:88`.

**Issue:** The parent added a path for a live strand that no run record names: `Send` and `Interrupt` consult `engineFor(RunState{})`, so they always use the default engine. The parent also stamps `Provider: engineName` on every strand it spawns, but nothing on this path reads it. The task (D1, acceptance criterion "`lyx shuttle open` with a model alias opens the session on the alias's engine") makes `Open` honor `spec.Engine`. `Open` writes no run record, so a session opened on a non-default engine is exactly a recordless strand. A later `lyx shuttle send` or `interrupt` to it would drive its pane through the default engine's idle read, paste and interrupt sequence.

Only one engine is registered today, so nothing misbehaves yet. This is the hand-off the two sides each assume the other makes: the parent's recordless doc says "the send goes through the runner's default engine", and the task's `Open` records the engine only in the provider tag and `OpenResult.Engine`. The gap surfaces when `codex-engine` lands.

The other focus items settled as follows:
- **Same engine name:** `Provider: engineName` and `Result.Engine` both derive from `engineFor` at `Start`, so they agree for the same run.
- **Removed `claude_deny_*` keys:** no task code or test reads them. The remaining mentions are the parent's retired-key refusals, the `deny_python` migration and `configreg/keymove.go`.
- **`PATTERN-model-choice-mapping`:** the scan passes on the merged tree. The parent's `shuttleengines`, `creelengine` and `creelfake` do not import `modelspec` in production. The only parent-side `modelspec` use is `envkit`'s test registry literal.
- **`PATTERN-shuttle-provider-seam` and `PATTERN-leaf-packages`:** `ModelChoice` holds no provider name and no forbidden import.
- **Loom and webster wiring:** the `loomcli` and `webstercli` wiring edits compose with the `ModelChoice` resolvers.
- **Test budget:** `cmd/lyx` `TestTestBudget_PackagesStayWithinBudget` passes on the merged tree.
- **`envkit`:** the parent's `nilLegal` realignment composes with the task's `Deps` fields.
