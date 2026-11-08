---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md:113
    summary: D2 never orders a hint's removal before its Slack read, so a hint the holder overwrites during the read is removed and the newer message waits for a resync or restart
    fix: In D2, after the sentence that one goroutine drains the inbox, add that the relay removes each hint before it makes the read that hint triggers, so a hint written during the read is a new file and a new wake. Add to D2's acceptance criteria that a hint written while its channel's read is in flight is read at the next wake.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:229
    summary: Reopening on name_taken an archived channel the map holds no entry for leaves its cursors undefined, and an empty cursor replays the channel's whole history to the orch
    fix: In D5 step 2, change the archived bullet to say the reopen (D6) writes a new entry with its cursor and thread floor at the channel's newest message, as adoption does. In D6, limit the sentence that the cursors stay where the entry left them to an entry the map already holds. Add to D5's acceptance criteria that a reopen on name_taken with no map entry forwards no message written before the reopen.
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md:487
    summary: With the poll gone an idle relay makes no Web API call, so LastSlackOK stops moving and the restart cap's lifetime reads a long-lived relay as one that died young
    fix: Add to D1 that the relay stamps the relay state's last-healthy time at each Socket Mode pong for the holder and at each outbound read's end for every relay, and that the restart cap's lifetime and `lym slack status` read that stamp. Change the Notes bullet on `slackengine.EnsureRelayWhileAway` to name this change. Add an acceptance criterion that a relay idle for over ten minutes, then ended, does not count as a young death.
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:160
    summary: On the upgrade re-exec D3 names, no channel map exists yet, so the missing-token line reaches no Slack channel and the away operator hears nothing
    fix: In D3, change the missing-token sentence so that, when the map does not hold the prime's channel and `setup.json` still carries an old channel key, the relay posts the same line to that old channel with the bot token, read once for that purpose only. Extend the D3 acceptance criterion to cover a relay re-exec'd from an older binary's state, posting to the old setup channel.
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md:221
    summary: Ensuring every listed beam when presence becomes slack has no presence baseline, so a relay started by lym slack away or restarted while away never ensures them
    fix: In D5, replace the third ensure bullet with every listed beam's channel at the relay's start when presence is slack, and at each wake that reads presence slack after it read terminal. Change the D5 acceptance criterion to cover a relay started by `lym slack away` and one restarted while presence is already slack.
  - id: F6
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:48
    summary: A failed read of the live-hub list itself has no stated outcome, so read as no hubs it lets every event be dropped and the user-token map test pass vacuously
    fix: In D1, extend the unreadable-map sentence to a `hubregistry.Live` read that fails, leaving every event unacknowledged for Slack's retry. In D3, extend the fail-closed paragraph the same way, so no unarchive, invite or adopt is made while the live-hub list cannot be read, with one notice naming the hubs record's path and its own way forward. Add both to the D1 and D3 acceptance criteria.
  - id: F7
    severity: MEDIUM
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md:236
    summary: The operator invite runs only when a channel is first made, so one that fails after the entry is written, or an operator changed by setup, is never invited, while the criterion says invited each time
    fix: In D5, add that the map entry records the operator it invited, and that each ensure of a channel the map holds invites the setup's operator when the entry records another or none, until the invite stands. Change the D5 criterion to say an invite that failed is retried at the next ensure, and that a channel the map holds invites a changed operator.
  - id: F8
    severity: LOW
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md:107
    summary: A hint written by the usual atomic write leaves a temp file and a locks directory in the inbox, which the drain removes as unparsed, failing the holder's rename
    fix: In D1, state that the holder writes a hint by creating a temp file in the hub's `.lym/slack/` directory, outside the inbox, then renaming it into the inbox. In D2, state that the drain ignores names starting with a dot.
  - id: F9
    severity: LOW
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md:185
    summary: Moving to a new app leaves running relays on the old tokens, and the new bot rejoins a map channel only through an outbound post refusal, so inbound stays dead until an orch posts
    fix: In D3, add that a login that changes a stored token ends every live relay of the machine's live hubs, so each restarts on the new tokens. Add to D8's `not_in_channel` handling that a start or resync read refused with `not_in_channel` invites the bot back under the same bound. Change the rationale sentence on the invite bringing the new bot back to name both paths.
  - id: F10
    severity: LOW
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md:147
    summary: The login match is undefined for an app-level or user token with no bot token stored, and for a bot token logged in after an app-level token, whose app id nothing stores
    fix: In D3, add that an app-level or user token with no bot token stored is refused, naming `lym slack login` with the bot token first, and that login stores the app-level token's app id from the hello in `slack.json` beside the token so a later bot-token login matches against it.
  - id: F11
    severity: LOW
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md:230
    summary: A live channel this bot created and no map holds, with the bot no longer a member, falls into the refusal with no way forward
    fix: In D5 step 2, change the live bullet to say a live channel the bot is not a member of is first joined by inviting the bot with the user token under D3's bound, then adopted. Add the user-token use to D3's list of calls.
  - id: F12
    severity: LOW
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md:101
    summary: The thread read does not say it skips the thread's parent, which conversations.replies can return
    fix: In D2, after the thread-hint sentence, add that a thread read skips the message whose timestamp equals its thread timestamp, since the top-level read handles it.
  - id: F13
    severity: LOW
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md:63
    summary: A holder whose re-exec fails has released the machine socket lock and only retakes its relay lock, so it never rejoins the socket lock wait
    fix: In D1, add that a holder whose re-exec fails rejoins the machine socket lock wait as it does after an exec.
  - id: F14
    severity: LOW
    class: consistency
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md:160
    summary: The missing-token notice and Slack line are not placed after the relay lock and the first creel read, against the relay's first-read-before-any-Slack-call invariant
    fix: In D3, state that a relay missing a token takes its relay lock and makes its first creel read before it mails the notice and posts the line, then exits.
  - id: F15
    severity: LOW
    class: consistency
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md:377
    summary: The rewording constraint covers only ways forward naming setup --channel, leaving the old bot-scope list and the refused-read notice pointing at setup for a channel
    fix: In the PATTERN-refusal-way-forward constraint, add that every way forward listing bot scopes names the D10 manifest's scopes and `docs/slack-app.md`, and that the relay's refused-read notice names `lym slack login` and the manifest, not `lym slack setup`.
  - id: F16
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:217
    summary: Two ensures of one beam channel in one relay, from the outbound read and the presence wake, are not serialized, so the loser adopts at the newest message and skips messages between
    fix: In D5, add that one relay ensures one orch's channel at a time, under a per-orch lock in the relay, and that an ensure finding the map now holds the channel stops there.
  - id: F17
    severity: LOW
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md:264
    summary: The record does not say where a beam's mail posts while the slackengine.Hub read fails
    fix: In D6, add that while the beams read fails, mail from a beam whose channel the map holds open posts there, and other beam mail waits for the next read.
  - id: F18
    severity: NIT
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md:419
    summary: The D2 criterion says the relay's record writes wake nothing, while D2's rationale says its presence writes do
    fix: In the D2 criterion, change its records to its relay state, outbox and channel map.
  - id: F19
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/discussion/decision-record.md:480
    summary: The new Slack boundary rule text, a requirement the diff must land, sits only in Notes for the plan writer, which the spec makes never a completeness requirement
    fix: Move the three sentences of the new Slack boundary rule from the first Notes bullet into the PATTERN-agent-addressing constraint in place of its pointer sentence, and drop that Notes bullet.
  - id: F20
    severity: NIT
    class: consistency
    origin: handler
    location: _lym/discussion/decision-record.md:487
    summary: The third Notes bullet restates D3's sentence on EnsureRelay, EnsureRelayWhileAway and dobby's restart
    fix: Drop the third Notes bullet, or keep only the part F3 adds.
  - id: F21
    severity: NIT
    class: consistency
    origin: handler
    location: _lym/discussion/support-log.md:76
    summary: The support log says the archived-name risk stays in Open risks, but Open risks reads None
    fix: In the support log's leanness 3 line, replace the sentence about the archived-name risk staying with one saying it moved to D5's rationale and the question ledger.
