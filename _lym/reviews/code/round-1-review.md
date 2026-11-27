---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:code-correctness
    location: internal/gatecli/memoryguard.go:264
    summary: The own share sums only the guard's root tree, so memory the same call holds elsewhere (sibling --copies copies, the report steps' test binaries beside the last tier) counts as other load, and a run that fills the slice itself is called an infrastructure stop, rerun, then left unjudged with no fixer.
    fix: "In memoryGuard.kills, compute the run's own share over every process of the calling gate call, not only sample. Give memoryGuard a field such as `own func() ([]proc.ProcessSample, error)` that newMemoryGuard and the tier and report guards set to proc.TreeSample of the lym verb's own pid (os.Getpid()), and sum RSS over that tree. Use the same set to exclude own pids in otherHolders, so sibling copies and report-step binaries are never named as other holders. Add a TestMemoryGuard row where the guarded tree is small but the call's whole tree (a sibling copy at 6000 MB) is over half of Used, and want an ordinary pressure kill (slicePressure false)."
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/gatecli/tierrun.go:234
    summary: slicePressure checks only len(record.Failures) == 0, which drops package-level failures, so a pressure kill beside a build or vet failure, TestMain exit or init panic in another package is rerun, and on a repeat labelled not a test failure, skips --stop-on-fail and spawns no fixer.
    fix: "In runTierCommands, set slicePressure only when no failure in the `failures` list belongs to a package other than the killed binaries' packages: compare each runFailure with test == \"\" against the packages of the guard's kills (carry the killed binary's package dir or import path on guardKill), and also require record.Failed == 0. Add a TestRunTierMemoryGuard row with a slice-pressure kill plus a package-level FAIL line of another package, and want slicePressure false."
  - id: F3
    severity: MEDIUM
    class: scope
    origin: lens:code-scope
    location: internal/verifytree/verify.go:530
    summary: The verify sites built on verifytree (loom verify, landing gates, weaver, lym webster verify) read only FindRunnerCrash, so a memory pressure stop reaches them as an ordinary failure and they may start a fixer that cannot clear it, the failure mode the board entry reports for the loom verify step; gateslot's package doc also claims every site tells the stop apart.
    fix: "In internal/verifytree/verify.go, have runnerCrashDetail (or a sibling it calls beside it at verify.go:440) also match gateslot.FindMemoryPressureStop and set the same not-judged flag with a detail naming the pressured runs, so the readers that key off RunnerCrash (loomshed/verifygate.go, landingshed/verifygate.go, websterengine/verifygate.go, webstercli/verify.go, weaverengine/verify.go, loomshed/redfirstgate.go) treat it as not judged; rename the flag and its doc if it now covers both. Add a verifytree test row for a log holding the pressure line. Remove the Bounds entry from _lym/landing/summary.md once done."
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/gatecli/run.go:283
    summary: No test pins that runTest closes with MemoryPressureLine when no failure judged the code, that it then leaves out RunnerCrashLine, or that stopsCall lets --stop-on-fail pass a pressure stop; the detached runner relies on that line to skip the fixer.
    fix: "Add rows beside the runner-crash closing-line rows in internal/gatecli/test_integration_test.go (around lines 1613-1685) and the --stop-on-fail crash row (around line 1832): a tier whose run and rerun both end on slice-pressure kills closes with gateslot.MemoryPressureLine and no RunnerCrashLine; one with a crash run and a pressure run closes with the pressure line only; one with a pressure stop and a failing lint prints neither; and a --stop-on-fail call runs the next tier after a pressure stop."
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/gatecli/copies_test.go:197
    summary: TestMergeCopies never sets or asserts slicePressure, so the new rule for a --copies slice-pressure stop is uncovered.
    fix: "Add rows to TestMergeCopies: every copy slicePressure gives merged.slicePressure true; one copy slicePressure and the rest passed gives true; one copy slicePressure and another failed otherwise (ExitCode 1, slicePressure false) gives false; assert merged.slicePressure in each."
  - id: F6
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/gatecli/tierrun_test.go:398
    summary: TestRunTierMemoryGuard has only a pure slice-pressure row and a pure over-cap row, so slicePressureOnly's exclusion of mixed stops (a pressure kill beside an over-cap kill, the memory cap or the process cap) and the no-failing-test condition are untested.
    fix: "Add TestRunTierMemoryGuard rows, or a table test of runStops.slicePressureOnly with the tierResult wiring, for a pressure kill plus an over-cap kill, a pressure kill plus a memoryCap line, a pressure kill plus a processCap line, and a pressure kill plus a failing test in the command's output; want slicePressure false for each."
  - id: F7
    severity: MEDIUM
    class: scope
    origin: lens:code-scope
    location: _lym/landing/summary.md:45
    summary: The board entry asks to check the memory use of TestTestOnlyCode_EveryProductionFunctionIsReachableFromAMain, and the summary says it was read, not measured, giving no figure for the memory the killed cmd/lym binary held.
    fix: "Measure the peak resident memory of the cmd/lym test binary running only that test (for example through lym gate test --tier with a -test.run filter and the guard's log, or the binary under /usr/bin/time -v), and record the figure in _lym/landing/summary.md beside gate.memory_quit_mb and the 1024 MB pressure kill floor, saying whether the test alone can reach the floor."
  - id: F8
    severity: LOW
    class: consistency
    origin: lens:code-correctness
    location: internal/gatecli/tierrun.go:498
    summary: A first run that was a slice-pressure stop gets no mark on its run record, while a crash gets record.RunnerCrash, so the records on file cannot tell a pressure stop from an ordinary guard kill.
    fix: "Add a tolerant field to gateslot.RunRecord such as `MemoryPressure bool` with yaml/json key memory_pressure, set it on the first run's record in retryTier where pressured is true and on the rerun's record when retried.pressureStop is set, extend the gatecli doc line on runner_crash to name it, and assert it in the TestRetryTier pressure rows."
  - id: F9
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/gatecli/memoryguard_test.go:462
    summary: No row pins otherHolders' fallback name for a holder whose executable could not be read; every holder in otherLoad carries a Path.
    fix: "Add a holder with an empty Path and an RSS among the three largest to a TestMemoryGuard slice-pressure row (or a new row), and pin the line literal holding \"a process whose executable could not be read (pid N) at M MB\"."
  - id: F10
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/proc/proc_linux_test.go:193
    summary: Neither withinCgroup's root-cgroup branch nor its exclusion of a sibling whose name shares the prefix is pinned, so a check that dropped the trailing slash would pass.
    fix: "In TestProcessTreeReads, assert that ResidentInCgroup(\"/\") lists the test process, and that ResidentInCgroup(ownCgroup + \"x\") (a sibling sharing the prefix) does not list it."
