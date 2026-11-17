---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/proc/memorycap_linux.go:160
    summary: Under a tier's memory_mb, a process-cap SIGKILL of the tier shell, or any outside SIGKILL of it, is reported as a kill at memory_mb, so the tier prints a false memory cap line beside the process cap line
    fix: In runMemoryCapped (internal/proc/memorycap_linux.go), compute treeKilled from scopeHadOOMKill(cgroup) alone, read before killScope, and keep killed as it is so RunMemoryCapped's contract is unchanged. In runInvocations (internal/gatecli/tierrun.go:404), set stops.memoryCap only when capKilled is true and stops.processCap is empty. Update the treeKilled doc comments in memorycap.go and memorycap_linux.go to say it reports a kernel OOM kill in the scope.
  - id: F2
    severity: MEDIUM
    class: design
    origin: handler
    location: internal/gatecli/memoryguard.go:376
    summary: In lym gate redundant and lym gate testmap a guard SIGKILL of a test binary reads in RunAlone as a kill at redundancy_test_memory_mb, so a pressure kill caused by memory held elsewhere counts as a mutant kill or sets FailsUnmutated and is reported as the 2048 MB cap
    fix: Let RunAlone tell a guard kill from a cap kill. Have guardOwnTests return, beside stop, a func(pid int) bool that reports whether the guard killed pid. Pass it to testcover.Toolchain as a new field. In RunAlone, check cmd.Process.Pid against it after the run, and return a new RunResult flag (for example GuardKilled) instead of MemoryCapped. In redundantmutants.go and redundancy.go, treat a GuardKilled run as not judged, with a reason naming the memory guard, never as a mutant kill or FailsUnmutated. In testmappackage.go, record it as failed with that reason.
  - id: F3
    severity: MEDIUM
    class: scope
    origin: handler
    location: internal/gatecli/run.go:216
    summary: The redundancy report step that lym gate test runs after its tiers starts test binaries through RunAlone with no memory guard, though the board entry asks for the guard over every test binary lym starts
    fix: In runTest (internal/gatecli/run.go:216), wrap exe.redundancyStep().run(red, inHub) as stop := guardOwnTests(ctx, request.memoryQuit, request.stderr); exe.redundancyStep().run(red, inHub); stop(). Add the report step to the Memory guard section of internal/gatecli/doc.go, to the memory_quit_mb comment in internal/gateslot/template.yaml and to the guard sentence in docs/overview.md. Drop the summary's design decision that the report step runs no guard.
  - id: F4
    severity: MEDIUM
    class: scope
    origin: handler
    location: internal/gatecli/tierrun.go:129
    summary: The board entry asks the kill report to name the test that was killed, but in a tier run without -run, the default, neither the guard's kill line nor the tier's causes name any test
    fix: In runTier (internal/gatecli/tierrun.go), for each guard kill line, append the tests the go-json counter saw start in that run and never pass, fail or skip. Expose them from the counter as a method returning the unfinished test names per package. Name them in the cause line as the tests that were running when the binary was killed. Keep the -test.run filter in memoryGuardLine for runs that pass one.
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/gatecli/child.go:64
    summary: A memory_mb cap that cannot be set fails the tier with no cause line and no way forward when the scope never ran the command, and refuses it with the wrong way forward, the shell PATH hint, when the start fails before the shell
    fix: In runInvocations (internal/gatecli/tierrun.go:407-411), when run.tier.MemoryLimit() is above 0, ctx has not ended and runErr is no *exec.ExitError, return a refusal naming the tier, its memory_mb, runErr and the way forward. The way forward is to remove tiers.<tier>.memory_mb with lym config gate from the prime, or to give the systemd user manager the memory controller, then re-run the same command. Return it before the existing shell-start refusal.
  - id: F6
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/gatecli/tierrun.go:386
    summary: No test pins how a tier run handles a memory guard kill or a kill at its memory_mb cap, that is exit 1 on a zero exit, the line in the log and causes, memoryStopped set and no rerun
    fix: Give testRequest a memoryGuardFor seam like processCapFor, so tierrun.go:390 builds the guard through it. Add rows to the tier-run test beside TestRunTierProcessCap (internal/gatecli/tierrun_test.go). One row has a guard that returns one kill line over a command that exits 0, and asserts exit 1, the line in the log and causes, memoryStopped set and failedOnShortage empty. Add a tier1 row in a _linux test file that runs a tier with memory_mb over a memory hog on a host whose user manager caps a scope, skipping otherwise, and asserts the tierMemoryCapLine cause.
  - id: F7
    severity: LOW
    class: decision
    origin: handler
    location: internal/gateslot/tiers.go:238
    summary: The board entry asks to put the gate tiers' go test runs under the kernel cap, but memory_mb has no default and the change sets no hub value, so every tier still runs uncapped
    fix: Leave the code in place and defer this to the run's parent. The parent either sets memory_mb on the hub's tiers with lym config gate from the prime, or records with lym loom decision add that tiers stay uncapped unless the hub sets memory_mb.
  - id: F8
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/gatecli/memoryguard.go:423
    summary: guardOwnTests has no test of its no-op stop when the guard does not apply, nor of the stop that cancels the guard and waits for its end
    fix: Give guardOwnTests its guard and applies decision as parameters, with the production caller passing newMemoryGuard and memoryGuardApplies. Add a TestMemoryGuard row or a small table that asserts the stop returns at once when the guard does not apply. A second row asserts that the stop cancels a running scripted guard and returns only after its run has ended.
  - id: F9
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/gatecli/doc.go:728
    summary: The sentence that a vstest tier is bounded by its timeout_min alone now follows the memory_mb paragraph, where it wrongly says the kernel cap watches .test processes only
    fix: Move the sentence It watches `.test` processes only, so a `vstest` tier is bounded by its `timeout_min` alone. back to the end of the CPU watchdog section in internal/gatecli/doc.go, after the outermost gate run sentence. Add to the Memory guard section that the guard watches `.test` processes only, while a tier's memory_mb caps every process of its command, vstest included.
  - id: F10
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/gatecli/cli.go:160
    summary: The stderr field comment lists the lines the verb prints there but leaves out the memory guard's kill lines and the tier's memory_mb cap line the diff now prints
    fix: Reword the comment at internal/gatecli/cli.go:160 to say stderr receives the verb's lines beside its output, namely the CPU watchdog's quit, the memory guard's kills, a tier's timeout or memory cap, and each tier's summary.