---

### [MEDIUM] Hint removal is not ordered before its read

**Location:** `_lym/discussion/decision-record.md:113`, with `:44-45` and `:128`

**Issue:** D1 overwrites a channel's hint on each event, and D2 removes hints, but the record never says which comes first: removing a hint or the read it triggers.
When the removal comes after the read, the holder can overwrite the hint for a newer message while the read is in flight, and the removal then deletes it.
That message waits for another message, a resync or a restart, which can be hours away, against the Goal of about a second.
D2's rationale lists the gaps push leaves and does not include this one.
Raised by interactions F1 and walkthrough F1.

### [MEDIUM] A name_taken reopen with no map entry has no cursor

**Location:** `_lym/discussion/decision-record.md:229`, `:263`

**Issue:** D6 keeps the cursors "where the entry left them", but D5's `name_taken` reopen runs for a channel the map does not hold.
That happens after a re-clone, since `.lym` is never tracked, and after D3's own way forward of moving an unreadable map aside.
The only cursor defined for a new entry is step 1's empty one, which reads from the channel's beginning and forwards the archived channel's whole operator history again.
Adoption sets its start at the newest message; the reopen does not.
Raised by bound-breaker F1 and walkthrough F2.

### [MEDIUM] The restart cap reads a long idle relay as a young death

