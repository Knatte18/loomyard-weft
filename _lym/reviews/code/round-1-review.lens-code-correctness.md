```yaml
lens: code-correctness
outcome: reported
```

**Lens: code-correctness. For each function or branch the diff changes, the case I ran and whether it was correct.**

**List**
- `capLine` (`failurereport.go:244`): a 301-byte line whose byte 300 is a UTF-8 continuation byte. `cut` backs up to the rune start and the result is `line[:cut]+"..."`. Correct. The empty and 300-byte cases return the line unchanged. Correct.
- `renderFailureReport` (`failurereport.go:168`), nil and empty `runs`: returns nil. Correct.
- `renderFailureReport`, one run with no failures and no unparsed lines: prints the "log holds the cause" line. Correct.
- `renderFailureReport`, more than 80 body lines: truncates and appends "... N more lines; the logs named above hold them". The count is right, but the message is wrong for lint findings (F1).
- `renderFailureReport`, touched list over 20 files: names 20 and appends " and N more". Correct. An empty `touched` omits the line. Correct.
- `qualify` (`failurereport.go:228`):
  - Run dir equal to root with the package equal to the module: `dir == "."`, line unchanged. Correct.
  - Package in a subdirectory: the directory is prefixed. Correct.
  - Package outside the module path: unchanged. Correct.
  - A module path that is only a string prefix, such as `a/b` and `a/bc`: the `rest` guard rejects it. Correct.
- `tierFailedRun` (`failurereport.go:62`):
  - Timed-out run: outcome `timed out`. Correct.
  - Empty `RunDir`: module fields stay empty and no qualification happens. Correct.
- `buildFailedRun` (`failurereport.go:77`): empty output yields nil lines, so the "log holds the cause" line prints. Correct. More than 12 lines keeps the last 12. Correct.
- `runLints` and `lintFailedRun` (`failurereport.go:95`, `:122`):
  - Lint error: a failed run with outcome `could not run`, which sets exit 1 in `run.go`. Correct.
  - No findings and no error: no failed run. Correct.
  - One finding: outcome `1 finding`. Correct.
  - Comment lint returning partial findings together with an error: the error wins and the findings are dropped. Acceptable.
  - Unknown lint name: ignored silently. Unreachable, because `decodeGates` validates names.
- `touchedFiles` (`failurereport.go:144`):
  - Merge-base error: warns and falls back to HEAD. Correct.
  - `ChangedPaths` error, including a cancelled `runCtx` on the halt path: returns nil, so the touched line is silently absent. Acceptable, since the doc says it only orients the reader.
- `run.go:174`, lint loop:
  - Tiers passed and a lint fails: code 1, `firstFailed` is the lint label. Correct.
  - A tier failed and a lint fails: the code stays the tier's. Correct.
  - `runCtx` cancelled: lints are skipped. Correct.
  - A halt skips the lints and still prints the report. Correct.
- `runBuild` (`build.go:75`): the failed run is appended before `buildFailed` is returned. The tiers it blocks add no run, so each failure is listed once. Correct.
- `report` (`run.go:507`): appends a failed run when `ExitCode != 0`. This covers guard, vet, timeout and shortage runs. Correct.
- `goJSONCounter.observeTest` (`counters.go:183`):
  - Up to 8 non-status lines are kept per test and deleted on pass, skip or fail. Correct.
  - `failedPackages` is set on any test fail, subtests included. Correct.
- `observePackageOutcome` (`counters.go:216`):
  - `build-output` with an `ImportPath` goes to `buildLines`. Correct.
  - A package-level fail with no failed test (a `TestMain` exit, or a build failure through `FailedBuild`) adds a package failure with its own and its build's lines. Correct.
  - Package-level fail after failed tests: skipped, so nothing is duplicated. Correct.
  - Package-level output with `Test == ""` goes to `packageLines`. Correct.
  - A package fail with no `FailedBuild`: `buildLines[""]` is nil. Correct.
- `Failures()` (`counters.go:157`): a package failure (`test == ""`) is never treated as a parent, so it is kept. Correct.
- `handle` (`counters.go:348`): `observeTest` runs only when `Test != ""`, so a package-level fail event cannot create a test failure. Correct. Non-JSON lines are kept up to 12 and the status lines are filtered. Correct.
- `testFailures` (`tierrun.go:763`): drops package failures and uses `lines[0]` as the first line; an empty slice gives `""`. Correct, and it matches the old record shape.
- `replayStdout` (`detachfront.go:304`):
  - Missing file: nil. Correct.
  - Size of 4096 or less: copied. Correct.
  - Over 4096: a note on stderr, and the last line printed when it starts with `{`. The empty-after-trim case is safe, because `LastIndexByte` returns -1 and the slice starts at 0. Correct.
  - A failing capped call still carries its failures on the stderr file, which `replay` copies after. Correct.
- `decodeGates` (`config.go:365`): an unknown lint is refused with the fix-it clause. Correct. `Config.Gate` clones `Lints`. Correct.

**Findings**

- **F1, MEDIUM, design.** `failurereport.go:207-210`, with `run.go:174-183`. The cut line says "the logs named above hold them", but a lint step has no log (`failurereport.go:52`). Lint runs last, so the truncation cuts it first. A tier-failure flood past 80 lines, or many comment findings, drops lint findings that appear nowhere else. They are listed only in the report, and the cut line then points at logs that do not hold them. This breaks the "every failure in one go" goal. Evidence: `failedRun.log` is empty for a lint, and the lints are appended after the tier runs.
- **F2, LOW, design.** `failurereport.go:196-205`. A run's `unparsed` lines (a vet or build error from outside any test event) are printed only when the run has no `failures`. A run with a read failure plus a non-event error, for example a failed test in one invocation and a vet error in another, hides the vet error. The same gate applies to `causes`: the timeout and no-tests-ran lines from `tierrun.go:731-749` go into `unparsed`, so they are hidden whenever a failure was read. Evidence: the `switch` at `failurereport.go:196`.
- **F3, LOW, design.** `counters.go:351`, `:190`. Non-event lines and per-test lines keep the first 12 and first 8, so earlier noise such as `go: downloading` lines or `t.Log` output can push out the real error or the got and want lines. This is a best-effort limit, not a logic error.
- **F4, LOW, design.** `failurereport.go:228-241`. `qualify` assumes bare file names are relative to the package directory, which holds for test error lines. Compile errors in a package failure's lines are relative to the go command's working directory, which is the run directory, not the worktree root. When the run directory is a nested module, those `file:line` values are not worktree-relative, though the report promises that.

I looked at no other lens's area.

**Tool calls (1 of 6)**
1. Read `internal/gatecli/counters.go:345-394`. My context did not hold the `handle` body, which I needed to confirm that `observeTest` runs only for `Test != ""` and that the package-level events are routed correctly.
