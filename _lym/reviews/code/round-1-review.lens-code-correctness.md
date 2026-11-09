```yaml
lens: code-correctness
outcome: reported
```

**Task:** code-correctness lens. For each function or branch the diff adds or changes, I ran one concrete case.

**Per-function results (case, verdict)**

- `run`, red tier: writes `RedGate` + `Note`, judges nothing. Correct.
- `run`, no hub: writes `noteNoHub`. Correct. `location` is non-nil because `loadGate` already used `location.AnchorRel`.
- `run`, scoper `none`: note only, no judging. Correct.
- `run`, write fails: `logger.Warn`, the exit code is untouched because `run` returns nothing. Correct.
- `run`, ctx cancelled before the step: the `runCtx.Err() == nil` guard skips the step. Correct.
- `judge`, errors from module, merge base, `Open`, `ChangedPaths` or `changedTests`: each becomes an `Errors` line and the report is written. The exit code is untouched. Correct.
- `judge`, no parent: sets `Note`, which retries. Correct.
- `judge`, skip-on-match: the check reads `workingTree`, not HEAD. `retries()` blocks a skip on an error, a note, an unjudged package or `reasonBudgetSpent`. `ChangedPaths` includes untracked files, so a new untracked test file is taken. Correct.
- `changedTests`:
  - A moved file's unchanged tests are untaken because `testHashesAt` is keyed by package directory and name. Correct.
  - A deleted file is skipped because `tree.file` reports not-found. Correct.
  - A file with no current tests skips the base read. Correct.
  - Module root `"."` and a nested module are both handled. Correct.
- `dropReason`: tier2, tier3, undeclared tier, keep and spawn drops match D7 step 2. An undeclared tier with `Tier()` erroring is handled. Correct.
- `judgeTests` cap: `kept[limit:]` goes to not judged, in file-then-name order because `changedTests` sorts. Correct.
- `holdSlot`:
  - The slot is taken after the tiers release theirs, and the deadline is set after `Acquire`.
  - If `MkdirTemp` fails after `Acquire`, the deferred `close` still releases the lease.
  - An inherited slot gives a nil lease, and `HolderEnv` handles that.
  - Correct.
- `judgePackage`: `stopReason` is checked before the build and before each test. A build failure lists the whole package as not judged. Every unrun test, including a whole unbuilt package, is listed as not judged. Correct.
- `buildLym`: runs once, caches the error, and a failure becomes a `reasonLymFailed` line per test. Correct.
- `runTest`:
  - `RunAlone` removes the stale profile first, so a profile is never reused across tests.
  - A timeout gives `reasonTimedOut`.
  - `secondUnit` is set at both `testRequest{` construction sites (`cli.go`, `rungate.go`), so the timeout is never zero.
  - Correct.
- `judgeRuns`: the covers are map tests with unchanged hash, minus names in `ran`, plus the other judged runs through `JudgeAgainst`. A judged test is never its own cover. Correct.
- `testcover.JudgeAgainst`:
  - Kept runs count toward `counts`, are never marked contained (`i < len(runs)`), and are never removed.
  - Two judged tests covering only each other: only the slowest is removable, so the second comes out unique.
  - `Judge` is unchanged for `kept == nil`.
  - Correct.
- `CoveredKeys`: keys every package block, so ordinals match a map built from a whole profile. Duplicate profile lines merge. Correct.
- `ParseTestFuncs`: `File` is the base name, and `ScanTestFuncs` delegates to it. Correct.
- `mappedPackages`: no map, an unreadable map or a build-failure map each give a package line plus not-judged tests. A judged package gets its map commit, age, drift and file hashes. If hashing fails, there is an `Errors` line but no `FileHashes` entry, which retries. Correct.
- `mapDrift`: counts a file present on one side only. Correct.
- `RedundancyReportMatches`: a note or red report never matches, and the selection runs over `rev` through `commitTree`. Correct.
- `gateslot` decode: the unknown-report refusal names `redundancy` and `lym config gate`. The two new keys are bounded at 1 or more. `Gate()` clones `Reports`. Correct.
- `state.WriteJSON`: keeps only keys the type does not declare, so omitempty fields do not leak stale values from the old report. Correct.

**Findings**

1. MEDIUM, design, `internal/gatecli/redundancy.go:445-452` with `redundancyreport.go:165-169`. A call cancelled mid-step makes `stopReason` return `reasonCancelled`. That reason is listed as not judged, with no `Errors` line.
   - `retries()` only checks `reasonBudgetSpent`, so the report is written as complete.
   - The next call on an unchanged tree skips the step through the match and never judges those tests.
   - D7 says a later call retries "what failed or ran short".
   - Evidence: `judgePackage` at redundancy.go:464-466 and 481-484, and `retries` at redundancyreport.go:165-169.
   - Fix: make `retries()` also true for `reasonCancelled`, or add an error line when the step stops on cancellation.

2. LOW, design, `internal/gatecli/redundancy.go:160-164`. The skip-on-match compares only test hashes and file hashes. It does not compare `redundancy_max_tests`, and a coverage map rebuilt since the report does not invalidate it.
   - Raising the cap, or a fresher map, never re-judges a tree whose report lists tests as `past redundancy_max_tests`.
   - This follows D7's wording ("D8's match"), so it is a possible gap, not a clear violation.

**Tool calls**
1. `state.WriteJSON` in `internal/state/state.go`: to see whether omitempty fields keep stale values from the old report. They do not.
2. `RunAlone` in `internal/testcover/runner.go`: to check profile reuse and timeout handling. The manifest lacked it.
3. `testRequest{` construction sites, `run.go` lines 60-120 and `impactset.ChangedPaths`: to check `secondUnit` is set, `location` is non-nil, and untracked test files are taken.
4. `gateslot.HolderEnv`: to check it accepts a nil lease for the inherited-slot case.
