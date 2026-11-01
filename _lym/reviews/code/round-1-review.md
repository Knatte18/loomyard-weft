---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/gatecli/failurereport.go:207
    summary: The 80-line cap cuts the lint findings first and its cut line says the logs hold them, but a lint step has no log
    fix: In renderFailureReport, stop cutting a lint step's findings silently. Either reserve room so each run with failures keeps at least its first findings, or make the cut line say how many lines were cut from which runs and name the lint steps as having no log (point at re-running `lym loom lint-comments` or `lym loom lint-pattern` for the full list). Add a row to the report-cap case in failurereport_test.go with a tier flood followed by a lint run, asserting the lint run still appears or the cut line names it.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/gatecli/build.go:75
    summary: No test drives a failing shared build step through the call and checks that the failure report lists the build run
    fix: Add a case to internal/gatecli/test_integration_test.go (or the nearest runGate-level test with a build step) where the hub's build command exits non-zero, and assert the stderr holds the failure report with a `build (exit N)` run, its log path and the last output lines, before the closing line.
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/gatecli/run.go:174
    summary: The lint loop's exit-code rule when a tier already failed and the halted-call report print have no test
    fix: Add to the gate-level test table in internal/gatecli/test_integration_test.go two cases. One has a failing tier and a failing lint, and asserts the exit code is the tier's and the closing line names both runs with the lint last. The other halts the call (a cancelled context) and asserts the failure report prints before the closing line and no lint runs.
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/gatecli/failurereport.go:144
    summary: touchedFiles hub branch (merge base with the pair's parent, and its fall back to HEAD on an error) has no test, and agents always run in a hub
    fix: Add a test for touchedFiles with a hub fixture whose pair has a parent branch, asserting the files differing from the merge base are listed, sorted and compacted, and a second row where the merge base cannot be read asserting the list falls back to files changed since HEAD. Use the repo's hubforge fixture per PATTERN-test-isolation.
  - id: F5
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/gatecli/failurereport.go:196
    summary: A run's unparsed lines, including its timeout and no-tests-ran cause lines, are dropped whenever any failure was read
    fix: In renderFailureReport, print the run's cause lines (those tierrun.go puts first in unparsed) even when failures were read, or keep the causes in their own failedRun field and always print them. Add a row with a run that has both a failed test and a cause line, asserting both appear.
  - id: F6
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/gatecli/failurereport.go:228
    summary: A build error line in a nested module is relative to the run directory, not the worktree, so the report's worktree-relative file:line promise fails for it
    fix: In qualify, also prefix a compile error line that opens with `./` or a path relative to the run directory with the run's moduleDir (the line's path joined to moduleDir), not only bare file names of test lines. Add a row to the qualify cases in failurereport_test.go for a build error from a nested module.
  - id: F7
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/gatecli/failurereport.go:107
    summary: runLints pattern branch, the patternlint.Lint call and the finding text, is driven by no test; the only scenario lists comments
    fix: Add a row to the lint scenario in internal/gatecli/test_integration_test.go (or a direct runLints test on a fixture repo) listing only the pattern lint and a PATTERN.md grown past its headroom line, asserting one `lint pattern (1 finding)` run with the PATTERN.md finding message and the way forward.
  - id: F8
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/gatecli/detachfront.go:291
    summary: front.replay's switch to replayStdout is pinned by no detached-front test; TestReplayStdout covers only the function
    fix: Add a case to the detached-front replay test that records a run with a stdout past replayStdoutBytes and asserts the front prints the size note to stderr, not the stdout body, and exits with the recorded code.
  - id: F9
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/loomcli/lintcomments.go:1
    summary: The lint-comments and lint-pattern verb docs still say every card gate runs them as its own steps, which the card gate no longer does
    fix: In internal/loomcli/lintcomments.go lines 1 and 34 and internal/loomcli/lintpattern.go lines 1 and 23, reword to say the verb is the standalone form of the lint that a gate key's `comments` or `pattern` lint step also runs inside `lym gate test`, as docs/overview.md and pattern/PATTERN-gate-slot.md now say.
  - id: F10
    severity: LOW
    class: consistency
    origin: handler
    location: contracts/specs/loom-plan-spec.md:336
    summary: The Tier 1 description still lists the comment and PATTERN lints as steps after the gate call, but the card gate is now the one call
    fix: Reword line 336 to say the per-card gate is the hub's `card` key call, `lym gate test --gate card --card <card id>` without `--wait`, which also runs the comment line-break lint and the PATTERN headroom lint the key lists.