**Location:** `_lym/discussion/decision-record.md:487`; `internal/slackengine/start.go:238-245`, `relay.go:117-133`

**Issue:** `lifetime` is `LastSlackOK` minus the spawn time, and `recordingSlack` stamps `LastSlackOK` only on a Web API call that succeeds.
Today the poll stamps it at least once a minute.
With push, an idle relay makes no Web API call after its start read, and a relay that does not hold the socket lock makes none at all.
A relay that ran for hours and then died counts as dying young, five in a row make `EnsureRelayWhileAway` answer `OutcomeGaveUp`, and `status`'s `last_slack_ok` stops showing health.
The Notes say dobby's restart keeps its logic, and nothing in the record stamps liveness.

### [MEDIUM] The upgrade's missing-token line reaches no channel

**Location:** `_lym/discussion/decision-record.md:160-162`, `:329`, `:436-437`

**Issue:** D3 names the case: a re-exec onto a binary of this task with only a bot token.
On that upgrade no `channels.json` exists, because the old relay used `setup.json`'s channel, so "when the map holds the prime's channel" never holds and the line posts nowhere in Slack.
The orch's notice cannot reach the operator either, because the relay that would post it has exited.
Presence stays `slack`, and `EnsureRelayWhileAway` answers `OutcomeNotConfigured` quietly.
The operator, away and watching the old channel, hears nothing.
Raised by bound-breaker F2 and interactions F4.

### [MEDIUM] Beam channels on presence slack have no baseline

**Location:** `_lym/discussion/decision-record.md:221`; `internal/slackengine/inbound.go:253-263`, `internal/slackcli/cli.go:426-446`

**Issue:** D5 ensures every listed beam "when presence becomes slack, seen at the next wake".
`lym slack away` starts the relay, waits only for its first creel read, then writes presence, so the relay can read presence for the first time already at `slack`.
The existing change detection counts a first read as no change.
A relay dobby restarts while presence is already `slack` sees no change either.
Neither ensures the beam channels, including ones `lym slack stop` archived.

### [MEDIUM] A failed live-hub list read has no bound

**Location:** `_lym/discussion/decision-record.md:48-49`, `:166`, `:171-172`; `internal/hubregistry/hubregistry.go:98-102`

**Issue:** `hubregistry.Live` returns an error for a `hubs.json` it cannot read or decode.
The record fails closed only for a single live hub's map that cannot be read.
If an implementer reads a failed list as no hubs, every event counts as unmapped while "every live map reads", so it is acknowledged and dropped.
The user-token test, a channel held by no other orch's entry in any live hub's map, then passes vacuously.
That is a guard bypass with no stated bound.

### [MEDIUM] The operator invite is never retried

**Location:** `_lym/discussion/decision-record.md:225`, `:236`, `:238-239`, `:443-444`

**Issue:** Step 3's operator invite sits under "Ensuring a channel the map does not hold", and the entry is written before it.
An invite that fails on a transport error or a rate limit therefore leaves a channel the map holds, which no later ensure touches, so the operator is never invited.
An operator changed later through `lym slack setup --operator` is never invited into the channels the map already holds either.
The acceptance criterion says "The operator is invited each time", which the decision does not do.
Raised by walkthrough F3 and interactions F6.

### [LOW] An atomic hint write races the drain

**Location:** `_lym/discussion/decision-record.md:107-109`, `:115`; `internal/fsx/fsx.go:57-73`

**Issue:** `state.WriteJSON` and `fsx.AtomicWriteBytes` create a `.tmp-` file in the target directory and rename it onto the name.
`state` also creates a `locks` directory beside the record.
A hint written that way puts a non-hint entry in the inbox, which the drain removes as unparsed, and the rename then fails.
The write counts as failed, so the hub is owed a resync and its orch gets a notice.
The record says neither how a hint is written nor which names the drain spares.
Raised by existing-paths F4 and bound-breaker F4.

### [LOW] Moving to a new app leaves relays and inbound stale

**Location:** `_lym/discussion/decision-record.md:152-153`, `:185-189`; `internal/slackcli/wiring.go:188`, `:198`

**Issue:** Each relay reads its tokens once at start.
A login that replaces the bot token and removes the others leaves the holder on the old app's connection and every relay on the old bot token, with "this machine's bot user" ambiguous.
The rationale says the invite brings the new bot back in, but the only invite path is a refused outbound post.
The start and resync reads have no invite path, and Socket Mode sends no events for channels the bot is not in, so a beam channel without outbound mail stays deaf.
Raised by interactions F3 and F5.

### [LOW] The login match is undefined for two orders

**Location:** `_lym/discussion/decision-record.md:143`, `:147-153`

