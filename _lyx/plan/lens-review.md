# Lens review — umbrella-anchor plan

Every lens fork returned a report; each is saved beside this file as `lens-review.lens-<lens>.md`.
Duplicates across lenses were merged and are marked as such on the later line.

## forced-edits

- FE1, MEDIUM, `anchor_stopped` missing from `cleanupcli`/`adoptcli` state switches: fixed as proposed; card 11 now edits `cleanupcli#sharedStopText`, `adoptcli#sharedStopText`, `adoptcli#stoppedBranches` and `adoptcli/doc.go`, with `TestAdopt` rows.
- FE2, MEDIUM, `ErrAnchorChange` has no push failure class: fixed as proposed (merged with WF1); card 9 adds `PushFailureAnchorChange`, `AnchorStopError`, a `classifyFailure` case ahead of the not-fast-forward one, `FailedPush.WayForward` and its `FailedPushWayForward` case; card 13's push stop uses the same class.
- FE3, MEDIUM, nil meaning of the new `MoveInProgress` seams: fixed as proposed; cards 16 and 19 state a nil seam reads as no record standing, and card 19 lists `adoptcli#world.deps` (merged with RV4).
- FE4, LOW, `ResolveWithAnchor`'s docs say two callers: fixed as proposed; card 18 edits `lyxcwd#ResolveWithAnchor`'s doc and `docs/shared-libs/lyxcwd.md`.
- FE5, LOW, lyxcwd's import cap stated elsewhere: fixed as proposed for the `leaf_enforcement_test.go` file comment (card 1 Intent); disputed, premise false, for `pattern/PATTERN-leaf-packages.md`, whose line 7 states `gitkit`'s import set, not `lyxcwd`'s.
- FE6, LOW, `pairLocation` caller names unverified: disputed, premise false: `lyx quarry resolve` found `internal/beamcli#dobbyRuns` (`dobbydeps.go:322`) and `internal/beamcli#dobbyWatchDirs` (`dobbywatch.go:17`), the enclosing functions of both calls.
- FE7, LOW, no role-guard registration needed: disputed, premise false: the report states no defect (`fabriccli/roleguard.go` needs no entry).

## coverage

- CV1, MEDIUM, push empty-marker refusal unreached: fixed as proposed; card 9's `TestPushFailureClasses` row for an empty own marker.
- CV2, MEDIUM, pair teardown at two anchors untested: fixed as proposed; card 4 adds two `TestRun_TeardownScenario` steps (a pair at `sub`, and an invalid own marker for the fallbacks).
- CV3, MEDIUM, card 2's new branches unreached: fixed as proposed; card 2 creates `TestDobbyRuns_PairAtItsOwnAnchor` (own anchor and invalid marker for `dobbyRuns` and `dobbyWatchDirs`), and card 4's teardown step reaches `driverLockBusyError`'s fallback.
- CV4, MEDIUM, `Menu`, `runReconcile` and `taskAnchor` fallback untested: fixed as proposed; card 4 adds a `TestMenuScenario` row, creates `TestRunCLI_ReconcileFromAPairAtAnotherAnchor`, and its invalid-marker teardown step reaches `taskAnchor`'s fallback.
- CV5, MEDIUM, exclude lines after a re-anchor unchecked: fixed as proposed; card 15's move row asserts the `.`-anchored lines another worktree needs stay.
- CV6, MEDIUM, child landing into the parent not driven: fixed differently: the proposed full batten landing is not driven, since `fabricengine/doc.go`'s merge surface keeps the weft out of every merge, so the code merge is what can touch the parent; card 17's row merges the child into the parent with `lyx fabric merge` and asserts the parent's marker and anchor unchanged, after the child resolves and commits at its own anchor.
- CV7, MEDIUM, Windows live-dobby refusal unreachable: fixed as proposed; card 17's `anchorDeps` gains a `GOOS` seam and a `TestChangeAnchor` row.
- CV8, MEDIUM, both-`.lyx` recovery refusal untested: fixed as proposed; card 15 row.
- CV9, MEDIUM, owner refusal's allowed branch untested: fixed as proposed; card 17 row.
- CV10, MEDIUM, spared push and sync roles unasserted: fixed as proposed; `TestChangeAnchor` asserts the dobby end spares them.
- CV11, MEDIUM, `treeAnchor` fallback and tie untested: fixed as proposed; card 8 creates `TestTreeAnchor` through `TreeAnchorForTest`.
- CV12, LOW, adopt of a branch with an empty committed marker: fixed as proposed (merged with GR5): card 8's adopt path counts an empty committed marker as none, as D8's census does, and writes D5's value, with a row.
- CV13, LOW, gone-weft branch unreached: fixed as proposed; card 3 adds a `TestRemove_TearsDownNestedJunction` subtest.
- CV14, LOW, D6 landed re-run untested and the hook row would roll the pair back: fixed as proposed; card 18 builds both kills by hand (before and after the commit) and drops the hook from its Uses.
- CV15, LOW, integration failure before the branch moves untested: fixed as proposed; card 15 row.
- CV16, LOW, no dobby start for a not-owned or lock-held beam untested: fixed as proposed; card 17 row.
- CV17, LOW, log sink order untested: fixed as proposed; card 17's order row includes the sink close, lock-directory open and reopen at `B`.
- CV18, LOW, reworded bootstrap message and probe fetch error unreached: fixed as proposed for the message (card 6 lists `TestCloneHub_OldOrderInvocationIsRefused`); disputed, premise false, for the fetch error, a propagated git error `refusal-spec.md` exempts as transient.
- CV19, LOW, prime-first migration and unwire leaving the marker unasserted: fixed as proposed; card 8's test asserts both.
- CV20, LOW, origin-only target branch untested: fixed as proposed (merged with GR1).
- CV21, LOW, never-tracked `A/.lyx` exemption untested: fixed as proposed; card 15's move row puts an untracked file in `.lyx` first.

