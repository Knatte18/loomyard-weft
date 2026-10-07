# Round 2 fixer report

Every finding of `round-2-review.md` is fixed, each in its own commit on this worktree; nothing is pushed.

## What changed

| Finding | Commit | Change |
|---|---|---|
| F1 | `planglyph: fix F1` | Rewrapped the code-index paragraph in `internal/planglyph/doc.go` one sentence per line. |
| F2 | `landingshed: fix F2` | Split the `checkPublishVerify` doc into one sentence per line. |
| F3 | `commentlint: fix F3` | The package doc states which breaks pass as one sentence and defines a finding in the next, with no break at a comma before `nor`. |
| F4 | `landingshed: fix F4` | The `TestPublishVerify_PublishVerifyCommand` doc states its two behaviors as two sentences. |
| F5 | `shedrecipe: fix F5` | `TestResolveGateSpec_PlanGatesRequirePlanIndex` and its subtests call `t.Parallel`. |
| F6 | `commentlint: fix F6` | Deleted `TestFindNewWrappedBreaks_FindingNamesFileLineAndText`, and lowered the `internal/commentlint` row of `cmd/lyx/testdata/test-budget.yaml` from 4 to 3. |
| F7 | `impactset: fix F7` | The error for a marker in a `tmux` or `llm` file now reads `<file>:<line>: //lyx:guard marks a guard test in a file no integration-tier run compiles`, and both rows of `TestFindGuardTests` assert `p_test.go:5`. |
| F8 | `commentlint: fix F8` | `isProseBlock` became `isProseParagraph` and the lint judges each paragraph of a block on its own, split at blank comment lines, so a heading, list item or code line exempts only its own paragraph. Two rows of `TestFindNewWrappedBreaks` show a wrapped paragraph beside a heading paragraph and beside a list paragraph is flagged, and `doc.go` states the paragraph rule. `adjacentProse` went, since a paragraph holds no blank line. |

## Verification

`go build ./...`, `go vet` under `integration`, `tmux` and `llm`, `go test ./...` and `go test -tags integration ./...` all pass after the last commit.
`lyx loom lint-comments --base <merge-base>`, built from this tree with the F8 lint, reports no finding.
The test-budget file still equals an independent count of top-level tests for every package.
A scan of every comment line the range adds finds no mid-sentence break beyond the carried-over ones the pattern exempts and independent-clause breaks.

## Deferred

Nothing is deferred.

## Notes

- The F8 lint is stricter than the one the earlier gates ran, so the card gates and the round gate now flag a wrapped paragraph that sits in a block with a heading or list.
  Nothing in this range tripped it, because F1 was the only such paragraph and is fixed.
- The pair-key false negative of the lint, a new break whose two boundary words also meet at a base line end, is prescribed by card 6 and left as it is, as in round 1.
  F2 passed the lint through it and is fixed at the comment.
