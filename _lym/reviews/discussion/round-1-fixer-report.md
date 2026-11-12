# Fixer report — discussion round 1

Every finding's premise held against the code it cites (`keeper.go:226-233`, `slackengine/outbound.go:106-108` and `:356-370`, `inbound.go:358-365` and `:606`, `holder.go:352-354`, `api.go:263-308`, `identity.go:121-129`, `machinename.go:79-85`, `creelengine/outbound.go:24`, the `SendingHub` branches).
All edits are in `_lym/discussion/decision-record.md`; `_lym/discussion/support-log.md` gains its Round 1 entry.

## Changes

- F1 (BLOCKING), fixed as proposed: D4 states that a channel adopted by `name_taken` or by a resync enters the bridge map with its cursor at the channel's start and no thread floor, and D8's resync points there; a D8 criterion covers a channel adopted after the peer posted and one added by resync.
- F2 (MEDIUM), fixed as proposed: D3 replaces "before the hour" with the bound that holds, a relay down past the hour leaves the message returned, and states creel journals a late settle delivered and posts the sender a `CREEL` notice; a D3 criterion added.
- F3 (MEDIUM), fixed as proposed: D7 introduces a bridge-post record of its own under `.lym/slack/`, apart from the outbox record, whose entry goes once creel answered the settle; D3's re-settle reads it; Constraints name it; the D3 and D7 criteria updated.
- F4 (MEDIUM), fixed differently in part: D6 matches the self-name refusal in orch channels and, in a bridge channel, handles `approve`/`revoke` only on the relay whose peer they name; criteria added.
  Not made: "change D8's non-command operator text rule to apply to orch channels only".
  An orch channel forwards the operator's text to the orch (`inbound.go:614-624`), so a not-delivered answer there would be wrong.
  Instead other operator text in a bridge channel is answered once, by the relay of the end that sorts first in the channel's name, which removes the double answer and keeps the operator told.
- F5 (MEDIUM), fixed as proposed: D8's resync takes the local orch only from an envelope message of this bot or of the pinned app of its `from` machine; the Bound says so; a criterion covers a human-posted envelope.
- F6 (MEDIUM), fixed as proposed: D8 names a bridge resync hint of its own that runs only the users.conversations listing and reads of unmapped `bridge-` channels, at most once per bridge-poll base interval per hub; the Bound states what a foreign-channel event can trigger; the criterion updated.
  "Whose channel name it cannot place" is stated as a channel no hub's map holds, since the holder sees the channel id.
- F7 (MEDIUM), fixed as proposed, with the notice in the prime orch's channel: an unmapped channel is not read, so an `approve` typed in it would never be handled.
- F8 (MEDIUM), fixed as proposed, except the not-delivered criterion is written for a bridge channel per F4 above; the role-slack.md content moved into D9 with its own Review criterion, and D11's criterion lost it.
- F9 (LOW), fixed as proposed: D7 says D4's holds count toward no cap; D4 retries the invite at each bridge poll with `already_in_channel` counting as invited (`api.go:320` already ignores it); criteria added.
- F10 (LOW), fixed with the proposal's second option: D4 states the listing's true rate, once at first find and at most once per bridge-poll ceiling while a `bridge-name-taken` hold lasts.
  The first option, keeping a refused find in memory until restart, would leave a hold the operator cleared by renaming the channel unnoticed until the relay restarts.
- F11 (LOW), fixed as proposed in D3's Bound.
- F12 (LOW), fixed as proposed: a D8 bullet before the pin checks skips a message whose `from` names this machine; criterion added.
- F13 (LOW), fixed as proposed: D8's resync never adopts a channel whose local orch is not live, and a per-message bullet skips mail to a retired orch with a notice; criterion added.
- F14 (LOW), fixed as proposed in D6 and its criterion.
- F15 (LOW), fixed as proposed, except the sweep journals such mail `undelivered`, not dropped, per the self-check below.
- F16 (NIT), fixed as proposed, with the check placed in the parser per the self-check below; the D1 criterion extended.

## Self-check

- fix-claims, MEDIUM, sweep journals swept off-hub mail `undelivered` (`sweep.go:239-244`), not dropped: fixed as proposed in D8 and its criterion.
- fix-consistency 1 + fix-way-forward 1, MEDIUM, F5's filter made F7's notice unreachable: fixed as proposed; the filter now applies to choosing the local orch only, and the notice reads the newest envelope whoever posted it, for routing only.
- fix-consistency 2, LOW, D6's "any channel the relay reads" disagreed with the bridge-channel rule: fixed as proposed.
- fix-consistency 3, LOW, D8 Bound "never read": fixed as proposed, now "never delivered from, and a resync reads it only for its notice".
- fix-consistency 4, LOW, stale reference and ambiguous resync: fixed as proposed; the Auto-mode assumption points at D6, and the start and ceiling resync is named a bridge resync.
- fix-way-forward 2, MEDIUM, a retired-orch channel was never read so no one was told: fixed as proposed; the resync posts one notice in such a channel naming the orch as retired.
- fix-proposal 1, MEDIUM, F16's check sat outside the sole parser: fixed as proposed; the parser takes this machine's names and makes the check.
- fix-proposal 2, LOW, a refused settle kept its entry forever: fixed as proposed; a refusal counts as an answer.
- fix-proposal 3, LOW, F12's skip had no way forward when a peer took a former name: fixed as proposed; the notice asks for a name this machine never held.

## Disputed

None.

## Deferred

None.

## Rulings

None.
