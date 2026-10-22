# Plan fixer report, round 1

Every finding from F1 to F9 is fixed.
This is a re-run after the loom halted at Plan-Burler-Fix.
In the earlier attempt, a self-check fork ran `git init` and `git worktree repair` in a scratch repository under `/tmp`, and nothing under the target was written by that fork.
The cards already held the earlier attempt's fixes, so this session checked each finding's premise against the code and confirmed each fix in the cards.
It then ran its own self-check and fixed the two findings that came back.
`lyx loom validate-plan` answered ok after the self-check edits.

## Changes

- F1 (card 10), fixed as proposed: the `TestWatcher_NoteRequestRendersTemplateFileBeforeTyping` bullet drops every assertion on stencil literals and plans no new one.
  It keeps the Go-filled template and `PendingHandoff` paths, and checks the family rule both ways: the hard request offers no `defer` and the soft one does.
  Review, not a test, guards the reworded stencils.
- F2 (card 7), fixed differently: the review proposed naming the repair step only for a gitfile `.git`.
  That part was made: a gitfile's Warn line names `git worktree repair <path>`, and the line ends the skip only when the admin directory stands.
  A `.git` directory's Warn line names the cause, `.git is a directory, not a worktree link`, and no repair step.
  The empty-`.git`-directory row asserts that line, and a new gitfile row asserts the repair line.
- F3 (card 11), fixed as proposed: when `ClearResumeNoticed` fails in `withdrawQueuedCompact`, the tick fails, the phase stays compacting, `Pending` is cleared and `ResumeLineWithdrawn` stays set.
  A qualifying boundary with `ResumeLineWithdrawn` set re-runs the clear before it completes.
  The unwritable-record row drives the following tick through that clear to completion.
- F4 (card 5), fixed as proposed: `internal/reedcli#reedCLI.statusCmd` is in the Edit list.
  Each row gains an `elsewhere` key, and the `Long` help names it.
  `internal/reedcli/doc.go` lists no row keys, so it stays unchanged.
- F5 (card 11), fixed as proposed: the Intent says `tick` runs `recoverPending` before `tickCompacting`, so the no-id branch holds only after a `Sent` or `Inject` failure in the same tick.
  A test row drives that case.
- F6 (card 3), fixed as proposed: `internal/shuttleengine#strandRemovable` is in the Edit list, and only its doc comment changes.
  The self-check widened this fix to the rule's two other statements (see Self-check).
- F7 (card 9), fixed as proposed: `unpausedSince` returns `pauserecord.AwakeSince(from, now, coverage.Intervals)`.
- F8 (card 4), fixed as proposed: the `Live: false, OwnWindow: false, Present: true` row is dropped.
- F9 (cards 3 and 12), fixed as proposed: the nil-`paneAge` fallback and its test row are gone, and `hubPauseHolds` has no empty-path branch.

## Self-check

- fix-affected-set, LOW, fixed as proposed: the removability rule F6 rewords is stated twice more, in `Runner.StopStepRuns`' doc comment (`internal/shuttleengine/stepstop.go:44`) and in the Stop verb section of `internal/shuttleengine/doc.go:177`.
  Card 3's Edit list now names both, and its Status bullet says they change the same way.
- fix-consistency, LOW, fixed as proposed: card 11 said the retry runs "even when the `/compact` outcome then reads withdrawn".
  After a clear failure `Pending` is nil, so no outcome is read.
  The sentence now says the precondition fails on the nil `Pending` and no withdraw is attempted.
- fix-claims: no findings.
- fix-proposal: no findings.

## Disputed

None.

## Deferred

None.
