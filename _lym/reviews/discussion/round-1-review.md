---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md D4 (name_taken adoption) and D8 (resync add)
    summary: No Decision says where an adopted or resync-added bridge channel's read cursor starts, and the existing adopt starts it at the newest message, so the first bridge message is lost on the race and first-contact paths, against Q6's no-lost-first-message purpose
    fix: In D4, after the adoption sentence, and in D8, after the resync add sentence, state that a bridge channel entered into the bridge map by adoption or resync starts its read cursor at the channel's start, with no thread floor, so every envelope message already in it is read through D8's per-message rules and the dedupe of D8 keeps a reread from delivering twice; add a D8 acceptance criterion that a channel adopted after the peer posted, and one added by resync, deliver the messages already in it once.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md D3 (Creel owns the hour; re-settle at start) with D10 (relay runs only while a button is on)
    summary: A posted message whose settle was lost is returned at its hour when the relay is down for over an hour, which D10 allows, and D3 does not say what creel does with a later settle-delivered for a message it already returned
    fix: In D3, replace the guarantee that a lost settle is made again before the hour with the bound that holds, namely that a relay down past the hour leaves a posted message returned, and state creel's answer to a settle-delivered for a returned message, recommended that creel journals it delivered and posts the sender a CREEL notice that the returned message did reach the peer; add a matching D3 acceptance criterion.
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md D7 (Posting is at least once) and D3 (relay re-settles from its outbox record)
    summary: The existing outbox record is wiped whole on every confirmed SLACK-mailbox read, so a bridge post kept there is forgotten at the next orch-mail read, defeating both the no-repost claim and the restart re-settle
    fix: In D7, state that the relay keeps each bridge message Slack accepted in a record of its own under the hub's .lym/slack/, written through internal/state, separate from the outbox record of orch mail, and that an entry is removed only once creel has answered its settle; make D3's restart re-settle read that record.
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md D6 (commands in any channel the relay reads) with D8 (both machines read a bridge channel)
    summary: An operator command or plain text in a bridge channel is handled by both machines' relays, so approve can pin a machine's own app under its own name, revoke acts on both, and every line is answered twice
    fix: In D6, state that a relay handles approve and revoke only when the named machine is not this machine's current or a former name, refusing one that is with an answer naming the bare command on the other machine, and that in a bridge channel a relay handles a command only when it names the peer machine of that channel, ignoring any other operator text there; change D8's non-command operator text rule to apply to orch channels only, and add acceptance criteria for both.
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D8 (resync adds a channel by its newest envelope message)
    summary: Resync adoption takes the local orch from the newest envelope message posted by anyone, so any workspace member can post a fake envelope in a public bridge channel and make another hub adopt it with the wrong local orch
    fix: In D8's resync sentence, replace the newest message holding an envelope with the newest message holding an envelope whose bot is this bot or whose app is the pinned app of the envelope's from machine, and extend the Bound line to say adoption trusts only such messages; extend the D8 resync acceptance criterion with a human-posted envelope that is ignored.
  - id: F6
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md D8 (an event in a channel no hub's map holds becomes a resync hint in every hub)
    summary: The new routing edge turns every message in any channel the bot was invited to into a hint on every hub, and the existing resync hint also rereads every orch channel with its thread scan, with no bound stated
    fix: In D8, name a bridge resync hint of its own, distinct from the existing resync hint, that runs only the users.conversations listing and the reads of unmapped bridge- channels; state that the holder writes it only for an event whose channel name it cannot place, at most once per bridge-poll base interval per hub, and add a Bound line saying what an event in a foreign channel can trigger.
  - id: F7
    severity: MEDIUM
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md D8 (resync admits only a channel made by this bot or an approved peer's bot; no-pin hold)
    summary: On first contact the receiving machine has not approved the sender, so the sender's channel is never adopted, the no-pin hold and its notice never run, and the receiving operator is told nothing
    fix: In D8, state that a resync that finds a bridge- channel made by a bot whose app no pin holds posts one notice per channel, in that channel or in the hub's orch channel, naming the app id and asking for approve <machine> or approve <machine> <app id>, and leaves the channel unmapped; once the app is approved the next resync adopts it per F1; add a D8 acceptance criterion for the notice.
  - id: F8
    severity: MEDIUM
    class: consistency
    origin: handler
    location: _lym/discussion/decision-record.md Acceptance criteria
    summary: Several Decision behaviours have no checkable criterion, namely D7's return after five Slack refusals, D8's receipt under a former machine name, D8's not-delivered answer to non-command operator text and D4's bridge-name-taken hold reason and notice, and the Review criterion for role-slack.md states content no Decision states
    fix: Add acceptance bullets - D7 a message Slack refuses five times is settled returned with the code; D8 a resync and a delivery accept an envelope naming this machine's former name as this machine; D8 non-command operator text in an orch channel is answered as not delivered; D4 a channel of the bridge name made by anyone else holds the message with reason bridge-name-taken and posts one notice naming the channel. Move the role-slack.md content of the D11 Review criterion into D11 or D9 as a stated decision, so the criterion checks a Decision.
  - id: F9
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md D4 (refused invite holds) and D7 (five Slack refusals return)
    summary: A refused invite and a bridge-name-taken hold are Slack error codes, so the record does not say whether they count toward D7's five refusals, nor how a hand invite clears the hold
    fix: In D7, state that D4's holds count toward no refusal cap and last until cleared or the hour, and in D4 state that the relay retries the invite at each bridge poll, an already_in_channel answer counting as invited, so a hand invite clears the hold.
  - id: F10
    severity: LOW
    class: consistency
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D4 (name_taken ... a listing made once per pair)
    summary: A bridge-name-taken hold retries create and the full paged channel listing at each bridge poll, so the once-per-pair listing claim is false on that path
    fix: In D4, state that a refused find is kept per pair in memory and not repeated until the hold's notice episode ends or the relay restarts, or else replace once per pair with the true rate, at most once per bridge-poll ceiling while the hold lasts.
  - id: F11
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D3 Bound (program claim from an orch pane or a process in no pane)
    summary: The stated forgery bound holds only on Linux, since off Linux identity is the environment alone and any strand pane can claim the SLACK program
    fix: Extend D3's Bound with the sentence that off Linux creel resolves identity from the environment, so any process, a strand pane included, can make the program claim and forge (other machine, trusted orch) mail, the same bound the existing SLACK program claim has there.
  - id: F12
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D8 (from machine with no pin is held)
    summary: A message whose from names this machine is held forever with a notice asking the operator to approve this machine itself
    fix: In D8's per-message list, before the pin checks, add that a message whose envelope from names this machine's current or a former name is skipped with one notice per episode in the bridge channel; add an acceptance bullet.
  - id: F13
    severity: LOW
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md D4 (retirement archives bridge channels) with D8 (resync and deliver)
    summary: After an orch retires, the peer recreates the channel, resync adopts it with the retired orch as local end, and nothing says what happens to mail delivered to an orch that no longer exists
    fix: In D8, state that a resync never adopts a channel whose local orch is not a live orch of this hub, and that a message to such an orch is skipped with one notice per episode in the bridge channel naming the orch as retired.
  - id: F14
    severity: LOW
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md D6 (approve handled like away, back and status)
    summary: The existing command handling lower-cases the whole text before matching, which would lower-case the app id of approve <machine> <app id> so it matches no app
    fix: In D6, state that the command word and machine name are matched case-insensitively and the app id argument is kept as written; extend the D6 acceptance criterion with an upper-case app id.
  - id: F15
    severity: LOW
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md D2, D8 and D9 (machine-inbound mail)
    summary: The record does not say how machine-inbound mail behaves where creel branches on another-hub origin, so a reply could reach a per-hub outbound worker or the forward guard and stamp could be skipped, and the removed-holder sweep has no stated outcome
    fix: In D8, state that machine-inbound mail is off-hub mail for the forward guard, the stamp and the removed-holder sweep, that a reply to it journals a machine-outbound message per D3 and never a per-hub outbound one, and that the sweep returns nothing to another machine and journals such mail dropped.
  - id: F16
    severity: NIT
    class: consistency
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D1 (refuse this machine's current or a former name)
    summary: machinename.Names keeps a former name that fails CheckName, and D1's parser refuses that machine part first, so the post gets the parser's error, not the stated way forward
    fix: In D1, state that the own-name check runs before CheckName on the machine part, so a post naming any former name, valid or not, is refused naming the bare name.
---

### [BLOCKING] Adopted or resync-added bridge channel loses its first message

**Location:** decision-record.md D4 and D8.
**Issue:** The existing adopt (`internal/slackengine/keeper.go:226-233`) starts the cursor and thread floor at the newest message.
The record leaves the bridge cursor's start unstated, so the existing behaviour is the default.
When both sides send at once, the side that adopts the channel skips the message already in it.
After first contact, the resync that adds the channel skips the message that caused the add.
Both paths defeat Q6's "no lost first message".

### [MEDIUM] Lost settle past the hour returns delivered mail

**Location:** decision-record.md D3 with D10.
**Issue:** D10 stops the relay when no button is on, so the relay can stay down past `outboundDeadline` (`internal/creelengine/outbound.go:24`).
D3's "before the hour" then fails: creel returns mail the peer received.
D3 does not say what creel does with the late settle-delivered.

### [MEDIUM] Bridge posts kept in the outbox record are wiped by orch-mail reads

**Location:** decision-record.md D7 and D3.
**Issue:** `outbound.read` saves an empty `outboxRecord` on every confirmed read (`internal/slackengine/outbound.go:106-108`).
A bridge entry kept there is gone at the next orch-mail read, so a restart reposts the message and the re-settle finds nothing.

### [MEDIUM] Operator commands in a bridge channel run on both machines

**Location:** decision-record.md D6 with D8.
**Issue:** Both relays read every bridge channel, and commands are gated on the operator alone (`internal/slackengine/inbound.go:599-613`).
So `approve osl-1046` typed there also runs on osl-1046 and pins its own app under its own name, and `revoke` acts on both machines.
Each machine also answers every line.

### [MEDIUM] Resync adoption trusts an envelope from any poster

**Location:** decision-record.md D8, resync.
**Issue:** Bridge channels are public (`api.go:263`).
The creator check passes, and then the newest envelope message, whoever wrote it, picks the local orch.

### [MEDIUM] Unmapped-channel events trigger full resyncs on every hub

**Location:** decision-record.md D8.
**Issue:** Today `route` drops such an event (`holder.go:352-354`), and `readsForHint` for `resync` rereads every open orch channel with its thread scan (`inbound.go:358-365`).
D8 sends every message in any channel the bot was invited to through that path on every hub.
Only the one-second floor bounds it.

### [MEDIUM] First contact leaves the receiving operator uninformed

**Location:** decision-record.md D8.
**Issue:** D7 makes the sender approve the receiver before it posts, and D8 adopts only a channel whose creator bot is approved.
On first contact the receiver has not approved the sender, so the sender's channel is never mapped.
The no-pin hold and its notice therefore never run.

### [MEDIUM] Acceptance criteria miss stated behaviours

**Location:** decision-record.md Acceptance criteria.
**Issue:** These behaviours have no criterion:
- D7's return after five refusals;
- D8's receipt under a former machine name;
- D8's not-delivered answer to plain operator text;
- D4's `bridge-name-taken` reason and notice.

The Review criterion under D11 checks `contracts/orchrole/role-slack.md` content that only Scope names.

### [LOW] D4 holds versus D7's five-refusal return

**Location:** decision-record.md D4, D7.
**Issue:** `refused` counts every API error and `errChannelRefused` toward `refusalCap` (`outbound.go:356-370`).
The record says neither which rule wins nor how a hand invite clears the hold.

### [LOW] "Once per pair" listing is false on the hold path

**Location:** decision-record.md D4.
**Issue:** `FindChannel` pages every public channel (`api.go:276-308`), and a held `bridge-name-taken` retries it at each poll.

### [LOW] Program-claim bound holds only on Linux

**Location:** decision-record.md D3 Bound.
**Issue:** `resolveFromEnvironment` (`identity.go:121-129`) accepts a program claim from any process off Linux.

### [LOW] A `from` naming this machine is held with a self-approve notice

**Location:** decision-record.md D8.
**Issue:** The no-pin hold applies to this machine's own name too, and with F4 it invites the operator to pin a foreign app under that name.

### [LOW] Mail to a retired orch through a recreated bridge

**Location:** decision-record.md D4, D8.
**Issue:** After retirement the peer recreates the channel under the freed name, and resync can adopt it for the retired orch.
No outcome is stated for that mail.

### [LOW] App id lower-cased by command matching

**Location:** decision-record.md D6.
**Issue:** `handle` lower-cases the whole text (`inbound.go:606`), and Slack app ids are upper case.

### [LOW] Machine-inbound mail in creel's other-hub branches

**Location:** decision-record.md D2, D8, D9.
**Issue:** The reply path, the forward guard, the stamp and the sweep all branch on `SendingHub` (`verbs.go:354-367`, `verbs.go:1035`, `fold.go:103`, `sweep.go:239`).
The record names only the stamp and states no outcome for the reply path or the sweep.

### [NIT] Former name failing CheckName

**Location:** decision-record.md D1.
**Issue:** `machinename.Names` keeps such names (`machinename.go:79-83`), and D1's parser refuses them before the own-name refusal can run.

## Rejected

- Walkthrough 6, nothing wakes the relay when an approval clears a hold: the relay's take loop rereads pending mail on its own five-to-sixty-second backoff, so a hold cleared by another hub's relay is posted within a minute without a wake.

## Focus notes

- Slack facts (D4, D10): both facts are recorded with their source and date, and neither is left for later.
  D10's poll fully bounds the bot-to-bot event gap.
  D4's fallback for a refused invite is bounded only once F9 says how the hold clears.
- Leanness (D3, D4, D6, D8): no deliberation narrative was found to relocate.
  The docs-gap sentences in D4 and D10 are the evidence the spec requires in the Decisions, so they stay.
- Consistency with the Turn 3 ledger: hold versus skip on a mismatched pin, creel owning the hour, and per-channel holds outside `retryOrGiveUp` all appear in D3 and D8 as written.
  Open risks "None" agrees with the Question ledger.
  The ledger's "one listing per pair" fix does not hold (F10).
