# Decision record — slack-machine-bridge

## Goal

Let the orchs on two machines, each running its own `lym-<machine>` Slack app in one workspace, exchange creel mail through a Slack bridge channel the operator watches.
Mail from another machine reaches an orch as cross-machine mail, never as the operator's words, and only the orch of a machine the operator approved is trusted.
The operator needs no setup beyond Slack itself.

## Scope

In:

- The `<machine>::<name>` address: its parser in `internal/agentname`, creel's post and reply to it, its stamp.
- The bridge channel: name, creation, invites, joining, archiving, and the hub's map of its bridge channels.
- The bridge message's form, its sole parser, outbound posting and inbound reading in `internal/slackengine`.
- The machine-wide record of approved peer machines and the operator's in-Slack `approve` and `revoke` commands.
- Creel's program-only verbs through which the relay takes outbound bridge mail, reports and settles it, and delivers inbound bridge mail.
- The orch role's Slack file, the trust passages that name the `(other hub, …)` stamps, `docs/slack-app.md`, and the `PATTERN-agent-addressing` change.

Out:

- Any strand but an orch at either end; a pair's orch, which has no slack button.
- More than one Slack workspace, Slack Connect channels, and a change to the app manifest.
- Delivery receipts from the other machine: a post Slack accepted is delivered as far as the sender learns.
- Mail between two hubs of one machine, which keeps its daemon-to-daemon path.

## Decisions

### D1 — The cross-machine address and its one parser

A name of another machine is `<machine>::<name>`, joined by `agentname.MachineAddressSeparator`, the form `agentname.FormatMachineAddress` already writes for display.
`internal/agentname` gains its parser, the only place that splits it.
The machine part must pass `machinename.CheckName`, and the name part must parse as an agent name; the parser does not judge the role.
A post to a `<machine>::` name whose machine is this machine's current or a former name (`machinename.Names`) is refused, naming the bare name as the way forward.
The parser takes this machine's names and makes this own-name check before `machinename.CheckName` judges the machine part,
so a post naming any former name, one that fails the name rule included, gets this refusal.
Rationale: the operator fixed the form on 2026-11-08; no agent name holds `::` today, so the existing parsers reject it and nothing misreads it.

### D2 — Only an orch posts across machines, and only to an orch

A post to a `<machine>::` name is accepted only from an orch strand that has a slack button: the prime's orch or a beam's orch.
Its receiver's name must have role `orch`.
`post` and `post --reply` carry it; `inject`, `interrupt`, `hold`, `release`, `post --interrupt` and `forward` refuse a `<machine>::` name, naming `lym creel post`, as they refuse a name of another hub.
A message from another machine is forwarded only to a strand of this hub, as other-hub mail is.
Bound: this opens one edge, orch to orch across machines; a program, EXTERNAL, a pair's orch or any other strand is refused at post with its own way forward.
Rationale: the task is orchs talking, and one bridge channel per pair of orchs stays readable only if no other strand uses it.

### D3 — Creel journals the post and the relay carries it

Creel journals a post to a `<machine>::` name as a machine-outbound message, an event of its own under a new id with the receiving machine recorded, and answers that id at once.
Retention carries a pending one under `of` naming that event, with its receiving machine and last hold reason,
so a binary that does not know the event reads it as an id only, as for machine-inbound mail (D8).
A machine-outbound message is never handed to creel's per-hub outbound workers, the machine record or the `deliver` and `confirm` verbs:
the scheduler's start scan and its wake on a post start no per-hub worker for it, and the relay's take verb is its only reader.
The relay uses three program-only creel Go-client verbs for it:

- take: the pending machine-outbound messages not yet past their hour, oldest first, each with its reply-to;
  a message past its hour is returned before take answers; the relay calls take at each bridge poll (D10);
- report: a hold reason for a pending message (`peer-unapproved`, `button-off`, or a Slack error code), which creel keeps as the message's last hold reason;
- settle: delivered, with the Slack timestamp in a field of its own, once Slack accepted the post; or returned with a reason code.

Each verb's journal form:
report journals a `machine-held` event only when the reason differs from the message's last hold reason,
as the `stalled` event is written on a change of reason, and that record is the last hold reason the retention carry holds.
Settle delivered journals a `machine-delivered` event with the Slack timestamp in a field of its own,
and settle returned, or creel's own return at the hour, a `machine-returned` event with its reason.
Settle delivered is accepted for a returned message, where the per-hub settle leaves a settled message alone.
A settle delivered for a message already journaled delivered writes no record and posts no notice.
A binary that does not know these events folds them to nothing, and the message stays an id only.

