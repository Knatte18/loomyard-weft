---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/reedcli/watchdog.go:624
    summary: "The OOM step runs inline in the discovery loop, and each session's Engine.Status takes that worktree's reed op lock with a blocking flock, so one long AddStrand or Resume stalls discovery, reaping and the binary check, and the journalctl read (up to 10 s) and the creel post add to the stall"
    fix: "Move the per-session OOM check out of the discovery loop: run it in each session's own goroutine, as enterSession already does for name repair, started for every entered session (the `watchdog: off` ones included), with its own backed-off timer. If it stays in the loop, have reportPaneOOMKills read the strands with a non-blocking op lock (lock.TryAcquireWriteLock, through an engine method that skips when the lock is held), and run the journalctl read and the NotifyAsker post off-loop. Update step 7 of the runWatchdogLoop doc to match."
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/reedcli/oomnotice.go:55
    summary: "planPaneOOMNotices keys kills by whatever cgroup a pane's shell sits in, so an uncapped pane that shares a cgroup with other panes or the tmux server gets a kill attributed to the first strand that reads it, and that scope's earlier kills are told as kills at a memory cap that never applied"
    fix: "In planPaneOOMNotices, skip a pane whose cgroup's base name does not start with the pane scope prefix (`lym-pane-`), so only capped panes are counted. Export the prefix once from reedengine, for example as `reedengine.PaneScopeUnitPrefix`, and reference it from both panememory.go and oomnotice.go. Add a row to TestPlanPaneOOMNotices with a pane whose cgroup is a `tmux-spawn-` scope holding kills, and assert that it gets no notice."
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/reedcli/oomnotice.go:84
    summary: "paneOOMNoticeLine names the last newKills journal entries whenever the journal holds that many, but the journal is cumulative and lags the memory.events counter, so a new kill read before its kernel line lands is named with an earlier, already-told kill's process and pid"
    fix: "Track, per cgroup in watchedSession, how many journal entries earlier notices already named (beside the kill count in paneOOMKills, for example as a struct value {kills, named int}). In paneOOMNoticeLine, name only entries past that told count, and fall back to the count-only form when fewer than newKills new entries exist. Add a TestPaneOOMNoticeLine row where the journal holds only an already-told kill and a new kill is counted, and assert the count-only line."
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/reedcli/oomnotice.go:101
    summary: "No test shows a new pane OOM kill becoming a creel notice to the parent orch; reportPaneOOMKills, its call in runWatchdogLoop and the oomKills map enterSession sets are reached by no assertion, although the Goal's fourth bullet asks for exactly this delivery"
    fix: "Give reportPaneOOMKills its three boundaries as parameters or fields: the strand reader (Status), the kill readers (proc.ProcessOOMKills and proc.OOMKillsInCgroup) and the poster (orchcli.NotifyAsker), with production passing today's functions. Add one tier0 test in internal/reedcli that feeds a strand with a new kill through fakes and asserts the poster is called once with program REED and the exact line, and that a second call with the same counts posts nothing."
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/reedengine/panememory.go:33
    summary: "No test pins the degraded path the Goal's third bullet asks for, a pane that cannot be capped still launching with a Warn naming the cause; the mutants at panememory.go:35 and :40 survive"
    fix: "Add a tier0 row to an existing launchStrandLocked test in internal/reedengine, with the fake tmux scripted to answer `display-message ... #{pane_pid}` with an error and then with a non-numeric answer. Capture the log through internal/testkit/logcapture and assert the Warn line for each cause, and that the launch still sends the launch keys."
  - id: F6
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/proc/processscope_linux.go:151
    summary: "busctl's StartTransientUnit returns once the job is queued, so a job the user manager fails later, such as one that cannot move a pid outside its tree, is seen only as a 5 s waitForScope timeout under the reed op lock, with a warning that names the symptom and not the cause"
    fix: "In waitForScope, also read the unit's state on each poll (busctl --user get-property on the unit's ActiveState and Result, or the job's state), and return at once with an ErrMemoryCap error naming the failed result when the unit is failed or inactive. Log that spawn per the caller's existing logging."
  - id: F7
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/reedcli/oomnotice.go:69
    summary: "planPaneOOMNotices advances seen before reportPaneOOMKills posts, so a NotifyAsker failure loses that kill's notice for good"
    fix: "Have planPaneOOMNotices leave seen at the old count for a pane with a notice, and have reportPaneOOMKills set seen[notice.cgroup] to the new count only after NotifyAsker succeeds. Adjust TestPlanPaneOOMNotices's wantSeen rows to the new contract."
  - id: F8
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/reedengine/panememory.go:46
    summary: "The busctl spawn on each pane launch is a lifecycle spawn, but it is logged at Debug; PATTERN-child-processes sets Info for a lifecycle spawn and Debug only inside a polling probe"
    fix: "Change the logger.Debug call announcing the busctl move in capPaneMemoryLocked to logger.Info."
  - id: F9
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/reedengine/panememory.go:20
    summary: "No test shows a hub's own pane_memory_max_mib reaching the cap in bytes, or a zero or negative value taking the 8192 MiB default; only the template default is asserted"
    fix: "Add one tier0 table-driven test of the cap's bytes for a Config with pane_memory_max_mib 1024, 0 and -1, expecting 1073741824, 8589934592 and 8589934592 as literals. Drive it through the surface the engine uses, or an export_test.go hook for paneMemoryMax if no surface reaches it at tier0."
  - id: F10
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/proc/processscope_linux_test.go:343
    summary: "The Goal names MemorySwapMax, yet no test reads the scope's memory.swap.max, so the mutant of the swap value at processscope_linux.go:82 survives"
    fix: "In TestCapProcessMemory_KillsARunawayChildInTheScopeAndTheProcessLivesOn, beside the memory.max check, read memory.swap.max from the same cgroup directory and assert it reads exactly 0."
  - id: F11
    severity: NIT
    class: consistency
    origin: lens:code-affected-set
    location: internal/reedengine/config_test.go:40
    summary: "The //testtiming:keep reason on TestLoadConfig_TemplateDefaultsResolve lists every shipped template default it pins, but leaves out pane_memory_max_mib 8192, which the test now asserts"
    fix: "Add `pane_memory_max_mib 8192` to the list in the //testtiming:keep reason above TestLoadConfig_TemplateDefaultsResolve."
  - id: F12
    severity: NIT
    class: design
    origin: lens:code-tests
    location: internal/proc/processscope_linux_test.go:390
    summary: "TestCapProcessMemory_AnEndedProcessIsACapError asserts only errors.Is ErrMemoryCap, so it passes whichever branch returns the error; the mutants at processscope_linux.go:47 and :52 survive"
    fix: "Also assert that the error text holds `refused the scope`, the busctl refusal path an ended pid takes."
