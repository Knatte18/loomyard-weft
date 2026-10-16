# Plan round 1 — fixer report

## Changed

- **F1** (fixed as proposed, `_lyx/plan/02-verify-package-missing.md`): Intent drops the pass for a result outside the worktree root.
  The missing-directory sentence now states that such a result is never under the root, so it is reported whatever lies beside the root, as D3 reads `..` and `../…`.
  Tests gain the reporting row `go test ../outside` in the overview chain, with an empty card.
- **F2** (fixed as proposed, `_lyx/plan/02-verify-package-missing.md`): the rows that report nothing gain `true | cd internal && go test ./nope`, `false || cd internal && go test ./nope` and `go -C .. test ./nope`.
  Checked against `splitChain` (`internal/planparser/verifymodule.go`): a segment's `Separator` is what preceded it, so the `cd` segments carry `|` and `||`, and `internal/nope` is absent from the materialized tree, so each row would report `./nope` if the scan went on.
- **F3** (fixed as proposed, `_lyx/plan/04-loop-status-stop.md`): the read-error bullet adds that a read error with the re-step already used stops at once with `status-unreadable: <error>` and the read-error way forward, with no re-read.
  Tests gain the subtest "a read error after a used re-step stops without a re-read".
  Its `ReadStatus` seam delegates to `readStatusAt` on its first call so the first child can start, then returns `read lock unavailable`.

## Gate findings

The gate reported three blocking `edit-test-file` findings on Edit entries naming external test files; all three are fixed.

- Card 1's `internal/planparser/targetdirs_test.go` and `internal/batcher/estimate_test.go` become the member glyphs `internal/planparser_test#TestCardTargetDirs` and `internal/batcher_test#TestPeakContext`, both resolved by `lyx quarry resolve`.
- Card 2's `internal/planparser/validate_test.go` Edit entry becomes the Create handle `plan:internal/planparser_test#TestValidate_VerifyPackageMissing`, and the Tests paragraph's pointer to the Shared Decision now says the card declares it as that handle.
- The overview's Shared Decision named external-test members by file only until card 1's mapping shipped in the running `lyx` binary.
  The installed binary already holds `planparser.goPackageDir` (found in `/home/knatte/go/bin/lyx`), so a `_test` glyph derives the package directory, and the decision now prescribes member glyphs and `plan:` handles on `<dir>_test`.

`lyx loom validate-plan` reports the plan clean.

## Disputed

None.

## Deferred

None.
