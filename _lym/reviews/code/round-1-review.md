---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    origin: lens:code-affected-set
    location: internal/proc/memorycap_linux.go:47
    summary: scopeStartScript repeats the literal /proc/self/cgroup although selfCgroupFile already names that path, against PATTERN-code-conventions' named-constants rule
    fix: In internal/proc/memorycap_linux.go, build scopeStartScript from the constant instead of the literal, as one constant expression concatenating selfCgroupFile (declared in internal/proc/tasklimit.go) into the script text; take the script body from F5's fix, which also drops the external cat, so the result reads `const scopeStartScript = "while IFS= read -r line; do echo \"$line\"; done < " + selfCgroupFile + ` >&3; exec 3>&-; exec "$@"``. Leave the doc comment above it as it is.
  - id: F2
    severity: BLOCKING
    class: consistency
    origin: lens:code-tests
    location: internal/proc/memorycap_linux_test.go:75
    summary: The new test fails when the killed sleep is still in /proc 10 s after the call, a measured-time bound a loaded gate machine can reach, against PATTERN-test-speed's Time is injectable rule
    fix: In TestRunMemoryCapped_AKillAtTheCapEndsWhatTheChildLeftInItsScope, change the poll deadline from 10 * time.Second to time.Minute, the generous deadline PATTERN-test-speed names and the cancel test in the same file uses, and change the t.Fatalf message from "10 s after the kill" to "a minute after the kill".
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/proc/memorycap_linux.go:155
    summary: With OOMPolicy=continue an OOM kill of a process other than the command no longer ends the scope, and runMemoryCapped calls killScope only when the command itself died of SIGKILL, so the scope's other processes outlive the call
    fix: "In runMemoryCapped, after the scopeCgroup check, decide the leftover kill separately from killed: kill the scope (killScope(cgroup)) when the command died of SIGKILL or when the scope's cgroup v2 file memory.events, read at filepath.Join(cgroupMount, filepath.FromSlash(cgroup), \"memory.events\"), holds an oom_kill line whose count is above 0; a missing file means the scope is already gone and nothing is left. Name the file with a constant beside cgroupKillFile, and parse the oom_kill line in one small helper. Keep killed true only for the command's own SIGKILL, join a killScope error into err on both paths as now, and reword the runMemoryCapped doc line 117 and RunMemoryCapped's doc in internal/proc/memorycap.go:11 to say every process left in the scope is killed after any kill at the cap. Add a row or case to TestRunMemoryCapped_AKillAtTheCapEndsWhatTheChildLeftInItsScope's file in which the shell starts the background sleep, runs the hog as a child with a plain call rather than exec, and then exits 0, asserting killed false and that the sleep is gone, polled with the same minute deadline as F2."
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/proc/memorycap_linux.go:101
    summary: No test fails when -p OOMPolicy=continue is dropped, so the behavior the task exists for, a kill at the cap that leaves the scope running instead of failing it, is unpinned
    fix: "Add a tier1 test to internal/proc/memorycap_linux_test.go, skipped like its siblings where proc.MemoryCapCoversTree is false, that pins the policy through RunMemoryCapped's public surface: the capped command is sh -c 'hog; echo survived' with the memoryHog binary over the cap as a plain child (no exec), its stdout captured in a bytes.Buffer. Under OOMPolicy=continue the kernel kills only the hog and the shell prints survived and exits; under the default stop policy systemd stops the scope and the shell never prints it. Assert killed is false and the captured output is exactly survived followed by a newline. If F3's fix adds a case of that same shape, fold this assertion into it instead of adding a second test."
  - id: F5
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/proc/memorycap_linux.go:47
    summary: The start script now runs an external cat found through the caller's PATH, so a cmd.Env whose PATH lacks cat makes every capped run, successful ones too, read as a scope that never ran the command
    fix: Replace the external cat in scopeStartScript with shell builtins that copy the listing line by line, `while IFS= read -r line; do echo "$line"; done < <selfCgroupFile> >&3`, written through the constant as F1's fix shows, so the script depends on no PATH lookup before it execs the command. Update the doc line above it only if it names cat.
  - id: F6
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/proc/memorycap_linux.go:182
    summary: killScope writes cgroup.kill through os.WriteFile, which opens with O_CREATE and O_TRUNC, so on a kernel without cgroup.kill the open fails with a permission error and every cap kill there carries a spurious joined error
    fix: In killScope, open the kill file with os.OpenFile(killFile, os.O_WRONLY, 0), write []byte("1") to it and close it, returning the first error, so a missing file reads as fs.ErrNotExist and stays no error under the existing check; keep the ENODEV tolerance as is.
  - id: F7
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/proc/memorycap_linux_test.go:46
    summary: No assertion fails when killScope joins an error into a cap kill whose scope is already empty or gone, so its ErrNotExist and ENODEV tolerance is unpinned
    fix: In TestRunMemoryCapped_EndsAChildOverTheCapByTheHostsMeans, add to the row assertion that err is exactly an *exec.ExitError by type assertion (err.(*exec.ExitError), not errors.As), so a joined kill error on the AScopeReportsTheKernelsKill row, whose scope holds nothing after the kill, fails the row; report the mismatch in the existing message's form, naming the error's type.
