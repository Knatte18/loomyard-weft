# Card 7 — attach-held-runs

**Edit:**
- `internal/shuttleengine/attach.go`
- `internal/shuttleengine/doc.go`
- `internal/shuttleengine/attach_asking_test.go`
- `internal/shuttleengine/attach_test.go`

**Uses:**
- `internal/shuttleengine/hold.go`

**Intent:**
D6: resuming a producer attaches to a held run, and a `run.json` an older binary left at outcome `asking` with a live strand is attached instead of respawned (#407).

A held run's `run.json` stays `running` (card 5), so the existing Attach probe every spawning adapter runs before archiving already finds it.
This card adapts the legacy `asking` record:

- `dispositionCandidate`: a candidate whose `Outcome` is `asking` and whose strand reed tracks as live is attachable, whether or not its events grew and whether or not it carries an `AskingOffset`.
  With a dead or untracked strand it is classified exactly as today.
  The file-contract-first check stays first.
- `reconstructAndWait`: an `asking` record is reset to `running` and its `AskingOffset` cleared.
  With an `AskingOffset`, the run replays from that offset and `NotifiedOffset` (card 6) is set to it, so the old ask turn end is neither read nor notified: the parent already saw that run halt.
  Without one, it replays from `PromptOffset`, so its last turn end is read as a held turn end and notified once, and the earlier turn ends in its events file are not (the first tick's backlog is one batch classified by its last event, card 5).
- An attached record with no `StrandName` takes the name reed's status reports for its guid, when the status was read.
- `attachCandidate.eventsSize` and the events-file stat in `collectAttachCandidates` existed only for the growth rule and go.

Bound, in the doc comments: only a live, tracked strand is attached; the attach starts a fresh deadline (D4).
Update `attach.go`'s header and `reconstructAndWait`'s offset comment, and the attach section of `shuttleengine/doc.go`.

Tests, in `attach_asking_test.go` by rewriting its three tests rather than adding beside them (card 5 already adapted `TestAttach_AskingReentryStartsAtRecordedOffset` so it passes at its commit; this card replaces that adaptation):
`TestDispositionCandidate_AskingReentry` becomes the table over legacy asking records: live with an offset, live without one, live with no events growth, all attachable; dead or untracked, classified as before;
`TestCollectAttachCandidates_RecordsEventsSize` is deleted with the field it pins;
`TestAttach_AskingReentryStartsAtRecordedOffset`, the end-to-end attach test, drives `Attach` with a recording notifier over a legacy record: with an offset, the old ask is not notified and a later Stop with outputs finalizes Done; without one, an events file holding several held turn ends notifies exactly one, the last, once;
and a `running` record whose `NotifiedOffset` covers its last held turn end is attached without a new notice, for a plain Stop and for an expired-shell waiting line once the shell bound has elapsed.
Adjust `attach_test.go` rows that build asking candidates to the new verdicts.

**ImpactSummary:** Attach now waits on live legacy asking runs instead of letting the caller respawn beside them; candidates no longer stat the events file.
