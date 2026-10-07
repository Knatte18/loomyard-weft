# Round 1 fixer report

Every finding of `round-1-review.md` is fixed, each in its own commit on this worktree; nothing is pushed.

## What changed

| Finding | Commit | Change |
|---|---|---|
| F1 | `planindex: fix F1` | Rewrapped the moved and edited comments in `internal/planindex/planindex.go`, `internal/planglyph/repo.go`, `internal/websterengine/beginbatch.go` and `internal/websterengine/runlevel.go` one sentence per line. |
| F2 | `tests: fix F2` | `t.Parallel()` and the shared-repository sentence added to `TestLint_OverARepository`, `TestDerive_OverARepository`, `TestLintCommentsCmd_OverARepository` and `TestTreeSitterLink_OnlyAllowedTestBinariesLinkIt`. |
| F3, F4 | `commentlint: fix F3, F4` | A directive line now ends the comment block it sits in instead of exempting the whole block; `yet` and `so` join the coordinating conjunctions. `doc.go` follows. The "directive comment block is never flagged" row became a row showing the prose above a directive is still checked, and two rows cover `yet` and `so`. |
| F5 | `loomshed: fix F5` | `impactset` returns a `*GuardScanError` (with `IsGuardScanError`) for a test file that does not parse, a misplaced marker or a marker in a `tmux`/`llm` file. `NewVerifyGate` turns only that into a finding, which names the file, the line and the way to place the marker; every other `Derive` error is a returned error. The existing marker step in `verifygate_integration_test.go` also asserts the way forward. |
| F6 | `refusal-spec: fix F6` | The "Webster-Burler verify gate exhausted" row in `contracts/specs/refusal-spec.md` now covers the lint and marker refusals and names `lyx loom lint-comments` as their check. |
| F8 | `websterengine: fix F8` | A nil-`Index` row in `TestBeginBatch_Refusals`, asserting the error names `Geometry.Index`. |
| F9 | `cmd/lyx: fix F9` | Deleted `TestParseGoListImports`; `TestTreeSitterLink_OnlyAllowedTestBinariesLinkIt` covers `parseGoListImports` over real `go list` output. |
| F7 | `cmd/lyx: fix F7` | `cmd/lyx` budget row set to 68, the count left after F9; a scan of every package's top-level test count against the file shows no other row differs. |

## Verification

`go build ./...`, `go vet` under `integration`, `tmux` and `llm`, `go test ./...` and `go test -tags integration ./...` all pass after the last commit.
`lyx loom lint-comments --base d536a1a4e`, built from this tree, reports no finding.

## Deferred

Nothing is deferred.

## Notes

- Before the review was saved I ran two read-only git commands with a `_lyx/config` pathspec (`git diff` and `git log`, both empty) to see whether the landing config sat in the diff; I ran no git command against `_lyx` in job B.
- The pair-key false negative in the lint (a new break whose two boundary words also meet at a base line end) is prescribed by card 6 and left as it is.
