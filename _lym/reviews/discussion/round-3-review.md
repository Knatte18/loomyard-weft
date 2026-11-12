---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lym/discussion/decision-record.md:61-64 (D3), :143 (D7)
    summary: No creel component is named to return a machine-outbound message at its hour, and the only code that enforces the hour excludes it
    fix: >-
      In D3, name the creel-side mechanism that returns a pending machine-outbound message at its hour,
      and require it to run without the relay: a timer in the scheduler woken at the earliest pending deadline,
      armed by a post and by the daemon's start scan.
      State that the hour is one hour from the post, as for a message to another hub.
      Add an acceptance criterion under D3 that a pending machine-outbound message is returned at its hour
      with the relay not running, under an injected clock.
      State in D3 that the take verb returns only messages not yet past their hour,
      and that a message past its hour is returned before take answers.
  - id: F2
    severity: MEDIUM
    class: design
    location: _lym/discussion/decision-record.md:85-89 (D4), :277 (acceptance criterion)
    summary: The bridge-archived way forward, rename the archived channel, cannot be done because Slack refuses a rename of an archived channel
    fix: >-
      In D4, replace the way forward of the bridge-archived notice with the sequence Slack allows:
      unarchive the channel in Slack, rename it to free the name, then archive it again if wanted;
      keep the rule that the relay never unarchives.
      Drop the clause that renaming the archived channel frees the name, and change the D4 acceptance criterion
      that reads once the name is free to say once the operator has unarchived and renamed the channel.
      Add the evidence line, docs.slack.dev conversations.rename page, error is_archived, read 2026-11-12.
  - id: F3
    severity: MEDIUM
    class: design
    location: _lym/discussion/decision-record.md:55-62 (D3), :268 (acceptance criterion)
    summary: The journal form of the report and settle events is unstated, so report may journal every poll and a settle after a return has no defined event
    fix: >-
      In D3, state how each of the three relay verbs is journaled.
      Report journals only when the reason changes, as the stalled event does, and its record is what retention carries
      as the last hold reason.
      Settle delivered and settle returned are events of their own for a machine-outbound message,
      with the Slack timestamp in a field of its own, and a settle delivered is accepted for a returned message
      where the existing settle leaves a settled one alone.
      State that a binary that does not know these events folds them to nothing and leaves the message an id only.
      Add a D3 acceptance criterion that an unchanged hold reason reported at repeated polls writes one record.
  - id: F4
    severity: MEDIUM
    class: design
    location: _lym/discussion/decision-record.md:81-82 (D4 bridge map), :186-194 and :202 (D8)
    summary: The hold and notice state of a bridge channel has no named home and no rule for when an approval moves the delivery cursor
    fix: >-
      In D4 add to the bridge map's fields the per-channel notice marks that make one notice per episode checkable
      across a relay restart, and state where the same mark of an unmapped channel is kept
      (the resync's notice for an unpinned creator or a retired orch), or that it is kept in memory and repeats once
      after a restart.
      In D8 state that an approval which delivers held messages moves the delivery cursor past them,
      so a later revoke does not hold an already delivered message again and post its notice a second time.
      Add a D8 acceptance criterion for approve then revoke under a held message:
      after both are handled, a reread posts no second notice for the delivered message.
---

### [BLOCKING] Nothing returns a machine-outbound message at its hour

**Location:** `_lym/discussion/decision-record.md:61-64` (D3), `:143` (D7)

**Issue:**
D3 says "Creel owns the hour" and counts on it while the relay is down (the sentence on a relay down past a message's hour)
and in D7, where the last button goes off and the relay stops.
The hour exists today in one place, `outboundWorker.run` in `internal/creelengine/outbound.go`:
it caps its wait at the message's remaining hour and calls `settleReturned` at the hour.
D3 states the scheduler's start scan and its wake on a post skip a machine-outbound message,
and that the take verb is its only reader.
So the code that returns a message to another hub at its hour never sees this message,
and the record names no replacement.
Read literally, the hour is applied when the relay calls take, which cannot happen once no button is on.
The sender's `--wait-reply` then never ends and no return notice is posted, which contradicts D7's
"creel returns it at its hour" and the D3 criterion that an unsettled message is returned at its hour.
The plan writer has to invent a timer, and whether it runs without the relay is the open design question.
This also settles the first part of the lifecycle focus:
the relay can post a message creel returned between its take and its post, which D3's late-settle notice covers;
the take again before posting a held message narrows that window and does not close it.
That path is sound.
The missing owner of the hour is the defect.

### [MEDIUM] The bridge-archived way forward is impossible

**Location:** `_lym/discussion/decision-record.md:85-89` (D4), `:277` (acceptance criterion)

**Issue:**
D4 tells the operator to "rename the archived channel in Slack, which frees the name".
Slack's `conversations.rename` page lists the error `is_archived`, "Cannot rename archived channel",
and Slack's help says to unarchive a channel before renaming it (searched 2026-11-12).
The name stays held by an archived channel until it is unarchived, so the stated way forward cannot clear the hold.
The hold then lasts until creel returns the message at its hour, and the acceptance criterion
"once the name is free, the next try creates the channel" has no path that reaches it.
`internal/slackengine/keeper.go:196` carries the same text for an archived orch channel;
that is outside this task, and the plan writer will copy the wording unless the record corrects it.
Verdict on the hand-archived focus item: the hold, the single notice per episode and the rate-limited listing are sound;
the way forward is the defect.

### [MEDIUM] The journal form of report and settle is unstated

**Location:** `_lym/discussion/decision-record.md:55-62` (D3), `:268`

**Issue:**
D3 fixes the event that creates a machine-outbound message and its retention carry under `of`,
and that is safe for an older binary: `applyCarried` in `internal/creelengine/fold.go` ignores an unknown `of`
and the id still counts in `maxID`.
The three relay verbs write more records, and the record is silent on them.
Report keeps a last hold reason that the carry also holds, so each report is a journal write;
the relay reports while a hold lasts and polls every five seconds, so without a change-only rule the journal grows by a record per poll.
The existing `stalled` event is written once per change of reason and is the precedent.
Settle delivered after a return is new behavior: today `settleDelivered` in `outbound.go` leaves a message that is not pending alone,
so the late-settle path D3 requires has no event to write.
The plan writer cannot test an older binary's read of these events without knowing their names and fields.

### [MEDIUM] Hold and notice state of a bridge channel has no home

**Location:** `_lym/discussion/decision-record.md:81-82` (D4), `:186-194` and `:202` (D8)

**Issue:**
D8 promises "one notice per episode" for a held message, a clashing machine name, a wrong app, a retired orch and an unpinned creator,
and keeps "the hold per channel".
D4 lists the bridge map's fields, and none holds an episode mark.
A channel left unmapped (an unpinned creator, a retired orch) is in no record at all,
yet its notice is "once per channel" across resyncs that run at start and at every ceiling.
A restart then either repeats every notice or the record must say where the mark lives.
The delivery cursor has a second gap on the exactly-once focus.
The operator cursor does stop an `approve`, a later `revoke` and a plain-text line from being handled again, and the revoke stands.
D8 does not say that an approval which delivers held messages moves the delivery cursor past them.
If it stays at the held message, the reread after the `revoke` finds no pin, holds the message again
(creel's dedupe makes the delivery harmless) and posts the notice a second time.
Verdict on the focus item: operator messages are handled once; the stuck cursor after an approval is the residue.

## Focus verdicts

- Operator messages past a held peer message (D6, D8): handled once each by the operator cursor; the revoke stands.
  The residue is the delivery cursor after an approval and the notice marks, in the last finding.
- Adopted or resync-added channel (D4, D8): sound.
  The delivery cursor starts at the channel's start and the seven-day skip precedes bots.info;
  a delivered id is held by creel's journal for at least seven days after delivery, and delivery is never before the post,
  so the skip and the dedupe leave no gap.
  The operator cursor and thread floor at the newest message keep an earlier `approve` from being handled.
  Not a defect.
- Held machine-outbound lifecycle (D3, D7, D8): the refusal count lives in the bridge-post record and is dropped once take stops returning the message, which is coherent;
  the post-after-return race is covered by D3's late-settle notice.
  The two defects are the owner of the hour and the journal form of report and settle.
- Unpinned-creator notice with no open prime channel (D8): sound.
  The relay runs only while some orch has its button on, so at least one channel receives the notice,
  and `approve` typed there is handled as D6 states.
  Not a defect.
