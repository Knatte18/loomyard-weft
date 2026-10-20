---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: scope
    location: internal/shuttleengine/attach_test.go:260
    summary: Nothing pins that AttachIfLiveGated removes no superseded strand, the contract its doc comment and the change description state
    fix: In TestAttach_RemovesSupersededStrands, extend the attach_if_live_removes_nothing row so it also runs runner.AttachIfLiveGated(spec, GateSpec{}), for example with a second flag beside ifLiveOnly that selects the gated entry point at the attach selection near line 293; add no new test function.
    origin: lens:code-tests
  - id: F2
    severity: LOW
    class: scope
    location: internal/sheddriver/agentshuttle.go:48
    summary: No test pins that RunnerShuttle forwards its gate to StartGated and AttachIfLiveGated; both fakes bypass the adapter, so a swap back to Start or AttachIfLive would drop the arbiter gate in production unnoticed
    fix: Add a row or small test over a runner fixture asserting that RunnerShuttle.Start and RunnerShuttle.AttachIfLive deliver the given GateSpec to the run, if the existing shuttleengine fixtures can be reached cheaply from sheddriver tests; otherwise leave the gap and accept it.
    origin: lens:code-tests
---

### [MEDIUM] AttachIfLiveGated's removes-nothing contract is untested

**Location:** internal/shuttleengine/attach_test.go:260, 293-295; internal/shuttleengine/attach.go:79-84
**Issue:** `AttachIfLiveGated` calls `attachAndWait(spec, gate, false)` and its doc comment says it removes nothing on any path.
The focus directive asks whether it keeps `AttachIfLive`'s rule.
The code does: `removeSuperseded` is false.
The only removal test, `TestAttach_RemovesSupersededStrands`, reaches `AttachIfLive` alone through its `ifLiveOnly` row.
The new `AttachIfLiveGated` row in `TestAttach_GateThreading` checks gate threading only.
A regression passing `true` would remove a live strand beside a live arbiter, and no test would fail.

### [LOW] RunnerShuttle's gate forwarding is untested

**Location:** internal/sheddriver/agentshuttle.go:48-58
**Issue:** `RunnerShuttle` is the only production path that carries the arbiter's gate to shuttle.
No test references it, and the fakes implement `AgentShuttle` directly.
The adapter is thin and reads correctly, so the risk is a later edit, not a present defect.

## Rejected

- Gate rewrite hint always mentions block scalars, even for a semantic failure such as an empty reason (code-correctness): the reader's error comes first in the same text, so the cause is visible; not a defect.
- `TestArbiterVerb_VerdictGate` repeats the three file rows on both the Start and Attach paths (code-tests): the closure is shared, but the per-path wiring is what each subtest pins, and no existing test covers it; not a redundancy.
- `TestArbiterPrompt` parses the stencil's yaml example (code-pattern-fit, borderline): it asserts that a value Go reads parses, not the stencil's prose; allowed under test economy.

## Holistic check against the fasit

- Gate wiring: both `settleFresh` (`arbiter.go:260`) and `attachAndConclude` (`arbiter.go:198`) pass `arbiterVerdictGate` with the verdict path the spec writes.
- Gate semantics: an absent file passes, an unreadable one fails with the reader's error and the path, and the entry is `PassOnCap` with 2 attempts.
  After the cap the verb's own read hands on with cause `unparseable`, as before.
- Signature change: every implementer (`RunnerShuttle`, `agentFake`, `arbiterFake`) and caller is updated, and the tackler passes `nil`.
  `TestTacklerCall_SpawnPath` asserts its gates are empty.
- Stencil: the diff only adds the block-scalar paragraph, the example and the re-prompt sentence; no existing rule is dropped or weakened.
- Docs: `internal/loomcli/doc.go` and `internal/shuttleengine/doc.go` match the code.
- Change description: its claims hold, and the attach-path coverage it points at exists for the arbiter's attach wiring and for gate threading; F1 covers the removes-nothing gap.
- Scope: the diff makes the changes the board entry asks for and nothing beyond them.
