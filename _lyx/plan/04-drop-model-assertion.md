# Card 4 — drop-model-assertion

**Edit:**
- `internal/websterengine/beginbatch.go`
- `internal/websterengine/state.go`
- `internal/websterengine/runlevel.go`
- `internal/websterengine/doc.go`
- `internal/webstercli/beginbatch.go`
- `internal/webstercli/cli.go`
- `internal/webstercli/wiring.go`
- `contracts/specs/refusal-spec.md`
- `contracts/specs/webster-spec.md`
- `contracts/stencils/webster/webster-template-master.md`
- `tools/sandbox/SANDBOX-WEBSTER-SUITE.md`
- `tools/sandbox/suite.go`
- `internal/websterengine/beginbatch_test.go`
- `internal/websterengine/runlevel_test.go`
- `internal/websterengine/state_test.go`
- `internal/webstercli/verbs_test.go`
- `internal/webstercli/acceptaudit_test.go`
- `internal/webstercli/restoreplan_test.go`
- `internal/shuttleengine/claudeengine/modelswitch_test.go`

**Intent:**
Implement decision D10 (`webster-batch-validation-speed`): `begin-batch` no longer asserts or injects a model.

Remove from `internal/websterengine`: the model-switch injection block in `BeginBatch` and the RoleMaster lookup that feeds it; the `Injector` interface; `BeginDeps.Injector`, and `BeginDeps.Roles` and `BeginDeps.Engine` when nothing else in `BeginBatch` reads them; `BeginResult.AssertedModel`; `State.AssertedModel`; and `Run`'s launch-time write of it (keeping the save that persists `MasterSessionID`).
Rewrite the comments that describe the assertion: `beginbatch.go`'s file comment and `BeginBatch`'s doc comment, and `doc.go`'s "idempotent per-batch model assertion" section, which goes, along with the "ONLY model-injection site" wording; Master's model is set once at spawn and nothing in begin-batch reads it.
In `internal/webstercli`, drop the `injector` field and seam from `cli.go` and `wiring.go`'s `setRunner`, stop passing it in `beginbatch.go`, drop the envelope's `model` key, and rewrite the `begin-batch` `Long` text without the model clause.
`shuttleengine.Engine.ModelSwitchSequence` stays; it is not webster's.

A state.json written before this change, still carrying `assertedModel`, must load unchanged: `LoadState` decodes with `encoding/json`'s default unknown-field tolerance, so dropping the field is enough; the test below pins it.

Docs: delete the refusal-spec row "model switch injection failed"; reword `webster-spec.md`'s "the model assertion" mention and step 66 of `webster-template-master.md` ("this asserts your own model for the batch") so neither claims an assertion, and drop the "model changes are injected by Go's own `begin-batch` call" tail of that stencil's "What you never do" line, keeping the clause "NEVER use a `/model` switch" that `contracts/stencils/claims_test.go` pins; in `tools/sandbox/SANDBOX-WEBSTER-SUITE.md` remove assertions W2a and W2b, the `assertedModel` tamper exception in "Controlled exceptions", and every reference to them (the intro, the report `ref` examples, the session-log lines, the closing notes), since the mechanism they armed is gone.
W2c is not about the model assertion: it checks that a fork's transcript file is already flushed when its report lands, which `record-batch`'s audit relies on and D10 does not touch.
It stays, moved into W1 as a further watched assertion with its own verdict line and `ref`, and the suite's "model-assertion" wording becomes "fork-loop" wording, including `tools/sandbox/suite.go`'s comment on `websterSuite`, which names "model-escalation" scenarios the suite no longer carries.
Grep the repository for `AssertedModel`, `assertedModel`, `Injector`, "model switch" and "injected" outside `docs/benchmarks/` (a measurement report, left as recorded) and clear every live reference.

Bound: the removal lets nothing through in the shipped flow, since the comparison was between a recorded model and the model `Run` launched Master on and never read the live pane; a Master pane an operator moved with `/model` stays there for the rest of the run, as it did before.

Tests:
delete `TestBeginBatch_ModelAssertion`, `TestBeginBatch_WayForward_ModelSwitchFailureIsTransient`, `TestBeginBatch_UnknownRoleErrors` (it pins the RoleMaster lookup this card removes) and `TestRun_AssertedModelInitializedToMasterRoleModel`, and the `AssertedModel` fields in the remaining fixtures.
In `state_test.go`, a state.json carrying `"assertedModel": "opus"` loads with every other field intact and saves back without the key.
In `internal/webstercli/verbs_test.go`, `TestBeginBatchCmd_HappyPath` asserts the begin-batch envelope has no `model` key and that the fake runner recorded no inject call; `initState` loses its model parameter, and its other callers, `acceptaudit_test.go` and `restoreplan_test.go`, drop the argument.
In `internal/shuttleengine/claudeengine/modelswitch_test.go`, reword the comment that cites "the W2b corruption mode" so it names no sandbox scenario.

**ImpactSummary:** Removes begin-batch's dormant model injection, its state field and CLI seam across websterengine and webstercli; one refusal row and two sandbox assertions go with it, and the live fork-transcript flush check is kept.