Creel owns the hour, one hour from the post as for a message to another hub:
a machine-outbound message unsettled at its hour is returned with its last hold reason, or `bridge-unsettled` when it has none.
A timer of creel's scheduler returns it, and runs whether or not the relay runs:
it wakes at the earliest hour among the pending machine-outbound messages, and a machine-outbound post and the scheduler's start scan arm it.
The return notice, the reply mark and `--wait-reply` work as for an outbound message to another hub.
The relay, at its start and before taking new mail, settles delivered every accepted message its bridge-post record (D7) holds, so a settle lost to a crash is made again once the relay runs.
A relay down past a message's hour, which D10 allows while no button is on, leaves that message returned although the peer may hold it.
Creel answers a settle delivered for a message it already returned by journaling its `machine-delivered` event and posting the sender a `CREEL` notice that the returned message did reach the peer.
The post is refused at once, before journaling, when the sending orch's slack button is off, or the machine lacks a token or the hub its setup;
the refusal names the button or the `lym slack` verb as the way forward.
Creel learns these through a seam its wiring in `internal/creelcli` fills from `internal/slackengine`.
Only the `<shortname>:SLACK` program may send the take, report, settle and deliver (D8) verbs; any other sender is refused.
Bound: on Linux a program claim through the Go client is accepted from an orch strand's pane and from a process in no pane, so either can take, settle or deliver bridge mail, and so forge `(other machine, trusted orch)` mail to an orch of this hub.
Off Linux creel resolves identity from the environment, so any process, a strand pane included, can make the program claim and forge such mail.
Both are creel's existing program-claim bounds, which the `SLACK` program claim has today; the CLI has no program flag.
Rationale: the outbound map, its hour and its notices already model mail that leaves the hub; the relay alone may call Slack (`PATTERN-agent-addressing`).

### D4 — One bridge channel per pair of orchs

The channel's name is `bridge-<end>--<end>`, each end being the machine and the orch's creel name sanitized as `ChannelName` sanitizes, the two ends sorted, so both machines compute the same name.
An example is `bridge-neve-ly-orch--osl-1046-ly-orch`.
The name is cut at Slack's 80 characters with a hash as `ChannelName` cuts it.
Bound: the sanitized ends are not injective, so two pairs can compute one name only when two machine names overlap at a hyphen, such as `neve` and `neve-ly`; D4's creator check and D8's envelope check then keep mail of the wrong pair out.
The hub's bridge map, a new record under the hub's `.lym/slack/` written through `internal/state`, holds each bridge channel's id, local orch, peer address, invited operator, delivery cursor, operator cursor, thread cursors and thread floor.
The hub's bridge-notice record, beside the bridge map and written through `internal/state`, holds the relay's notice marks:
one per kind of relay notice of D4, D7 and D8, set when an episode's notice is posted and cleared when the episode ends,
so a restarted relay posts no second notice for an episode.
A send-side hold's mark (D4, D7) is keyed by the pair, its local orch and peer address, since the hold can come before the pair has a channel;
a read-side notice's mark (D8) is keyed by channel id, whether or not the bridge map holds the channel.
Dropping a channel from the bridge map leaves its marks as they are.
The relay of the side that sends first creates the channel, then invites the operator and the approved peer's bot user (Slack's conversations.invite).
A channel the relay creates enters the bridge map with every cursor and its thread floor at the channel's start.
When the setup's operator changes, the relay invites the new operator into every channel of the bridge map whose invited operator is another or none, as it does into orch channels.
A `name_taken` answer finds the channel with `FindChannel`, a paged listing of the workspace's channels, archived ones included.
While a `bridge-name-taken` or `bridge-archived` hold lasts, the relay tries the create again at most once per bridge-poll ceiling (D10),
so it makes that listing at most once per ceiling.
A channel the listing finds archived holds the message with reason `bridge-archived` and one relay notice per episode naming the channel,
with the way forward: unarchive the channel in Slack, and the next try adopts it when this bot or the approved peer's bot made it;
for a channel anyone else made, also rename it to free the name, then archive it again if wanted.
The relay never unarchives.
Slack refuses a rename of an archived channel (docs.slack.dev, the conversations.rename page, error `is_archived`, read 2026-11-12).
Otherwise the relay adopts the channel only when its creator is this bot or the approved peer's bot user, and joins it with conversations.join.
A channel the relay adopts, here or by a resync (D8), enters the bridge map with its delivery cursor at the channel's start,
and its operator cursor and thread floor at the channel's newest message,
so every envelope message already in it is read through D8's per-message rules, D8's retention bound and dedupe keep a reread from delivering twice,
and no operator message already in it is handled.
A channel of that name made by anyone else holds the message with reason `bridge-name-taken` and one relay notice per episode naming the channel.
An invite of the peer's bot that Slack refuses holds the message with Slack's error code, and the notice tells the operator to invite `@lym-<machine>` by hand.
The relay retries the invite at each bridge poll, and an `already_in_channel` answer counts as invited, so a hand invite clears the hold;
the Slack docs do not say that one app's bot can invite another app's bot user (docs.slack.dev, the conversations.invite page, read 2026-11-12), and this path carries the bridge either way.
A post refused because the channel is archived or gone drops the channel from the bridge map, and the next send creates or finds it again per this decision.
When an orch retires, its relay archives its bridge channels under `ArchivedName`, as it archives the orch's own channel.
Rationale: a computed name lets the side that sends second, or both at once, find the channel; an invite makes the peer's bot a member with no discovery step.

