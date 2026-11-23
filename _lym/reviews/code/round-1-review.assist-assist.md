---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    location: internal/loomcli/amendduringwebster_test.go:61
    summary: The new state reader duplicates Webster's owned filename instead of using its declared contract.
    fix: Replace the hand-built state.json path and raw reads with websterengine.LoadState before and after the operations, checking errors, presence and equality of the complete state values. Update the test comment to describe state equality. If byte equality is retained, expose the path through websterengine using its existing stateFileName constant and use that accessor in both production persistence and this test.
    origin: lens:code-affected-set
  - id: F2
    severity: MEDIUM
    class: design
    location: internal/loomcli/amendduringwebster_test.go:70
    summary: The preservation assertion checks a record disconnected from the amendment reader and bypasses loom's goto hooks.
    fix: Set f.implementerSeam.AnchorRoot to anchorPath before f.deps(), and drive goto through the production loom spec and shedverbs.RunGoto with the same anchored fixture, including its production hooks. Keep fakes at slow boundaries, assert the resulting paused Plan-Burler-Review status, then run the plan gate and check the seeded Webster state after these operations. Update the end-to-end claim to describe the exact path exercised, without claiming the producer bodies ran.
    origin: lens:code-tests
---

### [BLOCKING] State filename ownership

**Location:** internal/loomcli/amendduringwebster_test.go:61

**Issue:** The new reader spells `state.json` independently of `websterengine.stateFileName`.
`internal/websterengine/state.go:66` owns the constant;
its persistence functions use that constant at lines 192 and 212.
`PATTERN-code-conventions` requires a perishable name or path to have one named constant referenced everywhere.
The round's constraints require BLOCKING severity for this rule violation.
This is a source-level ownership finding, not a failed mechanical check.

### [MEDIUM] The fixture misses the production state path

**Location:** internal/loomcli/amendduringwebster_test.go:70

**Issue:** The planted state and the amendment's implementer reader use different anchors.
`newDecisionFake` assigns its implementer seam a fresh temporary directory;
the new test redirects only the amendment path and approved-plan reader.
`notifyLiveRuns` reaches `ActiveOrSetting` at `internal/loomcli/decision.go:519`, using that empty directory.
The preservation assertion consequently checks a record that this amendment-side reader never sees.

The goto similarly supplies neither `Guard` nor `Admitted` and writes an independent status fixture.
Production wiring supplies those hooks at `internal/loomcli/arm.go:478`,
and `internal/shedverbs/goto.go:23` runs the pre-goto hook before forwarding them.
Generic `Goto` executes no producer bodies, regardless of their names in the routing list.
The test therefore misses regressions in the loom transition while its comment and commit claim the reported path is driven.
This combines the correctness lens's anchor finding with the tests lens's transition finding.

## Other review conclusions

The landed glyph fixture is coherent: `sub#FooN` identifies the corresponding function in `sub/a.go`.
The gate reads the active implementer's state and passes its done cards to the index.
The existing `TestNewPlanGate` rows cover both done-card exemption and collision without a run record.
There is no supported finding that these Creates inherently fail to collide.

The range adds only the test file and traces to both added decisions.
The record explicitly permits a guard that passes on base, so no missing production fix or Root-Cause trailer is raised.
The comment-formatting commits do not claim to fix the incident.
The distinct preservation assertion prevents treating the whole test as redundant with the existing gate tests.
The new test adds no wait, sleep or shared mutable fixture.

## Validation

The focused foreground gate command could not acquire a slot because its holder record was outside writable roots.
It failed creating a temporary record under `/home/knatte/Code/loomyard-LYMHUB/_bolt/.lym/gate`.
No test result from that attempted run is claimed.
The supplied current-tree redundancy report labels the new test unique;
the supplied reach and mutation reports identify no changed production function or block.
All four lens reports are saved beside this report.
