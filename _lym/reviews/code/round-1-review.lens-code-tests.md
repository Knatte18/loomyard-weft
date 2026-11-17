```yaml
lens: code-tests
outcome: reported
```

Task: the code-tests lens, listing for each behaviour the diff adds or changes the test that pins it, or none.

**Behaviour → pinning test**

| Behaviour | Test |
|---|---|
| Pull-first adopt write refuses on a rejected push over another machine's hold or an owner of these subjects | `TestWriteAdopt/TwoClonesAdoptingTheSameBeamEndWithOneOwnerAndTheLoserRefusesNamingTheWinner` (`holdwrite_integration_test.go:219`) |
| Same-set own hold kept as it is | `TestWriteAdopt/ThisMachinesSameSetHoldStaysWithItsKindFixTaskAndTries` (`:113`) |
| Overlapping and disjoint other-set own holds refuse naming the hold's subjects | `TestWriteAdopt/ThisMachinesOtherSetHoldRefusesNamingItsSubjects_*` (`:142-146`) and `TestAdopt/APlainAdoptOfOtherSubjectsUnderThisMachinesHoldRefusesNamingTheHoldsSubjects` (`adopt_test.go:396`) |
| Another machine's hold refuses; `--take-hold` rewrites it only for the same subject set | `TestWriteAdopt` rows `:167`, `:186`, `:206` |
| Refused registration writes nothing | `TestWriteAdopt/ARefusedRegistrationWritesNothing` (`:267`) |
| Replay refuses an adopt commit over another machine's new hold, and replays holds and registry | `TestFabricReplay` rows at `replay_test.go:253-358` |
| `FixHold` and `ClearHold`, including rejected push and lost replay | `TestWriteAdopt` rows `:298-436` |
| `WriteHold` | `TestWriteHold` rows `:437-539` |
| Takeover entry write, rejected-push retry, archive, conflict abort and teardown lock | `TestTakeOverRun` rows (`takeoverrun_integration_test.go:84-286`) |
| Hand-over mark dropped on takeover | `takeoverrun_integration_test.go:89-106` and `TestDropHandOverMark` |
| `OriginParentRunStatus` | `TestOriginRunStatus` (`:333`) |
| Moved-run test and steps 2 and 3 skipped for own owner with no takeover entry | `TestAdopt` rows `:487-505` |
| Re-run after a kill follows `--take-hold` | `TestAdopt` row `:558` |
| Hold re-read before each step | `TestAdopt` rows `:573`, `:584`, `:595` |
| Own `adopt-fix` re-run keeps kind, fix task and tries and files no second task | `TestAdopt` row `:519`, and `TestRunHoldCheck/AnOpenOwnFixTaskFilesNoSecondTask` (`holdcheck_test.go:416`) |
| Step order, catch-up and workspace failures, deploy, `--no-orch`, held beam | `TestAdopt` rows `:602-739` |
| Green check clears the hold, drops the `bolt-hold` entries and calls BeamUp | `TestRunHoldCheck/AllGreen...` (`:204`) and `TestHubHoldCheckDeps_DropBoltHoldLeavesNoBoltHoldEntry` |
| Try counts read from the hold | `TestRunHoldCheck/TheTryCountIsReadFromTheHold` (`:384`) |
| Limit notice | `TestRunHoldCheck/AtTheLimit...` (`:399`) |
| Hold lost during check | `TestRunHoldCheck` rows `:222-284` |
| `--take-hold` shell-guard row | `rows_integration_test.go:226-232`, `:311` |
| Steer and notice avoid the flag | The claudeengine settings tests at `settings_test.go:480` and `:640` check the steer is installed; nothing visible asserts the steer and notice omit `--take-hold` |
| `--here` `owned_here` and `no_subject` | `TestCommand_HereAdoptsTheSubjectOfItsWorktree` |
| Beam status shows the Bolt hold | `TestAdoptSectionOf` and `TestStatus_PrintsThePrimesDobbyAndEachLocalBeams` |
| Orch watcher reads the Bolt hold | `TestUnderOwnHold` |
| Dobby handover reads the hold | `TestHubDobbyDeps_ABeamWithNoOwnerOfItsOwnIsUnderARootAdopt` |
| Catch-up over hub run slugs | `TestHubCatchUp` |
| Wrong-subject check refused | `TestRunHoldCheck/ASubjectOutsideTheHoldRefusesNamingTheHoldsSubjects` (`:190`) |
| Standing anchor change refused | `TestAdopt` rows `:361-367` |
| `adoptFields` | `TestAdoptFields` |
| Second clone takes over end to end | `TestAdoptIntegration_SecondCloneTakesOver` |
| Retired flags and worker gone | no test; deletions only (assert-no-callers) |

**Findings**

1. **MEDIUM, design.** No visible test asserts that the `adopt-take-hold` row's steer and notice omit `--take-hold` and name asking the operator.
   - The D6 acceptance criterion requires both properties, and `PATTERN-shell-guard-table` requires the same.
   - Location: `internal/shellguard/rows.go` (the new row); `internal/shellguard/rows_integration_test.go:226`.
   - Evidence: the only tests of the row are the pattern matrix, `:226-232` and `:311`, and the claudeengine installed-steer checks.
   - A grep of `internal/shellguard/*_test.go` and the claudeengine tests found no `--take-hold` assertion on the steer or notice text.
   - A generic init check on quotes and backslashes may exist in `rows.go`; if no generic "no flag spelled" scan covers this row, the criterion is unpinned.
   - Caveat: I did not search the `no-denied-recovery` scan, so the handler should confirm it does not already cover this.

2. **LOW, scope.** The `Retired:` list (adopt record, worker and the rest) has no negative coverage, which is acceptable for deletions.
   - Not a finding unless the affected-set lens finds a leftover caller.
   - A `pauserecord.AdoptCaller` / `HoldCaller` reader remains for legacy entries, with tests at `record_test.go:360-380`.

I did not run the test economy candidates. `TestAdoptFields` is already marked kept in the redundancy report.

**Tool calls (3 of 6)**
1. Grepped test names and rows in the adoptcli, boardengine and fabricengine tests, since my context held only the code and not the tests.
2. Read the git diff of the shellguard, orchcli, dobbyengine and battencli tests, and the `TestAdopt` rows, because the first grep missed the `TestAdopt` rows.
3. Grepped for `take-hold` across the shellguard and claudeengine tests, plus the replay rows and hand-over mark tests, to settle finding 1.