---

### [BLOCKING] The run's own share misses the call's own processes outside the guarded tree

**Location:** `internal/gatecli/memoryguard.go:264`

**Issue:** `kills` sums RSS over `sample`, the tree under the guard's root.
A tier guard is rooted at its own command (`tierrun.go:625`), and each `--copies` copy runs its own guard over its own command (`copies.go:94`).
Four copies of 3.5 GB in a 16 GB slice each see own = 3.5 GB, under half of Used: every copy is a slice-pressure kill, `mergeCopies` marks the tier, and a repeat makes it a `pressureStop` that judges nothing and gets no fixer.
The line names the sibling copies as "other holders" and says "no change to the code under test frees it", which is false.
The report steps' test binaries, which run beside the last tier as children of the lym verb (`reports.go:99`, `run.go:199-200`), are miscounted the same way.
This breaks the board entry's ask to tell slice-wide pressure apart from the run's own growth.
Memory charged to `Used` that no RSS shows, such as tmpfs, shmem and kernel memory (`memorylimit.go:78`), and test-started daemons that left the tree, widen the gap further.
The fix addresses the in-call processes.
Those other gaps are a bound the summary should name.

### [MEDIUM] A package-level failure beside a pressure kill is classed as judging no code

**Location:** `internal/gatecli/tierrun.go:234`

