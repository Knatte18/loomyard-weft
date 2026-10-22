# Plan fixer report, round 1

Every finding F1 to F9 was fixed.
`lyx loom validate-plan` answered ok after the fixes and again after the self-check edits.

## Changes

- F1 (card 10), fixed as proposed: the `TestWatcher_NoteRequestRendersTemplateFileBeforeTyping` bullet drops every stencil-literal assertion and plans none, keeping the Go-filled template and `PendingHandoff` paths and the hard-versus-soft `defer` family rule; review guards the reworded stencils.
  The self-check widened the family rule to both directions (see Self-check).
- F2 (card 7), fixed as proposed: a `.git` directory's Warn line names the cause, `.git is a directory, not a worktree link`, and no repair step; a gitfile's Warn line names `git worktree repair <path>`.
  The empty-`.git`-directory row asserts the cause line and no repair, and a new row with a gitfile naming an existing empty directory asserts the repair line.
- F3 (card 11), fixed as proposed: a `ClearResumeNoticed` failure in `withdrawQueuedCompact` fails the tick with the phase compacting; a qualifying boundary with `ResumeLineWithdrawn` set re-runs the clear before completing, even when the outcome reads withdrawn.
  The unwritable-record row now drives the following tick to clear `ResumeNoticed` and complete.
- F4 (card 5), fixed as proposed: `internal/reedcli#reedCLI.statusCmd` joins the Edit list; each status row gains an `elsewhere` key and the `Long` help names it.
  `internal/reedcli/doc.go` lists no row keys, so it is unchanged.
  The ImpactSummary names `lyx reed status`.
- F5 (card 11), fixed as proposed: the Intent states that `tick` runs `recoverPending` before `tickCompacting`, so the no-id branch holds only after a same-tick `Sent` or `Inject` failure.
  A row where the `/compact` inject failed and `Sent` fails in the boundary tick asserts the withdraw runs.
  `Sent` failing alone is what leaves no id: `adoptPending` holds the step through `creelFailed`, and `Inject` is never reached.
- F6 (card 3), fixed as proposed: `internal/shuttleengine#strandRemovable` joins the Edit list, doc comment only; the Status bullet, the ImpactSummary and the overview's shared decision on `Present` say so.
- F7 (card 9), fixed as proposed: `unpausedSince` returns `pauserecord.AwakeSince(from, now, coverage.Intervals)`.
- F8 (card 4), fixed as proposed: the `Live: false, OwnWindow: false, Present: true` row is dropped.
- F9 (cards 3 and 12), fixed as proposed: the nil-`paneAge` fallback and its `TestEngineAliveIDs` row are dropped, since `New` sets the seam and tests build engines through `newTestEngine`; `hubPauseHolds` loses its empty-path branch.

## Self-check

- fix-consistency, MEDIUM, fixed: on a clear failure card 11 saved `Pending`, so a no-id `/compact` could be re-sent by the next tick's `recoverPending` after the boundary. `withdrawQueuedCompact` now clears `st.Pending` in that save, and the row asserts `Pending` nil.
- fix-proposal, MEDIUM (card 10 soft `defer`), fixed: the family rule is now pinned both ways, so the hard request offers no `defer` and the soft one does, a fragment Go chooses under `PATTERN-test-economy`.
- fix-proposal, MEDIUM (card 7 gitfile repair cannot end the skip), fixed differently: the proposed change was to name no repair step on any Warn line. I did not make it, because the fix-claims fork ran `git worktree repair` on the planned fixture, a gitfile naming an empty directory, and found that it rewrites the link and the worktree resolves again.
  The premise holds in part, since repair cannot fix a missing or unreadable admin directory. So the gitfile Warn line now names repair as the step to try, which ends the skip when the admin directory stands, and promises nothing for that other case.
- fix-claims: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.
