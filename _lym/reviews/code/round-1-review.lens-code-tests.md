```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, I looked for the test row that fails if that behaviour breaks.

**Per behaviour, the test that pins it**

Red-first gate (`internal/loomshed/redfirstgate.go`):
1. Exit 4 answers pending, with one line naming the way forward: `TestRedFirstGate_VerdictPerLabelRecordAndExitCode` row "a check that exits 4 holds the run…". This row also checks that `Send` is a single line.
2. Exit 5 answers pending, with one line naming the way forward: the same test, row "a check that exits 5 holds the run…".
3. `Final` turns exit 4 and exit 5 into failures with findings: the same test, rows "a final evaluation of a check that exits 4…" and "…exits 5…".
4. The first refusal holds and sends the line: `TestRedFirstGate_HoldRunsNothingUntilACommitOrTheRecordLands`, step 1.
5. An arrival on the same HEAD tree runs nothing and sends nothing: the same test, step 2.
6. A new commit runs the check again and holds without sending: the same test, step 3.
7. The `No test seam` record lifts an active hold: the same test, step 4.
8. After a hold ends in a non-hold verdict (pass, green failure, dirty tree, exempt), `heldTree` resets, so a later refusal sends the line again: **none**.
9. `Final` on an exempt entry (no bug label, or the record has landed) passes and runs nothing: **none**. Every `final: true` row uses exit 4 or 5 with a bug label and no record.
10. `holdsForCommitOrRecord` excludes `TimedOut` and `RunnerCrash`: **none** that would fail. The only timeout row ("…holds the tier timeout line fails terminally") exits 1, so it stays terminal even without the guard.
11. A `HeadTree` read error while held becomes the gate's returned error: **none**.
12. `SendFailedWayForward` is set on the pending result: none. It is only a log clause, so this is no finding.
13. `HeadTree` is now exported: a rename only, already reached by `TestVerify_PauseHold`.

Recipe (`internal/shedrecipe/entries_gate.go`):
14. The red-first entry is `MayHold` and carries `Final`: `TestResolveGateSpec_RedFirstEntryMayHoldAndCarriesAFinal`.
15. Every other entry keeps `MayHold` false and `Final` nil: **none**.
16. `resolveRedFirstGate` refuses a missing root or seam: the existing `TestResolveGateSpec_RedFirstRequiresItsEnv`, which the reach report lists.

Preflight (`internal/loomcli/darnwiring.go`):
17. A wrapped `*stencilstore.NewerBoardCopyError` keeps its own way forward alone, and every other error gains the config clause: `TestPreflightTableError_KeepsANewerStencilRefusalsOwnWayForward`, both rows. These rows also check the `errors.Is` wrap and that exactly one `way forward:` remains.
18. `PreflightStencils` in `wireDarn` actually goes through `preflightTableError`: no row would fail. `TestWire_DarnRecipe` reaches it, but nothing asserts a newer-stencil refusal at that seam.

Findings file (`internal/shedadapters`, `internal/loomcli`):
19. A later passing gate keeps the file: the `multillm_test.go:339-340` row, the `burlerfix_test.go:394-395` row, `requireAcceptedReviewRecorded` in `burlerreview_test.go:765`, and `TestBurlerHalfEntries_RunDirectory/ReviewWhoseGatePassedKeepsTheRowsFindingsFile`.
20. `latestDarnOutcome` names no kept file when the latest entry is not a Stuck: the new `TestLatestDarnOutcome` row "a Done latest entry names no findings file an earlier halt kept". The existing row "a resumed run, its error cleared, names the kept findings" covers the Stuck case.
21. The deleted `TestClearGateFindings` covered only `clearGateFindings`, which the diff also removes. No behaviour the diff keeps lost coverage. The four changed rows assert the new kept-file contract and are renamed to say so. None of them is a loosened check: each pins the opposite, intended behaviour exactly.

**Findings**

1. LOW, design: no test re-arms the hold after it ends.
   - Location: `internal/loomshed/redfirstgate.go`, `redFirstGate.arrival`, which resets `heldTree` on the exempt, verdict and non-hold failure paths.
   - No row pins that a refusal following one of those results answers pending again with a fresh `Send`.
   - Evidence: `internal/loomshed/redfirstgate_test.go:265-273`, where every step stays held or ends at the record.
2. LOW, design: no test covers `Final` on an exempt entry.
   - Location: `internal/loomshed/redfirstgate.go`, `redFirstGate.final`.
   - No row runs `Final` on a non-bug entry, or after the `No test seam` record has landed, to show it passes without running the check.
   - Evidence: `redfirstgate_test.go:122-142`. The `final: true` rows are exit 4 and exit 5 with a bug label only.
3. LOW, design: the `TimedOut`/`RunnerCrash` exclusion in `holdsForCommitOrRecord` is unpinned.
   - Location: `internal/loomshed/redfirstgate.go`, `holdsForCommitOrRecord`.
   - A timed-out run that reports exit 4 or 5 would hold instead of failing terminally, and no row would catch it.
   - Evidence: `redfirstgate_test.go:154-157`. The timeout row exits 1, which never reaches the exit-code comparison.
4. LOW, design: no test covers a `HeadTree` read failure while held.
   - Location: `internal/loomshed/redfirstgate.go`, `redFirstGate.arrival`, the `heldTree != ""` branch.
   - No row shows that the git error comes back as the gate's error rather than as pending.
   - Evidence: `redfirstgate_test.go:265-273`, `verify.go:542-546`.
5. LOW, design: no test shows that only the red-first entry may hold.
   - Location: `internal/shedrecipe/entries_gate.go:76`, `MayHold: final != nil`.
   - Nothing asserts that a plain entry (for example `verify` or `drift`) keeps `MayHold` false and `Final` nil. A mutant setting `MayHold: true` on every entry would survive, and must-pass entries would silently gain hold rights.
   - Evidence: `internal/shedrecipe/entries_gate_test.go:420-429` checks only red-first.
6. LOW, design: no test checks that `wireDarn` routes through `preflightTableError`.
   - Location: `internal/loomcli/darnwiring.go:357-361`, the `PreflightStencils` closure.
   - Reverting the closure to the old inline `fmt.Errorf` with the config clause would pass every test. The unit test pins the helper, not the wiring that issue #175 is about.
   - Evidence: the reach report lists only `TestWire_DarnRecipe` reaching `wireDarn`, and nothing asserts a newer-stencil refusal there; `wiring_darn_test.go:946-975` covers only the helper.

Tool calls:
1. `grep` of the row names and commands in `redfirstgate_test.go`, plus the `preWritten`/`wantFindings` rows in `multillm_test.go` and `burlerfix_test.go`. My context had only the diff hunks, not the unchanged rows such as the timeout row and the findings-file table setup.