---

The diff does what the Goal asks.
Every strand launch goes through `launchStrandLocked`, which now moves the pane's shell into a `lym-pane-<pid>.scope` with `MemoryMax`, `MemorySwapMax=0` and `OOMPolicy=continue`.
The cap is a reed config key with a default, and a pane the cap cannot reach launches with a Warn.
The watchdog posts a REED notice through `orchcli.NotifyAsker`.
I checked the busctl call live on this host: the scope held the given `memory.max` and `memory.swap.max` read 0.
Every finding is MEDIUM or below, so the verdict is APPROVED.
The OOM notice path still has correctness gaps and no wiring test, and the fixer should close those gaps.

### [MEDIUM] The OOM step can stall the watchdog's discovery loop

**Location:** `internal/reedcli/watchdog.go:624`

**Issue:** step 7 calls `reportPaneOOMKills` inline for every known session.
That calls `Engine.Status`, which goes through `withOpLock` and `lock.AcquireWriteLock`, a blocking `flock`.
A long reed operation on any worktree therefore holds the loop, and so do the journalctl read and the creel post.
While it waits, the loop does no discovery or reaping, makes no binary check and does not read `ctx.Done()`.
The loop's own doc runs reaps off-loop for exactly this reason.

### [MEDIUM] A kill in a shared, uncapped cgroup is attributed to the wrong strand

