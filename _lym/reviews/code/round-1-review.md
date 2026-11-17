---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/proc/memorycap_linux.go:170
    summary: A systemd-run that cannot start the scope exits non-zero and reads as a failing test, so a mutant counts as killed or a baseline as FailsUnmutated, against the RunMemoryCapped doc that a cap not set is reported as an error
    fix: In runMemoryCapped's scope path, tell a scope that never started from a command that ran and failed, and return that case as an error rather than as cmd's exit. One way is to have the scope run the command through a marker that only the command's own process writes once the scope exists. Another is to check the run's cgroup after Start, before Wait, and fail the call when it is not the new scope. Either way, RunMemoryCapped returns a non-ExitError error for a cap that could not be set, which makes RunAlone return an error. Add a row to TestRunMemoryCapped_EndsAChildOverTheCapByTheHostsMeans, or a new Linux row, that runs with a memoryScope whose starter fails at once, such as /bin/false through export_linux_test.go, and asserts an error that is not an *exec.ExitError.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/proc/memorycap_linux.go:134
    summary: The probe runs systemd-run with no bound, so a user manager that does not answer stalls the first capped run and memoryCappedParallel for ever, likeliest under the memory.high throttle this change targets
    fix: In probeMemoryScope, run the probe through exec.CommandContext with a context.WithTimeout of a named constant of a few seconds, such as probeTimeout = 5 * time.Second, with WaitDelay set. Treat a timeout as a failed probe that returns memoryScope{}, so the host falls back to the data segment limit.
  - id: F3
    severity: MEDIUM
    class: decision
    origin: handler
    location: internal/proc/memorycap_linux.go:192
    summary: The fallback sets RLIMIT_DATA where the board entry's Fix names RLIMIT_AS, and the decision record carries no Added after Discussion entry for the change
    fix: Leave the code in place and defer this finding to the run's parent, who records or refuses the switch from RLIMIT_AS to RLIMIT_DATA with lym loom decision add. The summary's measured reason, that a 2048 MB RLIMIT_AS fails healthy Go binaries at start-up, is the case to put to the parent. The fixer changes no file for this finding.
  - id: F4
    severity: MEDIUM
    class: scope
    origin: handler
    location: internal/gatecli/testmappackage.go:672
    summary: lym gate testmap runs tier0 parallel capped runs side by side with neither the parallel-1 rule for the per-process fallback nor the headroom clamp, though the board entry's Fix puts the fallback at parallel 1
    fix: Apply the same side-by-side bound in lym gate testmap as in the mutation check. Where testMapRun takes its tier0 parallel, clamp it through memoryCappedParallel, injected on the testmap request as cappedParallel is on redundantRequest, and run that many tests at once. Update internal/gatecli/doc.go's testmap paragraph and docs/overview.md's testmap sentence to say fewer tests go side by side when memory is short. If the parent decides testmap keeps its parallel, record that with lym loom decision add instead and drop the code change.
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/testcover/runner_test.go:250
    summary: The outOfMemoryLine branch of RunAlone, the only way the fallback and Windows paths set MemoryCapped, is never run on a host whose scope probe passes
    fix: Add a TestRunAlone row whose stand-in prints the Go runtime's fatal out-of-memory line to its output and exits 2 without allocating, run with MemoryLimit set, and want a RunResult with Action ActionFail and MemoryCapped true. Add the stand-in case to runStandIn in internal/testcover/testmain_test.go. The row then pins the regex branch on every host.
  - id: F6
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/gatecli/redundancy.go:553
    summary: No test pins the redundancy report step listing a capped test as not judged with reasonMemoryCapped, nor testmap passing MemoryLimit to its runs
    fix: Extend the existing redundancy report step test and the existing lym gate testmap test with a fixture test that keeps memory past a small redundancy_test_memory_mb, such as 128, and stops by itself at 384 MB. In the report step test, assert that the test is listed as not judged with the reason ended at the memory cap of 128 MB. In the testmap test, assert that the run is recorded as failed. Add rows or assertions to those tests rather than new test functions.
  - id: F7
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: cmd/lym/spawnobservability_test.go:64
    summary: The spawn-observability exemption for memorycap_linux.go says callers log the command and its end, but the probe's own systemd-run spawn is logged by no caller
    fix: Make the reason true. Either log the probe from its callers by exposing its outcome, for example through MemoryCapCoversTree being called and logged once before the first capped run in testcover and gatecli, and state that in the reason. Or reword the Why text and the internal/proc/doc.go sentence to say the one-time probe spawn goes unlogged and why.
  - id: F8
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/gateslot/template.yaml:6
    summary: The redundancy_test_memory_mb comment says a capped test then counts as failed, but the redundancy report step leaves it out as not judged
    fix: Reword the comment's clause to match each caller, for example that the test is left out, recorded as failed by lym gate testmap, and a mutant run in lym gate redundant counts as a kill. Mirror the wording of the redundancy_test_timeout_sec comment above it.
  - id: F9
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/proc/memorycap_windows.go:392
    summary: The command joins the memory-limited job only after Start, so a process it starts in that window escapes the job limit and the TerminateJobObject cancel
    fix: Start the command suspended, with CREATE_SUSPENDED in cmd.SysProcAttr.CreationFlags, assign it to the job, then resume its main thread. Or, if that is out of reach, state the window in RunMemoryCapped's doc comment beside the Windows sentence.
  - id: F10
    severity: LOW
    class: design
    origin: lens:code-tests
    location: cmd/testtiming/redundancy.go:68
    summary: No test pins the new time and memory bounds on measurePackage's runner, which the board entry asks for
    fix: In the existing cmd/testtiming redundancy test, add a row or assertion that a fixture test past the memory cap is classified as failed and the run goes on, or, if no such test drives measurePackage, accept the gap and say so in the change description.