### D5 — The bridge message and its sole parser

A bridge message is posted with `PostAs` under the username `<machine>::<sender>` and the sender's icon, as orch mail is today.
Its text opens with one envelope line, `[lym-bridge] from=<machine>::<name> to=<machine>::<name> id=<id>`, with ` reply-to=<id>` for a reply, followed by the body.
`internal/slackengine` holds the one formatter and the one parser of the envelope; both addresses go through D1's parser.
A message whose first line does not parse is no bridge message and is skipped.
The body and envelope must fit creel's line cap with the longest stamp, checked at post as for other-hub mail.
Rationale: the envelope rests on `chat:write` alone, which the app holds, and the operator sees it.

### D6 — Approved peer machines

A machine-wide record of approved peers sits beside the credentials file, slack.json, in the credential directory and is written through `internal/state`.
It maps a machine name to the peer's Slack app id and bot user id and when the operator approved it.
Only the operator's Slack commands write it:
`approve <machine>` and `approve <machine> <app id>`, and `revoke <machine>`, handled like `away`, `back` and `status` in an orch channel, in a bridge channel as stated below, and only from the configured operator.
The command word and the machine name are matched case-insensitively, and the app id is kept as written.
In an orch channel, `approve` or `revoke` naming this machine's current or a former name is refused,
and the answer names the command to type in a channel of the other machine.
In a bridge channel, a relay handles `approve` and `revoke` only when they name that channel's peer machine,
and leaves one naming its own machine to the peer's relay.
Any other operator text there is answered once, by the relay of the end that sorts first in the channel's name,
as not delivered, with a pointer to the orch's own channel.
`approve <machine>` finds the bot users named `lym-<machine>` (users.list) and reads each one's app id (the profile's api_app_id from users.info).
One candidate is pinned, and the answer names its app id; none or several is refused in Slack, listing each candidate's app id for `approve <machine> <app id>`.
`approve <machine> <app id>` pins that app under that name whatever the app is called, so a renamed machine is approved again this way.
The `status` answer names this machine's own app id, so the operator can compare an app id on one machine with the other machine's `status` before approving.
The relay answers each command in the channel it was written in.
Bound: anyone signed in as the operator's Slack account can approve a machine, the same authority that already instructs every orch.
An approved app is trusted to name its sender.
`approve <machine>` trusts the one candidate it finds; an app another workspace member named `lym-<machine>` before the real one exists is caught only by the operator comparing the app id with `status`.
Rationale: the operator wants no setup beyond Slack; the app id pins the peer, since a workspace member can create another app named `lym-<machine>`.

### D7 — Sending: the relay posts machine-outbound mail

The relay posts each machine-outbound message to its pair's bridge channel, creating or finding it per D4 when the bridge map lacks it.
A message is held unposted, and its hold reason reported to creel, while:

