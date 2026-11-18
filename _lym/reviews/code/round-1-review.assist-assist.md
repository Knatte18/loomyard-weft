---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: internal/proc/memorycap_linux.go:148
    summary: The no-scope fallback can still run the entire command uncapped and return success
    fix: In runMemoryCapped and runDataLimited, enforce the fallback limit before the target can execute or fork; if pre-execution enforcement is unavailable, return ErrMemoryCap before invoking start. Cover the delayed-start callback on the no-scope path with a bounded regression row and update the fallback contract to match.
    origin: lens:code-correctness
  - id: F2
    severity: BLOCKING
    class: consistency
    location: internal/proc/memorycap_linux.go:45
    summary: The new memory.max constant leaves another production reader using an independent literal
    fix: Move cgroupMemoryMaxFile to an existing platform-neutral proc file such as memorylimit.go and use it in readMemoryLevel in place of its memory.max literal, retaining its use in the scope script.
    origin: lens:code-affected-set
  - id: F3
    severity: MEDIUM
    class: scope
    location: internal/proc/memorycap_linux_test.go:140
    summary: The added delayed-start hog has a size bound but no execution deadline
    fix: Give the hog-running test commands bounded contexts and group cancellation, including the shell created in the RunMemoryCappedVia table and its delayed-start row. Register cancellation cleanup so both the shell and its hog end on timeout or test failure, while retaining the fixed allocation size.
    origin: lens:code-tests
  - id: F4
    severity: MEDIUM
    class: scope
    location: _lym/landing/summary.md:53
    summary: The claimed concurrent repeated validation has no overlapping gate run in the recorded intervals
    fix: Run the repeated memory-cap tests while another gate workload actually overlaps them, retain both run records, and replace the concurrent-validation claim in the landing summary with the observed count and overlapping run identifiers and intervals.
    origin: lens:code-scope
  - id: F5
    severity: MEDIUM
    class: design
    location: internal/proc/memorycap_linux.go:164
    summary: No regression row covers accepting a legitimate non-page-multiple cap
    fix: Extend the existing successful capped-run test with a scope-backed row using testMemoryCap plus one byte and assert successful command execution. Keep the existing aligned row and skip the scope-specific row only when the host lacks the required user manager.
    origin: lens:code-tests
  - id: F6
    severity: MEDIUM
    class: design
    location: internal/proc/memorycap_linux.go:162
    summary: No test asserts that the new holder ends after an ordinary successful command
    fix: Extend the existing successful-run coverage to capture the command's scope and assert eventual removal of that scope or absence of its processes after the capped call returns. Use a bounded wait and failure cleanup so removing release.Close makes the assertion fail without leaving a holder behind.
    origin: lens:code-tests
---

### [BLOCKING] The fallback leaves the requested enforcement guarantee open

**Location:** internal/proc/memorycap_linux.go:148

**Issue:** The scope check does not cover the fallback selected when the user-manager probe fails.
`runDataLimited` invokes `start` before applying `Prlimit` at lines 270–275.
The delayed-start callback newly exercised by this diff can finish the command before returning.
`Prlimit` then returns `ESRCH`, which the fallback ignores, and the saved successful wait result is returned.
An immediate starter also permits the child to fork before the limit is installed.
This is an unimplemented part of the Goal's explicit requirement to prevent silent uncapped execution.
The correctness and scope lenses independently identified this defect; they are consolidated here.

### [BLOCKING] Share the new cgroup filename constant

**Location:** internal/proc/memorycap_linux.go:45

**Issue:** `internal/proc/memorylimit.go:63` still reads the same filename through a separate literal.
`PATTERN-code-conventions` requires one named constant referenced by every reader.
The new constant needs a platform-neutral home because the existing reader is not Linux-only.
The directive makes violations of this constraint blocking regardless of runtime correctness.

### [MEDIUM] Bound the regression hog in time

**Location:** internal/proc/memorycap_linux_test.go:140

**Issue:** Both start variants wait without a deadline, including the newly added delayed-start row.
The shell uses `exec.Command`; so does `memoryHog` in `memorycap_test.go:33`.
`hoardMemory` in `testmain_test.go:30` bounds allocation size but contains no time bound.
Finite allocation work can stall under memory pressure and does not satisfy the Goal's separate time requirement.
The tests and scope lenses' findings describe the same omission.

### [MEDIUM] The concurrent validation claim needs evidence

**Location:** _lym/landing/summary.md:53

**Issue:** The cited gate record proves 50 repetitions and 800 passing results with no failures or skips.
Its interval is 15:07:04.435–15:07:19.469 local time.
An interval query over the recorded tier runs finds only that run overlapping this interval.
The next other-worktree gate begins at 15:07:19.772, after the repetitions ended.
The recorded run therefore does not establish the Goal's required validation beside other gate work.
This finding concerns the change description's claim and the missing validation, not the answer key.

### [MEDIUM] Pin non-page-aligned limits

**Location:** internal/proc/memorycap_linux.go:164

**Issue:** Every existing cap-test row passes the page-aligned `testMemoryCap`.
Removing the new rounding would leave those assertions passing while rejecting valid unaligned limits.
The rounding itself agrees with the kernel's byte-to-page division.
The missing regression is the finding, not an asserted rounding defect.

### [MEDIUM] Pin holder cleanup on success

**Location:** internal/proc/memorycap_linux.go:162

**Issue:** Successful commands bypass `killScope` and depend on deferred closure of `release` to end the holder.
Current assertions check return values, output, or selected command descendants, not this holder's departure.
The new delayed-observation row exercises an OOM path that also kills the scope.
It therefore cannot establish cleanup on ordinary success.

## Focus assessment

The shell's `read && comparison || refusal` grouping rejects unreadable and mismatched caps before command execution.
The `0::` case extracts the unified cgroup path; absent data normally makes the memory-limit read fail closed.
No supported kernel listing exposing a contrary case was found.
Bounded dash and bash probes preserved spaced, empty, and wildcard arguments after `shift`.
The expected maximum remains one quoted argument.

The first pipe failure starts nothing; the second closes the marker and deferred reader.
After `start` returns, the parent closes `marker` and `holderEnd` on success and failure.
Deferred `release.Close` covers normal return, start failure, cancellation, cap refusal, and wait errors.
The holder's standard streams are detached before its background read, avoiding an output-pipe wait cycle.
Scope cleanup sends a kill without waiting for the holder's read to complete.
No additional holder-lifetime implementation defect was established.

Linux divides the byte limit by page size, matching the downward mask used here.
The source is [page_counter_memparse](https://raw.githubusercontent.com/torvalds/linux/master/mm/page_counter.c).
Production callers examined by the affected-set lens use integer-megabyte limits.

The historical failure logs contain only the surviving shell's output, not the hog's completed-allocation message.
Together with the removed watcher code, they support the lost-observation explanation.
The previous boot's journal records `tail -8` at 11429 MB RSS, consistent with the separate pipeline-stage incident.
The description names the root cause and the holder-based fix.

The delayed-observation row and ignored-cap row cover different behaviors.
The advisory out-of-process result requires no `testtiming:keep` marker.
Searches found no stale watcher reference; `errors` and the memory-events filename remain used.
The changed documentation describes the scope path implemented by the diff.

## Verification limits

A bounded live scope probe could not connect to the systemd user bus in this review session.
No new cap test run is claimed; the repeated-run result above comes from its saved gate record.
The shell probes allocated no hog memory.
All four lens reports are saved separately with their returned text unchanged.
