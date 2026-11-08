# Fixer report — discussion round 1

Every finding's premise held against the code and Slack's docs.
All 22 are fixed in `_lym/discussion/decision-record.md`, and F18 also in `_lym/discussion/support-log.md`.

## Changes

- F1, as proposed: hints go into `.lym/slack/inbox/`, which D2 watches.
  D2 states the on-disk fallback when `fswatch.Watch` fails at open, and a D2 bullet says a hint wakes the relay through its file event alone.
- F2, fixed differently for the pruned case.
  A new thread's start value is its parent's timestamp, and an adopted channel's is its newest message, as proposed.
  For a pruned thread the proposal's start value, the channel's top-level cursor, was not used.
  That cursor can pass a reply not yet read: a reply at t1, a top-level message at t2 read first, then the thread read skips the reply.
  Instead each entry keeps a thread floor, which a prune raises to 24 hours before the prune and an adoption sets to the newest message.
  A thread with no cursor starts at the later of its parent's timestamp and the floor.
  D2 states the one draining goroutine and the bound, and three D2 bullets cover the prune, a reply after it, and adoption.
- F3, as proposed: the watch takes the inbox directory and only the presence file in the Slack directory, under a one-second floor.
  The `PATTERN-daemon-wakeups` Constraint carries the floor, and D6 retries a refused archive only at the outbound wait.
- F4, as proposed: D2 forwards a `thread_broadcast` from the top-level read only, and the D7 bullet asserts one forward.
- F5, as proposed: D2 acts only on a hint that parses and names a channel in the relay's own map, with a D2 bullet.
- F6, as proposed: D3 and D5 fail the guard closed on an unreadable map, with a notice.
  D1 leaves an event no readable map holds unacknowledged while a map is unreadable.
- F7, fixed differently.
  D1 sets a ten-second limit on one envelope's map reads and hint write, as proposed.
  The proposal's reaction to a passed limit, dropping the connection and reconnecting, was not used.
  One hub's persistently unwritable inbox would then fail every reconnect's resync write, so one hub would stall push for every hub.
  Instead the holder acknowledges the envelope, marks the hub as owed a resync hint, writes it at each ping until it stands, and notifies that hub's orch.
  D2's gap list names the lost hint and its cover.
- F8, as proposed for the `name_taken` path: D5 step 2 reopens or adopts only a channel whose `creator` is this machine's bot user, read from `auth.test`.
  D3's bound and D4's rationale rest on that test, and a D5 bullet covers another bot's archived channel.
- F9, as proposed, with the sources pinned to what Slack's docs give.
  `auth.test` names no app id, and `apps.connections.open` answers only `ok` and `url`.
  So the app-level token's app id comes from the Socket Mode `hello` message's `connection_info.app_id`, and the bot's from `bots.info`.
  The proposal refuses every mismatch; a bot token that mismatches is instead taken and the tokens it does not match are removed.
  Refusing it would leave a machine moving to a new app no way to log in any of the new app's tokens.
  The Constraints list the four Slack answers with the read date.
- F10, as proposed: D2 runs the binary check after each outbound read, and D1 says how a holder hands its connection and lock across the exec.
  The `PATTERN-daemon-wakeups` Note lists the check.
- F11, as proposed: D3's relay start mails the hub's orch a notice naming the missing kind, with a D3 bullet.
  The post to the prime's channel is made only when the map holds it.
- F12, as proposed: D3 names `RequireConfigured`, `lym slack start` and `lym slack away` as refusing, and `EnsureRelay` and `EnsureRelayWhileAway` as quiet.
  The third Notes bullet matches.
- F13, as proposed: D5 names `hubregistry`'s way forward, removing the extra clone.
- F14, as proposed: D8 posts the one stale line to the prime's channel, and the D8 bullet matches.
- F15, as proposed: D8 invites the bot back on `not_in_channel`, D3's user-token uses name that invite, and the D3 and D8 bullets match.
- F16, as proposed: the `PATTERN-refusal-way-forward` Constraint rewords every existing way forward that names `lym slack setup --channel`.
- F17, as proposed: D1 names the live-hubs seam `internal/slackcli` fills, and the `PATTERN-lym-files` Constraint says so.
- F18, as proposed: D3 cites `fd159ff2` and states the rewording; the support log's Turn 1 says the task has landed.
- F19, as proposed: a D1 bullet covers the connection-refused notice, and a D2 bullet covers the prune.
- F20, as proposed: Open risks reads "None."
- F21, as proposed: the ten-connection and name-grammar facts are gone from Constraints; the support log keeps the cap.
- F22, as proposed in naming `refusalCap`.
  The proposal's rule that a failed reopen counts as one refusal was narrowed to a Slack refusal, which adds no second count.
  A transport failure, server error or rate limit counts toward no cap, as `refused` in `internal/slackengine/outbound.go:182-186` keeps for a post.

## Self-check

- fix-consistency, MEDIUM, merged with fix-claims MEDIUM: the claim that the relay's own writes never wake it. Fixed as proposed: D2 says its records and log wake nothing, and its hint removals and presence writes wake it under the floor; the bullet matches.
- fix-consistency, LOW: the creator test had no source on the D6 reopen and D8 `not_in_channel` paths. Fixed with own change: a channel in this hub's map passes the guard without a creator test, and only D5's `name_taken` path reads the creator. That also settles fix-proposal finding 2.
- fix-consistency, LOW: the D3 bullet's "relay start before any spawn". Fixed as proposed: it names `lym slack start`, `lym slack away` and `RequireConfigured`.
- fix-consistency, outside its lens: login's Socket Mode connection can take an event. Fixed: D3 says login closes the connection after `hello`, and an event handed to it goes unacknowledged for Slack's retry.
- fix-claims, LOW, merged with fix-way-forward finding 1, MEDIUM: the prime's-channel post on the upgrade path has no map. Fixed as proposed: the Decision and the D3 bullet make the post conditional on the map holding the prime's channel.
- fix-claims, LOW: the `hello` app id is shown only in an example. Fixed as proposed: the Constraints line says so.
- fix-proposal, BLOCKING: a `thread_broadcast` hinted as a thread reply never triggers its top-level read. Fixed as proposed: D1 writes the channel hint for a broadcast, and the D7 bullet asserts it.
- fix-proposal, MEDIUM: an app move fails the creator test on every map channel. Fixed with own change, the same as the consistency LOW above.
- fix-proposal, MEDIUM: the inbox directory may not exist when the watch opens. Fixed as proposed: D2 creates it first.
- fix-proposal, MEDIUM: an event inside the one-second floor had no stated fate. Fixed as proposed: it wakes the relay at the floor's end, and a D2 bullet says so.
- fix-proposal, MEDIUM: a map lock held across a Slack call can stall the holder. Fixed as proposed: D2 makes no Slack call while a map lock is held.
- fix-proposal, LOW: D8 counted a reopen's transport failure toward the cap. Fixed as proposed, as F22 above states.
- fix-way-forward, MEDIUM: the unreadable-map notice named no way forward. Fixed as proposed: D3 names moving the file aside, after which that hub's relay ensures its channels anew.
- fix-way-forward, MEDIUM: one unwritable inbox made every reconnect fail. Fixed as proposed: the owed-resync change under F7 above.
- fix-way-forward, LOW: a refused prime's channel was never ensured again. Fixed with own change: in place of the proposed `lym slack stop` and `start`, D5 ensures the prime's channel again at each outbound wake until it stands. `lym slack stop` archives every beam channel, too much for that case.

## Disputed

None.

## Deferred

None.
