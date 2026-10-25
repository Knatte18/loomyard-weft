# Fixer report — plan, round 1

F10 and F13 are `decision` findings, left to the parent; its Added after Discussion entries keep cards 17 and 13 as written, and neither card changed for them.
`lyx loom validate-plan` passes on the edited plan.

## Changes

- **F1** (fixed as proposed): cards 9 and 13 add `Fabric.fastForwardWeft` and `TestFastForwardPair_MovesBehindPairToUpstream` to their Edit lists.
  Card 9: once the weft has an upstream, it refuses an empty own marker, fetches and runs `weftAnchorStop` before `PullWeft`, and a stop skips the weft pull with its existing warning while the code side goes on.
  Card 13 skips the pull the same way while a record stands.
  Each card adds a test row, and the overview's Stops bullet names the fast-forward.
- **F2** (fixed differently, in part): card 17's owner refusal also names `--set <the fetched upstream tip's marker>`, and TestChangeAnchor's owner-refusal row asserts it.
  The unfinished-run refusal does not offer that `--set` as a standalone step, because it would refuse the same `--set` again.
  Instead it says that, for an owned-elsewhere worktree, the `--set` after `lyx batten cancel` is to the upstream marker.
  For a present worktree with a reachable origin, the cancel itself runs (`internal/battencli/cancel.go:384-391` refuses an owned-elsewhere slug only when the worktree is absent or origin is unreachable).
  The overview's way-forward bullet names the new step.
- **F3** (fixed differently): card 15 re-signs `Apply` to `(landed bool, err error)`.
  A failure after the commit, or after the integration moved the branch, leaves the record standing and returns an error naming `lyx fabric anchor <slug> --set <B>`.
  The hook becomes `anchorFailHook(afterCommit bool)` with `SetAnchorFailHookForTest`.
  Card 17 restarts dobbies only when `landed` is false; when it is true, it releases the parent lock and returns the error, and the re-run's `Finish` starts the dobby.
  Both tests gain the after-commit row.
  The review proposed no way for card 17 to tell the two failures apart, so the `landed` result is my addition.
- **F4** (fixed as proposed): card 15's `validateAnchorTarget` names a next step for every refusal: another directory, the warp update, committing the directory, or moving the wired name out of `B`.
  Card 18 says its `--anchor` refusals carry the same text, and both tests assert it.
- **F5** (fixed differently): the fallbacks and card 4's invalid-marker teardown step are dropped, as are card 2's sentence that card 4 reaches the fallback.
  The texts are not built from a Location the teardown already resolved, because `driverLockHeld` is a seam that the in-package tests fake with no real worktree (`teardown.go:110,170`, `cancel_test.go:46`).
  So a `ResolveWorktree` there would break those tests.
  Instead `driverLockBusyError` returns the `NamedWorktree` error in place of the refusal.
  `taskAnchor` is re-signed to `(string, error)`: `busyError` returns the error, and `archivePairLogs` warns and archives nothing.
  Card 4's Edit list gains `busyError`, `archivePairLogs` and `pairWithLogs`.
- **F6** (fixed differently): card 6 puts `commitAnchorMarker` in a new `anchormarker.go`.
  It writes through an `os.Root` at the weft root (`root.WriteFile`, as `hubworkspace_write.go:39`), and card 15's `moveWeftContent` writes its marker the same way.
  So neither needs an allowlist entry.
  The allowlist's doc comment (`cmd/lyx/uncontainedwrite_test.go:56-62`) admits a new raw write only into a git-owned path or a directory minted in the same call, and an existing weft is neither.
  Card 6 lists `cmd/lyx#uncontainedWriteAllowlist`, the variable that holds the entries, rather than the test function, and drops the marker from the `clone.go` entry's reason, since clone.go no longer writes it.
  Card 15 needs no allowlist edit.
- **F7** (fixed as proposed): card 9's `weftAnchorStop` gains a `reset bool` parameter that ignores ancestry.
  Card 10's `DropLocal` sets it, and `TestDropLocal` gains the unpushed re-anchor row.
- **F8** (fixed as proposed): card 15 creates `moveWeftContent`, the step-4 move that `Apply` and card 18's Add share.
  Add runs it after the fork and does not call `BeginAnchorChange` or `Apply`, so their refusals do not apply.
  A markerless fork point moves from its resolved anchor, and TestAdd_AtAnchor gains that row.
  Card 18's `EnsureAnchor` recovers through `BeginAnchorChange`, `Apply` and `Finish`.
- **F9** (fixed differently, in name): the restore and the deletion of untracked paths both go through one new `destroy.go` executor.
  The proposed name `removeUntrackedMovePaths` describes only the deletion, so it is `restoreMovePathsToHead(rec, weftPath, pathspecs)`.
  Its gates are containment, the move's pathspecs and `dirtinessNA`.
- **F11** (fixed as proposed): card 18's Add removes the record before it returns, ahead of `createSteps`' push, workspace and dobby steps.
- **F12** (fixed as proposed and widened): card 6's Edit list gains `codebinding.go`, `Unwire`, `Command`, `staleFabricAnchorName` and `internal/fabriccli/doc.go`, and its docs sentence covers them and `CloneAndWire`'s doc.
  The `unwire.go` file entry is dropped in favour of the `Unwire` member glyph, as `validate-plan`'s redundant-file-target check requires.

## Self-check

- fix-claims, MEDIUM, the allowlist bound claim in card 6: fixed as proposed; the reason now rests on the allowlist's doc comment.
- fix-claims, MEDIUM, card 6 said `codebinding.go`'s header mentions unwire: fixed as proposed; the sentence is reworded.
- fix-claims, LOW, card 15 named no ownership constructor for the move's pathspecs: fixed as proposed; it creates `ownedMovePathspecs`.
- fix-affected-set, LOW, card 6 missed stale `weft:main` sites (`Unwire`, `CloneAndWire`, `fabriccli/doc.go`): fixed as proposed.
- fix-consistency, LOW, card 18's record removal against D6's wording: fixed as proposed; card 18 reads D6's "remaining create steps" as Add's own.
- fix-consistency, LOW, TestAdd_AtAnchor asserted a workspace folder that Add does not write: fixed as proposed; the assertion is dropped.
- fix-consistency, LOW, the card 9 index line and card 13's title did not name the fast-forward: fixed as proposed.
- fix-consistency and fix-proposal, LOW, `EnsureAnchor` did not handle `Apply`'s `landed` result (duplicate across both lenses): fixed as proposed; the error names `lyx beam create <slug>` again, and `Close` releases the lock.
- fix-proposal, LOW, the owner refusal named a no-op `--set` when the upstream marker equals the checked-out anchor: fixed as proposed; that clause now appears only when the two differ, and the test rows cover both cases.
- fix-proposal, LOW, card 15's warp-commit next steps named no cost: fixed as proposed.

## Disputed

None.

## Deferred

None.
