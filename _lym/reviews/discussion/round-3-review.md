---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lym/discussion/decision-record.md:124-127 (D2), acceptance criteria D2
    summary: A hint is removed before its read and nothing re-triggers a read that fails, so a transient Slack error strands the operator's message until a later message or a reconnect
    fix: "In D2, add a retry owed per channel and thread. When the read a hint triggered fails (transport error, server error, rate limit, or a refusal the relay recovers from), the relay keeps that channel or thread in an in-memory owed-read set and reads it again at each wake from the outbound read's wait, backing off from one second to sixty and honoring a rate limit's retry wait, until the read stands. A crash needs no record: the relay's start reads every open channel. Add to D2's Rationale one clause that the removal before the read stays, since it orders the read after any hint written meanwhile. Add an acceptance criterion under D2: a hinted read that fails is retried at the next outbound wait without a new hint, and forwards the operator message once it succeeds. Add the same to PATTERN-daemon-wakeups' Constraint line only if the retry wait is a new loop; it reuses the outbound wait, so no new loop is named."
  - id: F2
    severity: MEDIUM
    class: design
    location: _lym/discussion/decision-record.md:249-255 (D5), acceptance criteria D5
    summary: A held channel is ensured again only at relay start or a presence flip, so a failed invite or a changed operator waits for a restart and the notice's own way forward does not take effect
    fix: "In D5, name when a held channel's ensure runs. Add that the relay re-runs the ensure of every channel whose entry records another operator than setup.json's, or none, at each wake from the outbound read's wait, the same wake that already retries a refused prime channel, reading setup.json then. The wait backs off, so a persistently refused invite costs one bounded local read per wait and one Slack call per attempt, and the per-episode notice stays one. State in D5 that `lym slack setup --operator` takes effect at that wake with no relay restart, since the relay watches presence.json alone. Reword the acceptance criterion 'An operator invite that failed is retried at the next ensure' to name that wake, and add a row that a changed operator is invited at the next outbound wait with the relay still running."
  - id: F3
    severity: MEDIUM
    class: design
    location: _lym/discussion/decision-record.md:49-51 and 194-196 (D1, D3), acceptance criteria D1
    summary: A hubregistry.Live read that keeps failing leaves every event unacknowledged and the holder mails no notice, so push stops silently while the operator is away
    fix: "In D1, add that a holder whose hubregistry.Live read fails mails its own hub's orch one `[slack relay]` notice per episode naming the hubs record's path, moving that record aside as the way forward, and `lym creel inbox` in each hub as the record's repair, the wording D3 already uses for the ensure path; the episode ends at the next read that stands. The holder knows its own hub without the list. Add an acceptance criterion under D1 that a failing Live read mails that one notice and a read that stands ends the episode. Optionally have the notice also post to the holder hub's prime channel when its map reads, since an away operator reads Slack and not the orch's pane."
---

### [BLOCKING] A failed hinted read is never retried

**Location:** `_lym/discussion/decision-record.md:124-127` (D2), and the D2 acceptance criteria.

**Issue:**
D2 removes each hint before the read it triggers, which is right: a hint written during the read is a new file and a new wake, and a message sent before the hint is visible to a read made after the removal.
But the holder has already acknowledged the envelope (D1), so Slack will not send the event again, and the hint is gone.
D2 lists three triggers for a channel read: a hint, a resync hint, and the relay's start.
A read that fails after its hint was removed, on a network error, a 5xx or a rate limit, has none of the three left.
The operator's message stays unforwarded until the channel gets another message and hint, or until the next reconnect writes a resync hint, which is hours away (connections refresh every few hours, per Constraints).
D2's Rationale says reading after the cursor makes retries harmless, and that a lost hint is covered by the read for the next message.
That covers a lost hint only when a next message comes.
The operator is away and waiting for an answer in the common case, so no next message comes.
The poll this task replaces retried a failed read at its next pass, so this is a regression of the relay's delivery guarantee.
The focus question on ordering is settled in the record's favour: the removal is correctly ordered before the read.
The gap is what happens after a failed read.

### [MEDIUM] No trigger re-ensures a held channel for a failed invite or a changed operator

