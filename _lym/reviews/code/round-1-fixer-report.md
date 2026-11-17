# Fixer-report, code round 1

All ten findings are fixed as proposed unless marked below.
The gate (`lym gate test --tier 0 --tier 1` over `internal/gitrepo`, `internal/websterengine`, `internal/secretpath`, `internal/fabricengine`, `internal/webstercli`, `cmd/lym`) is green.
Commits group findings that touch the same files, so one commit covers several ids:
`websterengine: fix F1, F2, F4, F6, F8, F9`, `secretpath: fix F3, F10`, `fabricengine: fix F5, F7`, and one for the self-check fixes.
`_lym/landing/summary.md` is edited on disk and committed nowhere, since no git command may touch `_lym`.

## Changes

- **F1.**
  Added the constant `websterengine.SecretFileFailureReason`, which `failOnSecretPaths` opens its reason with.
  Added a Master stencil rung after the `batch_failed` rung: a message containing that text makes Master write `outcome: stuck` and stop without `recover-batch`.
  The way forward now reads `for the run's parent, once the run has stopped: 1) lym webster reset --to batch-start --batch NN; 2) lym webster recover-batch NN`.
  `TestRecordBatch_SecretFileFailsBatch` asserts the constant in the error.
- **F2.**
  Replaced `gitrepo.Repo.AddedFilesBetween` with `AddedFilesInCommits`, which runs `git log --no-renames --diff-filter=A --name-only -z base..head`.
  Updated the doc comments of `Git.AddedPaths` and `addedPaths`, `websterengine/doc.go` and the landing summary.
  Added the new method to the pinned method list in `cmd/lym/gitrepoboundary_test.go`, which lists every `Repo` method that calls `run` or `runChecked`.
  `gitrepo_test.go` commits a file and deletes it in the next commit and asserts the range lists it.
  `fakeGit.AddedPaths` keys on the (base, head) pair, so the secret test pins `bs.StartSHA` as the base.
- **F3.**
  `t.Parallel()` after `tiergate.Require` in `TestRecordBatch_SecretFileFailsBatch`, first in each subtest, and in `TestLeafInvariant_StandardLibraryOnly`.
- **F4.**
  The correctness branch of `RecordBatch` runs `failOnSecretPaths` first once the head resolves.
  `failOnSecretPaths` takes the batch's audit findings and the transcripts it consumes, so the secret failure also lists each finding as a reason, marks it failed in the ledger and consumes the transcripts, as `failFromFindings` does.
  New `CorrectnessFindingToo` row.
- **F5.**
  New `HubPattern` row in `TestCommit_SecretPath_RefusedAndCommitsNothing`, over a hub `fabric.yaml` with `secret_paths: "*.key"`.
  New tier-0 `TestWire_HubLoadsSecretPathsFromFabricYAML`, which runs the real `wire` and calls the loader it installed, for an added pattern and a malformed one.
  New tier-1 `TestRecordBatchCmd_SecretPaths`, with a hub-pattern failure row and a row whose loader fails with an invalid config.
  The `fabric.yaml` load refusal is asserted there, with its way forward.
  The two `webstercli` tests live in separate files because the tier-tag guard allows one tier per file.
  The CLI test sets `c.secretPaths` directly, so the loader's own wiring is pinned by the tier-0 test and the verb's use of it by the tier-1 test.
- **F6.**
  Dropped the delete step from the way forward, rewrote the `failOnSecretPaths` doc comment, `websterengine/doc.go` and the summary to say the reset drops the commits and the files they added.
  The test expects `2) lym webster recover-batch 01`.
- **F7.**
  The refusal test asserts `git diff --cached --name-only` is empty in both the warp and the weft checkout, through `gitkit.Git`.
- **F8.**
  The `NewPathsIn` step asserts `NewPathsIn(nil)` returns no paths while `fresh.txt` and `staged.txt` exist.
- **F9.**
  New `NeedsParentReport` row asserting `ErrBatchFailed` and a terminal failed batch.
- **F10.**
  The `ShippedPatternWinsOverAdded` row adds `.e*` and still wants `.env`.

## Self-check

Lens reports are saved beside this file as `round-1-fixer-report.lens-<lens>.md`, one per lens; every fork returned a report.

- fix-consistency, LOW: four comment lines at 120 characters or more (`doc.go` twice, `recordbatch.go`, `gitfake_test.go`). Fixed as proposed.
- fix-consistency, LOW: the new stencil rung's "over the two rungs above". Fixed as proposed, now "over the `batch_failed` rung above".
- fix-proposal, LOW: the stencil rung repeats the reason text with no test tying it to the constant. Fixed with my own change: `TestMasterTemplate_QuotesSecretFileFailureReason` asserts the rendered master template quotes `SecretFileFailureReason`. No templated marker was added, since the stencil carries no such constant.
- fix-proposal, LOW: the precedence wording. Merged with the fix-consistency finding above.
- fix-code-correctness, LOW: the stencil rung and the constant are not tied by a test. Merged with the fix-proposal finding above.
- fix-affected-set, MEDIUM: the new tier-0 wiring test and the tier-1 record-batch test call no `t.Parallel` and name no global state. Fixed with my own change: neither can run in parallel, since `wire` sets the process-global logger sink and the tier-1 subtests call `t.Setenv`, so each test's doc comment now says so.
- fix-affected-set, LOW: the `record-batch` long help does not say the way forward is the run's parent's. Fixed as proposed in shortened form, because the long help has a 2000-byte cap that the first wording exceeded.
- fix-affected-set, LOW: `recordbatch.go` header line 8 and the comment at the late secret check do not mention the new ordering. Not a finding: both remain true, and the `RecordBatch` doc comment and the inline comment at the correctness branch state the ordering. No change.
- fix-affected-set, clean sets: no stale `AddedFilesBetween`, delete-step wording or caller of the old `failOnSecretPaths` signature.

## Gate repair

The gate flagged four comment sites by line length or a wrapped break:
the header of `secretpaths_wiring_test.go`, the secret-file paragraph of `websterengine/doc.go`, the `SecretFileFailureReason` comment in `recordbatch.go`, and the doc comment of `TestMasterTemplate_QuotesSecretFileFailureReason`.
Each is rewritten as whole sentences, one per line, under 120 characters.
The `doc.go` sentence that opens the secret-file definition ended in a comma too and is split the same way.
No code changed.

## Disputed

None.

## Deferred

None.

## Rulings

None.