---

### [MEDIUM] A process-cap or outside SIGKILL of the shell reads as a memory_mb kill

**Location:** `internal/proc/memorycap_linux.go:160`, `internal/gatecli/tierrun.go:404`

**Issue:** `processCap.run` SIGKILLs every pid that carries the run marker, and the tier shell carries it, since `cmd.Env` holds `LYM_GATE_TEST_PID`.
Under `memory_mb`, `runMemoryCapped` then sees the command signalled with SIGKILL and no cancel, so it sets `killed` and returns `treeKilled` true.
`runInvocations` turns that into `stops.memoryCap`, and the tier prints `tierMemoryCapLine`, which wrongly says the kernel killed the run at its `memory_mb`, beside the process-cap line.
The kernel's own `oom_kill` count in the scope's `memory.events` is the signal that tells a cap kill apart.

### [MEDIUM] A guard kill in lym gate redundant and lym gate testmap counts as a kill at the memory cap

**Location:** `internal/gatecli/memoryguard.go:376`, `internal/testcover/runner.go:178`, `internal/gatecli/redundantmutants.go:397`

**Issue:** With a capping scope, the test binary is the scope command's own process.
A guard SIGKILL of it makes `runMemoryCapped` report `killed`, so `RunAlone` returns `MemoryCapped`, and the run is reported as ending at `redundancy_test_memory_mb`.
The pressure trigger kills the largest tree of 1 GB or more because memory is short elsewhere on the machine, which says nothing about the test.
In the mutation check such a run counts as the test killing the mutant, so a cover's pressure-killed run can confirm a candidate as redundant falsely.
On an unmutated run it sets `FailsUnmutated`.
The board entry's own premise is that the kernel cap is the bound whose kill counts.

### [MEDIUM] The redundancy report step's test binaries run unguarded

**Location:** `internal/gatecli/run.go:216`