**Issue:** `record.Failures` comes from `testFailures`, which drops every failure whose `test` is empty (`tierrun.go:399-404`).
A build or vet failure, a TestMain exit or an init panic in another package, beside a slice-pressure kill, still gives `slicePressure`.
After a repeat it is a `pressureStop`: the label says "not a test failure", `stopsCall` lets the call pass, `MemoryPressureLine` closes the call, and the detached runner spawns no fixer.
`runnerCrash` checks `record.Failed` too, and this check does not.

### [MEDIUM] Verify sites do not read the memory pressure line

**Location:** `internal/verifytree/verify.go:530`

**Issue:** Only `detachfixer.endedUnjudged` reads `FindMemoryPressureStop`.
`verifytree` sets its not-judged flag from `FindRunnerCrash` alone.
The loom verify, the landing gates, weaver and `lym webster verify` therefore see a double pressure stop as a failure and may start a fixer for it.
Issue #267 names a loom verify run among the kills, so the board's ask to report such a run as not judged stays unmet there.
The summary's `## Bounds` names the gap, but no `Added after Discussion` entry narrows the scope.
`internal/gateslot/doc.go` now says a site tells a full memory limit from a failing test, which holds for the detached runner only.

### [MEDIUM] The call's pressure closing line and `stopsCall` are untested

**Location:** `internal/gatecli/run.go:283`

**Issue:** No test covers the closing line's precedence over `RunnerCrashLine`, its `judged` gate, or `stopsCall` skipping a pressure stop.
The runner-crash path has integration rows for both.
The detached runner's fixer skip depends on this line, so dropping it would pass every test.

### [MEDIUM] `mergeCopies` slice-pressure rule is untested

**Location:** `internal/gatecli/copies_test.go:197`

**Issue:** `TestMergeCopies` asserts launches, exit and copies only.
The new `slicePressure` fold has no row for all copies under pressure, one with the rest passing, or one beside an otherwise failed copy.

### [MEDIUM] Mixed stops and the no-failing-test condition are untested

**Location:** `internal/gatecli/tierrun_test.go:398`

**Issue:** `slicePressureOnly` returns false when the memory cap, the process cap or an over-cap kill comes with a pressure kill.
`slicePressure` also requires that no test failed.
No row exercises either, so the tier could be rerun and marked unjudged for a run that judged code, and no test would fail.

### [MEDIUM] The ask to check the named test's memory use is not met

**Location:** `_lym/landing/summary.md:45`

**Issue:** The board entry asks to check the memory use of `TestTestOnlyCode_EveryProductionFunctionIsReachableFromAMain`, and stresses that the use was never measured.
The summary reads the test and calls its memory "large but bounded", but gives no figure.
Without one, a reader cannot tell whether that test alone pushes `cmd/lym` past the 1 GB pressure floor.

### [LOW] A pressure stop leaves no mark on its run records

**Location:** `internal/gatecli/tierrun.go:498`

**Issue:** A crash sets `record.RunnerCrash` on the first run's record.
A slice-pressure stop sets nothing, and `pressureStop` lives only in memory.
The records on file therefore read the same as those of an ordinary guard kill.

### [LOW] `otherHolders`' unreadable-executable name is unpinned

**Location:** `internal/gatecli/memoryguard_test.go:462`

**Issue:** Every holder in `otherLoad` has a `Path`, so the fallback name has no row.

### [LOW] `withinCgroup`'s root and sibling-prefix cases are unpinned

**Location:** `internal/proc/proc_linux_test.go:193`

**Issue:** `TestProcessTreeReads` checks the own cgroup, its parent and a missing child only.
A version that dropped the `+"/"` would still pass.

## Rejected

- Code-scope's BLOCKING severity for the verify sites is lowered to MEDIUM. The gate call itself reruns and labels the stop, and the summary names the gap, so the core asks hold. The verify-site gap is a partial build of one ask.
- Code-affected-set's verify-site finding is merged into F3. It describes the same defect.
- Merge danger `two-way`, blast radius `gate`: not a finding. The diff changes no record format or config meaning, and an older binary reads the new closing line as an ordinary failure.
- Crash-then-pressure runs that carry no flag: not a finding. This matches the documented shortage-then-crash precedent.
