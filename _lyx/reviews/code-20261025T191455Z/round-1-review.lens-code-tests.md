```yaml
lens: code-tests
outcome: reported
```

Task: the code-tests lens, listing the test that pins each behaviour the diff adds or changes, or none.

**Per behaviour, the test that pins it** (from test names and subtest rows only; I did not open test bodies)

| Behaviour | Test |
|---|---|
| lyxcwd own-marker read, fallback order, invalid marker | `internal/lyxcwd/anchorread_test.go` and `lyxcwd_test.go` (modified, not opened) |
| Scan for `Location` literals | `TestEnforcement_LocationLiterals` |
| Dobby geometry at each worktree's own anchor | `TestDobbyGeometry`, `TestDobbyRuns_PairAtItsOwnAnchor` |
| Remove / exclude census per anchor | `remove_junctions_integration_test.go` rows `PairAtItsOwnAnchorRemovedFromThePrime`, `PairWhoseRecordsWorktreeIsGone` |
| `Add` takes the fork point's anchor, refuses an empty marker and a standing move record | `addparent_integration_test.go` rows `EmptyForkPointMarker`, `MoveRecordOnForkPoint_LockFree`, `MoveRecordOnForkPoint_LockHeld`, and `add_runrecords_integration_test.go` row `ParentPairAtAnotherAnchor` |
| `Add --anchor` (D6) | `TestAdd_AtAnchor`, `TestEnsureAnchor` |
| Reconcile migrates markers and removes the Bolt marker | `TestReconcile_MigratesMarkerlessWefts` |
| Reconcile refuses an empty marker per pair | `TestReconcile_RefusesEmptyAnchorMarkerInsteadOfWiringAtTheRoot` |
| Reconcile from a pair at another anchor | `TestRunCLI_ReconcileFromAPairAtAnotherAnchor` |
| Weft pull stops on a marker change, an empty marker, or a move record; markerless tips and an unpushed re-anchor do not stop it | `pull_integration_test.go`, five subtests |
| Replaying push stops | `pushclasses_integration_test.go`, rows `OntoAnUpstreamCarryingAnotherMarker`, `WhileTheOwnMarkerIsEmpty`, `WhileAMoveRecordStands` |
| Checkout refuses another anchor | `TestCheckout_RefusesAnotherAnchor` |
| Drop-local refusal | `droplocal_integration_test.go`, rows `OriginsTipCarriesAnotherMarker`, `LocalWeftAheadWithAReAnchorNotYetPushed`, `OwnMarkerEmpty` |
| Origin reconcile reports `anchor_stopped` and the code branch still moves | `originreconcilemoves_integration_test.go`, two subtests; `parenttips_integration_test.go`, three new rows |
| Promote refuses a task at another anchor than its parent's | `promote_test.go`, rows `TaskAtAnotherAnchorThanItsParentsRecordsRefusedAfterTheRunIsEndedAndBeforeTheReset`, `UnreadableAnchorsRefusedNamingTheSlugAndTheRerun` |
| Batten cancel passes an entry stopped only by `anchor_stopped` | `cancel_test.go`, two rows |
| Move record read, remove, and `MoveInProgress` | `TestMoveRecord_ReadAndRemove`, `TestMoveInProgress` |
| Dobby loop starts no pass while a move stands | `TestRun_NoPassWhileAMoveStands` |
| `EnsureLive` skips and reports dobbies while a move stands | `dobbyengine/start_test.go` (modified, rows not opened) |
| Integration across anchors | `TestIntegrateWeft` |
| Anchor-change engine, move, recovery and failure paths | `TestAnchorChange` |
| `fabric anchor` verb, refusals and wait | `TestChangeAnchor`, `TestRunCLI_Anchor` |
| Adopt refuses during a move | `adoptcli/adopt_test.go` (+54 lines, subtests not matched by my grep) |
| `treeAnchor` (D5 markerless value) | `TestTreeAnchor` |
| `parentRecordsAnchor` | `TestParentRecordsAnchor` |
| `readTree`-style marker reads in clone (D4), `probeWeftBinding` reading `main-weft` | `clone_adopt_test.go`, `warpbinding_clone_integration_test.go` (modified, rows not opened) |
| Hub workspace files at each worktree's own anchor | `hubworkspace_integration_test.go` (modified, rows not opened) |
| Launcher move (D7 step 6) | none found by name |

**Findings:** none that meet the "name the covering test" bar. I could not confirm any duplicate test, and I found no new behaviour without a test.

**What I did not verify**
- I did not open any test body, so I did not check that each row fails if its behaviour breaks.
- I could not tell whether `TestEnsureAnchor` duplicates `TestAdd_AtAnchor`, or whether `TestMoveRecord_ReadAndRemove` and `TestMoveInProgress` overlap with the `TestAnchorChange` coverage of the move record.

Tool calls (2):
1. `git diff ... -- '*_test.go'` filtered to file names and `Test*` functions: my context held no test files.
2. `git diff` over eleven test files filtered to `t.Run` and row names, plus a `grep` for the stop helpers' direct tests: to get subtest rows my context lacked.