**Issue:** The board entry's Idea asks for the guard over every test binary lym starts, and its Built-since section lists the redundancy report step among the `RunAlone` paths.
`lym gate test` runs the report step after the tiers with no guard: the tier guards end with each command, and `guardOwnTests` is called only by `lym gate redundant` and `lym gate testmap`.
The summary justifies the gap by the kernel cap, but that reason applies equally to `lym gate redundant` and `lym gate testmap`, which got a guard.

### [MEDIUM] A tier run without -run never names the killed test

**Location:** `internal/gatecli/tierrun.go:129`, `internal/gatecli/memoryguard.go:390`

**Issue:** The board entry asks the guard to "report which test and package was killed".
The kill line names the package directory, and the test only through `-test.run`, which a normal tier run does not pass.
The run's go-json counter already sees each test's start and end in the package's output, so the tests running at the kill are known to the tier, but no line names them.

### [MEDIUM] A memory_mb cap that cannot be set fails the tier silently or with the wrong way forward

**Location:** `internal/gatecli/child.go:64`, `internal/gatecli/tierrun.go:407`

**Issue:** When the scope never runs the command, `runMemoryCapped` returns an error that is no `*exec.ExitError`.
`tierExitCodeOf` maps it to exit 1 and `cmd.Process` is set, so the tier fails with no cause line, nothing on stderr and no way forward; the error reaches only the Info log.
When the capped start fails before the shell starts, through a failed `os.Pipe` or a Windows job that cannot be created, `cmd.Process` is nil.
The refusal then tells the user to fix the tier's dir and put the shell on PATH, and leaves out the real cause.

### [MEDIUM] The tier-run wiring of guard kills and memory_mb kills has no test

**Location:** `internal/gatecli/tierrun.go:386`, `internal/gatecli/tierrun.go:117`, `internal/gatecli/tierrun.go:241`, `internal/gatecli/child.go:63`

**Issue:** `TestRunTierProcessCap` pins the process cap's wiring into exit, log, causes and the no-rerun rule.
No test pins the same for a memory guard kill or a kill at `memory_mb`.
`newMemoryGuard` is called directly in `runInvocations`, so no test can reach the guard's path, and nothing references `tierMemoryCapLine` or `runTiedChild` with a limit.
A regression that dropped `stops.memoryLines()` from the exit or causes, or the `memoryStopped` flag, would pass every test.

### [LOW] Every tier still runs uncapped

**Location:** `internal/gateslot/tiers.go:238`

**Issue:** The Idea's third bullet asks to put the gate tiers' `go test` runs under the kernel cap with a per-tier setting.
The diff adds the setting with no default, and the summary says the change sets no hub value, so no tier is capped yet.
The summary gives a reason, that the right size depends on the tier's parallel and packages, but no recorded decision narrows the board entry this way.

### [LOW] guardOwnTests' start and stop are untested

**Location:** `internal/gatecli/memoryguard.go:423`

**Issue:** `TestMemoryGuardApplies` covers the decision and `TestMemoryGuard` the loop.
Nothing pins that the stop is a no-op when the guard does not apply, or that it cancels the guard and waits for its end.
That wiring is what `lym gate redundant` and `lym gate testmap` rely on.

### [LOW] A CPU watchdog sentence now trails the memory_mb paragraph

**Location:** `internal/gatecli/doc.go:728`

**Issue:** The new Memory guard section was inserted ahead of the sentence "It watches `.test` processes only, so a `vstest` tier is bounded by its `timeout_min` alone.", which used to close the CPU watchdog section.
After the `memory_mb` paragraph, "It" reads as the kernel cap, which holds every process of the command, a `vstest` tier's included.

### [LOW] The stderr field comment misses the new lines

**Location:** `internal/gatecli/cli.go:160`

**Issue:** The comment lists what `stderr` receives: the CPU watchdog's quit, a tier's timeout and each tier's summary.
The diff also prints each memory guard kill line there, through the guard's `out`, and the tier's `memory_mb` cap line, at `tierrun.go:126`.

## Rejected

- The code-tests lens's unfiled notes, the pressure branch's skip of an already killed binary and `RunMemoryCappedVia` on the data segment fallback, are no findings: both paths are a line each and follow the tested ones.
