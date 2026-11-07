---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/gatecli/scope.go:192
    summary: "The --card scope watch skip (cardChanges, skipNames, sc.skip in the scopeCard branch of computeScope) has no test that fails if it breaks."
    fix: "Add a subtest to TestGateTest_Scopes in internal/gatecli/scopes_integration_test.go that runs a --card call over the `watched` fixture package with a parent branch, asserts the tier args carry `-skip ^(TestWatched)$`, then edits `watched/x_linux.go` and asserts the skip is gone. Add a row for a worktree with no parent branch asserting no skip."
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/gatecli/scope.go:524
    summary: "scanGuards dropping a watched guard test under sc.changesRead, through moduleRelative's prefix trimming, has no test; only the computed path through targetsFor is covered."
    fix: "Add a test for scanGuards or a card-scope scenario with a guard test that also carries //lym:watch, in a module whose dir is not the worktree root. Assert the guard is dropped when no changed path matches and kept when one does. Cover moduleRelative for the root and a nested module dir."
  - id: F3
    severity: MEDIUM
    class: consistency
    origin: lens:code-affected-set
    location: internal/gitexec/gitexec.go:57
    summary: "armDeadline is a new package-level var replaced by tests, which PATTERN-test-speed 'Time is injectable' forbids for a clock a test would wait out; the test must stay serial."
    fix: "Either carry the deadline context maker as a parameter or field of the gitexec attempt path so tests keep t.Parallel, or add to pattern/PATTERN-test-speed.md 'Time is injectable' the exception that gitexec's remote-attempt seams stay package-level and their tests run serially. The neighbouring remoteDeadline and startStallTimer vars are precedent, which is why this is MEDIUM and not BLOCKING."
  - id: F4
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/gitexec/gitexec_unix_test.go:141
    summary: "No test runs the production armDeadline default any more, so nothing pins that the real wall-clock deadline kills a hung remote."
    fix: "Export the default through export_test.go (for example `DefaultArmDeadlineForTest`) and add a case to TestRun_RemoteBounds or a small test that arms a 20 ms deadline and asserts the context is done with context.DeadlineExceeded. This waits on a context, not a process or binary start, so no bound shortens."
  - id: F5
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/verifyrun/run_integration_test.go:166
    summary: "TestRun_Cancellation cancels on the command's first output, which can arrive before `sleep 30` has started, so it no longer pins that a running child is killed."
    fix: "Change the long command to start the sleep before printing, for example `sleep 30 & echo up; wait` (and the ping equivalent on Windows), so the first write means the child is running."
  - id: F6
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: pattern/PATTERN-test-speed.md:117
    summary: "The guard section says the guard step runs every marked test whatever the impacted set, which is now wrong for a test carrying both //lym:guard and //lym:watch."
    fix: "In the 'Repository-scanning tests carry //lym:guard' section, or the watch section, add one sentence: a guard test that also carries //lym:watch is left out of a narrowed gate's guard step when no changed file concerns it."
  - id: F7
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/gatecli/cli.go:240
    summary: "The GuardMisplaced refusal now appends the scan error text, which for a watch marker already carries its own way-forward clause, so the refusal states the way forward twice."
    fix: "In guardScanRefusal, keep the single way-forward clause: either drop the `%v` of err from the GuardMisplaced case and restore a kind-neutral message, or strip the scan error's own way-forward text before appending it."
---

### [MEDIUM] --card watch skip is untested

**Location:** internal/gatecli/scope.go:192
**Issue:** The `scopeCard` branch of `computeScope` reads the merge-base diff and fills `sc.skip`, and `skipNames` names the tests. The only scope test of the skip is the computed one. The `tiers_test.go` rows that mention `scopeCard` hand-build scopes with no skip. A regression there would fail nothing.

### [MEDIUM] Guard-step watch drop is untested

**Location:** internal/gatecli/scope.go:524
**Issue:** `scanGuards` removes a watched guard test that the card scope's changes leave alone, using `moduleRelative` to align paths. `TestTargetsFor` covers the computed route only. The C# fixture test is exactly this case, a guard that is also watched.

### [MEDIUM] New package-level var for a test-waited clock

**Location:** internal/gitexec/gitexec.go:57
**Issue:** `PATTERN-test-speed` requires a clock a test would otherwise wait out to be a field or parameter, never a package `var`, so tests keep `t.Parallel`. `armDeadline` is a package `var`, and `SetDeadlineForTest` says its caller must not be parallel. The surrounding `remoteDeadline` and `startStallTimer` vars are precedent, but the written rule outranks inferred convention.

### [LOW] Production deadline path lost its test

**Location:** internal/gitexec/gitexec_unix_test.go:141
**Issue:** Every deadline row of `TestRun_RemoteBounds` now ends on a ready signal. The default `context.WithTimeout` in `armDeadline` runs in no test. The board entry says no test may lose what it pins, and the real deadline killing a hung remote was pinned before.

### [LOW] Cancellation test no longer proves a running child dies

**Location:** internal/verifyrun/run_integration_test.go:166
**Issue:** The first write of `echo up && sleep 30` can precede the `sleep` start. The old 300 ms pause let the child exist before the cancel.

### [LOW] Guard pattern text is stale

**Location:** pattern/PATTERN-test-speed.md:117
**Issue:** The text says marked tests run whatever the impacted set. `scanGuards` and `targetsFor` now drop a watched guard test. `internal/gatecli/doc.go` already says so; the pattern background does not.

### [LOW] Double way-forward in a refusal

**Location:** internal/gatecli/cli.go:240
**Issue:** The `GuardMisplaced` case now formats `err`, and a watch misplacement error carries its own way-forward text from `internal/impactset/watches.go`. The refusal then has two.

## Rejected

- Zero-value fallbacks of `detachSeams.frontPoll` and `leaseRead` untested (lens:code-tests, LOW): `defaultDetachSeams` sets both explicitly, so the fallback is a per-helper branch the public surface does not need, and the rubric's test-economy rule disfavours it.
- `isWatchLine` rows for `//lym:watchx` and a tab separator (lens:code-tests, NIT): the boundary is one line of code and the cost of a row is not warranted.
- `--base` scope has no scope-level skip row (lens:code-tests): it shares `DeriveTargets` with the computed scope, whose subtest and `TestTargetsFor` rows cover the behaviour.
- The `RunWithin` row depends on the earlier row's pid file (lens:code-correctness note): the row removes it and fails clearly if missing, and the earlier row always writes it.
- Waits that cover a process or binary start: the checked waits (`remoteDeadline`, `remoteWaitDelay`, the 10 s and 5 s test bounds) keep their lengths in the diff; the new `hangStarted` readiness only ends a deadline early in tests and never shortens a production bound.
