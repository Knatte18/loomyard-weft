# Card 9 — channel-keeper

**Create:**
- `plan:internal/slackengine#channelKeeper` -> `type channelKeeper struct`
- `plan:internal/slackengine#newChannelKeeper` -> `func newChannelKeeper(geom Geometry, slack SlackClient, liveHubs LiveHubs, mailbox Mailbox, clock Clock) *channelKeeper`
- `plan:internal/slackengine#channelKeeper.ensure` -> `func (k *channelKeeper) ensure(ctx context.Context, orch string) (ChannelEntry, error)`
- `plan:internal/slackengine#channelKeeper.create` -> `func (k *channelKeeper) create(ctx context.Context, orch string) (ChannelEntry, error)`
- `plan:internal/slackengine#channelKeeper.adopt` -> `func (k *channelKeeper) adopt(ctx context.Context, orch, name string) (ChannelEntry, error)`
- `plan:internal/slackengine#channelKeeper.heldElsewhere` -> `func (k *channelKeeper) heldElsewhere(orch, channelID string) (string, error)`
- `plan:internal/slackengine#channelKeeper.join` -> `func (k *channelKeeper) join(ctx context.Context, orch string) error`
- `plan:internal/slackengine#channelKeeper.inviteOperator` -> `func (k *channelKeeper) inviteOperator(ctx context.Context, orch string) error`
- `plan:internal/slackengine#channelKeeper.inviteOperators` -> `func (k *channelKeeper) inviteOperators(ctx context.Context) error`
- `plan:internal/slackengine#channelKeeper.retire` -> `func (k *channelKeeper) retire(ctx context.Context, beams []string, retryRefused bool) error`
- `plan:internal/slackengine#channelKeeper.botUser` -> `func (k *channelKeeper) botUser(ctx context.Context) (string, error)`
- `plan:internal/slackengine#channelKeeper.notice` -> `func (k *channelKeeper) notice(key, line string)`
- `plan:internal/slackengine#errChannelRefused` -> `var errChannelRefused = errors.New("slack: the channel was refused")`
- `plan:internal/slackengine#ChannelKeeper` -> `type ChannelKeeper = channelKeeper`
- `plan:internal/slackengine#NewChannelKeeper` -> `func NewChannelKeeper(geom Geometry, slack SlackClient, liveHubs LiveHubs, mailbox Mailbox, clock Clock) *ChannelKeeper`
- `plan:internal/slackengine#channelKeeper.Ensure` -> `func (k *channelKeeper) Ensure(ctx context.Context, orch string) (ChannelEntry, error)`
- `plan:internal/slackengine#channelKeeper.Join` -> `func (k *channelKeeper) Join(ctx context.Context, orch string) error`
- `plan:internal/slackengine#channelKeeper.InviteOperators` -> `func (k *channelKeeper) InviteOperators(ctx context.Context) error`
- `plan:internal/slackengine#channelKeeper.Retire` -> `func (k *channelKeeper) Retire(ctx context.Context, beams []string, retryRefused bool) error`
- `plan:internal/slackengine_test#keeperHarness` -> `type keeperHarness struct`
- `plan:internal/slackengine_test#newKeeperHarness` -> `func newKeeperHarness(t *testing.T) *keeperHarness`
- `plan:internal/slackengine_test#TestChannelKeeper` -> `func TestChannelKeeper(t *testing.T)`

**Uses:**
- `plan:internal/slackengine#ChannelName`
- `plan:internal/slackengine#ArchivedName`
- `plan:internal/slackengine#ChannelMap`
- `plan:internal/slackengine#ChannelEntry`
- `plan:internal/slackengine#ReadChannelMap`
- `plan:internal/slackengine#UpdateChannelMap`
- `plan:internal/slackengine#LiveHubs`
- `internal/slackengine#ReadSetup`
- `internal/slackengine#orchName`
- `internal/slackengine#isAPIError`
- `internal/slackengine#slackTimestamp`
- `internal/slackengine#relayNoticePrefix`
- `internal/slackengine_test#fakeSlack`
- `internal/slackengine_test#fakeMailbox`

**Intent:**
Make, adopt, join, invite and retire the hub's channels (D5, D6 and the D3 map test, as the operator-added entry amends them), in a new file `internal/slackengine/keeper.go`.
The relay wires the keeper in cards 10 to 12; no Slack call is made while a map lock is held, and every map write goes through `UpdateChannelMap`.

`ensure(orch)` runs under a per-orch mutex of the keeper, so one relay ensures one orch's channel at a time.
Holding it, it reads the map again: an open entry is a held channel, for which it only invites the setup's operator when the entry records another operator or none (`inviteOperator`) and returns the entry.
An absent or archived entry makes a new channel (`create`): `conversations.create` for `ChannelName(shortname, machine, slug)` as a public channel, then at once a new entry with an empty cursor, replacing an archived one, then the operator's invite.
The entry stands before the invite, so the operator's first message is routable; a refused invite fails nothing the entry holds and is retried at the next trigger.
An invite Slack refuses mails the hub's orch one `[slack relay]` notice per episode naming the code and `lym slack setup --operator <user-id>`.
`setup.json` is read at each ensure and each invite, so a changed operator applies at the next trigger.

