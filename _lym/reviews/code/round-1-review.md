---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    location: pattern/PATTERN-test-speed.md:225
    summary: The PATTERN-test-speed background still says the report steps run after the tiers released their slots and after the lints, which the diff no longer does.
    fix: "In pattern/PATTERN-test-speed.md, rewrite line 225 to say the redundancy report step judges beside the first run of the key's highest tier, taking its own gate slot beside the slots that tier holds, and writes its report after the lints. Rewrite line 329 to say the reach step judges beside the last tier, before the redundancy step, and its report is written after the lints. Rewrite line 344 to say the redundancy step judges beside the last tier and its report is written after the lints. Keep the slot rules and the reach, redundancy, mutation order."
    origin: lens:code-affected-set
  - id: F2
    severity: MEDIUM
    class: design
    location: internal/gatecli/processcap.go#runMarker
    summary: The redundancy step's processes carry the call's LYM_GATE_TEST_PID marker, so while they run beside the last tier they count toward that tier's process cap and can fail it, letting an advisory step change the call's result.
    fix: "Make the tier's process cap count only processes below a running tier command: in the process cap's run, after listing the marked pids, drop each pid with no ancestor that request.commands holds, using a process-tree sample as withoutForeign does. Thread request.commands into newProcessCap for that. Add a row to the process cap's existing test showing a marked process outside every running tier command is not counted."
    origin: lens:code-correctness
  - id: F3
    severity: MEDIUM
    class: design
    location: internal/gatecli/tierrun_test.go#TestRunTierMemoryGuard
    summary: No test pins the production wiring that spares the tier's test binaries; dropping request.commands.add, request.commands.remove or guard.foreign in reports.go fails no test, since TestMemoryGuard fills the foreign set by hand.
    fix: "In TestRunTierMemoryGuard, set the request's commands to a fresh runningCommands and, from the memory guard stub's run, record whether commands.holds(pid) is true for the tier command's pid while the command runs; after runInvocations returns, assert it was true then and that commands.holds(pid) is false now. That pins registration and removal in runInvocations."
    origin: lens:code-tests
  - id: F4
    severity: MEDIUM
    class: design
    location: internal/gatecli/reports_test.go#TestReportChain
    summary: Where the chain starts in a multi-run plan has no test; lastTierStart is reached by no mapped test and the only runTest scenario has a single tier, so a start index of 0 or the lowest tier passes.
    fix: "Add a tier0 table-driven test in internal/gatecli/reports_test.go over lastTierStart with literal plans and expected indexes: an empty plan gives -1, one run gives 0, tiers [0, 1, 2, 2] give 2, tiers [2, 0] give 0. The start index is not observable at runTest's surface without timing."
    origin: lens:code-tests
  - id: F5
    severity: MEDIUM
    class: consistency
    location: internal/gateslot/doc.go:45
    summary: internal/gateslot/doc.go still says the redundancy report step takes its slot after the call's tiers released theirs, and at line 79 that a key's reports run after its tiers, contradicting the diff's own gatecli doc and the Reports field doc.
    fix: "In internal/gateslot/doc.go, change line 45 to say the redundancy report step takes one slot beside the slots the call's last tier holds. Change line 79 to say a key's lints run after its tiers and its reports judge beside its last tier and are written after its lints."
    origin: lens:code-affected-set
  - id: F6
    severity: LOW
    class: design
    location: internal/gatecli/reports_test.go:760
    summary: The AStoppedChainWritesNothing row stops the chain only after wait returned, so it exercises no cancellation and asserts nothing the other row does not; the test's doc claims a stopped chain writes nothing.
    fix: "Delete the AStoppedChainWritesNothing row and the write field from TestReportChain, keep the AWaitedChainWritesItsReportsInStepOrder body as the test, and drop the doc sentence \"A stopped chain writes nothing.\""
    origin: lens:code-tests
  - id: F7
    severity: LOW
    class: design
    location: internal/gatecli/test_integration_test.go:1537
    summary: No test pins that a halted call whose key lists reports writes no report and returns only after the chain ended.
    fix: "In the existing TestGateTest_Scenario subtest for a gate key's redundancy report step, add a row whose tier halts the call. Reuse the halt fixture the scenario's existing halt subtests use. Assert that the call exits with the halt's code and that ReadRedundancyReport, or its package-internal reader, finds no report in the worktree's gate directory."
    origin: lens:code-tests
---

### [BLOCKING] PATTERN-test-speed background left describing the sequential tail

**Location:** pattern/PATTERN-test-speed.md:225

