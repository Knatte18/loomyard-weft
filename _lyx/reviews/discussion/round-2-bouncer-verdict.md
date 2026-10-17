---
verdict: CONTINUE
rationale: "Round 2 has four MEDIUM design findings and no BLOCKING one, but F2, F3 and F4 map to round-1 keys fresh-launch-leaves-hold-and-queued-injects, detached-notifier-refused-by-identity-rule and step-events-offset-at-queueing, still open in the new text, so the gate blocks; F1 is a new key."
---

Mapping of the latest review's findings to ledger keys:

- F1 (relaunch leaves the old cycle's unread note-request mail to be reminded to the new session) is the new key `relaunch-old-note-request-reminded`, first raised in round 2.
- F2 (a failed withdraw at relaunch leaves old `/clear` and `/compact` injects queued and the hold in place) reopens `fresh-launch-leaves-hold-and-queued-injects`: the round-1 fix added a withdraw and release, and the failure path still leaves the same leftovers.
- F3 (D6's in-pane test has no mechanism and a mismatch loses the notice) reopens `detached-notifier-refused-by-identity-rule`: the notice is still refused by creel's identity rule.
- F4 (the sending-time filter rests on event times shuttle does not provide) reopens `step-events-offset-at-queueing`: the step-events filter is still unsound.

All four are `design` at MEDIUM, so they gate, and three of them sit on keys open in an earlier round.
No finding was relabelled and there is no Focus departures section.

The round-1 keys the review rejected or left unraised are resolved in the ledger.