---

### [MEDIUM] A scope that never starts reads as a failing test

**Location:** `internal/proc/memorycap_linux.go:170`

**Issue:** In the scope path `runMemoryCapped` returns `cmd.Run`'s error as is.
When `systemd-run` cannot create the transient scope, for example because the run's `cmd.Env` lacks the user bus that the probe's own environment had, or the manager refuses the unit, `systemd-run` exits 1 before it execs the test.
`RunAlone` then sees an `*exec.ExitError` and returns `ActionFail`.
In `lym gate redundant` every baseline then reads as `FailsUnmutated`, or every mutant run as a kill, with no error surfaced.
That contradicts `RunMemoryCapped`'s doc at `internal/proc/memorycap.go:68`, which says the error reports a cap that could not be set.

### [MEDIUM] The probe has no time bound

**Location:** `internal/proc/memorycap_linux.go:134`

**Issue:** `exec.Command(starter, ...).Output()` waits for ever.
The probe runs once inside the first `RunMemoryCapped` or `MemoryCapCoversTree` call, before any test bound starts.
A user manager slowed by the slice-wide `memory.high` throttle, which is the condition the board entry describes, stalls the whole call there.

### [MEDIUM] RLIMIT_DATA replaces the board's RLIMIT_AS without a recorded decision

**Location:** `internal/proc/memorycap_linux.go:192`

**Issue:** The board entry's Fix says the fallback is `RLIMIT_AS` through `prlimit`.
The diff sets `RLIMIT_DATA`, and `_lym/landing/summary.md` argues for it under Design decisions.
The decision record holds no `Added after Discussion` entry for that change.
The reason looks sound: `RLIMIT_AS` counts reserved address space, which the Go runtime reserves far beyond what it uses.
But the change of scope is the parent's to record, not the writer's.

### [MEDIUM] testmap runs capped tests side by side without the fallback's parallel 1 or the clamp

**Location:** `internal/gatecli/testmappackage.go:672`

**Issue:** The board entry's Fix puts the `prlimit` fallback "at parallel 1" and clamps parallel to the memory headroom divided by the cap.
It also says the same `RunAlone` serves `lym gate testmap`.
The diff applies both bounds in the mutation check alone, through `holdSlot` and `memoryCappedParallel`.
`lym gate testmap` still runs up to tier0 parallel tests at once.
On a fallback host each process there has its own `RLIMIT_DATA`, and on a scope host four 2048 MB caps can pass the slice's `memory.high` headroom.
The summary names this a design decision, but no entry in the decision record licenses it.

### [MEDIUM] The out-of-memory branch of RunAlone is not pinned on a scope host

**Location:** `internal/testcover/runner_test.go:250`

**Issue:** `TestRunAlone/MemoryCapped` reaches the kernel-kill branch where the probe passes, and the `outOfMemoryLine` branch only where it fails.
On a host with a capable user manager no row runs the regex, which is the sole way the `RLIMIT_DATA` and Windows paths set `MemoryCapped`.

### [MEDIUM] The report step's and testmap's capped paths have no test

**Location:** `internal/gatecli/redundancy.go:553`

**Issue:** No test asserts the report step's `result.MemoryCapped` case and its `reasonMemoryCapped` text.
None asserts that testmap's runner carries `MemoryLimit` either: `internal/gatecli/testmappackage.go:672`.
Dropping the new `case` or either `MemoryLimit:` field fails no test.

### [LOW] The spawn-observability exemption's reason does not cover the probe

**Location:** `cmd/lym/spawnobservability_test.go:64`

**Issue:** The exemption says the callers of `RunMemoryCapped` log the command and its end.
`probeMemoryScope` spawns `systemd-run … cat /proc/self/cgroup` on its own.
No caller logs that spawn: `memoryCappedParallel` logs only the probe's conclusion.
PATTERN-child-processes requires the exemption's written reason to hold.

### [LOW] The template comment misstates the report step's handling of a capped test

**Location:** `internal/gateslot/template.yaml:6`

**Issue:** The comment says the test "then counts as failed".
The redundancy report step leaves it out as not judged, as `internal/gatecli/redundancy.go:553` and `internal/gatecli/doc.go:590` show.

### [LOW] The Windows job is joined after the command starts

**Location:** `internal/proc/memorycap_windows.go:392`

**Issue:** `cmd.Start` runs before `assignProcessToJob`, so a child the test spawns in between sits outside the job's memory limit and the cancel's `TerminateJobObject`.
The summary's Bounds section names the same window for `RLIMIT_DATA` but not for Windows.

### [LOW] cmd/testtiming's new bounds are untested

**Location:** `cmd/testtiming/redundancy.go:68`

**Issue:** The board entry asks for the report tool to get the same bounds.
The diff gives it a 2-minute, 2048 MB runner, and no test fails if either bound is dropped.

## Rejected

- code-tests finding 4, no assertion pins the 2 s floor: the floor is a tuned constant. A test that pins its value would only restate it, and the integration test's comment confirms the fixture timing still holds at the new floor.
- code-tests finding 5, `memoryCappedParallel` is stubbed in `TestGateRedundant`: the decision logic is all in `clampParallelToMemory`, which `TestClampParallelToMemory` pins row by row. The wrapper only reads two values and logs them.
