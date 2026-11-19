```yaml
lens: code-affected-set
outcome: reported
```

Lens: code-affected-set. For each change the diff makes, I looked for everything that calls, reads, asserts or documents it.

**Per change, the set found and whether it is covered**

1. **`runTier` now lists the worktree's untracked files before and after the run, and fails the run on new ones** (`tierrun.go:71-87`).
   - Callers: `run.go:590` (`runListed`, the empty-scope path) is covered, because a run with no invocation takes no listing. `run.go:629` (`runGreenAware`, inside `retryTier`) is not covered; see finding 2.
   - Readers of `result.record.ExitCode`, `Outcome` and `causes`: `report` at `run.go:703-728` is covered for a run that ran once. It writes the record, prints the summary line after the stray line, and calls `addFailedRuns` → `tierFailedRun`, which carries `result.causes`. `failedOnShortage` and `runnerCrash` in `tierrun.go` are covered, since a failing run keeps its own code.
   - Tests calling `runTier`: `tierrun_test.go` (tier0, `root` set to a `t.TempDir()` outside any repository, so the listing fails and is logged), `tierrun_memorycap_linux_test.go` and `copies_test.go`. All are covered: none asserts empty logs or stderr in a way the skipped check would break.
   - Docs: `doc.go` `# Exit codes` (1479+) is covered; a stray failure on a passing run is exit 1. `doc.go` `# Failure report` (820+) says "A run lists the verb's own cause line". It holds for a run that ran once and not for `--copies`; see finding 1.
2. **`runTier` split into `runTier` and `runTierCommands`; `copies.go` calls `runTierCommands`.**
   - `runTierCopies` (`copies.go:94`) is covered. Tests that call `runTierCopies` go through `runTier` or call it directly, and are unaffected.
   - Readers of the merged copies result: `printStartedLines` is covered. `addFailedRuns` (`copies.go`) is not; see finding 1.
3. **New stderr, log and cause line from `strayFilesLine`.** It is read by the tier's log record, `stderr` and `causes`. No stencil or doc quotes it. Covered, apart from finding 1.
4. **`doc.go` `# Stray files`.** It matches the code, apart from the copies failure-report gap and the rerun gap below.
5. **Repository rules and commit messages.**
   - `gitexec.Run` is the allowed entry point (PATTERN-gitrepo-client-boundary). The cap is the named constant `strayFilesListed` (PATTERN-code-conventions). The test is tier1 and needs git, so `tiergate.Require(t, 1)` is right (PATTERN-test-speed). No PATTERN changes are declared, so none are owed.
   - Commit messages: no claim of agreement or improvement, and no `Pattern-Departure:` or `Loosened-Check:` trailer. The summary's "93 of 93" claim gives its figure.

**Findings**

1. **MEDIUM, design — `internal/gatecli/copies.go`, `addFailedRuns` (with `checkStrayFiles` at `strayfiles.go:57-62`).**
   A `--copies` run that fails only on stray files gets no entry in the failure report, and the stray cause is never listed there.
   - `checkStrayFiles` adds the line to the merged result's `causes` and sets the merged `record.ExitCode` to 1.
   - `report` then calls `addFailedRuns`, which goes through `result.copies` only. It skips each copy whose `record.ExitCode == 0` and uses `one.causes`, never the merged `causes`.
   - When every copy passed, the call exits 1 but the failure report names no failed run. When some copy failed, the stray cause is still dropped from the report.
   - This contradicts `doc.go` `# Failure report`: "A run lists the verb's own cause line… One read of the call's result then shows every failure."
   - Evidence: `copies.go` `addFailedRuns` (`for i, one := range result.copies { if one.record.ExitCode == 0 { continue } … }`), `run.go:722-723`, `strayfiles.go:57-62`.
2. **MEDIUM, design — `internal/gatecli/run.go:628-633` (`retryTier`, a caller of the changed `runTier`).**
   Each attempt of a rerun takes its own listing before it runs. A file the first attempt left is therefore in the rerun's first listing and is never flagged.
   - `retryTier` reruns after a resource-shortage marker or a runner crash. A rerun that passes then passes the whole tier, and the stray file stays in the worktree.
   - This breaks the doc's rule "A file the second listing holds and the first lacks fails the run" for the run as a whole.
   - The first attempt's stray line survives only in its own run record (`recordFirst`). That record is never reported as a failure.
   - Evidence: `tierrun.go:72-76` (the listing taken inside each `runTier` call), `tierrun.go:344-378` (`retryTier` calls `attempt()` twice and returns `retried`), `run.go:628-633`.

**Tool calls**

1. Searched for the tests that call `runTier` or `runTierCopies`, and for docs that mention untracked or stray files. My context lacked the test callers and the doc sites outside `gatecli`.
2. Read the tier table in `PATTERN-test-speed`, the `root` and tier of the existing `runTier` tests, and the section list of `doc.go`. My context lacked the existing tests' fixtures and the doc sections that might describe the changed behaviour.
3. Read `doc.go`'s `# Failure report` and `# Exit codes` sections and the `gatecli` `TestMain`. My context lacked the doc text that the changed exit code and causes must still match.