---

### [MEDIUM] Lint findings are cut first and the cut line points at logs that do not hold them

**Location:** internal/gatecli/failurereport.go:207
**Issue:** Lint runs are appended after the tier runs, so past `failureReportLines` lines the lint findings are what get cut. The cut line says the logs named above hold the rest, yet a lint step has no log, so those findings exist nowhere else. This breaks the entry's aim that every failure is listed in one go.

### [MEDIUM] Build failure wiring is untested

**Location:** internal/gatecli/build.go:75
**Issue:** The `e.failed` append for a failed shared build is exercised by no call-level test. Removing the line would pass every visible test, and a build error is a failure class the entry names.

### [MEDIUM] Lint exit-code and halted-call report branches are untested

**Location:** internal/gatecli/run.go:174
**Issue:** The `code == 0` guard that keeps an earlier tier's exit code, and the report print on the halt path, have no row. The single integration lint scenario has no failing tier and no halt.

### [MEDIUM] touchedFiles hub branch is untested

**Location:** internal/gatecli/failurereport.go:144
**Issue:** Agent calls run in a hub, so the merge-base branch is the one that matters, yet only the HEAD base is covered.

### [LOW] Cause lines hidden when a failure was read

**Location:** internal/gatecli/failurereport.go:196
**Issue:** The `switch` prints `unparsed` only when a run has no failures. The timeout and no-tests-ran lines are placed in `unparsed`, so they disappear when a failure was also read.

### [LOW] Nested-module build errors are not worktree-relative

**Location:** internal/gatecli/failurereport.go:228
**Issue:** `qualify` matches bare `name.go:N` lines only. Compile errors print paths relative to the run directory, which differs from the worktree root for a nested module.

### [LOW] runLints pattern branch untested

**Location:** internal/gatecli/failurereport.go:107
**Issue:** `TestLintFailedRun` passes findings in by hand, and the integration scenario lists only `comments`, so the `patternlint.Lint` call and the `PATTERN.md: ` text are uncovered.

### [LOW] Replay wiring untested

**Location:** internal/gatecli/detachfront.go:291
**Issue:** Reverting `replay` to `copyRunFile` passes every visible test.

### [LOW] Verb docs still describe the old card gate

**Location:** internal/loomcli/lintcomments.go:1
**Issue:** Four doc lines say every card gate runs the verbs after its tests; the card gate now runs the same functions as lint steps inside its one call.

### [LOW] Plan spec Tier 1 still lists lints as separate steps

**Location:** contracts/specs/loom-plan-spec.md:336
**Issue:** The sentence reads as gate call, then two lints. The lints are now inside the call.

## Rejected

- Counters keep only the first 12 and 8 lines, so earlier noise can push out the real error (code-correctness): a stated best-effort bound, not a defect.
- No row for the `unparsedLines` cap on `goJSONCounter.handle` (code-tests): the cap is a constant guarding output size, and a cap regression changes nothing a caller acts on.
- No multibyte row for `capLine` (code-tests): the rune-boundary loop is a four-line guard, and no behavior the entry names depends on it.
- `tierFailedRun` timed-out and empty-`RunDir` rows missing (code-tests): the outcome string is a one-line branch, and the empty-`RunDir` case only skips qualification.
- `Config.Gate` returning `Lints` has no test (code-tests): no code reads `Lints` through `Gate`; the run reads `cfg.Gates` directly, so the behavior is unreachable.
- A .NET hub's tuned `card` key now runs both lints (code-affected-set, unverified): `patternlint.Lint` answers no finding for an absent `PATTERN.md` (internal/patternlint/lint.go:60-62, 80), so it is harmless.
- Other lens items marked correct or pinned were not findings.

Checked and not a defect: the Webster stencil change adds one sentence and drops or weakens no rule; the `internal/websterengine` and `internal/loomshed` edits follow from the card gate no longer rendering separate lint steps, so they are licensed; the commit carries no `Pattern-Departure:` trailer and the directive declares no PATTERN changes; `pattern/PATTERN-gate-slot.md` matches the code.
