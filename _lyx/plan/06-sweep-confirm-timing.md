# Card 6 — sweep-confirm-timing

**Edit:**
- `internal/creelengine#sweepBackoffCeiling`
- `internal/creelengine#sweeper`
- `internal/creelengine#Scheduler.sweepLoop`
- `internal/creelengine#Scheduler.sweepOnce` -> `func (s *Scheduler) sweepOnce(state *sweeper) (ok, changed, firstMiss bool)`
- `internal/creelengine_test#isProbeWait`
- `internal/creelengine_test#isSweepWait`
- `internal/creelengine_test#TestSweep_RemovedHolders`
- `internal/creelcli#isProbeWait`
- `internal/creelcli#isSweepWait`
- `internal/creelengine/doc.go`
- `pattern/PATTERN-daemon-wakeups.md`
- `tools/sandbox/SANDBOX-CREEL-SUITE.md`

**Create:**
- `internal/creelengine#sweepConfirmInterval`

**Uses:**
- `internal/creelengine#sweepBackoffBase`
- `internal/creelengine#Store.releaseGoneHolder`
- `internal/creelengine_test#env.sweepWaits`
- `internal/creelengine_test#env.sweepStep`
- `internal/creelengine_test#env.sweepLast`
- `internal/creelengine_test#awaitOutcome`

**Intent:** Fix #685 (D6): the removed-holder sweep confirms a missing holder fast and backs off less, keeping its two-misses-in-a-row rule and its rule that a failed listing counts toward no streak.

`sweepBackoffCeiling` drops from 4 minutes to 50 seconds, and the 30-second base stays.
`sweepOnce` also reports `firstMiss`, true when its successful listing missed some holder for the first time in a row.
`sweeper` records whether a confirm is pending.
In `sweepLoop`:
- A sweep with `firstMiss` sets the next wait to `sweepConfirmInterval` and marks a confirm pending, in place of any other wait, the changed-set base included.
- A failed listing while a confirm is pending sets the next wait to `sweepConfirmInterval` too; outside a pending confirm it sets the base, as today.
- A confirm stays pending from the sweep that set it until the next successful listing; that sweep ends it and sets its next wait as if the previous wait were the base, so an unchanged set doubles to the 50-second ceiling and a changed set returns to the base, and a new first miss at that sweep sets a new confirm.
- A new post, reply, forward, return or inject while a confirm is pending keeps the confirm deadline: the loop re-arms its timer at the time left to that deadline.
- Outside a pending confirm, such a wake re-arms the wait at the shorter of the base and the time left on the current wait, measured on the scheduler's clock, so new mail never lengthens the wait to the next sweep.
A removed holder is then settled within the ceiling plus the confirm interval, 60 seconds, while listings succeed, an orch-role holder included, which has no delivery worker; each failed listing between the removal and the second miss delays the release by at most its own wait.
Bound: the confirm interval and the ceiling stay above the one-second floor, the sweep still runs only while unread mail or a queued inject exists (`PATTERN-daemon-wakeups`), and a holder is released only after two successful listings at least 10 seconds apart both miss it.
The shorter guard now lets a strand that is absent from reed's persisted state for 10 seconds or more, for example while it is removed and re-added under the same name, have its held mail returned along its path, where before it had to be absent for at least 30 seconds.

Tests: `isSweepWait` in both test packages becomes "longer than the 8-second probe ceiling", which still tells every sweep wait but a post-shortened one from the probe waits on the shared clock; the rows that observe a post-shortened wait hold only orch-role mail, which has no worker, and read the clock's waits unfiltered.
The doc comment both predicates share, which sits on `isProbeWait`, says sweep waits are longer than the probe ceiling in place of "30 seconds and up".
Rows of `TestSweep_RemovedHolders`:
- "WaitsGrowToTheCeilingWhileTheStrandSetHoldsAndResetOnAPost" pins 30, 50, 50 seconds and a post with the full ceiling left re-arming at 30.
- New "PostWithLessThanTheBaseLeftDoesNotDelayTheSweep": orch mail only; with 5 seconds left on a 50-second wait a post re-arms the wait at 5 seconds, and the sweep runs when they pass.
- New "RemovedHolderIsSettledWithin60sAtTheCeiling": with mail held by `ly:orch` and by beta and the waits at the ceiling, both are removed right after a sweep; the next sweep at 50 seconds sets the 10-second confirm, and by 60 seconds after the removal both messages are back with alpha, marked ended unread; the releasing sweep arms its next wait at 50 seconds, as if the previous wait were the base, and the returns' own wake, a return outside a pending confirm, re-arms it at 30 seconds while alpha's mail keeps the sweep running.
- New "PostInsideTheConfirmIntervalKeepsTheConfirmDeadline": the removed holder is `ly:orch`, whose mail has no worker, so the row reads the clock's waits unfiltered; 5 seconds into a pending confirm a post to `ly:orch` arrives, the loop re-arms its timer at the 5 seconds left on the confirm deadline and records that wait, and the holder is released at the confirm deadline, within 60 seconds of the removal.
- "StrandRemovedAndReAddedKeepsItsMail" keeps pinning that a holder missing from one listing and present in the next keeps its mail, now with the confirm sweep 10 seconds after the miss, and that this confirm-ending sweep, whose strand-name set changed back, sets its next wait at the 30-second base.
  The row also holds mail for `ly:orch`; once the set holds again, beta is removed again for a first miss, and at the confirm sweep beta is back while `ly:orch` is gone, so the sweep that ends the confirm is `ly:orch`'s first miss and sets a new 10-second confirm.
- "FailedListingMovesNothingAndCountsTowardNoStreak" pins a failed listing after a first miss setting the next wait at 10 seconds, the mail unmoved, and the release at the first successful listing after it that misses the holder again; a failed listing with no miss pending still sets 30 seconds.
The rows in `internal/creelengine/server_test.go` and `internal/creelcli/cli_test.go` that drive a removal with `sweepStep` and `sweepLast` keep passing through the new `isSweepWait`, with the confirm wait as their second sweep.

Docs in the same commit: the Removed holders paragraph of `internal/creelengine/doc.go`, the creel sweep line of `pattern/PATTERN-daemon-wakeups.md` (from 30 seconds doubling to 50, a 10-second confirm after a first miss), and the C11 watch line of `tools/sandbox/SANDBOX-CREEL-SUITE.md` (each message steps back within 60 s of the removal).

**ImpactSummary:** The sweep loop's timing in `internal/creelengine` changes and its private `sweepOnce` gains one result, read only by `sweepLoop`; delivery, the release itself and every exported surface are unchanged, and two test predicates widen to see the 10-second wait.
