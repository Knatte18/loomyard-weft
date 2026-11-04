# Fixer-report, code review round 1

All eleven findings are fixed.
Verification: `lym gate test` over `internal/creelengine`, `internal/boardengine`, `internal/battenshed`, `internal/dobbyengine`, `internal/battencli` and `internal/boardcli` passes at tier 0, and `internal/boardcli` passes at tier 1.
Commits are grouped by touched files, not one per finding, because several findings share files (`landnotice.go`, `landnotice_test.go`, `cli.go`) and a hunk cannot be split without an interactive add.

## Changes

- **F1 — fixed differently.**
  Proposed: leave any hub orch holder alone in `sweepOnce` while no strand of that name is listed.
  Not made: that also stops returning unread mail from a strand or an EXTERNAL sender to a removed orch, and breaks the existing rows `RemovedHolderIsSettledWithin60sAtTheCeiling` (mail to `ly:orch` returns to `alpha`) and `PostInsideTheConfirmIntervalKeepsTheConfirmDeadline` (an EXTERNAL waiter is answered returned) in `internal/creelengine/deliver_test.go`.
  Made: `Store.waitsForHubOrch` in `internal/creelengine/sweep.go` names unread mail from a program, not EXTERNAL, that no strand forwarded, held under an orch name of this hub.
  `sweepSubjects` counts no holder for it and `releaseGoneHolder` skips it, so it stays unread until a strand of that name is listed.
  A program has no mailbox to take a return, which is why only its mail waits.
  The `sweepLoop`, `sweepNeeded`, `sweepSubjects` and `releaseGoneHolder` comments and the "Removed holders" section of `internal/creelengine/doc.go` say so.
  Tests in `deliver_test.go`: `UnreadProgramMailToAnUnlistedHubOrchOutlivesTheSweepAndWaitsForItsStrand` (fails with the guard removed: the mail is journaled undelivered and the strand reads nothing) and `ProgramMailForwardedToAnUnlistedHubOrchReturnsToItsForwarder` (pins the no-forwarder condition).
  A row for a listed-then-removed orch was tried and dropped: mail from a program to a listed strand is typed into its pane and settles as delivered, so the row could not hold unread mail, and the guard does not read listing history.
- **F2 — fixed as proposed.**
  `DobbyTasks` in `internal/battencli/dobbyreads.go` reads the parent and the owner of an entry carrying `notify_on_land` through the new `readTaskPlacement`; `TaskView.OwnedElsewhere` in `internal/dobbyengine/starts.go` says so; `takeDoneEntries` in `internal/dobbyengine/landnotice.go` skips an owned-elsewhere entry.
  Tests: `ADoneEntryOwnedByAnotherMachineIsLeftToItsDobby` row and the `landed` and new `landed-here` assertions in `TestDobbyTasks_ReadsDependenciesParentAndNotifyOnLand`.
- **F3 — fixed as proposed.**
  `TestLandNotice_ReachesAnOrchStrandListedAfterTheLanding` in `internal/dobbyengine/landnotice_creel_test.go`, tier 0: a dobby daemon's `Deps.Notice` posts as a program to `ly:orch` through a real `creelengine.Store`, scheduler and sweep over `creelfake`; the sweeps run past the point where a removed holder's mail would return; the orch strand is then listed and reads the line once, the request is cleared and no second line goes out.
  One extra listed strand holding mail keeps the sweep running, since the F1 change leaves the orch mail as no sweep subject.
  `_lym/landing/summary.md` names the test as the end-to-end demonstration and says no hand run was made.
- **F4 — fixed as proposed.**
  Rows in `TestLandNotices`: `AFailedClearIsRetriedWithoutAnotherPost`, `AFailedModeReadDecidesNothingUntilItReads`, `ADoneEntryIsNotTakenUpWhileTheModeReadFails`, `AnOwedLineIsPostedOnceByARebuiltDaemon`; fields `failClears`, `failConfigOnTick`, `rebuildBeforeTick`.
- **F5 — fixed as proposed.**
  `landNotices` takes up done entries only while `Deps.ClearNotifyOnLand` is set; the `Deps.ClearNotifyOnLand` comment in `internal/dobbyengine/loop.go` says so.
- **F6 — fixed as proposed, with a deviation in the log call.**
  An unreadable parent or owner of a non-released flagged entry leaves it with no parent and no owner and does not fail the read; a released task still fails it.
  It logs through the new `unreadableNoticeEntry` episode set, not a bare `logger.Warn`, so a flag that stays on the entry warns once and not at every dobby tick; the review's own premise is a failure that lasts as long as the flag.
  Test: `TestDobbyTasks_AnUnreadableParentFailsOnlyAReleasedTask`.
- **F7 — fixed as proposed.**
  `prune` Short, Long and `--apply` text in `internal/boardcli/cli.go`; prose in `docs/overview.md` (both prune clauses of the Land row), `contracts/orchrole/orch-role.md` and `plugins/ly/skills/board/SKILL.md`.
  Also the `Store.Prune` comment in `internal/boardengine/store.go` and the `PruneBoard` field comment in `internal/battenshed/land.go`, which a self-check fork named.
- **F8 — fixed as proposed.**
  `landedNotice(task, fields)` in `internal/boardcli/cli.go` answers only for a payload that sets `notify_on_land` true on a done entry, with the reworded text; `stepNotifyOnLandOnALandedEntry` in `internal/boardcli/verbs_test.go` pins the text and a title-only upsert returning no notice.
- **F9 — fixed as proposed.**
  The `Post_ToAnOrchOfTheHubWithNoStrandWaitsForItsStrand` row in `internal/creelengine/verbs_test.go` posts to `ly:orch` and `ly:gone:orch`, forwards to an unlisted orch, and refuses another hub's orch, an orch thread and a mistyped slug.
- **F10 — fixed as proposed.**
  The `AFailedPostIsRetriedOnTheNextTick` row is deleted.
- **F11 — fixed as proposed.**
  The `takeDoneEntries` comment and the landing-notice list of `internal/dobbyengine/doc.go` say a done entry is read as landed whatever set its status.

## Self-check

- fix-code-correctness, MEDIUM: forwarded program mail to a removed orch stayed under the orch's name — fixed as proposed (`len(msg.Path) == 0` in `waitsForHubOrch`), pinned by `ProgramMailForwardedToAnUnlistedHubOrchReturnsToItsForwarder`.
- fix-code-correctness, LOW: no row for an orch removed after it was listed — fixed with my own change: the row was written and dropped, since a listed strand's program mail is typed in and settles delivered, and the guard does not depend on listing history; the unlisted row and the forwarded row pin the guard.
- fix-consistency, LOW: the `sweepLoop`, `sweepNeeded` and `doc.go` sentences still said unread mail always keeps the sweep running — fixed as proposed.
- fix-affected-set, LOW: the `Prune` and `PruneBoard` comments — fixed as proposed (the `Prune` comment already named the exception in its third line, so only its first sentence changed).
- fix-proposal: no findings.

Lens reports are saved beside this file as `round-1-fixer-report.lens-<lens>.md`.

## Disputed

None.

## Deferred

None.
