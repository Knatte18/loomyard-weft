```yaml
lens: code-scope
outcome: reported
```

Task: trace each change in the diff to the board entry's goal points and the record's Added-after-Discussion entry (code-scope lens).

**Decisions**
- G1 (board entry, "What to do"): the fixture stops or waits for every `lym board` process it starts before the test returns. The fixture is the one around `smoke_prebootstrap_integration_test.go`.
- A1 (Added after Discussion, parent): no test seam. The defect and the fix are both test-only, in `waitForFixtureProcessesToExit` and `newBadReedUpFixture`, and production code stays unchanged.

**Changes and what they trace to**
- C1, `waitForFixtureProcessesToExit` rewritten to wait, then kill, then fail, with a `patience` parameter, the `fixtureProcessPatience` and `fixtureProcessPoll` constants, the `processesUnder` extraction and the `EvalSymlinks` root (`smoke_helpers_test.go:269-323`). Traces to G1 and A1.
- C2, `newBadReedUpFixture` registers the cleanup itself (`smoke_helpers_test.go:254-259`). Traces to G1 and A1. This brings `TestLoomFailedReedUpRefuses` under the cleanup.
- C3, call-site updates in `smoke_prebootstrap_integration_test.go:36,127,175` and `smoke_starttail_test.go:121`. These are forced by C1's signature change and by C2, which makes the explicit call redundant.
- C4, the new test in `fixtureprocesses_integration_linux_test.go`. This is the test of C1, so C1 forces it. A1 says no test can fail on the base's production code; that is about the red-first gate, so A1 does not forbid a test of the fixture itself.
- No production file changes, which matches A1.

**What builds each decision**
- G1: C1, C2 and C3. It is only partly built; see F1.
- A1: C1 and C2, with no production change.

**Findings**

F1 — MEDIUM, class `scope`, `internal/loomcli/smoke_helpers_test.go:63` (`newWiredPairFixture`)
- Summary: G1 asks that the fixture end every `lym board` process it starts. The diff adds the cleanup only for the tier1 callers of `newWiredPairFixture`; its tier2 callers run verbs that reconcile the hub and spawn the same detached board sync, and nothing ends it.
- Callers left out:
  - `smoke_test.go:364,400,530,571`: `loom start` and `loom run`
  - `smoke_drivingsurface_test.go:187,232,277,358`: `loom resume` and `loom start`
  - `smoke_resumearming_test.go:36`: `loom resume`
- Their only teardown is `registerBootstrapTeardown` (`smoke_test.go:86-104`). It kills the Go driver, the step processes and the watchdog, then runs reed's `Down`. It never kills a board sync.
- `tmuxkit.Main` fails any package that leaves a process running, tier2 runs included (`pattern/PATTERN-test-isolation.md`, "Private temp directory and leftovers"), so the intermittent gate failure the board entry describes stays possible in tier2.
- `_lym/landing/summary.md` ("`newWiredPairFixture` keeps its callers' explicit calls…") says the narrowing is deliberate. No `Added after Discussion` entry and no `Pattern-Departure:` trailer licenses it.
- Fix: register the cleanup for the tier2 callers before `registerBootstrapTeardown`, as `TestSmokeWatchdog_StartSpawnsTheDaemon` already does (`smoke_starttail_test.go:118-122`), so it runs after the teardown's kills. Or get a recorded parent decision that narrows scope to tier1.

No change in the diff traces to nothing.

Tool calls:
1. grep of the `loom start`/`resume`/`run` invocations in the tier2 smoke files that call `newWiredPairFixture`, and of any board-sync cleanup in them. My context held those files' fixture lines but not the verbs they run.