**Issue:** The background of a listed constraint states that the redundancy step "runs after the key's tiers released their slots" (line 225).
It also says reach runs "after its lints" (line 329) and redundancy "after its tiers and lints" (line 344).
The diff starts all three steps beside the first run of the plan's highest tier (`internal/gatecli/run.go`, the `reportsStart` branch).
The steps take their slots while that tier holds its own.
The constraint's own text now contradicts the code, and the diff does not land the doc change its behavior change needs.

### [MEDIUM] Report steps now share the last tier's process cap

**Location:** internal/gatecli/processcap.go#runMarker

**Issue:** `redundancyStep.env` is `e.env` (`redundancy.go:92`), which carries the call's `LYM_GATE_TEST_PID=<pid>` entry.
`runMarker` reads that same entry from the tier's env (`processcap.go:94`), and the cap counts every live process carrying it, up to `runProcessCap` (1024).
Before the diff the steps ran after every tier ended, so they never shared the cap.
Now a step's builds and coverage tests running beside the last tier add to the tier's count.
When the cap trips, the tier fails, which breaks the contract that a report step never changes the call's result (`internal/gatecli/doc.go`, "Each step is advisory").
The margin is wide, so the risk is low, but the failure would be misattributed to the tier.

### [MEDIUM] Production wiring of the guard's foreign set is unpinned

**Location:** internal/gatecli/tierrun_test.go#TestRunTierMemoryGuard

**Issue:** The new `TestMemoryGuard` row builds `runningCommands` by hand (`memoryguard_test.go:183-187`).
No test runs `runInvocations` with a `commands` set.
Removing `request.commands.add` or `remove` (`tierrun.go:534`, `:558`) leaves every test green, as does removing `guard.foreign = j.tierCommands.holds` (`reports.go`, `judge`).
Without that wiring, the report steps' guard can kill the last tier's test binaries, the hazard the change adds the filter for.

### [MEDIUM] The chain's start index has no test

**Location:** internal/gatecli/reports_test.go#TestReportChain

**Issue:** The reach report lists `lastTierStart` as reached by no mapped test.
The only `runTest` scenario with `reports` uses a single tier (`test_integration_test.go:1539`), where every candidate index is 0.
A `lastTierStart` that returned the first or lowest tier would pass every test and start the steps beside tier0, against the Goal's "concurrently with the tier 2 tests".

### [MEDIUM] gateslot package doc left describing the sequential tail

**Location:** internal/gateslot/doc.go:45

**Issue:** Line 45 says the redundancy step takes one slot "after the call's tiers released theirs".
Line 79 says a key's `reports` are steps "its call runs after its tiers".
The diff updated `GateKey.Reports` in `tiers.go:118` and the gatecli doc to say the steps judge beside the last tier.
These two lines now contradict them.

### [LOW] The stopped-chain row tests nothing of its own

**Location:** internal/gatecli/reports_test.go:760

**Issue:** The row calls `chain.wait()` before `chain.stop()`, so the steps have ended and no cancellation runs.
Its stat-before-write check repeats the other row, and its empty-stderr check holds whatever `stop` does.
The test's doc comment claims "A stopped chain writes nothing", a behavior the row does not exercise.

### [LOW] A halted call with reports has no test

**Location:** internal/gatecli/test_integration_test.go:1537

**Issue:** The diff replaces the old early return with a deferred `chain.stop()` on the halt path (`run.go`, the `end.halt` branch).
No scenario combines a halt with a key that lists `reports`, so neither half of that path is pinned:
that no report is written, and that the call returns only once the chain has ended.

## Rejected

- The code-tests lens's finding that the held step and guard lines in `judgedReports` lack a test is rejected. Those lines are stderr a reader reads, and their order is layout no other code relies on, so per the testing rules it gets no test.
- The assistant's F2, which asks for a synchronised scenario proving concurrent scheduling, is merged into F4 at a lower cost. A literal-table test of the start index pins the decision, and a timing-coupled scenario would breach PATTERN-test-speed's time rules.
- The assistant's F3, which asks for an active-cancellation lifecycle test, is merged into F6 and F7. The red-gate path after a failure that follows the start is already pinned by the `ARedTier` row of the redundancy scenario.
- The assistant rated its F2, F3 and F4 BLOCKING. They are coverage gaps under the rubric's test-economy bullet, not violations of a listed constraint, so they are kept at MEDIUM or LOW.
- The code-affected-set lens's `docs/overview.md:959` is still true, since the reports are written last.
- The code-scope lens found no scope finding, and none is raised. The added entry drops the before-and-after timing, red-first is untouched, and every change traces to the Goal's concurrency option or is forced by it.

## Focus departures

- "Test economy: ... Settle whether each is covered through TestReportChain or another test": `reportSteps.any` and `reportStepsOf` are reached on the redundancy scenario's path through `runTest`, which runs out of process, so the map cannot see them. A separate test for them is not asked for. Only `lastTierStart` is raised, as F4.
