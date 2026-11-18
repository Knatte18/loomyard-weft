---
verdict: BLOCKING
findings:
  - id: F8
    severity: BLOCKING
    class: consistency
    location: internal/proc/memorycap_linux_test.go:178
    summary: Retained contained tests lack the keep explanations required by PATTERN-test-speed.
    fix: In memorycap_linux_test.go add a //testtiming:keep directive directly above TestRunMemoryCapped_TheHolderEndsOnceACappedRunWithNoKillReturns explaining that it pins scope removal after successful completion. Add one above TestRunMemoryCapped_AScopeWithoutSetsidIsAnErrorAndNeverRunsTheCommand explaining that it pins refusal when the cap is valid but the holder dependency is unavailable. Retain both behavioral checks.
    origin: lens:code-tests
  - id: F9
    severity: BLOCKING
    class: consistency
    location: internal/proc/memorycap_linux_test.go:185
    summary: The new holder cleanup test repeats the cgroup paths already owned by package constants.
    fix: Expose selfCgroupFile and cgroupMount through test-only constant aliases in internal/proc/export_linux_test.go, then use those exports instead of the path literals in the cat command and filepath.Join in TestRunMemoryCapped_TheHolderEndsOnceACappedRunWithNoKillReturns.
    origin: lens:code-affected-set
  - id: F5
    severity: LOW
    class: design
    location: internal/proc/memorycap_linux.go:61
    summary: The command can signal its group before the asynchronous holder has entered its separate session.
    fix: Synchronize holder readiness in scopeStartScript before executing the command. Have the holder acknowledge startup only after setsid succeeds and its reader is ready; refuse startup if that acknowledgement fails. Preserve reparenting and detached stdio. Extend the holder behavior test to cover a command-group signal after command startup while the command survives.
    origin: lens:code-correctness
    carries: F5
prior:
  - id: F1
    status: resolved
    answer: The platform-neutral memorylimit.go owns cgroupMemoryMaxFile and both production readers use it.
  - id: F2
    status: resolved
    answer: The gate hog has fixed 256 MB input and a tier timeout; the stale sleep comments are gone.
  - id: F3
    status: resolved
    answer: Both named gate JSON records confirm the stated successful repetition count and approximately 16.1 seconds of overlap.
  - id: F4
    status: resolved
    answer: The new scope-only test waits for removal of the scope after an ordinary successful run.
  - id: F5
    status: open
    answer: Starting setsid asynchronously does not establish session isolation before the command starts.
  - id: F6
    status: resolved
    answer: TestRunMemoryCapped now includes the within-cap child under testMemoryCap plus one byte.
  - id: F7
    status: resolved
    answer: The new scope-only children test asserts an empty child list and a successful command.
---

### [BLOCKING] Explain retained contained tests

**Location:** internal/proc/memorycap_linux_test.go:178

**Issue:** The current redundancy report lists the holder-lifetime and missing-setsid tests as contained.
The holder-lifetime test asserts scope disappearance, which its covering children test does not assert.
The missing-setsid test exercises refusal after cap verification succeeds;
the covering cap-mismatch and children tests do not establish that refusal.
These are useful checks, but neither carries the keep explanation required by PATTERN-test-speed and the rubric.
The second affected declaration is at line 282.

### [BLOCKING] Reuse the existing cgroup path constants

**Location:** internal/proc/memorycap_linux_test.go:185

**Issue:** The new test repeats /proc/self/cgroup here and /sys/fs/cgroup at line 194.
internal/proc/tasklimit.go already declares both through selfCgroupFile and cgroupMount.
PATTERN-code-conventions requires a single declaration referenced everywhere.
The prior F4 prescription named literal paths, but the review instructions give repository constraints precedence.

### [LOW] Complete holder isolation before starting the command

**Location:** internal/proc/memorycap_linux.go:61

**Issue:** The intermediate shell starts setsid in the background and exits without waiting for session creation.
The outer shell can execute the command while the holder still belongs to the original process group.
A command that survives TERM and sends TERM to its group can kill the holder in that interval.
When that command later exits, the scope can disappear before the event read at line 194.
This leaves the process-group portion of prior F5 open despite the added setsid invocation.

## Validation

The recorded proc run 20261118T142443Z-tier1-444250 reports 950 passes, no failures and no skips for 50 repetitions.
It spans 15:24:25.305722506 to 15:24:43.427392354 local.
The successful gatecli run 20261118T142527Z-tier1-444438 starts at 15:24:27.325958680 local,
and it ends at 15:25:27.029447520 local.
Their overlap supports the summary's concurrency claim.

A fresh targeted gate attempt could not acquire a slot because the sandbox makes the hub's gate records read-only.
It ran no tests; no build or test failure is asserted by this review.

## Rejected

- A separate new coverage finding for holder session identity is below the incremental round's severity threshold.
  The surviving startup defect is carried under F5 instead.