**Location:** `_lym/discussion/decision-record.md:249-255` (D5), and the D5 acceptance criteria.

**Issue:**
D5 says each ensure of a held channel invites the setup's operator when the entry records another operator or none, and that a refused invite is retried only at that channel's next ensure.
D5's list of ensures gives the prime's channel at relay start only, a beam's at its first routed mail, and listed beams at relay start or a presence flip to `slack`.
The prime's channel, once its entry is written, is therefore ensured again only at the next relay restart.
`lym slack setup --operator <user-id>` writes setup.json, which the relay does not watch (D2 watches presence.json alone), and nothing restarts the relay.
The notice D5 mails on a refused invite names that command as the way forward, but running it changes nothing until a restart.
A changed operator is likewise not invited until a restart, and the inbound filter forwards only the configured operator's messages.
The acceptance criterion 'retried at the next ensure' leaves the plan writer no event to implement.
The record already has the right mechanism for a refused prime channel, a re-ensure at each wake from the outbound read's wait; the fix reuses it.

### [MEDIUM] A failing `hubregistry.Live` read gives no notice

**Location:** `_lym/discussion/decision-record.md:49-51` (D1) and `194-196` (D3).

**Issue:**
D1 leaves every event unacknowledged while the `hubregistry.Live` read fails, and owes each hub a resync once it reads again.
Slack's retries of an unacknowledged event are bounded, so the resync is what recovers the messages, and it runs only after the read stands.
D3 mails a notice naming the hubs record's path, but only on the ensure path (an unarchive, invite or adopt).
A holder that has no ensure to make, the usual state of a quiet relay, mails nothing, so a corrupt or unreadable hubs record stops push for every hub on the machine with no signal.
The holder knows its own hub's geometry without the list, so it can name the record and mail its own orch.
The focus question on the outcome when `Live` fails to read is settled for events (unacknowledged, resynced later) and for ensures (fail closed with a notice); only the holder's silence is the gap.

## Rejected

- Focus 1, the upgrade path of D3: not a defect.
  A relay re-exec'd from an older binary has the bot token under the kept `token` key, and `setup.json` keeps `channel_id`, which `state.WriteJSON` preserves as an unknown key and which the old `lym slack setup` verified the bot is a member of.
  D3 posts the missing-token line there first, with the bot token, and then mails the orch, so a Slack channel the away operator watches gets the line.
  `OutcomeNotConfigured` keeps dobby from respawning the relay, so the line posts once.
  The relay makes the check after its lock and first creel read, so the orch mail is deliverable.
- Focus 2, the holder's hint write: not a defect.
  The record has the holder create its temp file in `.lym/slack/`, outside the inbox, then rename it in; the `internal/state` lock files sit under `.lym/slack/locks/` (`recordLockPath`), also outside the inbox, and `fsx.AtomicWriteBytes` writes a `.tmp-` dotfile that the drain ignores.
  The relay's watch on `.lym/slack/` takes the presence file alone, so the temp file wakes nothing.
  A hint renamed over one the relay has listed but not yet removed is removed unread, but the read that follows the removal sees the message that caused it, since the message precedes the event.
- Focus 3, `name_taken` reopen with no map entry: not a defect.
  D5 step 2 writes a new entry with the cursor and the thread floor at the channel's newest message, D2 and the acceptance criteria say the same, and D6's 'cursors stay' sentence applies only when the map already holds the entry.
- Focus 3, beam channels for a relay started while presence is already `slack`: not a defect.
  D5 ensures every listed beam's channel at the relay's start when presence is `slack`, and the acceptance criteria name that case.
- Focus 3, two ensures of one channel in one relay: not a defect.
  The per-orch lock and the re-check once it is held serialize them, and the acceptance criterion pins that the second makes no create or adopt call.
- Focus 4, the restart cap's lifetime for an idle relay: not a defect.
  Every relay stamps the last-healthy time at the end of each outbound read, which comes at least every sixty seconds (`waitCeiling` in `backoff.go`), and the holder stamps at each pong, so an idle relay that makes no Web API call still reads as alive.
  `lifetime` in `start.go` subtracts the spawn time from that stamp and compares it with `relayHealthyLifetime`, so the 'idle for over ten minutes, then ended' criterion holds.
