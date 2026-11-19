---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: internal/gatecli/tierrun.go:75
    summary: A retry can accept files left by the first attempt and return a passing gate.
    fix: Preserve the pre-run untracked-file baseline across retryTier in runGreenAware and check the final result against it, or explicitly prevent a detected stray-file failure from being cleared by a retry. Keep the first failure code when applicable. Add a regression scenario whose first attempt leaves a file and reports a resource shortage and whose retry succeeds without removing it.
    origin: lens:code-correctness
  - id: F2
    severity: BLOCKING
    class: design
    location: internal/gatecli/tierrun.go:75
    summary: The new baseline probe adds real git subprocesses to existing tier0 tests.
    fix: Move TestRunTierProcessCap, TestRunTierMemoryGuard, TestRunTierStartFailureKeepsTheShellRefusal, TestRunTierNoTestNotice and TestRunTierCopies to tier1 files, keeping each file in one tier. Retain tier0 tests only for behavior exercised without subprocesses.
    origin: lens:code-affected-set
  - id: F3
    severity: BLOCKING
    class: consistency
    location: internal/gatecli/strayfiles_integration_test.go:26
    summary: The new test skips Windows by design instead of using a platform build constraint.
    fix: Add a !windows build constraint to strayfiles_integration_test.go and remove its runtime.GOOS skip and unused runtime import, as PATTERN-test-speed requires for an OS-specific test.
    origin: handler
  - id: F4
    severity: MEDIUM
    class: design
    location: internal/gatecli/strayfiles.go:57
    summary: A copies run loses the aggregate stray-file cause from its failure report.
    fix: Update copies.go addFailedRuns to retain aggregate stray-file failures and causes separately from individual copy results, without assigning ownership to a copy. Extend execution-level coverage with git-backed copies that all pass but leave a file, and copies where one already fails; assert the aggregate cause in the failure report and log and its diagnostic before the summary.
    origin: lens:code-correctness
  - id: F5
    severity: MEDIUM
    class: design
    location: internal/gatecli/strayfiles_integration_test.go:45
    summary: Eligibility and failed-listing branches have no distinguishing regression coverage.
    fix: Extend the stray-file behavior coverage for an unavailable initial baseline followed by a valid repository, a successful baseline followed by a listing failure, no invocations, a refusal after a successful baseline, and cancellation or timeout leaving a file. Assert retained exit and outcome fields and permitted diagnostics; use a controlled boundary to assert that no-invocation and refusal paths skip the intended listing rather than waiting on timing.
    origin: lens:code-tests
  - id: F6
    severity: LOW
    class: design
    location: internal/gatecli/strayfiles.go:74
    summary: Raw filenames can break the promised single-line diagnostic and make removal paths ambiguous.
    fix: Quote or escape displayed filenames in strayFilesLine while comparing their original bytes. Extend the existing behavior table with a filename containing a newline and a delimiter, asserting one diagnostic line and an unambiguous escaped filename.
    origin: lens:code-correctness
  - id: F7
    severity: LOW
    class: design
    location: internal/gatecli/strayfiles_integration_test.go:51
    summary: The truncation expectation repeats the production slice calculation.
    fix: Use a fixed fixture of f00 through f11 and literal expected names f00 through f09 with remainder 2 in the truncation row, instead of deriving fixture length and expected slice from strayFilesListed.
    origin: lens:code-tests
---

### [BLOCKING] Retry loses the original baseline

**Location:** internal/gatecli/tierrun.go:75

**Issue:** `runGreenAware` calls `runTier` inside `retryTier` in `internal/gatecli/run.go:628`.
Each attempt therefore takes a new baseline.
An attempt that leaves `a.lock` and exits after printing `cannot fork` fails the new check but still qualifies for retry.
The next attempt treats `a.lock` as pre-existing and can pass while leaving it in the tree.
`retryTier` returns that second result and discards the earlier stray-file cause.
The diagnostic itself can trigger this path when a new filename contains `cannot fork`,
because `failedOnShortage` searches the combined log.
This defeats the amended decision's required gate failure.