**Location:** `internal/reedcli/oomnotice.go:55`

**Issue:** the planner reads any pane's cgroup.
A pane the cap missed can share its cgroup with other panes or with the tmux server.
The first strand read takes the notice, and the cgroup's earlier kills are counted from 0 and told as "at the memory cap of strand X".

### [MEDIUM] The notice can name an earlier kill as the new one

**Location:** `internal/reedcli/oomnotice.go:84`

**Issue:** the journal list is cumulative and can lag the `oom_kill` counter.
With one kill already told and one new kill whose line has not landed yet, the journal holds one entry, and the line names the old kill's process and pid.

### [MEDIUM] No test of the notice reaching the parent orch

**Location:** `internal/reedcli/oomnotice.go:101`

**Issue:** `TestPlanPaneOOMNotices` and `TestPaneOOMNoticeLine` pin the planner and the line.
No test reaches `reportPaneOOMKills` or its step-7 call, so a broken wiring passes every test.
A dropped call, a nil `oomKills` map, or a wrong signer or receiver would all go unseen.

### [MEDIUM] No test of the degraded launch's warning

**Location:** `internal/reedengine/panememory.go:33`

**Issue:** the tier0 launch tests run the uncapped path only because the fake tmux answers `display-message` with nothing usable.
No test asserts the Warn or its cause, and the mutants at lines 35 and 40 survive.

### [LOW] A failed scope job is seen only as a 5 s timeout

**Location:** `internal/proc/processscope_linux.go:151`

**Issue:** busctl returns once the job is queued.
If the user manager then fails to move the pid, `waitForScope` polls until `scopeJoinTimeout` while the reed op lock is held.
The Warn then reports that the pid was still in its old cgroup, not why the job failed.

### [LOW] A failed post loses the notice

**Location:** `internal/reedcli/oomnotice.go:69`

**Issue:** `seen` is advanced before `NotifyAsker` runs, so a post error leaves the kill told to no one.

### [LOW] The busctl spawn is logged at Debug

**Location:** `internal/reedengine/panememory.go:46`

**Issue:** a spawn on each pane launch is a lifecycle spawn, which PATTERN-child-processes logs at Info.

### [LOW] No test of a hub's own cap value

**Location:** `internal/reedengine/panememory.go:20`

**Issue:** only the template's 8192 is asserted.
A non-default value, and the floor for zero or a negative value, have no test.

### [LOW] MemorySwapMax is not asserted

**Location:** `internal/proc/processscope_linux_test.go:343`

**Issue:** the capped-process test reads `memory.max` but not `memory.swap.max`, so the survivor at `processscope_linux.go:82` stands.

### [NIT] The keep reason misses the new default

**Location:** `internal/reedengine/config_test.go:40`

**Issue:** the reason lists each pinned default, but `pane_memory_max_mib` is missing from it.

### [NIT] The ended-process test cannot tell its branch

**Location:** `internal/proc/processscope_linux_test.go:390`

**Issue:** asserting only `ErrMemoryCap` lets the LookPath and busctl-refusal branches swap without a failure.

## Rejected

- code-affected-set F1, "Reed never imports creel": `reedcli` already reached creel before this diff, through `hubgeom`, `orchengine` and `pairteardown`. `go list -deps` on each package that `reedcli` imported at the merge base shows it. The new `orchcli` import adds no layering edge that was not already there.
- code-correctness, the `paneMemoryMax` overflow for a huge value: the value is the operator's hub config, and busctl's refusal of it already gives the Warn path. It was not filed by the lens either.