- the peer is not approved (`peer-unapproved`); the relay posts one notice per held episode in the sending orch's channel asking for `approve <machine>`;
- the sending orch's button is off (`button-off`), switched off after the post was accepted;
  the relay reports it only while another orch's button keeps it running, and once no button is on the relay stops (D10),
  so creel returns the message at its hour with its last hold reason or `bridge-unsettled`;
- D4 holds it (`bridge-name-taken`, `bridge-archived` or the refused invite's code).

A message Slack refuses with an error code five times is settled returned with that code, as the relay drops orch mail today.
The relay counts a message's Slack refusals in its bridge-post record, never in the outbox record of orch mail.
D4's holds count toward no refusal cap: they last until they clear or creel returns the message at its hour.
A held message is posted once its hold clears, or creel returns it at its hour (D3).
Before posting a held message the relay takes again, and drops unposted a message take no longer returns, since creel has returned it.
Posting is at least once: the relay keeps each bridge message Slack accepted, with its Slack timestamp, in a bridge-post record of its own.
That record sits under the hub's `.lym/slack/`, written through `internal/state` and apart from the outbox record of orch mail,
so a restart never posts the message again once the record is written.
An accepted message's entry is removed only once creel has answered its settle, a refusal included;
a refusal count is removed once take no longer returns its message.

### D8 — Receiving: the relay reads bridge channels and delivers to creel

The relay reads each bridge channel of its bridge map after the channel's delivery cursor, on a wake hint for the channel, at its start and on its bridge poll (D10).
The holder routes an event in a channel a hub's bridge map holds to that hub, as it routes orch channels.
An event in a channel no hub's map holds becomes a bridge resync hint in the inbox of every hub of the machine whose relay runs,
at most once per bridge-poll base interval (D10) per hub.
A bridge resync hint is a hint of its own: it asks for a bridge resync only, never the existing resync hint's reread of every orch channel.
A bridge resync lists the channels the bot is in (users.conversations) and reads each channel of the bridge name's shape its map lacks:
a name with the `bridge-` prefix and a `--` between two ends, or for a name cut at 80 characters the `bridge-` prefix alone,
and a channel no orch channel map of a hub of this machine holds.
It adds the channel only when its creator is this bot or an approved peer's bot user,
and when the newest envelope message whose bot is this bot or whose app is the pinned app of its envelope's `from` machine
names, in its `from` or `to`, this machine's current or a former name with this hub's shortname.
That end is the channel's local orch, and the channel enters the map as D4 states for an adopted channel.
A channel whose end names an orch that is not a live orch of this hub is left unmapped,
and the resync posts one notice per channel in it naming the orch as retired.
A bridge-shaped channel whose creator bot's app no pin holds is left unmapped.
When its newest envelope message, read whoever posted it and only to route this notice, names this hub's shortname at either end,
the resync posts one notice per channel in the prime orch's channel,
or, when the prime orch has no open channel, in the open channel of each orch of the hub whose button is on;
it also mails the notice to the prime orch, as the holder mails its relay notices.
The notice names the channel, the app id and the envelope's `from` machine, and asks for `approve <machine>` or `approve <machine> <app id>` typed in the orch channel the notice is posted in;
once the app is approved, the next resync adds the channel.
A relay also runs a bridge resync at its start and at each bridge-poll ceiling.

Per message after the delivery cursor, in order:

- Its own bot's posts are skipped.
- The operator's messages are handled as D6 states for a bridge channel, each once:
  the relay handles one only when it is after the channel's operator cursor, then moves that cursor to it.
  So operator messages past a held peer message are handled, and a reread from the held delivery cursor handles none of them again.
- A message from any other human, or with no `bot_id`, is skipped.
- An envelope message posted longer ago than creel's journal retention, seven days, is skipped, since creel's dedupe is sure to hold a delivered id only within it.
- A message whose envelope `from` names this machine's current or a former name is skipped with one notice per episode in the bridge channel,
  naming the clash and asking the operator to give the peer machine a name this machine never held.
- A bot message's app is read with bots.info (`slackengine.API.BotAppID`) and cached per bot id.
- When the envelope's `from` machine has a pin and the app is not the pinned one, the message is skipped with one notice per episode in the bridge channel naming the app.
- When the `from` machine has no pin, the message is held, and the relay posts one notice per held episode in the bridge channel asking for `approve <machine>`; an approval delivers, in order, every held message posted within creel's journal retention.
- When the envelope's `to` is not this channel's local orch, the message is skipped.
- When the local orch is no longer a live orch of this hub, the message is skipped with one notice per episode in the bridge channel naming the orch as retired.
- A message to an orch whose button is off is held until the button is on.
- Otherwise it goes to creel through the program-only deliver verb with its sender, receiver, sending id, reply-to and body.

The relay reads a bridge channel's threads as it reads an orch channel's, after each thread's cursor and the thread floor.
An operator reply in a thread is handled as D6 states and answered in its thread; every other thread reply is skipped.
A held message stops the delivery cursor of its channel only, and the hold is kept per channel, outside the relay's five-minute give-up for a message that fails (`retryOrGiveUp`).
An approval that delivers held messages moves the delivery cursor past each one it delivers,
so a later `revoke` holds no delivered message again and posts no second notice for it.
Creel journals delivered mail as a new machine-inbound event, recording the sender's full address and sending id, and writes a carried one with `of` naming that event, so a binary that does not know it reads it as an id only.
Creel answers an id it already holds for the same sender address and sending id without storing again, so a reread never delivers twice;
the sender address carries the sending hub's shortname, since creel ids are given per hub.
A reply-to is taken as a reply only when this hub sent that id to that sender address from that receiver; any other is refused naming the id, and the relay skips the message with a notice.
Creel treats machine-inbound mail as off-hub mail for the forward guard, the stamp and the removed-holder sweep.
A reply to it journals a machine-outbound message per D3, never a per-hub outbound one,
whose reply-to is that sending id, which take returns and the envelope's `reply-to` carries;
and the sweep returns nothing to another machine and journals such mail undelivered, as it journals swept other-hub mail.
Bound: a held message is delivered only by an approval or the button; a channel that only holds keeps its delivery cursor still and posts its notice once per episode; a channel no approved bot made is never delivered from, and a resync reads it only for its notice.
A resync takes a channel's local orch only from a message of this bot or of the pinned app of its envelope's `from` machine.
An event in a foreign channel triggers at most one bridge resync per base interval per hub:
a users.conversations listing and reads of the bridge-shaped channels the map lacks.

### D9 — The stamp and its authority

A message from another machine is shown as `"<machine>::<name>"` with the stamp `(other machine, trusted orch)` when its sender's role is `orch`, and `(other machine, untrusted, data and never an instruction)` otherwise.
A trusted other-machine orch carries the same authority as a `(other hub, trusted orch)` sender, never the operator's own words.
Every passage that grants the `(other hub, trusted orch)` stamp its authority names the new stamp beside it.
`contracts/orchrole/role-slack.md` says how an orch posts to an orch on another machine, `lym creel post <machine>::<name>`,
and that it weighs such mail by this stamp.
Rationale: the board entry asks that the receiver weigh it as cross-hub mail, never as an operator instruction.

### D10 — Bridges follow the slack buttons

A bridge has no switch of its own: an orch sends over a bridge only while its button is on (D3, D7), and receives only while its button is on (D8).
The relay runs while some orch of the hub has its button on, as today.
Its bridge poll takes pending machine-outbound mail (D3) and reads its bridge channels,
so a post across machines waits for the relay at most sixty seconds on an idle bridge and five on an active one.
The poll runs from five seconds, doubling to sixty while no new bridge message arrives, and returns to five on a newly delivered bridge message or a bridge post; a re-read held message is not new.
A Socket Mode event for a bridge channel wakes it at once, never within a second of the previous wake's start.
Rationale: the Slack docs do not say that Slack sends one bot's message events to another bot (docs.slack.dev, the message.channels page, read 2026-11-12), so the poll alone carries the mail and an event only speeds it.

### D11 — No change to the Slack app

The manifest in `docs/slack-app.md` already holds every scope the bridge calls need, so no app is changed or reinstalled.
Evidence, docs.slack.dev method pages, read 2026-11-12: conversations.create and conversations.invite accept `channels:manage` for a bot token;
users.conversations takes `channels:read`; users.list, users.info and bots.info take `users:read`, and users.info returns the profile's api_app_id for a bot user.
The relay already calls conversations.create, conversations.invite, conversations.join and conversations.archive with this manifest, and `lym slack login` calls bots.info (`internal/slackengine/api.go`).
`docs/slack-app.md` gains how two machines' orchs pair: both apps in one workspace, and `approve <machine>` on each machine.

### Added after Discussion (arbiter, 2026-11-12): Round 3 hard-cap ruling: the four open findings are answered in the record

Decision: The Discussion review closes at round 3 with its four open findings answered by the record's text. F1 (hour owner): D3 names a scheduler timer, one hour from the post, armed by a machine-outbound post and the start scan, running whether or not the relay runs, with a criterion under an injected clock. In D3, 'the relay's take verb is its only reader' means the only reader that posts the message: creel's hour timer also reads a pending machine-outbound message, only to return it, as D3's hour paragraph and the plan-writer note on the hour timer say. F2 (archived way forward): D4 gives unarchive for a channel either bot made and unarchive then rename for one anyone else made, with the conversations.rename is_archived evidence. F3 (journal form): D3 names machine-held on a changed reason only, machine-delivered with the Slack timestamp, machine-returned, an idempotent late settle and an older binary folding them to nothing. F4 (notice state): D4's bridge-notice record keys send-side marks by pair and read-side marks by channel id and survives a map drop; D8 moves the delivery cursor past what an approval delivers.

Rationale: The round is at the hard cap, so the segment can only be accepted, and the arbiter accepts only when the fix in place answers each open design-wrong finding. Each finding's requested fix, or the fixer's justified variant (F2, F4), is present in the record with an acceptance criterion; the only residue is the phrase 'only reader' in D3, settled here by reading it as the relay's only reader.

## Constraints

- `PATTERN-agent-addressing`: only `internal/slackengine` calls Slack, and an agent reaches Slack only through creel; the bridge keeps both, adding creel's hand-off of a `<machine>::` post to the relay (see PATTERN changes).
- `PATTERN-agent-addressing`: only `internal/agentname` forms, parses or matches the `<machine>::` name.
- `PATTERN-tolerant-records`: the bridge map, the bridge-notice record, the bridge-post record, the approved-peer record and creel's new journal fields are written through `internal/state` and never refuse an unknown key.
- `PATTERN-lym-files`: the bridge map, the bridge-notice record and the bridge-post record sit in the hub's `.lym/slack/` with the other relay records, named through the geometry `internal/hubgeom` builds; the approved-peer record sits in the credential directory beside slack.json, never in a config layer.
- `PATTERN-daemon-wakeups`: the bridge poll is an unbounded wait and joins the background's list of the Slack relay's loops.
- `PATTERN-sole-parsers`: the envelope line has one parser, in `internal/slackengine`.

## Auto-mode assumptions

- Approval on both machines, once per peer (D6).
- Trust from the approved app alone, without `trusted_hubs` (D9).
- An operator message in a bridge channel is not mail (D6).
- No receipt from the other machine (Scope).

## Open risks

None.

<!-- lym:carry-over Discussion-Review -->
- Discussion-Review, round 3: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lym/reviews/discussion/round-3-review.md; fixer report: _lym/reviews/discussion/round-3-fixer-report.md.
  - bridge-archived-way-forward-impossible (class design, severity MEDIUM, open in rounds 3)
  - bridge-channel-hold-state-has-no-home (class design, severity MEDIUM, open in rounds 3)
  - machine-outbound-hour-has-no-owner (class design, severity BLOCKING, open in rounds 3)
  - relay-verb-journal-form-unstated (class design, severity MEDIUM, open in rounds 3)
<!-- /lym:carry-over Discussion-Review -->

## Acceptance criteria

- D1: `<machine>::<name>` parses into its machine and name; a bad machine part or a name that does not parse is refused by the parser; a post naming this machine's current or a former name with the prefix, a former name that fails the name rule included, is refused naming the bare name.
- D2: a post to a `<machine>::` name from a non-orch strand, a program, EXTERNAL or a pair's orch, or to a non-orch name, is refused with its way forward, and so are `inject`, `interrupt`, `hold`, `release`, `post --interrupt` and `forward` naming one.
- D3: a post from an orch with its button on journals a machine-outbound message and answers its id; with the button off, or a token or setup missing, it is refused before journaling, naming the way forward.
- D3: a pending machine-outbound message survives the retention carry with its receiving machine and last hold reason; a fold that does not know the event reads it as an id only and starts no worker for it, and folds the `machine-held`, `machine-delivered` and `machine-returned` events to nothing.
- D3: a pending machine-outbound message is returned at its hour with the relay not running, under an injected clock; take never returns a message past its hour.
- D3: an unchanged hold reason reported at repeated polls writes one `machine-held` record; a changed reason writes another.
- D3: no per-hub outbound worker starts for a machine-outbound message, at the scheduler's start or on a post.
- D3: the take verb returns pending machine-outbound messages oldest first; a settled-delivered message is not returned at its hour; an unsettled one is returned at its hour with its last reported hold reason, or `bridge-unsettled`, and a `CREEL` notice to the sender.
- D3: a relay restarted after posting settles delivered, before taking new mail, every message its bridge-post record holds.
- D3: a settle delivered for a message creel returned at its hour journals it delivered and posts the sender a `CREEL` notice that it reached the peer; a second settle delivered for it writes nothing and posts no second notice.
- D3: any sender but `<shortname>:SLACK` is refused by the take, report, settle and deliver verbs.
- D4: both sides compute the same channel name for a pair, whatever the order of the ends, within 80 characters.
- D4: the first send creates the channel and invites the operator and the approved peer's bot; `name_taken` adopts a channel made by either bot; a refused invite holds the message with Slack's code.
- D4: a channel of the bridge name made by anyone else holds the message with reason `bridge-name-taken` and posts one notice naming the channel.
- D4: a `name_taken` listing that finds the channel archived holds the message with reason `bridge-archived`, counted toward no refusal cap, and posts one notice naming the channel and its way forward; once the operator has unarchived a channel either bot made, the next try adopts it, and once the operator has unarchived and renamed one anyone else made, the next try creates the channel.
- D4: a relay restarted during a hold episode posts no second notice for it, a `peer-unapproved`, `bridge-name-taken` or `bridge-archived` hold of a pair with no channel included, and a resync after a restart posts no second notice for an unmapped channel.
- D4: a lasting `bridge-name-taken` hold makes at most one `FindChannel` per bridge-poll ceiling, under an injected clock.
- D4: an operator change invites the new operator into every channel of the bridge map that records another operator or none.
- D4: a refused invite is retried at each bridge poll, and an `already_in_channel` answer clears its hold.
- D4: a post refused for an archived channel drops it from the bridge map, and the next send creates or finds the channel again.
- D4: an orch's retirement archives its bridge channels.
- D5: the envelope formatter and parser round-trip `from`, `to`, `id` and `reply-to`; a first line that does not parse is skipped.
- D6: `approve <machine>` from the operator pins the one candidate's app and bot user and names its app id; none or several candidates is answered listing them; `approve <machine> <app id>` pins that app, an upper-case app id kept as written; `revoke` removes the pin; `status` names this machine's app id; the commands from anyone else are not handled.
- D6: `approve` or `revoke` naming this machine in an orch channel is refused naming the command for the other machine.
- D6: in a bridge channel, `approve <machine>` is handled only by the relay whose peer it names, and other operator text is answered as not delivered once, by the relay of the end that sorts first.
- D7: a message to an unapproved peer is not posted, reports `peer-unapproved`, posts one notice per episode, and is posted once the peer is approved.
- D7: while another orch's button keeps the relay running, pending mail of an orch whose button went off is not posted and reports `button-off`; once the hub's last button goes off, creel returns it at its hour with its last hold reason or `bridge-unsettled`.
- D7: a held message creel returned at its hour is not posted when its hold clears.
- D7: a bridge message Slack accepted is not posted again after a relay restart, nor after a confirmed read of orch mail; its bridge-post entry goes only once creel answered its settle.
- D7: a message Slack refuses five times is settled returned with the code, a confirmed read of orch mail between the refusals included; a D4 hold counts toward no refusal cap.
- D8: a bot message from the pinned app of its `from` machine is delivered once as inbound mail from `<machine>::<name>`; a reread of the same message stores nothing new.
- D8: two hubs of one sending machine that send the same id to one orch both have their messages delivered.
- D8: a message from an app other than its `from` machine's pin is skipped with one notice, and later messages of the pinned app are delivered.
- D8: a message from a machine with no pin holds the delivery cursor and posts one notice; an operator `approve` written after it in the same channel is handled; the approval delivers the held messages in order, and skips one posted longer ago than creel's journal retention.
- D8: under a held peer message, an operator `approve`, a later `revoke` and a plain-text line after it are each handled once across repeated polls, and the `revoke` stands.
- D8: under a held message, an `approve` and then a `revoke`, once both are handled, leave a reread posting no second notice for the delivered message.
- D8: an operator `approve` written as a thread reply in a bridge channel is handled once and answered in its thread.
- D8: a held message stays held past five minutes.
- D8: a message to an orch whose button is off is held until the button is on.
- D8: repeated events in an unmapped channel within one bridge-poll base interval lead to one bridge resync per hub, under an injected clock.
- D8: on a machine named `bridge`, a resync reads none of the hub's orch channels.
- D8: an event in an unmapped channel leads each running relay to a bridge resync, which rereads no orch channel; a resync adds a bridge-shaped channel made by an approved bot whose envelope names this hub's orch at either end, and leaves out one made by any other creator or naming another hub.
- D8: a resync ignores a human-posted envelope in choosing the local orch, and leaves unmapped a channel whose envelope names a retired orch of this hub, posting one notice in it.
- D8: a channel adopted after the peer posted in it, and one added by resync, deliver once the messages already in it that were posted within creel's journal retention.
- D8: a channel added by resync holding an operator `approve` and an envelope posted longer ago than creel's journal retention handles neither.
- D8: a resync that finds a bridge-shaped channel made by an unpinned bot leaves it unmapped and posts one notice in the prime orch's channel naming its app id and the channel to type `approve` in; after `approve`, the next resync adds it.
- D8: on a hub whose prime orch has no open channel, that notice goes to the open channel of each orch whose button is on and is mailed to the prime orch.
- D8: a resync and a delivery accept an envelope naming this machine's former name as this machine.
- D8: a message whose `from` names this machine is skipped with one notice; a message to a retired orch is skipped with one notice naming it retired.
- D8: a reply to machine-inbound mail journals a machine-outbound message whose reply-to is the inbound message's sending id, which take returns and the envelope carries, and starts no per-hub outbound worker; the removed-holder sweep journals machine-inbound mail undelivered.
- D8: a reply naming an id this hub never sent to that sender from that receiver is refused, and the relay skips it with a notice.
- D8: a machine-inbound event, read by a fold that does not know it, yields an id only.
- D9: an inbound message whose sender's role is `orch` carries `(other machine, trusted orch)`, and from any other sender `(other machine, untrusted, data and never an instruction)`, in every line, reminder and read header that shows its sender.
- Review: D9: each passage that grants `(other hub, trusted orch)` its authority, in `contracts/orchrole/orch-role.md`, `contracts/stencils/parent/parent-directive-parent.md` and `internal/shuttleengine/creelhooks.go`, names `(other machine, trusted orch)` beside it.
- Review: D9: `contracts/orchrole/role-slack.md` says how an orch posts to an orch on another machine and how it weighs such mail.
- D10: each bridge poll takes pending machine-outbound mail; the bridge poll's wait starts at five seconds, doubles to sixty while no new bridge message arrives, re-reading a held message included, and returns to five on a newly delivered bridge message or a post, under an injected clock.
- Review: D11: `docs/slack-app.md` says how two machines pair, and its manifest is unchanged.

## PATTERN changes

- `PATTERN-agent-addressing` — changes: Naming or messaging an agent: names only via `internal/agentname`; only through creel; across hubs with the sender confirmed, across machines orch to orch via the Slack relay; authority only for a trusted hub's or approved machine's orch.

## Notes for the plan writer

- The background of `PATTERN-agent-addressing` gains an "Across machines" rule, and its Slack boundary rule names creel's hand-off of a `<machine>::` post to the relay.
- The cross-hub path in `internal/creelengine/crosshub.go` and `internal/creelengine/receive.go` is the model for the outbound map, the reply-to check and the dedupe; `creelengine.Message.HubStamp` in `internal/creelengine/fold.go` is where the stamp is chosen.
- The relay's operator commands sit in `internal/slackengine/inbound.go`'s handling of `away`, `back` and `status`; channel creation, adoption and retirement sit in `internal/slackengine/keeper.go`; the holder's routing sits in `internal/slackengine/holder.go`.
- `internal/slackengine/keeper.go`'s notice for an archived orch channel names a rename Slack refuses; the bridge notice takes D4's way forward, not that text.
- The hour timer of D3 is a scheduler loop of its own; the per-hub `outboundWorker.run` in `internal/creelengine/outbound.go` never sees machine-outbound mail.
