# Webster round 1 — fixer report

Every finding in `round-1-review.md` is fixed, each in its own commit.
After the last commit, `go build ./...`, `go vet -tags smoke ./...`, `go test ./...` and `go test -tags integration ./...` all pass.

## Fixed

- **F1 (MEDIUM)**, `4ea72a75d`: `fabricengine.MergeUntrackedFiles` lists the warp checkout's untracked files only.
  It no longer reads the weft or resolves merge geometry, and its doc says why: the weft is no merge participant.
  `mergestagetracked_integration_test.go` now writes an untracked file at the weft root and asserts that it is neither listed nor an error.
- **F2 (LOW)**, `0d53f6f1f`: after a pass, `verifytree.Verify` re-reads HEAD's tree and writes the record only when the tree is unchanged.
  A moved HEAD still returns `StatusPassed`, with no record, so the next call runs again.
  The `Verify` and `StatusPassed` docs say so, and `TestVerify_CommitDuringRunLeavesNoRecord` pins it.
- **F3 (LOW)**, `6a9f4288e`: `DirtyPaths` runs `git status --porcelain -z` and parses it with the new `parsePorcelainZ`.
  Paths come back verbatim, and a rename or copy record's source is skipped.
  `porcelain_test.go` (tier 1) covers special characters, an arrow inside a name, and index, worktree and copy renames.
  `TestDirtyPaths_NamesSpecialAndRenamedPathsVerbatim` checks the same against real git.
- **F4 (LOW)**, `b1c47b14b`: reworded the prose that still named the retired mechanisms:
  - "webster integration fork" became "webster verify-gate fixer fork" in `friction/friction.go`, `friction/doc.go` and the implementer friction stencil's banner.
  - "the integration report" became the verify gate's report in `loomshed/webster.go`, `loomcli/wiring.go` and `recordbatch_test.go`.
  - `validateReportFields`' doc no longer names the deleted `ParseIntegrationReport`.
- **F5 (NIT)**, `9fedd120b`: every comment F5 lists now breaks at sentence and independent-clause boundaries.
  The new F1 comment in `mergestage.go` follows the same rule.
- **F6 (NIT)**, `44214954f`: the knob comment in `websterengine/config.go` now says a zero `verify_gate_attempts` turns the must-pass verify gate off.

## Deferred

None.
