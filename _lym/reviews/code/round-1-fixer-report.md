# Fixer report — code review round 1

All seven findings are fixed as proposed, plus the stale twins the self-check found.
`lym gate test ./internal/shuttleengine ./internal/websterengine` passed after the code round (tier0: 2303 passed, 0 failed), and `lym gate test ./internal/shuttleengine` passed after the self-check round (1230 passed, 0 failed).

## Changes

- **F1** (`wait.go`, `pollEventsTick`): a turn start read past the recorded waiting turn end clears `waitingTasks` and `waitingTurnEnd` before any expiry runs, moving the tasks to `countedTasks` when there are any, so the gated expiry cannot count Done in a turn already under way.
  The gated shell-only branch with every output file present does the same when the turn start is in the same batch as the waiting turn end, and returns without nudging or expiring.
  Test: `TestPollEventsTick_TurnStartEndsTheGatedShellWait` appends a turn start after the waiting line, advances past the bound and expects no Done until the next turn end.
- **F2** (`waiting_test.go`): row "a turn start past the waiting turn end types nothing" of the table-driven `TestWait_GatedShellNudge`, asserting no `SendText` call.
- **F3** (`waiting_test.go`): rows for a pause covering the run (the line is dropped, nothing sent while covered) and a failed send (one attempt per waiting turn end, no retry in place, the run still reaches Done).
  The former `TestWait_GatedShellNudgeNamesEachShellOnce` is now the first row of that table, so the three variants share one test.
- **F4** (`wait.go`, `run.go`): `queueShellNudge` no longer marks shells named. It records the ids it names in the new `run.shellNudgeIDs`, and `sendShellNudge` marks them in `run.nudgedShells` only after `sendWithin` succeeds.
  A dropped or failed line leaves the shells unnamed, so the next waiting turn end over them queues it again; the pause and failed-send rows assert this.
- **F5** (`waitmark.go`), **F6** (`spec.go`), **F7** (`websterengine/runlevel.go`): doc comments reworded as the review proposed.

Twins found by the self-check and fixed in the same round: `docs/overview.md` (shuttle entry), `internal/shuttleengine/doc.go` (the nudge paragraph and the `AwaitedShellPrefixes` sentence), the `completionsignal_enforcement_test.go` comment on the third file-contract site, and the `expiredTurnEnd` doc, which now mentions the turn-start clear.

Commits: F5, F6 and F7 each have their own commit.
F1 to F4 share one commit, because they overlap in `wait.go`, `run.go` and `waiting_test.go` and cannot be split without interactive staging; the self-check twins have a fifth commit.

## Disputed

- Self-check, fix-proposal lens, finding 1 (LOW): the shell's first-seen stamp survives the F1 clear, so the agent gets no new wait window after the nudge.
  Ground: every fix worse.
  Evidence: `ShellWaitBound` defines the bound as how long a shell "stays outstanding", `endedShells` measures `Outstanding` from `shellFirstSeen`, and the staggered rows of `TestPollEventsTick_OutstandingBackgroundWork` pin first-sighting stamps.
  Resetting the stamps on a turn start would let an agent that keeps ending turns over a still-running shell restart the bound each time, so the bound would no longer limit the wait, and it would misreport `Outstanding`.

## Deferred with reason

Nothing deferred.

## Self-check

- fix-code-correctness: no findings.
  It noted an out-of-scope case, not a finding: an ungated run with an output file missing, with a waiting turn end and a turn start in one batch, can still expire into a held turn end when the shells are already past the bound.
- fix-consistency, LOW: `docs/overview.md` shuttle entry still said a gated shell-only turn end is judged as a plain turn end. Fixed as proposed.
- fix-consistency, LOW: `completionsignal_enforcement_test.go:189` comment, same premise. Fixed as proposed.
- fix-proposal, LOW: first-seen stamp survives the clear. Disputed, see above.
- fix-proposal, LOW: `expiredTurnEnd` doc names only counting and expiring as clearing the waiting list. Fixed as proposed.
- fix-affected-set, MEDIUM: `docs/overview.md:423`, duplicate of the fix-consistency overview finding. Fixed.
- fix-affected-set, LOW: `doc.go:440` still said "out of the plain-turn-end judgment". Fixed.
- fix-affected-set, LOW: `doc.go:430-431` said a failed send is not retried and a shell is named once by any earlier line. Fixed to the F4 behavior.
- fix-affected-set, LOW: `completionsignal_enforcement_test.go:189`, duplicate of the fix-consistency finding. Fixed.

All four forks returned a report.
