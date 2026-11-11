# Fixer-report, code review round 1

Gate: `lym gate test --tier 0 --tier 1` over `internal/slackengine`, `internal/slackcli` and `internal/boardengine` passed after the fixes;
`internal/slackengine` passed again after the self-check fixes.
A later gate flagged one comment line of `wakeLoop.run` in `relay.go` that reached 120 characters;
it is split into two sentences on two lines, a comment-only change.

## Changes

- F1, fixed as proposed (with one refinement):
  `wakeLoop.wake` in `internal/slackengine/relay.go` re-reads the forwarding file after the retire and sets `exiting` when no button is on, so `RunRelay` returns nil.
  It re-reads instead of using the list read before the retire, because the retire itself drops the last button.
  The `RunRelay`, `run`, `wake` and `exiting` comments name the new exit.
  New row `RelayEndsWhenTheRetireDropsTheLastButton` in `TestRunRelay/RetiredBeams`.
  No existing relay test started with every button off.
- F2 and F5, fixed as proposed, in one change:
  `channelKeeper.retire` in `internal/slackengine/keeper.go` also walks the forwarded orchs that have no map entry, and drops the button of each unlisted beam's orch that has no open channel.
  The drop runs only for an orch whose button is on, so it never rewrites the file for nothing and wakes no relay.
  A failed drop after an archive is retried at the next retire pass.
  New `Retire` rows in `keeper_test.go`: a pressed beam with no entry and an archived entry with its button on.
- F3, a ruling from the parent (see Rulings).
- F4, fixed as proposed: `TestWorktreeOrch` and `TestOrchProject` in `internal/slackcli/wiring_test.go` over a real hub.
  `TestWorktreeOrch` covers the prime, a listed beam (a pair made with `Beam: true`), a pair that is no beam and an unlisted worktree.
  `TestOrchProject` covers the empty slug, a beam with a project entry, a beam with none and an undecodable board.
- F6, fixed as proposed: `pattern/PATTERN-daemon-wakeups.md` names the inbox and forwarding-file watch.
- F7, fixed as proposed: `PressOfAButtonIsSeenOnDiskWhenTheWatchCannotBeOpened` in `TestRunRelay`.
- F8, fixed as proposed: a `TestHook` row for the beam orch `hub:b:orch` with its own button on.
- F9, fixed as proposed: an inbound row with an undecodable forwarding file that holds the message and posts nothing.
- Own change, forced by F2: the three `RetiredBeams` rows (`UnlistedBeamsChannelIsRenamedAndArchivedAtAWake`, `FailingBeamsReadRetiresNothing`, `RefusedArchiveIsRetriedAtTheNextOutboundWakeAndNotAtAFileEventWake`) woke the relay by pressing the button of an unlisted beam.
  Retire now drops that button, which added a wake and made the third row flaky, so they wake the relay with a hint on the prime's channel instead.

## Disputed

None.

## Deferred

- Self-check finding on `rootProjectHandle = "P0"` duplicating the board's unexported root handle (lenses fix-code-correctness and fix-proposal): the only sound change is an export from the board module, which the parent's ruling on F3 refuses.

## Rulings

- F3, ruling recorded by the parent: refuse the exported `boardengine.RootProjectHandle`; derive the root part of the channel name inside the Slack packages; the test of `orchProject` named in F4 covers it.
  Carried out: `RootProjectHandle` is removed from `internal/boardengine/render.go`, and `internal/slackcli/wiring.go` holds the constant `rootProjectHandle = "P0"`.
  `TestOrchProject` pins the root row.
  Nothing was left undone, and no flag went to the parent.

## Self-check

- fix-code-correctness, LOW, `retire` returned early on a forwarding read error: fixed with my own change.
  The error joins the failures and the channels still retire; a new `Retire` row covers it.
- fix-code-correctness, LOW, `spareOrch` comment incomplete: fixed.
  Merged with the same finding of fix-consistency (LOW) and fix-affected-set (LOW), and with the same note of fix-proposal.
- fix-code-correctness, LOW, `rootProjectHandle` duplicates the board's root handle: deferred, see above.
  Merged with fix-proposal's LOW.
- fix-consistency, LOW, `RunRelay` doc missing the new exit: fixed.
  Merged with fix-proposal's LOW.
- fix-proposal, LOW, same `RunRelay` doc: fixed, merged above.
- fix-affected-set: no further finding; it checked F1, F2, F3, F5 and F6.
- All four forks returned a report.
- fix-code-correctness noted that `TestRunRelay/WatchThatFailsToOpenFallsBackToTheOnDiskCheckWithItsOwnBackoff` failed once under `-count=5` load and passed 40 of 40 alone, and I saw the same once; I judge it a race of that existing row and left it.
