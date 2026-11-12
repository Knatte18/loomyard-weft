---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D8 (held cursor, operator messages past a hold) with D6 (bridge-channel commands, answered once)
    summary: Operator messages after a held peer message are reread and handled again on every bridge poll, so answers repeat and a stale approve re-pins a peer after a later revoke
    fix: In D8, state that operator messages in a bridge channel are tracked apart from the delivery cursor, by a per-channel operator cursor or a per-channel set of handled message timestamps kept in the bridge map, so each operator message is handled exactly once whatever the delivery cursor does; add a D8 acceptance criterion that, under a held peer message, an operator approve, a later revoke and a plain-text line are each handled once across repeated polls and the revoke stands.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D4 (name_taken adoption, archived or gone channel dropped and found again)
    summary: A bridge channel archived by hand keeps its name, so the drop-then-create path meets name_taken, FindChannel returns the archived channel, and D4 states no outcome for it
    fix: In D4, state that a name_taken lookup that finds an archived channel holds the message with a reason of its own, such as bridge-archived, counting toward no refusal cap, with one relay notice per episode naming the channel and the way forward (rename the archived channel in Slack, which frees the name), as the orch-channel adopt does; add a D4 acceptance criterion for it.
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D4 (adopted channel reads from its start) and D8 (resync add, dedupe)
    summary: Reading an adopted or resync-added channel from its start rehandles every operator command in it and redelivers envelopes older than creel's seven-day dedupe window
    fix: In D4, bound the start read of an adopted or resync-added channel, so envelopes are delivered only when posted within creel's journal retention, older ones are skipped, and operator messages already in the channel when it enters the bridge map are skipped, not handled; add a D8 acceptance criterion that a channel added by resync with an old approve and an envelope older than the retention handles neither.
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md D3 (hour return) with D7 (held message posted once its hold clears)
    summary: Nothing tells the relay that creel returned a held message at its hour, so the relay posts it once the hold clears and the peer may get it twice after the sender reposts
    fix: In D7, state that before posting a held message the relay checks it is still pending, by taking again or by the report verb's answer naming a returned message, and drops a message creel already returned without posting it; add a D7 acceptance criterion that a held message returned at its hour is not posted when its hold clears.
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md D3 (journals a post as an outbound message with the receiving machine recorded)
    summary: A machine-outbound message journaled as an outbound event is misread by the retention carry, which copies a fixed field list, and by a binary that does not know the machine field, which starts a per-hub worker for an empty hub
    fix: In D3, state that a machine-outbound message is journaled as a new event of its own, carried with of naming that event and with its receiving machine and last hold reason, so a binary that does not know it reads it as an id only, as D8 does for machine-inbound; extend the D3 acceptance criterion so a machine-outbound message survives the retention carry and starts no worker in a fold that does not know it.
  - id: F6
    severity: MEDIUM
    class: design
    origin: handler
    location: _lym/discussion/decision-record.md D8 (unpinned-creator notice in the prime orch's channel)
    summary: The first-contact notice goes only to the prime orch's channel, which exists only once the prime's button was pressed, while the relay may run for a beam's orch alone, so the operator is never told in Slack to approve
    fix: In D8, state where the unpinned-creator notice goes when the prime orch has no open channel, for example in the channel of each orch of the hub whose button is on, and that it is also mailed to the prime orch as the holder's notices are today; add a D8 acceptance criterion for a hub whose prime orch has no channel.
  - id: F7
    severity: MEDIUM
    class: consistency
    origin: handler
    location: _lym/discussion/decision-record.md Acceptance criteria for D4 and D8
    summary: The rate bounds on paths any workspace member can trigger, one bridge resync hint per base interval per hub and one name_taken lookup per ceiling, have no acceptance criterion
    fix: Add a D8 acceptance criterion that repeated events in an unmapped channel within one bridge-poll base interval lead to one bridge resync per hub under an injected clock, and a D4 criterion that a lasting bridge-name-taken hold makes at most one FindChannel per bridge-poll ceiling.
  - id: F8
    severity: LOW
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md D3 (take verb) with D10 (bridge poll)
    summary: The record never says what makes the relay call take, so outbound bridge mail on an idle bridge waits up to the sixty-second poll ceiling with that latency unstated
    fix: In D3 or D10, state the relay's trigger for take, either creel wakes the relay on a machine-outbound post, or the relay takes on each bridge poll, and then state the resulting worst-case send latency.
  - id: F9
    severity: LOW
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md D4 and D8 (bridge channels read after the channel cursor, no thread floor)
    summary: Thread replies in a bridge channel are never read, so an operator approve or revoke written as a thread reply is silently ignored
    fix: In D8, state that operator messages in a bridge channel's threads are read and handled as D6 states, or that a thread reply is answered once in its thread as not handled with a pointer to write the command in the channel itself.
  - id: F10
    severity: LOW
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md D4 (invites the operator at creation only)
    summary: When the configured operator changes, the existing re-invite reads only the orch channel map, so the new operator is never invited into existing bridge channels
    fix: In D4, state that an operator change also invites the new operator into every channel of the bridge map, as orch channels are.
  - id: F11
    severity: LOW
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md D8 (resync reads each bridge- channel its map lacks)
    summary: Orch channels are named machine-first, so on a machine whose name is bridge or starts with bridge- every orch channel the bot made matches the resync's bridge- test
    fix: In D8, key the resync's channel test on the full bridge name shape, the bridge- prefix and the -- separator between two ends, and exclude every channel the hub's orch channel map holds.
  - id: F12
    severity: LOW
    class: consistency
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D7 (five refusals settle returned)
    summary: D7 does not say where the five-refusal count of bridge mail is kept, and the existing count lives in the outbox record every confirmed orch-mail read empties
    fix: In D7, state that the refusal count of a bridge message is kept in the bridge-post record, or in creel beside its last hold reason, never in the outbox record.
  - id: F13
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md D7 (button-off hold) with D10 (relay stops when no button is on); acceptance criterion D7 button-off
    summary: When the orch whose button went off was the hub's last one on, the relay stops and nothing reports button-off, so the criterion holds only while another orch keeps the relay running
    fix: In D7, state that button-off is reported only while the relay runs and that a message of a hub whose last button went off is returned at its hour as bridge-unsettled; restate the D7 button-off criterion with another orch's button on.
  - id: F14
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md D4 (bridge map holds the peer app) with D6 (approved-peer pin)
    summary: The bridge map keeps a copy of the peer app beside the approved-peer pin, and no decision says which read uses the copy or updates it after a re-approval by app id
    fix: In D4, drop the peer app from the bridge map's fields, since D8's checks read the approved-peer record's pin.
  - id: F15
    severity: LOW
    class: consistency
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md D8 (reply-to check) with D3 (take) and D5 (envelope reply-to)
    summary: The record never says that a machine-inbound event keeps the sender's sending id, or that a reply's machine-outbound message carries it as the envelope's reply-to the peer checks
    fix: In D8, state that the machine-inbound event keeps the sending id, and that a reply to it journals a machine-outbound message whose reply-to is that sending id, which take returns and the envelope carries.
---

### [BLOCKING] Operator messages past a held message are handled again on every poll

**Location:** `decision-record.md` D8 lines 169 and 181, D6 lines 113-116, the bridge map's one read cursor at D4 line 79.

**Issue:** A held peer message keeps the channel's one delivery cursor still, and D8 has operator messages past it handled anyway.
With no second cursor and no handled set, every bridge poll reads those operator messages again.
A plain-text line gets its "not delivered" answer on each poll, against D6's "answered once".
Worse, a held `button-off` message, an operator `approve osl-1046` in the bridge channel after it, then `revoke osl-1046` in an orch channel: the next poll handles the old `approve` again and re-pins the peer, so a revoke does not stand.
The existing inbound model keeps no memory of handled messages and leaves the cursor before an unhandled one (`internal/slackengine/inbound.go:698-715`).
The interactions and walkthrough lenses found the same defect at MEDIUM; the revoke undone silently on a trust pin makes it BLOCKING.

### [MEDIUM] A hand-archived bridge channel has no stated outcome

**Location:** `decision-record.md` D4 lines 81-91.

**Issue:** Archiving in Slack keeps the channel's name, unlike retirement, which renames under `ArchivedName`.
D4's drop then leads to create, `name_taken`, and `FindChannel`, which lists archived channels too (`internal/slackengine/api.go:284`, `exclude_archived` false).
D4's adoption checks only the creator, so the archived channel is either adopted, refused at the post, dropped and found again in a loop until the refusal cap returns the message, or meets an outcome D4 leaves unstated.
The orch-channel adopt refuses an archived channel with a rename way forward (`internal/slackengine/keeper.go:194-197`), and D4 carries neither the refusal nor the notice.
Found by the bound-breaker and interactions lenses and the handler.

### [MEDIUM] Reading an adopted or resync-added channel from its start has no lower bound

**Location:** `decision-record.md` D4 lines 85-86, D8 lines 151-155 and 183.

**Issue:** D4 reads every adopted or resync-added channel from its start and leans on D8's dedupe.
That dedupe covers only envelope ids creel still journals, which retention drops after seven days (`internal/creelengine/journal.go:68`), so a lost bridge map under the never-tracked `.lym/slack/` re-adds every channel at the startup resync and redelivers months of mail.
The dedupe covers no operator message at all, so the same reread handles every old `approve`, `revoke` and plain-text line again, which can re-pin a peer the operator later revoked.
The focus's first question, where the cursor starts, is answered by the record; this is what that answer lets through.
Merged from the bound-breaker and interactions lenses.

### [MEDIUM] A held message creel returned at its hour is still posted when its hold clears

**Location:** `decision-record.md` D3 lines 59-63, D7 line 138.

**Issue:** The take, report and settle verbs carry no answer telling the relay that a message it holds was returned.
Once the hold clears the relay posts it; the sender, told to "post it again" by the return notice (`internal/creelengine/outbound.go:169`), may already have, so the peer gets it twice under two ids.
D3's late-settle notice was built for a settle lost to a crash, and it does not stop this repost.
D7 line 138 reads as if a returned message is never posted, but no mechanism makes that true.

### [MEDIUM] Machine-outbound mail journaled as an outbound event is misread by existing readers

**Location:** `decision-record.md` D3 line 50.

**Issue:** The retention carry of a pending outbound entry copies a fixed field list ending at `ReceivingHub` (`internal/creelengine/fold.go:724-729`), so the receiving machine and last hold reason are lost on carry unless added.
A binary that does not know the machine field folds it as a per-hub outbound entry with an empty `ReceivingHub`, and `outboundHubs` (`internal/creelengine/outbound.go:108-121`) then yields an empty hub path that a worker starts for.
D8 avoids exactly this for machine-inbound mail with an event of its own carried under `of`; D3 is silent for outbound.

### [MEDIUM] The first-contact notice can land in a channel that does not exist

**Location:** `decision-record.md` D8 lines 158-162.

**Issue:** D8 posts the unpinned-creator notice in the prime orch's channel.
The holder's `postToPrimeChannel` posts only when the hub's channel map holds an open prime entry (`internal/slackengine/holder.go:539-551`), and per the board entry an orch has no channel until its button is first pressed.
The relay runs while any orch's button is on, a beam's alone included, so the operator who watches Slack is never told that a peer's channel waits on `approve`, and first contact stalls until the sender's message returns at its hour.
This settles the focus's fourth question: the rate is bounded, and the operator is told only when the prime orch has a channel.

### [MEDIUM] Rate bounds on member-triggerable paths have no acceptance criterion

**Location:** `decision-record.md` Acceptance criteria, D4 and D8 bullets.

**Issue:** Any workspace member can invite the bot into a channel and post there, which D8 turns into bridge resync hints.
D8's bound, one hint per bridge-poll base interval per hub, and D4's one `name_taken` lookup per bridge-poll ceiling, are the bounds the record gives for those paths, and no criterion checks either.
The D8 criterion at line 269 checks that a resync follows an event, not its rate.
Every other behaviour the focus's fifth question names has a criterion: the return after five refusals (line 262), receipt under a former name (line 273), the `bridge-name-taken` hold (line 251).
No Decision states an answer to non-command operator text in an orch channel, so none is owed there.

### [LOW] The relay's trigger for take is unstated

**Location:** `decision-record.md` D3 lines 51-53, D10 line 207.

**Issue:** D3 removes the scheduler's wake for machine-outbound mail and makes take its only reader, and D10 resets the poll on a bridge post, which only follows a take.
No decision says when the relay takes, so on an idle bridge a reply between orchs may wait up to the sixty-second ceiling, a latency the record neither chooses nor states.

### [LOW] Thread replies in a bridge channel are never read

**Location:** `decision-record.md` D4 line 85, D8 line 146.

**Issue:** Orch channels read threads through `scanThreads` and `ThreadFloor`; a bridge channel enters the map with no thread floor and is read after the channel cursor alone.
An operator `approve` written as a thread reply is ignored with no answer, against D6's answer in the channel.

### [LOW] An operator change is not invited into bridge channels

**Location:** `decision-record.md` D4 line 80.

**Issue:** `inviteOperators` in `internal/slackengine/keeper.go` iterates the orch channel map only, and D4 invites the operator only at creation, so a new operator set by `lym slack setup --operator` never joins existing bridge channels.

### [LOW] The resync's bridge- test also matches orch channels on some machine names

**Location:** `decision-record.md` D8 line 151.

**Issue:** Orch channels are named `<machine>-<shortname>-…` and made by this bot, so on a machine named `bridge` or `bridge-…` each one passes the resync's test for a `bridge-` channel the map lacks.
Each resync then reads every orch channel, and one whose newest bot post opens with an envelope line, which a verbatim orch-mail body can, enters the bridge map.

### [LOW] D7 leaves the refusal count's home unstated

**Location:** `decision-record.md` D7 line 136.

**Issue:** "As the relay drops orch mail today" points at counts kept in the outbox record, which `outbound.read` replaces with an empty record after each confirmed orch-mail read (`internal/slackengine/outbound.go:107`).
Kept there, a bridge message's count resets on every orch-mail read and the five-refusal return never fires before the hour.

### [LOW] The button-off criterion holds only while another orch keeps the relay up

**Location:** `decision-record.md` D7 line 133, D10 line 206, acceptance criterion at line 260.

**Issue:** When the orch whose button went off was the hub's last, D10 stops the relay, nothing reports `button-off`, and creel returns the message as `bridge-unsettled`.
The criterion "reports `button-off`" does not say it needs another orch's button on.

### [LOW] The bridge map's peer app duplicates the approved-peer pin

**Location:** `decision-record.md` D4 line 79, D6 line 119.

**Issue:** The bridge map stores each channel's peer app, and D6's record pins the peer's app; D8's checks read the pin.
No decision reads the map's copy, and nothing updates it after `approve <machine> <app id>` pins a new app, so it is a second source that goes stale.

### [LOW] The reply's sending id is never carried across

**Location:** `decision-record.md` D8 lines 182-187, D3 line 54, D5 line 98.

**Issue:** The peer accepts a reply only when its `reply-to` is an id that hub sent; that is the peer's sending id, not the receiver's inbound event id.
D8 records the sender's full address on the inbound event, and its dedupe implies the sending id is kept, but no decision says a reply's machine-outbound message carries the sending id as `reply-to`, or that take returns it.

## Rejected

- existing-paths 1 (MEDIUM, D3 "creel owns the hour" has no named mechanism): D3 states the behaviour and its acceptance criterion at line 245 checks it; which creel timer fires it, and how it re-arms on a daemon restart, is a mechanism Plan designs.
