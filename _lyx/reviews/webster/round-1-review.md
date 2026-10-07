---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    location: tools/tokencount/tokencount
    summary: an 11 MB compiled ELF binary was committed by cards 3 and 5, against the repo's never-commit-a-binary rule in .gitignore
  - id: F2
    severity: BLOCKING
    class: consistency
    location: internal/fabricengine/refscanner_test.go:78
    summary: TestReferenceRule_Match and its subtests omit t.Parallel and name no global state, against PATTERN-test-speed
  - id: F3
    severity: MEDIUM
    class: scope
    location: tools/tokencount/calibrate_test.go:135
    summary: no test pins that a peak row's Estimate and EstimatedGrowth are priced at the fork's own position, since TestCalibrate runs a profile with batch_growth 0
  - id: F4
    severity: LOW
    class: consistency
    location: internal/batcher/cost.go:87
    summary: the inner loop's before variable shadows the Batch parameter before, which the code below the loop still means as the int
  - id: F5
    severity: LOW
    class: consistency
    location: internal/fabricengine/refscanner_test.go:93
    summary: the row "quoted search pattern returns neither" repeats what TestRefScanner_Matches already covers
---

Range: `git diff $(git merge-base main HEAD)..HEAD`, since `_lyx/loom/rework/` does not exist and `_lyx/shed/batcher-v2/status.json` carries `product.parent: main`.
Eleven commits, one per card.

Evidence gathered: `go build ./...`, `go vet ./...` and `go vet -tags integration|tmux|llm ./...` pass; `go test ./...` passes; `go test -tags integration` passes over the packages the diff touches; `lyx loom lint-comments` passes.
No card has a `Rename` or `Delete` group, so no AST-script or `assert-no-callers` check applies; every card's Edit and Create targets appear in its commit.

Checked against the focus directive and found sound, so no finding:

- No `startup_context` reference survives outside the retired-key handling the plan asks for: the load error in `profileWeights`, the decode-only `RetiredStartupContext` field, and the tests that pin both.
- The position convention is one-based everywhere: `PeakContext` and `startContext` take a 1-based position, `Batch` prices its k-th batch at `before + k`, `rebaselineBatches` passes the kept count, and calibration's `forkPositions` starts at 1.
- The dynamic program keeps the old objective and tie-breaks: fewest batches, then the lowest largest peak, then the shortest last batch, because `begin` descends and the comparison is strict.
- The `history` flag is gone from code, docs and usage; the only mentions left are the refusal test and a recorded-command fixture string in `acceptbatch_test.go`.
  `repositoryFlag` is a non-test constant, and the tests build the flag from it.
- `NewReferenceRule` backs both `RefScanner.Matches` and the plan check, so the patterns are compiled in one place, and `Matches` answers as before.
  `CheckCardFabricReference` covers Verify values, the verify section, fenced blocks and inline spans, and `planglyph`'s index runs it in `ValidateFormat` and `ValidateRework`, which are the paths both plan gates and `validate-plan` take.
- `IsReadOnlyCommand` refuses redirections, substitutions, backgrounding and every command outside the closed list; `AcceptBatchFabricReference` refuses the whole call on one non-read-only or command-less entry.

Notes that are not findings, since the target is the diff:

- The live `_lyx/config/batcher.yaml` (in the fabric repo, outside this diff) still carries `startup_context: 60000` in its `cautious` profile, and that profile is active.
  After this branch is deployed, `batcher.Active` refuses to load that file until it is changed to `master_base` and `batch_growth`; the template values are 52000 and 7000.

### [BLOCKING] Compiled binary committed

**Location:** `tools/tokencount/tokencount`
**Issue:** Commit `340ed2e13` (card 3) added an 11.8 MB ELF executable, and `a39151dac` (card 5) rewrote it.
No card names it, and `.gitignore` states compiled binaries are never committed; it ignores `/lyx` and `/godocreflow` for the same reason, but has no entry for this one, which `go build ./tools/tokencount` produces beside the sources.
**Fix:** Remove the file from the tree and add `/tools/tokencount/tokencount` to `.gitignore`.
The two commits stay in the branch history, which a fixer must not rewrite; the orchestrator decides whether to squash them away before landing.

### [BLOCKING] New test omits `t.Parallel`

**Location:** `internal/fabricengine/refscanner_test.go:78`
**Issue:** `PATTERN-test-speed` requires `t.Parallel` unless a comment names global state.
`TestReferenceRule_Match` and its subtests call neither, and no comment names state; the rule is stateless, so nothing bars it.
The neighbouring `TestIsReadOnlyCommand` does it right.
**Fix:** Call `t.Parallel()` at the top of the test and in each subtest.

### [MEDIUM] Calibration's per-position peak pricing is untested

**Location:** `tools/tokencount/calibrate_test.go:135`
**Issue:** `addRun` prices each fork with `batcher.PeakContext(..., position)` and derives `EstimatedGrowth` from `startEstimate(c.Weights, position)`.
`TestCalibrate` runs the `fit` profile, whose `batch_growth` is 0, so passing a constant position, or the wrong one, leaves every assertion true.
The `grow` profile is used only in `TestCalibrateStartFit`, which has no plan history and so produces no peak rows.
The behavior the diff adds, a peak row priced at its fork's position, has no covering test (`PATTERN-test-economy`, a behavior the diff adds with no coverage).
**Fix:** Add a subtest to `TestCalibrate` that runs `Calibrate` with the `grow` profile over the same alpha run and asserts each row's `Estimate` and `EstimatedGrowth` at positions 1 to 4.

### [LOW] Shadowed `before` in the dynamic program

**Location:** `internal/batcher/cost.go:87`
**Issue:** `before := table[k-1][begin]` declares a `split` named like the `before int` parameter, which `peak.start(w, before+1)` and `breakdownOf(..., before+k)` use as a count.
Both uses are correct today, but one more line in that loop reading `before` as the count would silently read the wrong value.
**Fix:** Rename the loop variable to `prefix`.

### [LOW] Redundant row in the new rule test

**Location:** `internal/fabricengine/refscanner_test.go:93`
**Issue:** `RefScanner.Matches` is the disjunction of the two rules, so `TestRefScanner_Matches` rows "quoted search pattern is text" and "echoed prose is text" already show that a quoted `lyx fabric` matches neither.
The card asks the new table to cover only what that test cannot see, the matched text and the rules called apart; the heredoc row it names stays, and this row adds nothing to it.
**Fix:** Delete the row "quoted search pattern returns neither".
