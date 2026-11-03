---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/gatecli/child.go:57
    summary: the branch that maps exec.ErrWaitDelay after a zero exit to nil has no test, because the only orphan scenario exits 134 and so never reaches it
    fix: In internal/gatecli/test_integration_test.go, turn the subtest at line 1550 into a table over the shell's exit code with rows 134 and 0, each with helperOrphanEnv=1 and helperPipeDelayEnv=100ms and the row's code in helperExitEnv, so it stays one behavior's test and adds no new test function. The 134 row keeps its current assertions. The 0 row asserts the call exits 0, the run record is outcome passed with ExitCode 0 and TimedOut false, the orphan is killed, and requireFree holds. Removing the nil mapping, or mapping the delay to a failure, then fails the 0 row.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/gatecli/build.go:47
    summary: the build step's bounded end has no test, since the stand-in starts an orphan only for a tier invocation and nothing would fail if runBuild passed a zero delay
    fix: In internal/gatecli/test_integration_test.go runStandIn, let the orphan branch also apply to the shared build invocation, for example by giving helperOrphanEnv a value that names the tier or the build, and keep the existing tier value working for the tier row. Then add a row to the existing scenario that runs the shared build step, found by grepping helperBuildExitEnv with sharedbuild, with helperBuildExitEnv=1, the orphan and helperPipeDelayEnv=100ms. The row asserts the call ends with the build's exit code, the build's run record is failed, the orphan is gone and requireFree holds.
  - id: F3
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/landing/summary.md:18
    summary: the change description says the start-time check of proc.KillTree leaves a reused group id alone, but the check only skips a process that started before the recorded child, so a later process that reused the id passes it
    fix: In _lym/landing/summary.md line 18, replace the clause that the start-time check leaves a reused group id alone with the true protection, which is that Linux keeps a group id pinned while any member lives and the sweep finds nothing once the group is empty. Leave the code as it is.
---

### [MEDIUM] The zero-exit ErrWaitDelay mapping is untested

**Location:** internal/gatecli/child.go:57, internal/gatecli/test_integration_test.go:1550
**Issue:** `runTiedChild` returns nil when `Wait` reports `exec.ErrWaitDelay`, and `exec.Cmd.Wait` reports that error only after a zero exit.
The new orphan subtest sets `helperExitEnv=134`, so `Wait` returns the `*exec.ExitError` and the nil-mapping branch never runs.
Deleting the branch, or turning it into a failure, leaves every test green.
It is the branch that decides whether a passing tier with an orphan still passes, so its behavior is pinned by nothing.

### [MEDIUM] The build step's bounded end is untested

**Location:** internal/gatecli/build.go:47, internal/gatecli/test_integration_test.go:275
**Issue:** The decision record's Expected section names both the tier and the build command.
`runStandIn` sets `orphan := !isBuild && ...`, so no test drives an orphan through a build invocation.
`runBuild` could pass a zero delay and the build would hang again with every test still green.
The build call shares `runTiedChild` with the tier, but its own argument at `build.go:47` is a separate behavior of the diff.

### [LOW] The change description overstates the start-time check

**Location:** _lym/landing/summary.md:18
**Issue:** The summary says `proc.KillTree` has a start-time check that leaves a reused group id alone.
`killTreePlatform` in internal/proc/tied_linux.go skips only a process whose start time is before the recorded one, so a process started after the recorded child, such as one that reused the pid and leads its own group, would pass it.
The sweep is still safe in practice, because Linux keeps a group id unassigned while any member lives and the window with an empty group is microseconds, but the stated reason is not what the code does.

## Rejected

- Lens code-tests finding 3, that `RunGate` passing `defaultPipeWaitDelay` at `rungate.go:28` is untested: a test would have to wait out the production 10 seconds, which `PATTERN-test-speed` forbids, and the value is a named constant used in two places with the injected path already covered by the orphan subtest.
- Lens code-correctness finding 1, that an unreadable start time at `internal/proc/tied_linux.go:31` makes the leftover sweep skip silently: the code is unchanged by the diff, `/proc/<pid>/stat` of a just-started child is practically always readable, and the tier still ends within the delay.
- The 10 second cost of killing only after the wait delay: it is a latency choice, the pipe read is already cut, and no leftover in the shell's group survives `runTiedChild`'s return, so it is no defect.

## Focus verdicts

- `killLeftovers` after a clean exit: it calls `proc.KillTree` on the shell's record. On Linux a live group member pins the group id, so a group that still has members cannot belong to a reused pid; an empty group has nothing to signal, and only a pid-space wrap inside a window of microseconds, up to the delay on the orphan path, could hit a new group leader with that id. Not a defect in practice; the summary's stated reason is F3.
- `ErrWaitDelay` handling: `Wait` reports it only after a zero exit and only when the context had no error, so the branch returns nil only for exit 0. A failing exit keeps its `*exec.ExitError`, and a timeout or cancel keeps the context error. The callers record exit 0 for nil, the shell's code for an `ExitError` and exit 1 with `TimedOut` for a timeout, so nothing is misrecorded. The zero-exit branch is untested, F1.
- Ordering: the kill runs after `Wait` returns, which is after the delay has cut the pipe read, and no path leaves a leftover of the shell's group holding the slot or the pipe past the call. The kill also runs on timeout and cancel, where it finds an already empty group, and a failed start returns before it with nothing to kill. A leftover that left the group is bounded by the delay alone, as `doc.go` says.
- Expected section: the diff carries the bounded delay on both commands through the shared `runTiedChild`, the group kill, and the shell's exit code on failure. The build path has no test of its own, F2.
- The new orphan scenario: before the change the sleeper held the pipe and the call never ended, so the scenario fails on the old code. It covers the tier with a failing exit only, so it misses the zero-exit shell, F1, and the build path, F2.
- `doc.go` and the `newCommand` doc: the 10 seconds, the group kill, the Windows job close after the delay and the session-leaver bound are each borne out by `child.go`, `tied_linux.go` and `tied_windows.go`. No spec doc that the repository's constraints name for gate behavior is left unchanged.
