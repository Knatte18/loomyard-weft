---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: internal/proc/memorycap_linux.go:50
    summary: the scope's OOM-kill poll wakes every 20 ms for the whole life of a capped command, breaking PATTERN-daemon-wakeups' one-second floor
    fix: In internal/proc/memorycap_linux.go, stop polling memory.events at oomKillPollPeriod (20 ms) in the goroutine runMemoryCapped starts. Either wake on the kernel's modify event of the scope's memory.events through internal/fswatch, the one file-event source, with a final read once wait returns, or raise oomKillPollPeriod to the one-second floor and keep the final read after wait. A one-second poll can miss a kill in the last second before the scope is removed; if that loss is not acceptable, use the fswatch route. If the loop stays a timer, add internal/proc/memorycap_linux.go to unboundedWaitFiles in cmd/lym/daemonwakeups_test.go and add a bullet for it to pattern/PATTERN-daemon-wakeups.md's What counts list.
  - id: F2
    severity: MEDIUM
    class: design
    location: internal/proc/memorycap_linux.go:164
    summary: scopeCgroup's one-second read deadline now runs while systemd-run is still starting the scope, so a slow scope start reads as a scope that never ran the command
    fix: In runMemoryCapped (internal/proc/memorycap_linux.go), read the scope's cgroup listing without a deadline while the command runs. The read ends at EOF once the shell closes fd 3 or systemd-run exits, and the wait's end closes every writer anyway. Keep a bound only on a read that happens after wait has returned. Update the doc comments of markerReadBound and scopeCgroup to match.
  - id: F3
    severity: MEDIUM
    class: design
    location: internal/gatecli/memoryguard.go:276
    summary: when the guard kills an inner test binary that a RunAlone binary started, guardKilled never names the outer pid, so the outer run's failure counts as a mutant kill, sets FailsUnmutated or reads as an ordinary failure
    fix: In startMemoryGuard (internal/gatecli/memoryguard.go), note as killed both the killed binary's pid and the pid of every Go test binary that is its ancestor in the sample the kill came from. For example, pass killTree the sample's parent links and walk Parent from the killed binary, noting each process whose Path ends in testBinarySuffix. Then RunAlone's GuardKilled(cmd.Process.Pid) is true for an outer binary whose inner binary the guard killed. Add a row to TestStartMemoryGuard whose tree has an outer test binary holding an inner one over the cap, asserting guardKilled is true for both pids.
  - id: F4
    severity: LOW
    class: design
    location: internal/proc/memorycap_linux.go:203
    summary: scopeHadOOMKill reads memory.events oom_kill, which counts kills by any OOM killer, so a kill at an enclosing slice's limit or a global OOM is reported as a kill at the tier's memory_mb
    fix: In scopeHadOOMKill (internal/proc/memorycap_linux.go), report a kill at the cap only when the scope's memory.events shows both oom_kill above 0 and oom above 0. The oom counter is raised on the cgroup whose own limit ran out, and on its ancestors, never on a child of the cgroup that hit its limit. Update the doc comments of scopeHadOOMKill and runMemoryCapped to say the count is of kills at the scope's own limit.
  - id: F5
    severity: MEDIUM
    class: consistency
    location: _lym/landing/summary.md:47
    summary: the change description makes three claims the diff contradicts, about the redundancy report step's guard, about naming the killed test, and about the absence of a real memory_mb runaway test
    fix: In _lym/landing/summary.md, replace the design decision that says the redundancy report step of lym gate test runs no guard of its own with what run.go does. run.go starts the guard over the gate call's own process for the step and hands the step its guardKilled. Replace the decision that says the guard names the test only through -test.run, so a tier run without -run gets only the package. A go-json tier's log and causes now also name the tests that started and never ended. Remove the To check by hand claim that no test drives a real runaway through a tier's memory_mb. TestRunTierMemoryCapKillsARunawayCommandAndNamesTheCap in internal/gatecli/tierrun_memorycap_linux_test.go does that.
  - id: F6
    severity: LOW
    class: consistency
    location: internal/gatecli/doc.go:723
    summary: doc.go and docs/overview.md say a test run the guard kills is not judged, but lym gate testmap records it as a failed entry
    fix: In internal/gatecli/doc.go:723, say that a test run the guard kills in the redundancy report step or lym gate redundant is not judged, a mutant run it kills counts as no mutant kill, and lym gate testmap records the test's entry as failed. Make the same change to the clause in docs/overview.md:969.
  - id: F7
    severity: LOW
    class: consistency
    location: internal/gatecli/doc.go:730
    summary: doc.go and template.yaml say memory_mb bounds a tier's builds and test binaries together and that a kill at it prints the tier's cap line, but on a host without a capping user manager each process gets memory_mb alone, nothing is killed and no cap line prints
    fix: In internal/gatecli/doc.go, after the sentences at 730-731, add that on a host whose user manager cannot cap a scope, the cap limits each process's data segment to memory_mb alone. An allocation past it fails in that process, and no cap line is printed. Add the same qualification to the memory_mb clause of internal/gateslot/template.yaml's tier key comment.
---

### [BLOCKING] OOM-kill poll wakes every 20 ms for the life of a capped command

**Location:** internal/proc/memorycap_linux.go:50, the loop at 170-176