---

### [BLOCKING] scopeStartScript repeats the path selfCgroupFile already names

**Location:** `internal/proc/memorycap_linux.go:47`

**Issue:** The script reads `/proc/self/cgroup` as a literal, while `selfCgroupFile` in `internal/proc/tasklimit.go:16` declares that path once for the package and `probeMemoryScope` uses it.
`PATTERN-code-conventions` says a path is "one named constant, declared once and referenced everywhere, never a repeated literal".
The constraint is a listed one, so the violation blocks however small it is.

### [BLOCKING] The new test bounds the leftover's end by 10 seconds of real time

**Location:** `internal/proc/memorycap_linux_test.go:75`

**Issue:** The poll fails the test when `/proc/<pid>` is still present 10 s after the call returns.
`PATTERN-test-speed` says no test compares a measured real time to a bound a loaded machine can reach, and it names a minute as the generous deadline.
The gate runs this test on the hosts whose memory pressure the cap exists for, and the cancel test in the same file already uses a minute.

### [MEDIUM] An OOM kill of a child leaves the scope's other processes running

**Location:** `internal/proc/memorycap_linux.go:155`

**Issue:** Before the diff the default `OOMPolicy=stop` stopped the whole scope after any OOM kill inside it.
With `continue`, the kernel's victim is the largest process in the scope's memcg, which can be a child the command started rather than the command.
The command then ends with no SIGKILL of its own, `runMemoryCapped` returns at line 157 without calling `killScope`, and every other process in the scope keeps running.
The new doc at `internal/proc/memorycap.go:11` promises that every process the command left in the scope is killed, which holds only when the command itself was the victim.

### [MEDIUM] Nothing pins the OOM policy the task exists for

**Location:** `internal/proc/memorycap_linux.go:101`

**Issue:** The board entry asks that a kill at the cap stop raising the desktop notice while the cap's tests keep working, and `-p OOMPolicy=continue` is the whole of that change.
Dropping it leaves every test green: `AScopeReportsTheKernelsKill` still sees SIGKILL, and the new leftover test still passes because the default `stop` policy kills the leftover sleep itself.
A command that outlives a child's OOM kill is the observable difference between the two policies at the public surface.

### [LOW] The start script depends on cat in the caller's PATH

**Location:** `internal/proc/memorycap_linux.go:47`

**Issue:** The old script wrote its marker with the builtin `echo`; the new one runs `cat`, looked up in the `PATH` of `cmd.Env`.
Where that `PATH` lacks `cat`, the shell writes nothing to fd 3 and still runs the command, so `scopeCgroup` returns false and a run that exited 0 is reported as a scope that never ran.

### [LOW] killScope's write creates and truncates

**Location:** `internal/proc/memorycap_linux.go:182`

**Issue:** `os.WriteFile` opens with `O_CREATE|O_TRUNC`.
On a kernel before 5.14, whose cgroups have no `cgroup.kill`, kernfs refuses the create with a permission error instead of reporting the file missing, so the error escapes the `ErrNotExist` check and is joined into every cap kill.

### [LOW] The vanished-scope tolerance is unpinned

**Location:** `internal/proc/memorycap_linux_test.go:46`

**Issue:** On every SIGKILL path `err` is already a non-nil `*exec.ExitError`, and the rows assert only `err == nil` and `killed`.
A `killScope` that stopped tolerating `ErrNotExist` or `ENODEV` would join an error into the result, and no row would fail.

## Rejected

- code-correctness finding 2, a leftover that holds the command's stdout keeps `cmd.Wait` blocked until `WaitDelay` before `killScope` runs: `RunMemoryCapped` promises `cmd.Run`'s semantics, every production caller sets a `WaitDelay`, and F3's fix moves no wait; the delay is bounded by the caller's own setting and is not a defect of this diff.
- The `scopeCgroup` read under focus item 1: no process holds fd 3 after the shell's `exec 3>&-`, a failed `systemd-run` closes it on exit, and an empty or partial listing reads as a scope that never ran, as the old one-byte read did; no stall and no wrong `ok`.
- Joining a `killScope` error into `err` under focus item 2: `errors.As` still finds the `*exec.ExitError`, `killed` stays true, and testcover keys its capped result on `killed`, so callers see only a longer error text.
- The new test's pid poll under focus item 4: a recycled pid or a zombie keeps `/proc/<pid>` present, which can only fail the test, never pass it falsely; the redundancy report lists the test as not judged, and no existing test covers a leftover process, so it is not redundant.