On `name_taken`, `adopt` reads `conversations.list` for that name (D5 step 2).
- A live channel whose `creator` is this machine's bot user, the `user_id` `auth.test` gives (`botUser`, read once and kept), and that `heldElsewhere` clears, is adopted:
  when the bot is no member it first joins with `conversations.join`, then the entry is written with its cursor and thread floor at the channel's newest message (`Latest`, or the clock's now for an empty channel).
- An archived channel of that name is never unarchived and is refused like another bot's channel: one notice naming the channel and renaming it in Slack, which frees the name.
- A channel another hub's map holds is refused with one notice naming the duplicate-shortname way forward, removing the extra clone of the hub since a recorded shortname never changes;
  one another orch of this hub holds is refused with one notice naming the channel, the orch holding it, and renaming that channel in Slack, which frees the name.
- A name `conversations.list` no longer finds is an error, retried at the next trigger.
Each refusal returns an error wrapping `errChannelRefused`, so callers tell it from a transient failure.

`heldElsewhere(orch, id)` reads this hub's map and every map `LiveHubs` lists, and answers the orch or hub path holding the channel for another orch.
A nil `LiveHubs` lists the keeper's own hub alone; the relay's wiring always sets it.
A map that cannot be read or decoded, or a failed `LiveHubs` read, fails closed: no join or adopt is made, and the hub's orch gets one notice naming the map's path, with moving that file aside as the way forward,
or naming the hubs record, with moving it aside and `lym creel inbox` in each hub a minute on as the way forward.
`join(orch)` is the D8 recovery after a `not_in_channel`: it calls `conversations.join` for the orch's open entry only when `heldElsewhere` clears it.
The bound: the map test sees only the hubs the live-hub list names, so a hub missing from it is unseen until it records itself.

`inviteOperators` invites the setup's operator into every open entry that records another operator or none, recording it in each entry once the invite stands; card 12 calls it at each wake from the outbound read's wait.
`retire(beams)` takes each open beam entry, never the prime's (`ly:orch`), whose slug `beams` does not list;
it renames the channel with `conversations.rename` to `ArchivedName(name, now, n)`, trying n 1, 2 and on while Slack answers `name_taken`, and records the new name in the entry as soon as the rename stands;
it then archives it with `conversations.archive` and marks the entry archived.
An entry whose name is already its dated name is not renamed again, so a retry after a refused archive goes straight to the archive.
A rename Slack refuses with another code, or an archive it refuses, leaves the entry open, returns the error and marks the orch refused in the keeper's memory;
a later call skips a refused orch unless `retryRefused` is set, which card 12 sets only at a wake from the outbound read's wait (D6).
The caller calls `retire` only after a `Hub.Beams` read that stood.

`export_test.go` exposes the keeper as `ChannelKeeper` with `NewChannelKeeper`, `Ensure`, `Join`, `InviteOperators` and `Retire`.

Tests, tier0, `TestChannelKeeper` over `keeperHarness`: `newHarness`'s hub plus a second live hub's geometry, a `LiveHubs` seam answering both or a scripted error, and `fakeSlack`'s channels, a row seeding foreign and archived ones with `seedChannel`.
- a new orch gets `ly-neve-prime` created; an `onMethod` hook reading the map at `conversations.invite` sees the entry already written with an empty cursor; the entry records `U1`; `already_in_channel` counts as done;
- `Ensure` of an open entry recording `U0`, or no operator, invites `U1` and records it, and creates nothing;
- a failing `AuthTest` on `name_taken` returns its error with no join or adopt; a `name_taken` whose name `FindChannel` no longer finds returns an error and writes no entry;
- a second ensure of the same orch makes no `CreateChannel` or `FindChannel` call, and two concurrent ensures make one `CreateChannel`;
- `name_taken` on a live channel of `UBOT` where the bot is no member joins, then adopts with cursor and floor at its newest message;
- `name_taken` on an archived channel of that name makes no join or adopt and one notice naming renaming it in Slack; so does one created by another user;
- `name_taken` on a channel the second hub's map holds is refused with a notice naming the extra clone; one another orch of this hub holds is refused too;
- with the second hub's map holding `{`, and with `LiveHubs` failing, neither adopt nor `Join` reaches Slack and one notice names the path;
- an archived entry gets a new channel under its D4 name that replaces it; no method of the seam can unarchive;
- an entry recording `U0` is invited for `U1` by `InviteOperators`, a refused invite mails one notice naming `lym slack setup --operator <user-id>` across two calls, and the next call that stands records `U1`;
- `Retire` with no beams listed over an open `payments` entry renames it to `ly-neve-payments-2026-11-05` and archives it; a `name_taken` rename goes on to `-2`; the prime's entry is never renamed;
  a refused archive leaves the entry open under its dated name, a next `Retire` without `retryRefused` makes no Slack call for it, and one with it archives it with no second rename;
  a rename refused with `not_authed` leaves the entry open under its D4 name and is retried only with `retryRefused`.
