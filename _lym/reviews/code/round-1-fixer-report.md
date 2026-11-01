# Round 1 fixer-report

## Changes

All ten findings fixed as proposed unless marked.
Commits: `44ca8d34` (F9, F10), `9278ee3a` (F2, F3, F4, F7, F8), `36b9079b` (F1, F5, F6), and one for the self-check findings below.
F1, F5 and F6 share `failurereport.go` and one commit; the per-finding commit rule could not split them.

- **F1** — `renderFailureReport` keeps each run's block whole in turn until the 80-line budget is spent, and `cutLine` closes a cut report naming the lines each run lost, pointing at the logs for a run with one and at `lym loom lint-<name>` for a lint step.
  Row added to the report-cap case: a tier flood followed by lint runs.
- **F2** — the shared-build stand-in prints `helperBuildFailureLine` on failure, and the existing failing-build scenario asserts the `build (exit 3), log …` run with that output line between the report header and the closing line.
- **F3** — two scenario rows: a failing tier with a failing lint (exit code is the tier's, lint named last), and a halted call (report before the closing line, no lint).
- **F4** — `TestTouchedFiles` over a hubforge hub with a pair: merge-base row and renamed-parent fallback row.
- **F5** — `tierResult` and `failedRun` carry `causes` apart from `unparsed`; causes always print. Rows added for a run with a failure and a cause, and a run with a cause alone.
- **F6** — `qualify` takes the `runFailure` and prefixes a run-directory-relative Go path (`./x.go`, `dir/x.go`, either separator) in a failure outside any test with the run's `moduleDir`. Row added.
- **F7** — `TestRunLints_PatternHeadroom` over a hub whose `PATTERN.md` grows past the headroom line.
- **F8** — `TestFrontReplay_NamesALongStdoutOnStderrInsteadOfPrintingIt`.
- **F9** — the four verb doc lines reworded.
- **F10** — plan spec line 336 reworded.

## Self-check

- fix-code-correctness: no findings.
- fix-affected-set: no findings.
- fix-consistency, LOW, `failureReportLines` comment says the logs hold the rest — fixed as proposed.
- fix-consistency, LOW, `renderFailureReport` doc names only bare file names — fixed as proposed.
- fix-consistency, LOW, stand-in role comment omits the printed failure line — fixed as proposed.
- fix-consistency, LOW, and fix-proposal, LOW (merged), the cut line is capped at 300 bytes and loses its lint hints — fixed: the cut line is no longer capped; the cap test grew to a case longer than the cap.
- fix-proposal, LOW, `runRelativeGoFile` ignores backslash paths — fixed: either separator matches and the prefix joins with `/`; row added.
- fix-proposal, LOW, a package-relative path with a directory in a no-test failure is wrongly prefixed — disputed, see below.

## Disputed

- **Self-check fix-proposal finding 3, ground `premise false`.**
  The finding rests on a hypothetical line (`testdata/x.go:3` printed package-relative outside any test) with no cited printed instance.
  The review's F6 premise is that the go command prints a build error's path relative to the run directory for a package that failed outside any test; the other lines such a failure carries are absolute (`/abs/z.go` row) or bare file names (qualified by package directory).
  The new rows pin the `/abs` and in-test boundaries.

## Deferred

Nothing deferred.
