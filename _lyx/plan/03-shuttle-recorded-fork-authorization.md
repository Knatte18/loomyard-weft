# Card 3 — shuttle-recorded-fork-authorization

**Edit:**
- `internal/shuttleengine#RunState`
- `internal/shuttleengine#Runner.start`
- `internal/shuttleengine#Run.finalize`
- `internal/shuttleengine#GateEntry`
- `internal/shuttleengine#Run.evaluateGate`
- `internal/shuttleengine#Result`
- `internal/shuttleengine#Spec`
- `internal/shuttleengine#seedAttachRunOpts`
- `internal/shuttleengine#seedAttachRun`
- `internal/shuttleengine#TestAttach_GateThreading`
- `internal/shuttleengine#TestRun_Wait_ForkAudit`
- `internal/shuttleengine#TestRun_Wait_ForkAuditFailure_KeepsTheClassifiedOutcome`
- `internal/shuttleengine#TestRunner_Start_HappyPath`
- `internal/shuttleengine/doc.go`

**Intent:**
D3's shuttle half: one run has one answer to whether it was authorized to fork, and it is the answer recorded when the run was spawned, never the Spec of a later Call that attaches it.

`RunState` gains `ForkSubagents bool` with JSON key `forkSubagents,omitempty`, documented as the spawning Spec's `ForkSubagents`, copied at spawn; a record an older binary wrote decodes false and reads as not authorized.
`Runner.start` sets it from `spec.ForkSubagents` in the `RunState` literal it saves.
An attach loads the record and keeps the field through every later save, since every save writes the whole `RunState`.

`Run.finalize`'s fork audit condition becomes `outcome == OutcomeDone && run.state.ForkSubagents` in place of `run.spec.ForkSubagents`.
`Result.ForkAudit`'s doc says the audit runs when the run's record carries fork authorization, not when the Spec that started this Call set it; `Spec.ForkSubagents`' doc gains one sentence that the run records it at spawn and the record, not a later Call's Spec, decides the fork audit and a gate entry's `Forked` closure.

`GateEntry` gains `Forked Gate`: when non-nil, the closure evaluated in place of `Gate` for a run whose record carries fork authorization.
`Run.evaluateGate` picks the closure as `entry.Gate`, then `entry.Forked` when it is non-nil and `run.state.ForkSubagents` is true, then `entry.Final` when the evaluation is final and `Final` is non-nil, so `Final` keeps its precedence.
The gate evaluation thereby receives the judged run's recorded authorization, since the `GateSpec` itself is fixed at wire time.
Bound: `Forked` replaces one entry's closure under one recorded fact; it adds no entry, changes no budget, and an entry without it is evaluated as today, so every existing gate is unchanged.
Webster's Master is always spawned fresh through `StartMaster` and never attached, so its record always carries the authorization its spec set and `runExitAuditCrossCheck`'s nil-audit error keeps its meaning.
The same evaluation serves a spawned and an attached run, so a `plan_producer` edit between spawn and gate changes nothing.
`doc.go`'s gated-run paragraph (the lines on `GateSpec` entries answering passed, failed or pending) gains one sentence naming `GateEntry.Forked` and the recorded authorization that selects it, and the fork-audit sentence wherever `doc.go` states which runs are audited follows the record.

Tests:
- `seedAttachRunOpts` gains `forkSubagents bool`, and `seedAttachRun` writes the `forkSubagents` key only when it is true, so a seed without it is the older record's shape.
- `TestAttach_GateThreading` gains two rows; its table gains `recordedFork`, `specFork` and `wantForked` fields and its gate entry a `Forked` closure that counts its own calls and passes, beside the existing counting `Gate`, and the engine is the existing `&fakeEngine{}`, whose `AuditForksCalls` records every audit:
  - "AttachGated of a fork-authorized record runs Forked and the fork audit, though the attaching spec does not authorize forks": `recordedFork: true`, `specFork: false`; want the `Forked` closure called once, `Gate` never, one `AuditForksCalls` entry and a non-nil `Result.ForkAudit`.
  - "AttachGated of a record without fork authorization runs Gate and no fork audit, though the attaching spec authorizes forks": `recordedFork: false`, `specFork: true`; want `Gate` called once, `Forked` never, no `AuditForksCalls` entry and a nil `Result.ForkAudit`.
  Both new rows set `gated: true`, so they go through `runner.AttachGated`; the existing rows keep their assertions with both flags false.
  Its `//testtiming:keep` reason gains that it pins the recorded authorization, not the attaching spec, choosing the gate closure and the fork audit.
- `TestRun_Wait_ForkAudit` and `TestRun_Wait_ForkAuditFailure_KeepsTheClassifiedOutcome` move their authorization from `Spec.ForkSubagents` onto the `RunState` they pass through `withRunState`, and their doc comments say a fork-authorized run record rather than a fork-mode spec.
- `TestRunner_Start_HappyPath` starts its run with `ForkSubagents: true` on the Spec, and its "persists the running outcome" subtest asserts that the loaded record's `ForkSubagents` is true; its `//testtiming:keep` reason gains the persisted fork authorization beside the persisted Started and running Outcome.

**ImpactSummary:** one additive `omitempty` record field and one additive `GateEntry` field nil everywhere but where card 5 sets it; the fork audit's source moves from the Spec to the record, which changes only an attach whose Call's Spec disagrees with the spawn's.

**Commit:** 3: shuttle: record fork authorization at spawn and judge the run by it