### [BLOCKING] Existing tier0 callers now invoke git

**Location:** internal/gatecli/tierrun.go:75

**Issue:** The affected tests in `tierrun_test.go` and `copies_test.go` call the wrapper with invocations.
The added baseline probe reaches `gitexec.Run` even when their temporary roots are outside repositories.
`PATTERN-test-speed` forbids subprocesses in tier0 and explicitly leaves indirect spawning to review.
Their existing shell usage does not authorize adding a git dependency.

### [BLOCKING] Runtime platform skip violates the testing rule

**Location:** internal/gatecli/strayfiles_integration_test.go:26

**Issue:** The new test calls `t.Skip` on Windows because its commands require a POSIX shell.
`pattern/PATTERN-test-speed.md`, under “No test skips by design,” requires a platform build constraint instead.
This is an explicit review-enforced repository rule.

### [MEDIUM] Copies lose aggregate failure reporting

**Location:** internal/gatecli/strayfiles.go:57

**Issue:** The check changes the aggregate exit code and causes after `mergeCopies` has completed.
`execution.report` sees the failure and calls `addFailedRuns`,
but that function selects only child copies whose exit code is nonzero.
If all copies passed, no failed run enters the report at all.
If one copy failed independently, its report still lacks the aggregate stray-file cause.
The immediate stderr line and aggregate log survive; the failure report does not.
The copies coverage gap reported by the tests lens is consolidated here.

### [MEDIUM] Missing checks for eligibility and failure paths

**Location:** internal/gatecli/strayfiles_integration_test.go:45

**Issue:** Every new row has an invocation and uses an uncancelled context.
The non-repository row makes both listings fail, so it cannot distinguish the two guards.
Existing refusal tests also start outside a repository, making `!listed` short-circuit the refusal guard.
No row pins the newly documented no-invocation, refusal or final-listing-failure behavior.
No row combines cancellation or timeout with a leftover file.
The successful integration rows do cover baseline acquisition;
the mutation report alone is not evidence that those rows lack assertions.

### [LOW] Diagnostic paths need escaping

**Location:** internal/gatecli/strayfiles.go:74

**Issue:** NUL-delimited parsing preserves a valid filename containing a newline,
but `strings.Join` emits that newline verbatim.
The promised single line becomes multiple lines, and commas in filenames resemble separators between paths.
The output no longer gives an unambiguous removal path.

### [LOW] Truncation expectation mirrors implementation

**Location:** internal/gatecli/strayfiles_integration_test.go:51

**Issue:** The fixture has `strayFilesListed+2` entries and expects `many[:strayFilesListed]`.
Production truncates with the same slice expression.
The rubric requires an independent expected value, which this row lacks for the displayed names.

## Other focus conclusions

The wrapper extraction and the copies call change serve the amended scope.
The omitted hardenshed fix is explicitly authorized by the parent's recorded amendment.
No commit carries a departure or loosened-check trailer, and no improvement comparison needs figures.
Prebuilds use the process temp directory or the hub's store; copies introduce no separate scratch write.
Gate records and locks use their existing cache paths, and this checkout ignores `.lym` and `.scratch`.
Run records are written after the comparison.
Concurrent writers can be blamed, as the package documentation expressly states.
Git lists paths under the supplied root and does not recursively inspect tracked submodule worktrees.
No amended requirement separately requests recursive submodule checking.
A failed second listing leaves the result unchanged, as the package documents; its missing test is F5.
Cancellation and timeout retain their existing nonzero codes when stray files are found.
Ordinary-run diagnostics precede `execution.report` and its summary.

## Validation

The committed range is `7346d6a8c8b38d0b26987969ac76b6b3843727c6..HEAD`.
The board entry was read through `lym board get` and agrees with the snapshot plus recorded amendment.
The focused gate attempt reported a read-only filesystem error registering its runner under the hub's runproc directory.
A foreground `--wait` attempt produced no result before it was interrupted.
No test pass is claimed; the findings above are grounded in the execution and reporting code.
