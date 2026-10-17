# Fixer report — discussion round 3

Every finding's premise held against the code; every finding is fixed.

## Changes

- **F1** (fixed as proposed, taking the Decision option): D5's Fresh launch bullet now has a failed first withdraw, a refusal or an unreachable daemon alike, make `lyx orch start` start and bind no session and fail with creel's error, the way forward, and the stated cost.
  A failed second withdraw or either release keeps the recorded watcher retry.
  The old TTL sentence is replaced by the bound that holds: an old inject is typed on the first idle, unpaused probe, since the TTL counts only live, unpaused time, and old mail is reminded until a retry succeeds; the exposure is accepted in so many words.
  The "at most one line of the old cycle" bound now says it holds once both withdraws have succeeded.
  The withdraw acceptance criterion is split into a first-withdraw refusal criterion and a second-withdraw retry criterion, and the relaunch criterion is qualified to both withdraws succeeding.
- **F2** (fixed differently): the proposal offered two mechanisms, a held-back cursor in the claude engine's reading or the signal reading; the second was taken.
  D5 now has `Runner.ReadEvents` take each event's time from the engine's `ParseSessionSignals`, pair them in file order, and resume from that reading's consumed count, which holds back a trailing stamp run (`internal/shuttleengine/claudeengine/signals.go`, `holdFrom`).
  Not made: a second stamp pairing in the claude `ParseEvents`, which would duplicate the rule `ParseSessionSignals` already owns.
  An acceptance criterion for a stamp read in one tick and its payload in the next is added.
- **F3** (fixed differently): the Scope bullet now lists the time field on `shuttleengine.Event`, `Runner.ReadEvents` setting it from `ParseSessionSignals` and resuming from its consumed count, the fake engine gaining a `ParseSessionSignals` that times each Stop line, and the orchengine `Session` seam passing the field through unchanged.
  Not made: the claude `ParseEvents` pairing each Stop with its stamp, for F2's reason.
  D5 names `ParseSessionSignals`'s stamp rule as the one place that defines a Stop with no readable stamp.
- **F4** (fixed as proposed): a Scope bullet adds a typed refusal code on creel's response (`protocol.go`), set only on the no-strand-pane refusal, which the Go client (`client.go`) returns as an exported error value.
  D6 says `NotifyPrime` matches it with `errors.Is`, never by message text.
  The docs bullet names the code in the `creelengine` and `creelcli` docs, and the sender acceptance criterion covers a detached Linux caller whose spawning parent has exited.
- **F5** (fixed as proposed, the claim about the orch role dropped in self-check): D6 adds the drop case of a non-orch pane with no strand claim, and states that a dropped notice survives only in the caller's warning, carried by `NotifyPrime`'s returned error.
  D8's fallback says a refused notice is not in `lyx creel log`.
- **F6** (fixed as proposed): support-log Turn 4 and the Q7 ledger line state the current sender rule.
- Support log: a Round 3 entry records the F1, F2/F3 and F4 picks, and Q16 records the first-withdraw launch refusal.

## Self-check

- fix-consistency, LOW, Scope's observable CLI list misses the `lyx orch start` refusal: fixed as proposed.
- fix-consistency, LOW, Scope's orch-state item still records a failed first withdraw: fixed as proposed.
- fix-consistency, LOW, D9's withdrawn-line sentence not qualified for a failed second withdraw: fixed as proposed.
- fix-consistency, LOW, Q16 ledger line stale: fixed as proposed.
- fix-claims, MEDIUM, the exposure is not limited to the window between the withdraws, because a tick that re-read before the save can queue after the second withdraw: fixed as proposed. D5 now states the one step per old watcher, the success-path bound and the relaunch criterion admit it, and the "between the two withdraws" limit is gone.
- fix-claims and fix-proposal (merged), MEDIUM, the pause-resume post has no state re-read, so the acceptance reason does not cover it: fixed with my own change. D5 says it can arrive whenever an old watcher runs and is informational to a session that never saw the pause. I did not add a state re-read to D9's post, which would change a decision outside what the finding asked for.
- fix-proposal and fix-way-forward (merged), MEDIUM, the orch role cannot read a caller's log, and a detached caller's error lacks the line: fixed as proposed. `NotifyPrime`'s returned error carries the full line, and the recovery names the operator only.
- fix-proposal and fix-way-forward (merged), LOW, a persistent sender-resolution refusal or a dead daemon blocks every orch start, and the way forward only said to wait: fixed as proposed. The way forward says to repair what the error names; a daemon that would not start has its log lines quoted by the client's connect failure (`internal/creelcli/doc.go`). The cost is stated.

## Disputed

None.

## Deferred

None.
