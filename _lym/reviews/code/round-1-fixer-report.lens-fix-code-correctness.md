```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the correctness of the changed code (`strayfiles.go`, `tierrun.go`, `run.go`, `copies.go`, `strayfiles_integration_test.go`) under the fix-code-correctness lens.

No findings that reach BLOCKING or MEDIUM. Each changed path, run with one concrete input:

- **`untrackedBeforeRun` and the empty-scope call.** `runListed`'s empty-scope `runTier` call gets `listed=false` because the run has no invocation, so it takes no listing and runs no check. The other `runTier` callers in `tierrun_test.go`, `copies_test.go` and the Linux memory-cap test keep the old signature and behave as before.
- **`runGreenAware`.** It takes one listing per tier run, after any shared build step, and every `retryTier` attempt shares it. With a first attempt of `touch r.lock; exit 1` and a passing rerun, the rerun still fails and names `r.lock`.
- **`--copies` run.** Two copies both run `touch c.lock`, so the merged exit is 0 until `checkStrayFiles` sets it to 1 and fills `strayLine`. `report` then calls `addFailedRuns`: no copy is red, and the `strayLine` branch appends one failed run labelled `runLabel(run)`. The line is in `causes` once, because the copies add none of their own.
- **`strayFilesLine`.** `strconv.Quote` turns the name `a\nb` into `"a\nb"` on one line. The truncation still cuts the raw sorted list before quoting.
- **Mutation checks, each by reading what the defect would do:**
  - **`TestRunTierStrayFilesSkipsARefusedRun`:** with the `result.refusal != ""` skip removed, `r.lock` would add a stray line to stderr and `causes`, and the test would fail.
  - **`TestRunTierCheckedHoldsEveryAttemptToTheFirstListing`:** with the listing taken per attempt, the second attempt would see `r.lock` as present and exit 0, and the test would fail.
  - **`TestRunTierCopiesStrayFile`:** without the `strayLine` branch, `exe.failed` would be empty, and the test would fail.
  - **`ANameWithANewlineStaysOneQuotedItem`:** with raw names, the stderr line would hold a real newline and fail the one-line check.

Remarks I did not rate as findings:

- **Duplicate line on rerun.** When a first attempt that left a file is then rerun, the stray line prints to stderr once per attempt. This is harmless duplication.
- **Failed run without module fields.** The copies stray-file failed run carries no `modulePath` or `moduleDir`, unlike `tierFailedRun`. The report takes these as unknown, which the struct doc allows.

Tool calls: none; the context already held every changed file and the gate result.
