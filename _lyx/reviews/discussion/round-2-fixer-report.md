## Changes

- F1 (BLOCKING), fixed differently in one part: D8 now has the discussion writer message its parent, `ly:orch`, during this run, asking it to file a board note at once through `lyx board upsert` with kind `note`, naming this task's slug and carrying the same measurement content.
  The Rationale's last sentence now says the orch files proposals as board notes at once, the note survives the entry's close, and a filing at landing would be lost because an agent-approved run lands with no orch turn.
  Premise checked: `orch-template-role.md` says "A run the agent approved lands with no orch turn" and "A proposal goes on the board as a note at once", and the hub's `loom.yaml` sets `alignment_pr_approve: true`.
  Not made as proposed: the fix asks the support log to record that the request was sent.
  It was not sent, because this fixer's prompt forbids messaging any other session about the review, so recording it as sent would be false.
  Ledger item Q9 instead names the new carrier and records that the request is still to be sent by the discussion writer.
- F2 (LOW), fixed as proposed: D3's replacement statement now excepts a rules file a card of this run targets, which the fork reads fresh like code.
- F3 (NIT), fixed as proposed: the Goal limits the `pattern/` read to the background files that touch what the forks will do.
- F4 (NIT), fixed as proposed: D3's last Decision sentence now reads "Every other sentence of each fragment is unchanged, apart from the plain fragment's re-read to read."

## Disputed

None.

## Deferred

- F1, the send itself: the message to `ly:orch` that D8 now requires is not sent this round, because the fixer may message no other session.
  The discussion writer sends it; Q9 in `support-log.md` records it as pending.
