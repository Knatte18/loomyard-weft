---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: internal/fabricengine/refscanner.go:195
    summary: IsReadOnlyCommand reads a quoted command name as blank, so a quoted non-reader followed by a reader name passes as read-only
  - id: F2
    severity: MEDIUM
    class: consistency
    location: internal/websterengine/recoverbatch.go:336
    summary: recover-batch refusal still names accept-audit --batch only for HEAD at the start commit, while the refusal-spec row now also names the committed read-only route
  - id: F3
    severity: LOW
    class: consistency
    location: internal/webstercli/cli.go:226
    summary: the webster usage line for accept-audit --batch still says once HEAD is back at its start, though the verb now also accepts a committed read-only batch
  - id: F4
    severity: LOW
    class: scope
    location: tools/tokencount/calibrate.go:360
    summary: three behaviors the diff adds have no test, the n/a growth ratio when estimated growth is not positive, an unparseable review file, and a no-partition run whose plan is unreadable
---

The range is `git diff $(git merge-base main HEAD)..HEAD`: `_lyx/loom/rework/` does not exist, so step 4 of the rubric applies, and `product.parent` is `main`.
Every card's commit is present and in card order.
`go build ./...`, `go vet` (default, `integration`, `tmux`, `llm` tags), `go test ./...`, `go test -tags integration` over the touched packages and `lyx loom lint-comments --base 35e36281e` all pass.
No card has a `Rename` or `Delete` group, so the per-type mechanical checks reduce to the `ParseRoundReviewName` export in card 6, and no reference to `parseRoundReviewName` remains.

### [MEDIUM] IsReadOnlyCommand reads a quoted command name as blank

**Location:** `internal/fabricengine/refscanner.go:195` (`IsReadOnlyCommand`), `internal/fabricengine/refscanner_test.go:113` (`TestIsReadOnlyCommand`)
**Issue:** The classifier splits the quote-blanked copy of the command into segments and tests each segment's first word against the reader list.
A quoted word is blanked, so a quoted command name vanishes and the next word becomes "the command".
Run against a copy of the function, `'tee' cat`, `"rm" cat x`, `ls; 'git' cat` and `cat a | 'tee' cat` all return true, though the shell runs `tee cat`, `rm cat x`, `git cat` and `tee cat`.
Card 11 requires each reader "in command position" and a closed list; the bound the code and `AcceptBatchFabricReference` state, "no listed reader can write", does not hold for these commands.
`TestIsReadOnlyCommand` has no row with a quoted word in command position, so the gap is uncovered.
**Fix:** Decide the command word from the unblanked text of each segment, or refuse a segment whose first non-space character in the original text is a quote or a backslash, so a quoted or escaped command word is never read-only.
Add rows to `TestIsReadOnlyCommand` for a quoted non-reader before a reader name (`'tee' cat`, `"rm" cat x`) and, as the passing counterpart, a quoted argument after a reader (`grep "a b" x`, already present, stays).

### [MEDIUM] recover-batch refusal does not name the committed read-only route

**Location:** `internal/websterengine/recoverbatch.go:336`
**Issue:** `contracts/specs/refusal-spec.md`'s `audit: fabric-reference` row, changed by this diff, says the recover-batch refusal names `accept-audit --batch NN` then `recover-batch NN` "when the worktree is clean and either HEAD is the batch's start commit, or the start is an ancestor of HEAD and every entry's recorded command is read-only".
The message built at `recoverbatch.go:336` still says "when HEAD is the batch's start commit (git reset --keep to it if the batch committed) and the worktree is clean".
For a committed batch of read-only commands it therefore steers the operator to discard the batch's commits, the very outcome the new route exists to avoid, and the spec row and the message disagree (`PATTERN-refusal-way-forward`: the message names the way forward and the spec row describes it).
**Fix:** Reword the `wayForward` text to name both routes as the spec row does, keeping `resetToStartSteps(stepRunFresh)` as the otherwise-branch, and extend the existing assertion in `recoverbatch_test.go` (around line 794) so it pins the committed read-only wording.

### [LOW] webster usage line for accept-audit --batch is stale

**Location:** `internal/webstercli/cli.go:226`
**Issue:** The usage line reads "accept failed batch 8's pathless fabric references once HEAD is back at its start".
The verb's own `Long` help, updated in this diff, says it also accepts a batch whose commits are kept when every command only reads.
**Fix:** Reword the usage line to cover both routes in the same short form, for example "accept failed batch 8's pathless fabric references, once HEAD is at its start or its commands only read".

### [LOW] Added behaviors with no test

**Location:** `tools/tokencount/calibrate.go:360` (growth ratio), `tools/tokencount/profile.go:271` (`countFindings`), `tools/tokencount/profile.go:225` (`readRunProfile`)
**Issue:** `PATTERN-test-economy` and the rubric require each added behavior to have coverage, and three have none:
- `CalibrationRow.HasGrowthRatio` is false when the estimated growth is not positive; the `n/a` cell in the peak table and `GrowthFitOf` skipping such a row are never exercised, since every `TestCalibrate` row has a positive estimated growth (`TestCalibrate` asserts `HasGrowthRatio` true on each).
- `countFindings` marks a run's findings not read with "does not parse" when a `round-<n>-review.md` file is malformed; `TestBuildProfileReport` has the no-review-files mark only. Card 6 specifies this mark.
- `readRunProfile` marks a run "no partition recorded and its plan is unreadable" when the run has no partition and no plan commit; `TestBuildProfileReport` covers the no-partition run only with a readable plan.
**Fix:** Add a row to the existing tests rather than new tests: in `TestCalibrate`, a fork whose card makes the estimated growth zero (a profile or fixture with no target message weight) asserting `HasGrowthRatio` false, the `n/a` cell and the growth fit counting only the other rows; in `TestBuildProfileReport`, a run whose review file is malformed and a no-partition run with no plan commit, each asserting its marked row and that it enters no findings figure.