**Issue:** "An app-level or user token that does not match the stored bot token is refused" assumes a stored bot token, so a first `xapp-` or `xoxp-` login on a fresh machine has no stated outcome.
A bot token logged in after an app-level token must be matched against that token's app id.
The record gets that id only from a live connection's `hello`, and it neither stores the id nor says the bot-token login opens a connection with the stored token.
Raised by walkthrough F5 and bound-breaker F8.

### [LOW] An own-created channel the bot left falls through

**Location:** `_lym/discussion/decision-record.md:227-234`

**Issue:** The creator branch admits an archived channel or a live one the bot is a member of.
A live channel this bot created and no map holds, from which the bot was removed, falls under "Any other is refused".
That branch's notice names a way forward only for another hub's map or another bot's channel, so this case gets none.
Raised by walkthrough F6 and bound-breaker F7.

### [LOW] The thread read does not skip the parent

**Location:** `_lym/discussion/decision-record.md:101`

**Issue:** `conversations.replies` can return the thread's parent message as its first item.
The record does not say the thread read skips it, so the parent, already handled by the top-level read, could be forwarded twice.

### [LOW] A failed re-exec leaves the socket lock wait

**Location:** `_lym/discussion/decision-record.md:63-64`; `internal/slackengine/relay.go:330-339`

**Issue:** The holder releases the machine socket lock before a re-exec.
The existing failure callback retakes only the relay lock, so a holder whose exec fails runs on without a connection and never rejoins the socket lock wait.
When it is the machine's only relay, push stops until a restart.

### [LOW] The missing-token notice has no place in the start order

**Location:** `_lym/discussion/decision-record.md:160-162`; `internal/slackengine/relay.go:185-187`, `internal/slackcli/wiring.go:188`

**Issue:** Today the relay refuses a missing token before its lock, its mailbox and its first creel read.
D3 has it mail and post before it exits, but it does not place that after the lock and the first read.
The relay's documented invariant is a first creel read before any Slack call.

### [LOW] Old ways forward still name the removed setup model

**Location:** `_lym/discussion/decision-record.md:377`; `internal/slackcli/cli.go:31`, `:330-332`, `internal/slackengine/inbound.go:297`

**Issue:** The constraint rewords only ways forward that name `lym slack setup --channel`.
`botScopes` lists the old scope set, `groups:*` included, and leaves out `channels:manage` and `chat:write.customize`.
The refused-read notice sends the operator to `lym slack setup` for a channel the bot cannot read.
Under D10, setup takes no channel, so both texts point the operator the wrong way.

### [LOW] Two ensures of one channel race in one relay

**Location:** `_lym/discussion/decision-record.md:113`, `:217-230`

**Issue:** The outbound read, on a beam's first routed mail, and the drain, on presence `slack`, can both ensure the same beam channel.
The loser gets `name_taken`, passes the map test because the holder is its own orch, and adopts the channel at the newest message.
That skips any operator message written in between and invites the operator twice.
D2's single-goroutine rule covers reads only.

### [LOW] Beam mail routing while the beams read fails

**Location:** `_lym/discussion/decision-record.md:215`, `:254`, `:264`

**Issue:** A beam counts as listed only after a successful read.
While the read fails, the record says neither whether a beam's mail posts to its channel or to the prime's, nor whether it waits.

### [NIT] The D2 criterion contradicts its rationale on records

**Location:** `_lym/discussion/decision-record.md:126-127`, `:419`

**Issue:** The criterion says the relay's writes to its records wake nothing.
The rationale says its presence writes do wake it, and `presence.json` is one of the hub's records.

### [LOW] The new Slack boundary rule sits only in the Notes

**Location:** `_lym/discussion/decision-record.md:359-361`, `:480-483`

**Issue:** The diff has to land the background's new Slack boundary rule, which makes it a requirement.
The Constraints line only points at the Notes for it, and the discussion spec says the Notes are design hints, never a completeness requirement.
The completeness-before-leanness test puts a requirement in Decisions or Constraints.

### [NIT] A Notes bullet restates D3

**Location:** `_lym/discussion/decision-record.md:487-488`, `:158-159`

**Issue:** The third Notes bullet repeats D3: the shared configured check answers `OutcomeNotConfigured` quietly, and dobby's restart spawns nothing for it.

### [NIT] The support log disagrees with Open risks

**Location:** `_lym/discussion/support-log.md:76-77`; `_lym/discussion/decision-record.md:400-402`

**Issue:** Turn 3's leanness line says "Only the archived-name risk stays" in Open risks, but Open risks reads `None.`
The question ledger and D5's rationale hold that risk.

## Rejected

- interactions F2, a fail-closed invite stalling the outbound queue with no cap: each repost Slack refuses with `not_in_channel` counts once toward `refusalCap` (D8, `outbound.go:184-200`), so the message drops after five refusals and the queue does not stall for 24 hours.
