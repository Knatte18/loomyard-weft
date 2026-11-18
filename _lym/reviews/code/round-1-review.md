---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    location: internal/proc/memorycap_linux.go:45
    summary: The new cgroupMemoryMaxFile constant names memory.max while internal/proc/memorylimit.go:63 keeps its own memory.max literal, so the name is declared twice, against PATTERN-code-conventions Named constants
    fix: Move the cgroupMemoryMaxFile declaration, with its doc comment, out of internal/proc/memorycap_linux.go into internal/proc/memorylimit.go, which builds on every platform. Replace the literal "memory.max" in readMemoryLevel at internal/proc/memorylimit.go:63 with cgroupMemoryMaxFile. Keep its use in scopeStartScript and in the ranUnderCap error of runMemoryCapped.
    origin: assist:assist
  - id: F2
    severity: MEDIUM
    class: scope
    location: internal/gatecli/tierrun_memorycap_linux_test.go:58
    summary: TestRunTierMemoryCapKillsARunawayCommandAndNamesTheCap still runs an unbounded cat /dev/zero | tail hog, against the board entry's rule that every test hog is bounded in size, and its comment about a background sleep keeping the scope alive describes the race the holder now closes
    fix: In TestRunTierMemoryCapKillsARunawayCommandAndNamesTheCap, replace the tier command with a hog bounded to a fixed size above the 64 MB memoryMB, for example a fixed-count head -c of /dev/zero piped into tail, so a cap that does not hold fails the test and never the machine. Drop the background sleep 300 and the two comment lines that say it keeps the scope alive for the OOM-kill read; keep the comment lines that explain why tail grows.
    origin: lens:code-affected-set
  - id: F3
    severity: MEDIUM
    class: consistency
    location: _lym/landing/summary.md:52
    summary: The summary says the 50 runs passed with tier-1 gate runs of other packages beside them, but the gate records show no run of any hub overlapping 20261118T140719Z-tier1-124652 (15:07:04 to 15:07:19 local), so the Done-when's other-gate-work condition is unproven
    fix: Run lym gate test --tier 1 ./internal/proc -- -count=50 -run TestRunMemoryCapped while another gate run, such as a tier-1 run of another package, runs at the same time, and check from both runs' json records that their start-to-end spans overlap. Then rewrite summary.md lines 52-54 to name the count, both run ids and their overlapping spans. If no overlapping run can be made, drop the beside-it claim.
    origin: handler
  - id: F4
    severity: MEDIUM
    class: design
    location: internal/proc/memorycap_linux.go:162
    summary: No test asserts that the scope holder ends once a capped run with no kill returns, so dropping release.Close or a holder that ignores end of file would leak a process and its scope on every run with every test passing
    fix: In internal/proc/memorycap_linux_test.go, add a scope-only test, skipped where MemoryCapCoversTree is false, whose capped command is cat /proc/self/cgroup with Stdout captured. Once RunMemoryCapped returns nil, take the cgroup v2 path from the output's line that opens with zero and two colons and poll /sys/fs/cgroup plus that path with a short tick and a one-minute deadline, failing when the directory still exists at the deadline.
    origin: lens:code-tests
  - id: F5
    severity: LOW
    class: design
    location: internal/proc/memorycap_linux.go:62
    summary: The holder stays in the command's process group, so gatecli's killLeftovers after RunMemoryCappedVia can find it before it reads end of file and log a leftover the command never left, and a command that signals its own group ends the holder before the memory.events read
    fix: In scopeStartScript, start the holder in a session of its own, for example by running read -r line <&4 under setsid inside the existing /dev/null subshell, so it shares no process group with the command. Update the scopeStartScript doc comment to say the holder leaves the command's process group as well as its tree.
    origin: lens:code-affected-set
  - id: F6
    severity: LOW
    class: design
    location: internal/proc/memorycap_linux.go:164
    summary: Every scope test passes the page-aligned testMemoryCap, so no test would fail if the round-down of wantMax broke and a legitimate cap that is not a page multiple were refused with ErrMemoryCap
    fix: In TestRunMemoryCapped in internal/proc/memorycap_test.go, or in a scope-only row of an existing linux test, run the hogWithinCap child under testMemoryCap plus 1 byte on a host where MemoryCapCoversTree is true and assert a nil error and no kill.
    origin: lens:code-tests
  - id: F7
    severity: LOW
    class: design
    location: internal/proc/memorycap_linux.go:53
    summary: No test pins that the holder is never a child of the command, the reason for starting it inside a subshell
    fix: Add a scope-only row to internal/proc/memorycap_linux_test.go whose capped command is sh -c 'exec cat /proc/self/task/*/children' with Stdout captured, so cat lists the children of the command's own pid, and assert the output is empty and the error is nil.
    origin: lens:code-tests
---

