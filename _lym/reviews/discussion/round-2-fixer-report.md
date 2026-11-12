# Round 2 fixer report — discussion

Every finding was fixed in `_lym/discussion/decision-record.md`, and the support log gained a Round 2 entry.

## Changes

- F1 (BLOCKING), fixed as proposed: the bridge map gains a per-channel operator cursor (D4).
  D8 handles an operator message only past that cursor, then advances the cursor, so a reread from a held delivery cursor handles nothing again.
  A D8 criterion checks approve, revoke and plain text under a held message.
- F2 (MEDIUM), fixed as proposed: D4 holds the message as `bridge-archived` when the listing finds the channel archived, posting a rename way forward.
  It retries the create once per ceiling while a name hold lasts. D7's hold list names the reason, and a D4 criterion covers it.
- F3 (MEDIUM), fixed as proposed: an adopted or resync-added channel's operator cursor and thread floor start at its newest message (D4).
  D8 skips an envelope posted longer ago than creel's journal retention, and the D8 criteria were added and adjusted.
- F4 (MEDIUM), fixed as proposed: D7 takes again before posting a held message and drops one take no longer returns. A D7 criterion covers it.
- F5 (MEDIUM), fixed as proposed: D3 journals machine-outbound mail as an event of its own, carried under `of` with its receiving machine and last hold reason.
  A D3 criterion covers the carry and the unknowing fold.
- F6 (MEDIUM), fixed as proposed: D8's unpinned-creator notice falls back to the open channel of each orch whose button is on, and is also mailed to the prime orch.
  A D8 criterion covers it.
- F7 (MEDIUM), fixed as proposed: two criteria were added, the D8 resync-hint rate and the D4 one-`FindChannel`-per-ceiling bound.
- F8 (LOW), fixed as proposed: the relay takes at each bridge poll (D3, D10). D10 states the worst-case wait of sixty seconds on an idle bridge, and the D10 criterion names the take.
- F9 (LOW), fixed as proposed: D8 reads bridge-channel threads as orch channels' threads are read, and handles an operator thread reply once in its thread. A D8 criterion covers it.
- F10 (LOW), fixed as proposed: D4 invites a changed operator into bridge channels. A D4 criterion covers it.
- F11 (LOW), fixed as proposed: the D8 resync test uses the bridge name's shape, the `bridge-` prefix with a `--` between ends or the prefix alone for a cut name.
  It excludes every channel an orch channel map of the machine holds, and a D8 criterion covers a machine named `bridge`.
- F12 (LOW), fixed as proposed: D7 keeps the refusal count in the bridge-post record, never in the outbox record.
  D3's startup settle is narrowed to accepted messages, and the D7 criterion names a confirmed orch-mail read between refusals.
- F13 (LOW), fixed as proposed: D7 states that `button-off` is reported only while another orch keeps the relay running, and that a hub whose last button went off returns at the hour.
  The D7 criterion is restated.
- F14 (LOW), fixed as proposed: the peer app was dropped from the bridge map's fields (D4).
- F15 (LOW), fixed as proposed: D8 keeps the sending id on the machine-inbound event, and a reply's machine-outbound reply-to is that id, which take returns.
  D3's take bullet and a D8 criterion carry it.

## Self-check

- fix-consistency, LOW, "the cursor" now ambiguous in D8: fixed as proposed ("delivery cursor" in both sentences).
- fix-consistency, LOW, retention skip contradicts "an approval delivers every held message": fixed as proposed. D8 and its criterion bound the approval to messages within retention.
  This was merged with the third fix-proposal finding.
- fix-consistency, LOW, refusal-count entry has no removal rule: fixed as proposed (merged with the second fix-way-forward finding and the second fix-proposal finding).
  D7 removes a refusal count once take no longer returns its message. An accepted message's entry keeps the settle rule, so a late settle still happens.
- fix-consistency, LOW, three `bridge-` channel twins left: fixed as proposed. They now say bridge-shaped in D8 and in both criteria.
- fix-claims, MEDIUM, operator re-invite needs an operator field the bridge map lacked: fixed as proposed. D4 adds an invited-operator field and invites where it records another or none, and the criterion matches.
- fix-claims, LOW, "dedupe holds ids only within it" overstated: fixed as proposed, reworded to "is sure to hold a delivered id only within it".
- fix-way-forward, MEDIUM, an `approve` typed in an unmapped bridge channel is never answered: fixed with my own change, merged with the first fix-proposal finding.
  The unpinned-creator notice names the orch channel to type the command in.
  The alternative, answering skipped operator commands at adoption, would reintroduce the reread of old commands that F3 removed.
- fix-way-forward, MEDIUM, refusal-count entry never removed: merged above.
- fix-proposal, MEDIUM, F3's skip loses a bridge-channel `approve`: merged with the first fix-way-forward finding.
- fix-proposal, LOW, refusal-count removal: merged above.
- fix-proposal, LOW, retention skip against "every held message": merged above.

## Disputed

None.

## Deferred

None.

## Rulings

None.