**Issue:** Commit 4e7e4ee2 added a goroutine to `runMemoryCapped` that reads the scope's `memory.events` every `oomKillPollPeriod`, 20 ms, until the command's wait returns.
Under a tier's `memory_mb`, that command is the tier's whole shell, which can run for its `timeout_min`, an hour or more.
So the loop is a wait with no deadline of its own that lives as long as a run.
That makes it an unbounded wait in `PATTERN-daemon-wakeups`' sense, whose rule is a one-second floor on any periodic wake.
The loop wakes 50 times a second.
`memorycap_linux.go` is not in `unboundedWaitFiles`, so the scan's literal check never sees it.
The pattern's Bound section says such a loop is caught by review.

### [MEDIUM] A slow scope start now reads as a scope that never ran the command

**Location:** internal/proc/memorycap_linux.go:164, with `markerReadBound` at 53 and `scopeCgroup` at 219-226

**Issue:** Before this branch, `scopeCgroup` ran after `cmd.Wait()`, when every writer of the start pipe had closed.
Its one-second read deadline could not trip then.
The goroutine now calls it right after `start`, while `systemd-run` is still asking the user manager over D-Bus for the transient scope.
If that takes more than `markerReadBound`, the read fails and `cgroupFound` gets `""`.
A user manager slows down most when the machine is short of memory, which is when the cap matters.
The command still runs to its end, but `runMemoryCapped` then returns `ErrMemoryCap` with "the scope did not run the command".
`runInvocations` turns that into `tierMemoryCapRefusal`, whose way forward is to remove `memory_mb`, for a tier that ran and may have passed.
In `RunAlone`, `lym gate redundant` and `lym gate testmap` read it as a binary that did not start.
The OOM-kill poll also never starts in that case, so a kill at the cap goes unreported.

### [MEDIUM] A guard kill of an inner test binary leaves the outer RunAlone run judged

**Location:** internal/gatecli/memoryguard.go:276 (`onKill` notes only the killed binary), memoryguard.go:161-171, internal/testcover/runner.go:187

**Issue:** `kills` spares a test binary whose tree holds another tree over `memory_quit_mb` and kills the inner binary instead, as the row "an inner test binary over the cap is killed, not the outer one" pins.
`onKill` records only the inner pid.
`RunAlone` asks `GuardKilled(cmd.Process.Pid)`, the outer binary's pid, which reads false.
So the outer test's failure, caused only by its child's death, is a plain `ActionFail`.
In `runMutants` that is a kill of the mutant when a mutant is on, and `FailsUnmutated` when none is.
In the redundancy report step it is a failing test, and in testmap a failed entry with no guard warning.
Lym's own tests start nested `go test` binaries, so this case is real.
It contradicts the docs' claim that a mutant run the guard kills counts as no mutant kill, and focus entry 2 asks about exactly this.

### [LOW] The cap line attributes any OOM kill in the scope to memory_mb

**Location:** internal/proc/memorycap_linux.go:203-215

**Issue:** The cgroup v2 `memory.events` field `oom_kill` counts processes of the cgroup "killed by any kind of OOM killer".
If an enclosing slice reaches its `memory.max`, or the host runs out, and the kernel picks a process in the scope, the count rises.
`runInvocations` then prints `tierMemoryCapLine`, which says the tier held more than its `memory_mb` and tells the operator to fix the package or raise `memory_mb`.
Neither is the cause.
Focus entry 1 asks whether the kill is attributed from the scope's OOM-kill count.
It is, and that count does not separate a kill at the scope's own limit from any other OOM kill.

### [MEDIUM] Change description contradicts the diff

**Location:** _lym/landing/summary.md:47-48, 42-44, 53-55

**Issue:** The summary says the redundancy report step of `lym gate test` runs no guard of its own.
But `run.go` wraps `step.run` in `startMemoryGuard` and hands the step `guardKilled`, and `doc.go`, `docs/overview.md` and `template.yaml` all say the step runs the guard.
It says the guard names the test only through `-test.run`, so a tier run without `-run` gets the package and not the test.
`unfinishedTestsLine` now names the tests a memory kill left running in the log and causes.
It says no test drives a real runaway through a tier's `memory_mb`.
`TestRunTierMemoryCapKillsARunawayCommandAndNamesTheCap` does that through a real scope.

### [LOW] "Not judged" is wrong for lym gate testmap

**Location:** internal/gatecli/doc.go:723, docs/overview.md:969

**Issue:** Both say a test run the guard kills under `lym gate redundant`, `lym gate testmap` and the report step is not judged.
`testMapRun.runTest` (`internal/gatecli/testmappackage.go`) records a guard-killed test as `testcover.OutcomeFailed`, the same entry an ordinary failure writes.
Only the redundancy paths list it as not judged.

### [LOW] memory_mb docs do not hold on the data-segment fallback

**Location:** internal/gatecli/doc.go:730-731, internal/gateslot/template.yaml:15

**Issue:** On a host where `hostMemoryScope` finds no capping user manager, `runMemoryCapped` falls back to `runDataLimited`.
That fallback sets `RLIMIT_DATA` to `memory_mb` on the shell, and every process it starts inherits the limit as its own.
So the builds and test binaries are not bounded together.
An allocation past the limit fails in its process, `treeKilled` stays false, and no tier cap line is printed.
`doc.go` says the cap holds them together and that a kill at it fails the run with the tier's line.
`template.yaml` says the kernel kills the largest of them.
Neither names the fallback, although `proc`'s own docs do.
