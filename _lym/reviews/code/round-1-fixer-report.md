# Fixer report — code review round 1

Every finding but the `decision`-class F2 is fixed.
The gate (tier 0, tier 1 and the guards over the touched packages) passes after the last change.
The commits group findings that share a file, so they are not one per finding.
Commits: 4662725c (F1, F5, F9, F13, F14, F18), 61ae9bd1 (F3, F4, F6, F7), 854b53d6 (F8, F17), baf722ab (F11), d093e9c2 (F10, F12), 9624351a (F16, F19), 22d512c1 (F10, F15).

## Gate repair

The first gate refused the round because comment lines I added or renamed were over 120 characters or wrapped mid-sentence.
I split each at a sentence boundary in `cmd/lym/testonlycode_linux_test.go`, `internal/discussionparser/append.go`, `internal/fabriccli/cli_test.go`, `internal/fabricengine/pushanchored_integration_test.go`, `internal/fabricengine/snapshot_integration_test.go`, `internal/loomrecipe/recipe_test.go`, `internal/shuttlecli/cli_test.go` and `internal/webstercli/cli_tmux_test.go`.
A second gate still found three lines at or over 120 characters in `internal/shuttlecli/cli_test.go` and `internal/webstercli/cli_tmux_test.go`.
I split those into shorter sentences.
Only comment text changed.

## Verify failure in `internal/pullrequest`

A later verify run failed tier 0 in `internal/pullrequest` with `tmuxkit: process ... left running`.
The leftover processes came from `TestRunButton_RealTmux` in `internal/reedengine`'s integration tests.
This round changed neither package's Linux build (`internal/reedengine/proctree_windows.go` compiles on Windows only), and the diff touches nothing in `internal/pullrequest`.
`PATTERN-test-isolation` records the bound that a concurrently running package fails for a process another binary left.
I reran `internal/pullrequest` alone and ran all of tier 0 over the module; both pass, so I made no code change.

## Changes

- **F1** — moved `internal/shedcheck` to `internal/testkit/shedcheckkit` (package, doc and tests renamed), deleted `shedbuild.Check` and its test, and pointed `internal/loomrecipe`'s tests at `shedcheckkit.Check(built, recipe.Entry, recipe.Terminals)`.
  Removed the ten allowlist entries.
  Updated the `shedbuild` comments (`build.go`, `doc.go`, `parse.go`, `recipe.go`), its import allowlist in `seam_enforcement_test.go`, the `battenrecipe` and `loomrecipe` comments that named `shedbuild.Check`, the shed-recipe spec and `docs/overview.md`.
  The kit imports `internal/shedengine` and the standard library only, which `PATTERN-test-isolation` allows.
- **F3** — added `TestReplayRequest_RefusesABoardWorktreeOfNoHub` to `internal/fabricengine/destroy_landedwork_integration_test.go`, which asserts a `CheckOwnership` refusal from `checkPathRequest(replayRequest(foreign, ownedBoardWorktree(), nil))`.
- **F4** — deleted `PushAnchored` from `export_test.go`.
  Re-targeted the callers at `PushPairAnchored`.
  Deleted `TestPushAnchored_PushesAndRecordsBranchPush`, since the `PushPairAnchored` walk covers its success path and record entries.
  Renamed the diverged-remote test to `TestPushPairAnchored_PrimeDivergedWeftRemote_ReturnsErrPushRejected`.
  Dropped the `PushAnchored` row of the push-lock test, which left a single case, so the table became straight-line code.
  Made `countBranchPushed` assert the `" remote="` part of the detail the deleted test used to assert.
  Updated the file header and the lock-test doc.
- **F5** — cut `never shedbuild.Load` and the "Unused entry" paragraph from the spec.
  Also removed the `Load` and `Check` entries of the spec's exported-function list, which the deletions made false.
- **F6** — `internal/fabricengine/doc.go` names `SnapshotWarpSHAForTest` in the read-half sentence and every other occurrence, and the consumer sentences no longer describe a production reader.
- **F7** — deleted `TestSnapshotWarpSHA_PerBranchScoping` and `TestSnapshotWarpSHA_DanglingWarpSHA_ReturnsRawWithSHAExistsFalse`.
  Re-targeted the unborn-HEAD test at `RebuildIndex` as `TestRebuildIndex_UnbornWeftHEAD`.
  Kept `TestSnapshotWarpSHA_TopologicalOrderBeatsCommitDate` whole (see Self-check).
- **F8** — `CheckAddedDecisionsKept`'s doc names `AppendDecisions`.
- **F9** — `docs/overview.md` says "planglyph's plan gates call `planparser.ValidateFormat`/`Validate`".
- **F10** — added `testtiming:keep` markers to the eight listed tests.
- **F11** — restored `hibernateengine.TerminalPhases` (a clone of `terminalPhases`) with an allowlist entry.
  `TestHibernateEndedPhases_AreTheEnginesTerminalPhases` calls it again.
