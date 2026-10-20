---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: scope
    location: internal/shuttleengine/attach.go (AttachIfLiveGated), internal/sheddriver/agentshuttle.go:57
    summary: The real attach path carries the gate with no test; only the fake shuttle sees it
    fix: Add one row to the existing shuttle attach tests in internal/shuttleengine that attaches to a live run through AttachIfLiveGated with a failing gate entry and asserts the run is re-prompted, so a regression that drops the gate in attachAndWait fails.
    origin: lens:code-tests
  - id: F2
    severity: LOW
    class: scope
    location: internal/sheddriver/tackler_test.go:39
    summary: Nothing pins that the tackler's start and attach stay ungated, which the change description claims
    fix: Have agentFake in tackler_test.go record the gate it is handed and add an assertion in an existing tackler start test and attach test that the recorded gate is empty.
    origin: lens:code-tests
  - id: F3
    severity: NIT
    class: consistency
    location: internal/shuttleengine/attach.go (AttachIfLiveGated doc comment)
    summary: The doc comment defines AttachIfLiveGated in terms of another named symbol
    fix: Reword the doc comment to stand alone, for example "AttachIfLiveGated waits on a live run matching spec's output files, gated by gate, and reports false when none is live; it removes nothing on any path."
    origin: lens:code-pattern-fit
---

### [MEDIUM] Real attach path carries the gate with no test

**Location:** `internal/shuttleengine/attach.go` (`AttachIfLiveGated`), `internal/sheddriver/agentshuttle.go:57`
**Issue:** `TestArbiterVerb_VerdictGate/Attach` captures the gate through `arbiterFake`, which hides both `RunnerShuttle.AttachIfLive` and `Runner.AttachIfLiveGated`.
No test file references either, so a change that passes `GateSpec{}` on the attach path would pass every test.

### [LOW] Tackler staying ungated is unpinned

**Location:** `internal/sheddriver/tackler_test.go:39`
**Issue:** `agentFake` discards the gate, so nothing asserts the summary's claim that the tackler passes an empty spec.

### [NIT] Comment defined against another symbol

**Location:** `internal/shuttleengine/attach.go` (`AttachIfLiveGated`)
**Issue:** The first line reads "AttachIfLiveGated is AttachIfLive, gated", which `code-quality`'s self-contained rule disallows.
The sibling comments phrase it the same way, so this is low value.

## Checked and held

- `arbiterVerdictGate` runs `ReadArbiterVerdict` on the attempt's own path at both call sites, passes an absent file, and is pass-on-cap with a budget of 2, so a still-unreadable verdict reaches `conclude` as `unparseable`.
- Every implementer and caller of `AgentShuttle` passes the spec; the tackler passes `nil`; `shedadapters/bouncer.go` calls `Runner.AttachIfLive` directly and is unaffected.
- The stencil example parses through the reader, and the diff only adds lines to the stencil.
- `internal/loomcli/doc.go` and `internal/shuttleengine/doc.go` match the code; no `PATTERN.md` or `docs/overview.md` change is owed.
- `TestArbiterVerb_VerdictGate` does not duplicate `TestArbiterVerb_VerdictOutcomes`, which pins only the post-run hand-on.
- The diff stays within the board entry's scope, and the tackler's verdict file is left out as the entry's out-of-scope note says.
