---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: scope
    origin: lens:code-scope
    location: internal/loomcli/smoke_test.go#registerBootstrapTeardown
    summary: The tier2 callers of newWiredPairFixture run verbs that reconcile the hub and spawn a detached lym board sync, but no cleanup ends that sync, so the board entry's every lym board process it starts is built for tier1 only, and the summary's claim that their teardown order covers them is not borne out.
    fix: "Register waitForFixtureProcessesToExit(t, fixtureProcessPatience) for every tier2 caller of newWiredPairFixture, after the fixture and before its registerBootstrapTeardown call so it runs after the teardown's kills, as TestSmokeWatchdog_StartSpawnsTheDaemon already does in internal/loomcli/smoke_starttail_test.go. The callers are in smoke_test.go, smoke_drivingsurface_test.go, smoke_resumearming_test.go, smoke_gate_test.go, smoke_attachprobe_test.go and smoke_bootstrapwiring_test.go. A simpler route is to call it inside registerBootstrapTeardown in internal/loomcli/smoke_test.go, before its own t.Cleanup. Then drop the explicit call in TestSmokeWatchdog_StartSpawnsTheDaemon, which would otherwise register it twice. Rewrite the summary's newWiredPairFixture bullet in _lym/landing/summary.md to match."
  - id: F2
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/loomcli/smoke_helpers_test.go#waitForFixtureProcessesToExit
    summary: A pid that first appears under the root after the kill pass, such as a git child of a killed board sync, is never killed and is then reported as still running after it was killed, contrary to the doc comment's claim that the cleanup ends every process.
    fix: "In the cleanup loop of waitForFixtureProcessesToExit, keep a set of the pids already killed. On every scan past the deadline, kill each pid in left that is not yet in the set, add it, and reset the deadline to now plus patience. Fail with t.Errorf only when every pid in left is in the set, naming those pids, so the message names only processes that were killed and still run."
  - id: F3
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/loomcli/fixtureprocesses_integration_linux_test.go:26
    summary: The redundancy report lists the new test as contained by TestAgentActivity_LiveLockWaitReadsAsActivity, which asserts nothing about process cleanup; the new test pins a SIGKILL side effect of test-only code, which coverage cannot see.
    fix: "Add a line `//testtiming:keep pins that the fixture cleanup SIGKILLs a process left under the test's temp root` directly above `func TestWaitForFixtureProcessesToExit_KillsAProcessThatOutlivesItsPatience` in internal/loomcli/fixtureprocesses_integration_linux_test.go, below its doc comment."
---

### [MEDIUM] Tier2 users of the fixture still leave the detached board sync running

**Location:** `internal/loomcli/smoke_test.go#registerBootstrapTeardown`

**Issue:** The board entry asks that the fixture end every `lym board` process it starts before the test returns.
The diff registers the wait-and-kill only for the tier1 callers of `newWiredPairFixture` and for `newBadReedUpFixture`.
The tier2 callers in `smoke_test.go`, `smoke_drivingsurface_test.go` and `smoke_resumearming_test.go` run `loom start`, `loom run` and `loom resume`.
Each of those verbs reconciles the hub and spawns the detached sync.
Their only teardown, `registerBootstrapTeardown` (`smoke_test.go:86-104`), kills the driver, the step processes and the watchdog, then runs reed's `Down`; it never ends a board sync.
`tmuxkit.Main` fails a package on a leftover process in every tier, so the intermittent failure stays possible in tier2.
The summary says these users "have their own teardown order, which this change leaves as it was", but that order ends no board sync.
No added entry narrows the scope to tier1.

### [LOW] A process that appears after the kill pass is never killed and is misreported

**Location:** `internal/loomcli/smoke_helpers_test.go#waitForFixtureProcessesToExit`

**Issue:** The loop kills `left` once, sets `killed`, and from then on only rescans.
`proc.KillPID` kills one pid, so a git child that the detached sync forked just before its kill can still be running in the board dir.
Such a pid was never signalled.
If it lives past the second patience, the cleanup reports it as running "after they were killed" and returns with it still alive.
The doc comment says the cleanup ends every process and fails only on a killed one that survives.

### [LOW] The new test is a contained candidate that pins what coverage cannot see

**Location:** `internal/loomcli/fixtureprocesses_integration_linux_test.go:26`

**Issue:** The redundancy report calls the test contained by `TestAgentActivity_LiveLockWaitReadsAsActivity` (`reviewstatus_test.go:414`), which tests agent activity records.
Coverage does not instrument `_test.go` files, so the containment is vacuous.
The new test asserts a SIGKILL exit status for a process under the fixture's temp root, which no other test checks.
The rubric's rule for such a test is to keep it with a `//testtiming:keep` marker naming what it pins.

## Rejected

- code-tests: the `EvalSymlinks` resolution of the temp root has no failing test. A test would need a symlinked `TMPDIR`, a process-global `t.Setenv` that costs the test its parallelism. The resolution only matches the root to what `/proc` reports, the way `hubforge`'s `copyTemplate` already resolves its copy root. That is too little behaviour to justify a serial test of test-support code.
- code-tests: nothing pins that `newBadReedUpFixture` registers the cleanup. The record's "No test seam" entry accepts that the leak has no deterministic test, and a fake sync to pin one registration line would be a per-helper test of fixture code.

Holistic checks that raised nothing:
- The review range is `8641cd50..HEAD`, two non-merge commits.
- The `Root-Cause:` trailer names the defect the diff removes: the silent give-up after the deadline and the missing registration in `newBadReedUpFixture`.
- No assertion of a base test is loosened.
- The directive declares no PATTERN changes.
- The new file's `_linux` suffix, its `tiergate.Require(t, 1)` and its lack of a tier build constraint follow `PATTERN-test-speed`.
- `proc.KillPID` in fixture teardown is test-fixture machinery, outside `PATTERN-child-processes`, and is already used by `registerBootstrapTeardown`.
- Cleanup order holds at every caller: the wait is registered after the temp dir's removal, and in `TestSmokeWatchdog_StartSpawnsTheDaemon` before the bootstrap teardown, so it runs after the teardown's kills.
- The scan sees the sync: `spawnSync` sets the sync's cwd to the board dir of the hub copy, and `copyTemplate` places that copy under the resolved `t.TempDir()` of the test or subtest that called the fixture.
- The "two-way" merge-danger call holds: the diff changes test files only.