- **F12** — `TestExitCode_UnknownModule` has a row that calls `run` with separate stdout and stderr buffers and asserts the envelope is on stdout alone.
- **F13**, **F14** — `PATTERN-test-only-code.md` states the generic-function bound, the generated-file and marker-method exemptions, and the corrected third bound.
  The guard's `packageFunctions` doc states the generic bound.
- **F15** — renamed every `TestRunCLI*` and `TestRunCLIIn*` test under `internal/*cli` to `TestCommand_*`, with the references in comments.
- **F16** — `TestNewPreflightWithChecks_CarriesToldName` calls `NewPreflightWithChecks("Some-Row", "/some/cwd", Checks{})`.
- **F17** — `docs/benchmarks/board-performance.md` says the rows drive `clihelp.Execute(boardcli.Command(), ...)`.
- **F18** — the `seatengine.Prompts` allowlist reason names the group of packages that start seats.
- **F19** — `tmuxProcessName` uses `filepath.Base`, and the two comment lines are gone.

## Fixed differently

- **F7** — the review proposed deleting the four `TestSnapshotWarpSHA_*` tests, re-targeting any that pin an ordering or scoping behaviour of `scanWarpSHATrailers`.
  `TestSnapshotWarpSHA_TopologicalOrderBeatsCommitDate` stays whole, because the production `--topo-order` scan is its subject and `SnapshotWarpSHAForTest` is the oracle that reaches it.
  A `RebuildIndex` assertion alone could not fail on a date-ordered scan, since index entries are keyed by warp SHA and this history holds two.
  The unborn-HEAD test is re-targeted at `RebuildIndex` rather than deleted.

## Disputed

None.

## Deferred

- **F1**, **F4** — the edits to `_lym/landing/summary.md` (taking `shedcheck` and `shedbuild.Check` out of the allowlist list and the claim about kept entries, and taking `PushAnchored` out of the Setup helpers list).
  My instructions bar touching `_lym`, and the file belongs to the loop that spawned this round.
  The parent updates it.

## Rulings

- **F2** — the parent recorded (creel #1331) that `internal/gitkit` and `internal/hubforge` count as test-support packages and that an allowlisted cross-package oracle or record writer may stay in its production file.
  The ruling asks for no code change and the code already holds it, so nothing changed.
  Nothing to flag to the parent.

## Self-check

Forks: `fix-code-correctness`, `fix-consistency`, `fix-proposal`, `fix-affected-set`. All four returned a report.

- fix-code-correctness, MEDIUM — the retargeted topological test no longer pins `--topo-order`. Fixed differently: restored the `SnapshotWarpSHAForTest` assertion and the original name.
- fix-code-correctness, MEDIUM — the unborn-HEAD tolerance had no test. Fixed: `TestRebuildIndex_UnbornWeftHEAD`.
- fix-code-correctness, LOW — the `GOOS=windows go vet ./internal/reedengine` failure on `mustTmuxOutput` is not caused by this round (only `proctree_windows.go` changed there). No change.
- fix-consistency, LOW — `shedcheckkit/doc.go` still said "shedcheck's diagnosis". Fixed.
- fix-consistency, LOW — `commit_integration_test.go` messages named `snapshotWarpSHA()`. Fixed.
- fix-consistency, LOW — `PATTERN-test-isolation.md` was silent on the new kit. Fixed with one sentence. `internal/testkit/doc.go` names no kit beyond the exempt ones, so it is unchanged.
- fix-proposal, MEDIUM — same finding as the correctness fork's first. Merged, fixed.
- fix-proposal, MEDIUM — `PerBranchScoping` and the unborn-HEAD test were deleted without a coverage check. The unborn one is re-targeted. `PerBranchScoping` stays deleted: the current-branch scope comes from `git log` over the checked-out branch, and `RebuildIndex` reads the same scan.
- fix-proposal, LOW — the `" remote="` assertion was lost with the deleted test. Fixed in `countBranchPushed`.
- fix-proposal, LOW — `filepath.Base("")` gives `"."`, so an empty binary yields `"..exe"`. Not changed: the caller passes a real binary path and the review asked for `filepath.Base`.
- fix-proposal, LOW — the one-row push-lock table. Fixed: the table is gone.
- fix-affected-set, LOW — the same stale `snapshotWarpSHA()` messages. Merged, fixed.
- fix-affected-set, LOW — `docs/benchmarks/test-redundancy.md` lists keep markers of deleted tests. Not changed: it is a measurement report.