## guard-reach

- GR1, BLOCKING, checkout misses a target branch only on origin: fixed as proposed; card 10 reads the target tip from origin through `resolveWeftBranch`'s probe and fetches it into the tracking ref only, with a `hubforge.CloneSecond` row.
- GR2, MEDIUM, promote compares working-tree markers, wrong when the parent pair is absent: fixed as proposed; card 12 creates `fabricengine.ParentRecordsAnchor`, which compares against `treeAnchor` of the parent records tip `recordsBranchTip` reads, with `TestParentRecordsAnchor` including a parent with no worktree.
- GR3, MEDIUM, board-tidy agent still spawns while a record stands: fixed as proposed; card 16's tick also skips `tidyStep`, with a row.
- GR4, LOW, `beam create` re-run without `--anchor` skips recovery: fixed as proposed; card 18 calls `EnsureAnchor` on every present pair, an empty anchor meaning the pair's own.
- GR5, LOW, adopt over an empty committed marker: merged with CV12, fixed there.
- GR6, LOW, checkout not refused while a move record stands: fixed as proposed; card 13 refuses it and adds a `TestCheckout_RefusesAnotherAnchor` row.

## way-forward

- WF1, BLOCKING, failed push stop records the not-fast-forward way forward naming `git merge`: merged with FE2, fixed there; the row asserts class `anchor_change` and no `git merge` in its way forward.
- WF2, MEDIUM, `--set` and `lyx batten cancel` refuse each other for a pair with a live run: fixed as proposed; card 11 lets `battencli#judgeOrigin` pass an entry stopped only by `anchor_stopped` branches, with `TestCancelRun` rows, and card 17's refusal names that cancel.
- WF3, MEDIUM, promote's `--set` way forward is refused for a held run and cancel destroys the promote: fixed differently: the proposal names the conflict only; card 12 moves the check into `promoteSteps` right before `ResetRecords`, after promote's own steps end and clear the run, so the `--set` it names passes and the re-run skips the done steps.
- WF4, MEDIUM, adopt and `--set` point at each other on a recovery owned elsewhere: fixed as proposed; card 17 exempts a worktree with a standing record from the owner refusal, bounded to the record's own `B`.
- WF5, MEDIUM, `EnsureAnchor`'s refusal names no re-run of `beam create`: fixed as proposed; it names the `--set` then `lyx beam create <slug> --anchor <anchor>`.
- WF6, LOW, three prime-only refusals name no next step: fixed as proposed; card 17 names `lyx reed down`, waiting for or pausing each named batten run, and waiting for the gate slot's holder, each then the `--set`.
- WF7, LOW, adopt refusal names no adopt re-run: fixed as proposed; card 19 names `lyx adopt` again.

## row-validity

- RV1, BLOCKING, `TestDobbyGeometry`'s existing beam row breaks: fixed as proposed; card 4's existing row writes the beam's own marker `sub`.
- RV2, BLOCKING, the new two-worktree step breaks the stale-marker step: fixed as proposed; card 1's step runs last.
- RV3, MEDIUM, card 5's rows pass on today's code: fixed as proposed; card 5's discriminating row keeps a legacy bare `_lyx` line that today's census strips.
- RV4, MEDIUM, fake deps builders missing from cards 12, 18 and 19: fixed as proposed; `promoteWorld.deps`, `createWorld.deps` and `world.deps` are listed with their defaults.
- RV5, MEDIUM, prime-record propagation to beams untested: fixed as proposed; card 16's `TestDobbyGeometry` drives `DobbyStart(prime).MoveInProgress`, and the `fakeStart` row that could not fail was replaced by a read-error row.
- RV6, MEDIUM, `newDivergedWeftFixture` cannot skip the local commit: fixed as proposed; an empty `localRel` means none.
- RV7, MEDIUM, target anchors are not directories git tracks: fixed as proposed; `ReanchorPair` commits a placeholder, and the card 10, 15, 17 and 18 tests commit `<anchor>/.keep` in the prime's warp first.
- RV8, MEDIUM, the prime `--set`'s dobby start fails under test: fixed as proposed; card 17 reports a failed start after `Finish` with the way forward `lyx beam start` and exits 0.
- RV9, MEDIUM, card 8's push spawn, network assertion and markerless fixture were unspecified: fixed as proposed; it uses `SwapPushRecorderForTest`, points the weft remotes at a missing path, removes markers with a commit, and `ReanchorPair` covers the prime.
- RV10, MEDIUM, the child-landing row reaches no landing: merged with CV6, fixed differently there.
- RV11, LOW, the fast-forward push row cannot fail: fixed as proposed; the row is dropped.
- RV12, LOW, card 7's re-anchored-parent row helper unstated: fixed as proposed; the row commits run records in the beam pair's own records worktree and uses `--parent`.
- RV13, LOW, `newLoopHarness` and the due-run fakes unnamed: fixed as proposed; card 16 uses `newLoopHarness` with faked `Runs` and `SpawnPass`.
- RV14, LOW, `CloneSecond` not under Uses: fixed as proposed on cards 15 and 17.
- RV15, LOW, the sweep test's new case should be a `from` value: fixed as proposed; card 3 adds a third `from` value.
