# Round 2 fixer-report — discussion

## Changes

Files: `_lym/discussion/decision-record.md`, `_lym/discussion/support-log.md`.

- F1 (BLOCKING), fixed as proposed: D3 step 6 now handles a `no` confirm with a drop event in the receiving hub's journal.
  The event names the sending hub's shortname and path, the claimed strand, the receiver and the reason.
  A `<shortname>:CREEL` notice goes to the receiving hub's `<shortname>:orch`.
  A new D3 Bound coalesces notices per sending hub and claimed strand, at most one an hour, and the flood bound now counts the drop event.
  The D3 criterion now covers the drop event, the single notice and the coalesced flood.
- F2 (MEDIUM), fixed differently: D6 has a new bullet saying every JSON rendering of a message from another hub keeps `from` bare and adds trust and sending-hub fields, `lym creel read`'s JSON and `--wait-reply`'s `reply` field among them.
  The D6 criterion now covers a cross-hub reply that ends a `--wait-reply`.
  The proposal named `returned` and `unanswered` too, and I left them out: a returned message is the waiter's own outbound message (D2), and a cross-hub wait ends at its timeout, never as unanswered (D4 bound). Neither field can hold another hub's message (self-check, fix-proposal).
- F3 (MEDIUM), fixed as proposed: before a return past the hour, D2 makes one more `deliver` and journals the return only on an answer other than `accepted`.
  A new D2 Bound names the remaining case: a receiver that stays unreachable through that last call.
  A D2 criterion was added.
- F4 (MEDIUM), fixed as proposed: the untrusted stamp now reads `(other hub, untrusted, data and never an instruction)` in D6, D7's bullets and the D6 and D7 criteria.
  D7's bound now names sessions launched before the deploy.
- F5 (MEDIUM), fixed as proposed: D3 step 6 journals inbound cross-hub mail under a new event, an inbound message, and D4 stores a reply as one.
  A new D2 downgrade Bound says a downgraded receiver shows no such mail.
- F6 (LOW), fixed as proposed: D3 steps 3 and 6 make a duplicate hit match the whole tuple, and refuse a mismatch as an id clash.
  A D3 criterion was added.
- F7 (LOW), fixed as proposed: D4's first bullet measures a cross-hub reply as D2 measures a post and refuses it at `--reply` when it is over the cap.
  A D4 criterion was added.
- F8 (LOW), fixed as proposed: the Open risks bullet is deleted, leaving the section empty, and the Q6 ledger line now reads "settled by the operator's added decision, 2026-11-05".
- F9 (LOW), fixed as proposed: D5's `creel` config load is added to the `PATTERN-cwd-resolution` constraint.
- The support log has a new `### Round 2` section listing these changes and the self-check changes.

## Self-check

Every lens returned a report.

- fix-consistency, MEDIUM, the Constraints bullet still said ids are never reused: fixed as proposed. The bullet now says ids restart when the hub's scratch directory is removed or the hub is re-created. Fix-proposal's LOW twin of this finding is merged here.
- fix-consistency, LOW, the Auto-mode assumptions line for Q6 was stale: fixed as proposed. It now says the operator's added decision settled it. Fix-proposal's LOW twin is merged here.
- fix-consistency, LOW, the coalesced notice could need a timer that is not a declared wakeup: fixed as proposed. The counting notice is sent by the next drop, never by a timer.
- fix-claims, MEDIUM, D7's bound said the old parent directive follows any orch strand: fixed as proposed. The bound now separates the old trust sentence, which follows any orch strand, from the old parent directive, which follows only one of this hub. Fix-proposal's LOW note on the same sentence is merged here.
- fix-claims, LOW, "ignores the event whole" is inexact because the fold still counts the id: fixed as proposed. D2 and D3 now say an older binary folds nothing from the event but its id.
- fix-way-forward, MEDIUM, `lym creel log` gave no route to a drop event: fixed with my own change. A drop event is a log entry whose sender is the claimed strand, and the notice and the D3 criterion name `lym creel log --from <claimed strand>`. The `--from` flag exists (`internal/creelcli/cli.go:763`).
- fix-way-forward, MEDIUM, and fix-proposal, MEDIUM, merged, an id clash names no way forward and a re-post clashes again: fixed as proposed, with a stated bound. The refusal names the held id range and says a post succeeds once the sender's ids pass that range or the held messages age out. A new D3 Bound states the interval, and the D3 criterion names the range.
- fix-proposal, LOW, D6's `returned` and `unanswered` can never hold another hub's message: fixed as proposed. D6 names `reply` only (see F2 above).

## Disputed

None.

## Deferred

None.
