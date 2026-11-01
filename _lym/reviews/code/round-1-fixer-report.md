# Round 1 fixer-report

## Changes

Commit `c66eebf8` holds both fixes; they share one test file, so they were committed together rather than one per finding.

- **F1** — fixed as proposed (`tools/boltrename/main.go`).
  `main` resolves the `filepath.Abs` path with `filepath.EvalSymlinks`, failing on error, and passes the resolved path to `liveProcesses` and `inspectHub`.
  `liveProcesses` takes the resolved path and the Abs path: the cwd test uses the resolved one, the command-line test uses both.
  `TestLiveProcesses` gains a row naming the Abs path on a command line; the existing rows have cwds under the resolved path.
- **F2** — fixed as proposed (`tools/boltrename/rename_test.go`).
  `TestInspectHub_StopsOnWhatItDoesNotExpect` gains rows `StrayFileInTheHub`, `NoPrimeWithWeft`, `CheckoutGitLinkPointsElsewhere` and `NewAdminDirAlreadyPresent`.
  `TestLiveProcesses` gains a row with PID `os.Getpid()` that must not be listed.

`lym gate test ./tools/boltrename --tier 0` passed.

## Self-check

- fix-code-correctness — no findings.
- fix-affected-set — no findings.
- fix-consistency — LOW, `inspectHub` compares git link files with the resolved path: no change made; the proposal stands (see below).
- fix-consistency — LOW, F1 and F2 committed together: no change; the commit cannot be split without rewriting history, and the code is unaffected.
- fix-proposal — LOW, same resolved-path concern as the consistency lens (merged with it): no change made.
- fix-proposal — LOW, the resolved-path `Cwd` row cannot fail with the defect in place, because the defect lives in `main`: no change; `main` is untested by design per the review's Rejected section, and the row the review asked for is in place.

The resolved-path concern fails closed: a hub whose git links hold the unresolved spelling stops with a message naming the path, and the operator fixes it by hand.
It stays as the reviewer proposed because the fix-everything rule gives no dispute ground (no cited evidence of a wrong result), and passing the Abs path instead would equally stop a hub whose links hold the resolved spelling.

## Disputed

None.

## Deferred

None.

## Gate repair

The verify gate failed `TestEnforcement_FabricVocabulary`: `tools/boltrename/rename_test.go` held the bare token weft in the `NoPrimeWithWeft` row name and its expected message.
The row is now `NoPrimeWithRecords` and expects the message part "no prime with its", which the same stop message contains.
`lym gate test ./tools/boltrename ./internal/lymcwd --tier 0` passes.