The diff fixes the root cause the summary names, and it fixes it correctly.
The holder keeps the scope until the parent reads `memory.events`, and the `memory.max` check stops a scope that is not held to the cap before the command runs.
I ran the start script under dash and bash for a mismatch, a match with spaced and empty arguments, and a listing with no `0::` line, and each ran as its comment says.
The verdict is BLOCKING on F1 alone.

### [BLOCKING] memory.max is now both a constant and a literal

**Location:** internal/proc/memorycap_linux.go:45

**Issue:** The diff adds `cgroupMemoryMaxFile = "memory.max"`.
`readMemoryLevel` at `internal/proc/memorylimit.go:63` in the same package still reads `filepath.Join(dir, "memory.max")`.
`PATTERN-code-conventions` says a name that can go stale is "one named constant, declared once and referenced everywhere, never a repeated literal".
`memorycap_linux.go` builds only on Linux and `memorylimit.go` builds on every platform, so the constant has to live in the platform-neutral file.

### [MEDIUM] The gate's own cap test still runs an unbounded hog

**Location:** internal/gatecli/tierrun_memorycap_linux_test.go:58

**Issue:** The tier command `sleep 300 >/dev/null 2>&1 & cat /dev/zero | tail` grows until something kills it.
That is the same `tail /dev/zero` shape that reached 11.4 GB on the day of the hang, and the board entry's Fix says every probe and test hog is bounded to a fixed number of MB.
Its comment says the background sleep keeps the scope alive so the OOM-kill count stays readable.
The holder now does that, so the comment describes a workaround for the race this diff closes.

### [MEDIUM] The "other gate work beside it" claim has no overlapping run

**Location:** _lym/landing/summary.md:52

**Issue:** `20261118T140719Z-tier1-124652.json` records 800 passed and 0 failed over 15:07:04.435 to 15:07:19.469 local time, so the count holds.
I searched the gate run records of every hub under `/home/knatte/Code` for a run whose span overlaps that window, and found only that run.
The nearest other run, `20261118T140721Z-tier1-130772`, starts at 15:07:19.772, after the 50 runs ended.
The Done-when asks for passes with other gate work running beside them, and the cited run does not show that.

### [MEDIUM] Nothing pins that the holder ends

**Location:** internal/proc/memorycap_linux.go:162

**Issue:** On a run with no kill, only the deferred `release.Close` ends the holder; `killScope` never runs.
`TestRunMemoryCapped/AChildWithinTheCapEndsOnItsOwn` and the other no-kill rows assert only return values and output.
If the close were dropped, each capped gate run would leave a scope and a process behind, and every test would still pass.

### [LOW] The holder shares the command's process group

**Location:** internal/proc/memorycap_linux.go:62

**Issue:** A background job of a non-interactive shell stays in the shell's process group, so the holder shares the command's group, which `StartTied` gives the command.
`runTiedChild` calls `killLeftovers` as soon as `RunMemoryCappedVia` returns.
`killTreePlatform` matches any live process in that group that started after the record, so it can find the holder before the holder reads end of file.
It then logs `gate test: killed what the shell left in its process group` for a leftover the command never left.
A command that runs `kill 0` also ends the holder before the parent reads `memory.events`, which reopens the race the diff closes for that command.

### [LOW] No test with a cap that is not a page multiple

**Location:** internal/proc/memorycap_linux.go:164

**Issue:** The round-down matches the kernel, which stores `bytes / PAGE_SIZE` pages and shows `pages * PAGE_SIZE`.
Every test passes `testMemoryCap = 128 << 20`, which is page-aligned, so a broken round-down would refuse unaligned caps with no test failing.

### [LOW] No test that the holder is never the command's child

**Location:** internal/proc/memorycap_linux.go:53

**Issue:** The subshell `( … & )` exists so the holder is reparented out of the shell's tree before `exec "$@"`.
No test command inspects its own children, so a holder started as a plain `&` child of the shell would pass every test.

## Rejected

- correctness lens, LOW, the subshell becomes a child of the command: `( … )` is a foreground subshell, and the shell waits for it to exit before `exec 4<&-; shift; exec "$@"`, so the command never inherits it. The affected-set lens reads it the same way.
- assist F1, BLOCKING, the RLIMIT_DATA fallback runs uncapped: `runDataLimited` is unchanged by the diff and documented at `doc.go:67` and `memorycap.go:24` as set once the command started. `MemoryCapCoversTree` reports it, so it is not silent. No production start returns only after the command has ended, and the `ESRCH` case it relies on needs one.
- assist F3, MEDIUM, the hog has no time bound: `hoardMemory` stops after a fixed count of 1 MB blocks and exits, and the gate's `-timeout=20m` bounds the test binary. The board entry's concern was a hog that grows without end, which this one does not.
- assist F4, F5, F6: merged into F3, F6 and F4 respectively.
