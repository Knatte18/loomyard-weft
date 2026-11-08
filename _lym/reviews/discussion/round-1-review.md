---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md:40
    summary: >-
      D1 writes hints into an inbox directory under .lym/slack/, but D2 wakes on an fswatch event on the Slack
      directory, and fswatch.Watch does not recurse, so a hint wakes nothing and waits up to the outbound read's 60 s
    fix: >-
      In D1 and D2 name one directory: the directory the hints are written into is the directory the relay's fswatch
      watches (either the hints sit in .lym/slack/ itself under a fixed name prefix, or D2 watches the inbox directory).
      In D2 also state what the relay does when fswatch.Watch fails at open, such as falling back to a bounded poll
      of the inbox from one second doubling to sixty, and add an acceptance bullet that a hint written into the
      inbox wakes the relay through its file event alone.
  - id: F2
    severity: BLOCKING
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md:72
    summary: >-
      D2 reads a thread after its thread cursor, but a pruned cursor, an adopted channel and a new thread have none,
      so a read replays every earlier operator reply in the thread, or every reply of the last 24 hours, to the orch
    fix: >-
      In D2 state the start value of a thread cursor that is missing. A new thread's cursor starts at the thread's
      parent timestamp, and a pruned or adopted thread's at the channel's top-level cursor, so no reply at or before
      what the channel already handled is forwarded. D2's 24-hour scan reads a thread with no cursor only for
      replies after that start value. State that one goroutine drains the inbox, so a file-event wake and an
      outbound-wait wake never read the same channel at once. Add acceptance bullets for a reply after a pruned
      cursor and for a channel adopted on name_taken, each forwarding no earlier reply.
  - id: F3
    severity: BLOCKING
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md:83
    summary: >-
      D2 wakes on any fswatch event in the Slack directory, where the relay itself writes relay.json, relay.log,
      outbox.json and the map, so a failing archive can loop on its own log write with no floor, against
      PATTERN-daemon-wakeups
    fix: >-
      In D2 state that the watch takes only the hint and presence file names (fswatch.Watch's names filter), or
      watches a directory the relay writes nothing else into, and that a file-event wake comes never within a second
      of the previous wake's start. In the PATTERN-daemon-wakeups Constraint add the same floor. State that a D6
      archive Slack refused is retried on the outbound wait's backoff, never on a file event.
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md:198
    summary: >-
      D7 admits thread_broadcast, which Slack returns both in conversations.history and in the thread's replies, and
      D2 reads the two routes with separate cursors, so a broadcast is forwarded twice
    fix: >-
      In D7 or D2 state that a thread_broadcast is forwarded only from the top-level read and is skipped by the
      thread read, and that a thread read moves the thread cursor past it.
      Extend the D7 acceptance bullet to assert one forward of a broadcast across a start read.
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:41
    summary: >-
      D1 and D2 never say a relay checks a hint's channel or thread against its own map, so D1's bound that a planted
      hint can only make a relay read its own channel rests on an unstated check
    fix: >-
      In D2 state that a relay acts on a hint only when its name parses as a channel id, or a channel id and a
      thread timestamp, and the channel is in the relay's own map. Any other hint is removed unread with a Debug log
      line. Add an acceptance bullet that a hint naming a channel outside the map reads nothing from Slack.
  - id: F6
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:106
    summary: >-
      D1 routing and the D3 and D5 no-other-map test read every live hub's map, and the record never says what an
      unreadable map does, so the user-token guard could pass on a map it could not read
    fix: >-
      In D3 and D5 state that a live hub's map that cannot be read or decoded fails the guard closed: no unarchive,
      invite or adopt is made, and the hub's orch gets the one [slack relay] notice. In D1 state that an event while
      any live map is unreadable is acknowledged only when a readable map holds its channel, and otherwise left
      unacknowledged for Slack's retry.
  - id: F7
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:45
    summary: >-
      D1 bounds a failed hint write only by Slack's retries and states no limit on a holder that stalls with the lock
      held, and D2's gap list leaves out a hint lost once those retries run out
    fix: >-
      In D1 state a time limit on one envelope's map reads and hint write. When the limit passes, the holder drops
      the connection and reconnects, so the resync hint on the new connection covers what was lost.
      Add a lost hint after Slack's retries to D2's list of gaps that a reconnect's catch-up covers.
  - id: F8
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:151
    summary: >-
      D4 names are not unique across machines (FromHostname keeps the part before the dot, and a dash is legal inside
      segments), and D5 step 2 reopens an archived channel of that name with the user token, letting two machines
      relay one channel against the board
    fix: >-
      In D5 step 2 reopen an archived channel, or adopt a live one, only when conversations.list shows its creator is
      this machine's bot user, read from auth.test. Any other channel is refused with the one [slack relay] notice.
      Restate D3's user-token bound and D4's rationale to rest on that creator check, not on the name alone.
      Add an acceptance bullet: an archived channel of the D4 name that another bot created is not unarchived.
  - id: F9
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:98
    summary: >-
      D3 login confirms each token alone and never that the bot, app-level and user tokens belong to one app and one
      workspace, so another machine's xapp token makes this holder take half of that app's events and drop them
    fix: >-
      In D3 state that login writes a token only when it matches the tokens already stored. The team id from
      auth.test must be the same for the bot and user tokens, and the app id must be the same for the app-level and
      bot tokens. A mismatch is refused, naming the kind already stored and lym slack login as the way forward.
      Record in Constraints which Slack answer carries each id, with its docs.slack.dev page and read date.
  - id: F10
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md:83
    summary: >-
      D2 removes the inbound poll, which is the only loop that runs CheckBinary, so the relay never re-execs onto a
      replaced binary, and the record says nothing of the machine socket lock across an exec
    fix: >-
      In D2 name the loop that runs the binary check after the poll goes, such as after each outbound read. In D1
      state that a holder closes its connection and releases the machine socket lock before the exec and takes its
      turn waiting again after it. Add the binary check to the loops the PATTERN-daemon-wakeups Note lists.
  - id: F11
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md:101
    summary: >-
      On an upgrade, a live relay re-execs, fails the new three-token check and exits with the refusal only in
      relay.log, while presence stays slack and dobby's restart answers not-configured silently
    fix: >-
      In D3 state that a relay that finds a token missing at its start, before it exits, mails the hub's orch one
      [slack relay] notice naming the missing kind and lym slack login, and posts the same line to the prime's
      channel with the bot token when it has one. Add an acceptance bullet for a relay started with only a bot token.
  - id: F12
    severity: MEDIUM
    class: consistency
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md:101
    summary: >-
      D3 says the relay start and RequireConfigured apply one check that refuses, but ensure does its own reads and
      answers OutcomeNotConfigured with no error, which orch bring-up and the anchor change rely on
    fix: >-
      In D3 state which paths refuse and which stay quiet. RequireConfigured and the CLI verbs start and away refuse,
      naming the missing kind, while EnsureRelay and EnsureRelayWhileAway answer OutcomeNotConfigured for a machine
      missing any token, so orch bring-up and the anchor change neither warn nor fail. Reword the third Notes
      bullet to match.
  - id: F13
    severity: MEDIUM
    class: consistency
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:156
    summary: >-
      D5 names lym fabric shortname as the way forward when another hub's map holds the channel, but that verb
      refuses to change a recorded shortname
    fix: >-
      In D5 step 2 replace the way forward with the one hubregistry's duplicate-shortname refusal names: remove the
      extra clone of the hub, since a recorded shortname never changes.
  - id: F14
    severity: LOW
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md:218
    summary: >-
      D8 posts the relay's stale line to that message's channel, but the outbox keeps stale mail as one hub-wide
      count posted as one line
    fix: >-
      In D8 state the stale line's channel. Either the outbox counts stale mail per channel and posts one line to each,
      or the one aggregate line goes to the prime's channel. Match the D8 acceptance bullet to that choice.
  - id: F15
    severity: LOW
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md:219
    summary: >-
      D8 recovers from is_archived and channel_not_found only, so a bot removed from a channel has every post dropped
      after the refusal cap and the dropped line fails in the same channel
    fix: >-
      In D8 add that a post refused with not_in_channel invites the bot back with the user token under D3's bound and
      posts again, counting once toward the refusal cap. Add not_in_channel to the D8 acceptance bullet, and widen
      D3's list of user-token uses to name this invite.
  - id: F16
    severity: LOW
    class: consistency
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md:284
    summary: >-
      D10 removes setup's channel flag, but the PATTERN-refusal-way-forward Constraint covers only new refusals, so
      existing ways forward that name setup --channel stay pointing at a flag that is gone
    fix: >-
      In the PATTERN-refusal-way-forward Constraint add that every existing refusal or notice whose way forward names
      lym slack setup --channel is reworded to the operator-only form D10 leaves.
  - id: F17
    severity: LOW
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md:280
    summary: >-
      The PATTERN-lym-files Constraint says paths come from hubgeom.SlackGeometry, but the holder in slackengine must
      reach every live hub's map and inbox and cannot import hubgeom, and no Decision names how it is told them
    fix: >-
      In D1 state that the relay is told, as a seam like its SlackClient, how to list the machine's live hubs with
      each hub's Slack geometry, and that slackcli fills it from hubregistry.Live and hubgeom.SlackGeometry.
      Reword the PATTERN-lym-files Constraint to say so.
  - id: F18
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/discussion/decision-record.md:96
    summary: >-
      D3 defers the login read to whatever way slack-login-prompt lands, but that task landed on main as fd159ff2 with
      a prompt and an empty-input refusal that name only the bot token, and the support log still calls it running
    fix: >-
      In D3 cite fd159ff2 on main. State that login keeps its terminal read and piped read, and rewords its tokenPrompt
      and its empty-token refusals to name the three kinds. Drop the deferral wording.
      In support-log.md Interview Turn 1, say slack-login-prompt has landed as fd159ff2.
  - id: F19
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/discussion/decision-record.md:312
    summary: >-
      Two D1 and D2 rules have no acceptance bullet that would fail if they broke, the connection-refused notice
      once per episode and the 24-hour thread cursor prune
    fix: >-
      Under Acceptance criteria add a D1 bullet that a connection Slack refuses or disables mails the holder hub's orch
      one [slack relay] notice per episode naming lym slack login. Add a D2 bullet that a thread cursor with no reply
      for 24 hours is pruned from the map, worded to fit F2's start value.
  - id: F20
    severity: NIT
    class: consistency
    origin: handler
    location: _lym/discussion/decision-record.md:307
    summary: >-
      The one Open risks line is no risk, since D5 step 2 handles both answers and its rationale already says so,
      and the question stays in the support log's ledger
    fix: >-
      Under Open risks replace the archived-name line with none. Keep D5's rationale sentence and the ledger entry
      as they stand.
  - id: F21
    severity: NIT
    class: consistency
    origin: handler
    location: _lym/discussion/decision-record.md:287
    summary: >-
      Two Slack facts in Constraints carry nothing the plan writer acts on, the ten-connection cap, which only argues
      against a rejected alternative, and the channel name grammar, which D4 already states
    fix: >-
      Delete the ten-connection and conversations.create name bullets from the Constraints Slack-facts list.
      Keep the ten-connection cap in support-log.md's Rejected alternatives entry for a connection per relay, which
      already names it.
  - id: F22
    severity: NIT
    class: consistency
    origin: handler
    location: _lym/discussion/decision-record.md:221
    summary: >-
      D8 counts each reopen or ensure re-post toward the refusal cap without saying which cap, though it is the
      existing per-message cap of five
    fix: >-
      In D8 replace the refusal cap with the existing per-message cap of five Slack refusals in
      internal/slackengine/outbound.go, and state that a reopen or ensure that itself fails counts as one refusal
      of the message.
---

### [BLOCKING] Hints land in a directory the relay's file watch does not see

**Location:** `_lym/discussion/decision-record.md:40`, `:83`

**Issue:** D1 writes each hint into "that hub's inbox directory under `.lym/slack/`".
D2 wakes the relay on "an `internal/fswatch` event on its Slack directory".
`fswatch.Watch` adds one directory to fsnotify and does not recurse (`internal/fswatch/fswatch.go:36-44`), so a hint in a subdirectory raises no event.
The relay then wakes only when the outbound read ends, after up to 60 s (`internal/slackengine/outbound.go:25`).
That misses the Goal's "within about a second" and the D2 acceptance bullet.
The record also gives no wake bound for when `fswatch.Watch` fails at open.
Every lens raised this one; the existing-paths, walkthrough and interactions lenses agree on it, and so does bound-breaker's finding D.

### [BLOCKING] A missing thread cursor replays old operator replies

**Location:** `_lym/discussion/decision-record.md:72`, `:79-80`, `:154`

**Issue:** D2 prunes a thread cursor after 24 hours with no reply, and reads a thread hint "after its thread cursor".
After a prune, a new reply makes the relay read the whole thread, and every earlier operator reply in it reaches the orch again.
D5 adopts a live channel with only its top-level cursor set, and D2's start scan takes every thread that "has no cursor", so the first start after an adoption forwards every operator thread reply of the last day.
A new thread's first read has no cursor either.
Replayed operator text carries the operator's authority to the orch, so this is more than a duplicate.
The record also never says the inbox drains are serialised, which "cursors only move forward" does not cover.
The interactions, walkthrough and bound-breaker lenses all raised this.

### [BLOCKING] The file watch wakes on the relay's own writes, with no floor

**Location:** `_lym/discussion/decision-record.md:83`, `:277`

**Issue:** Every wake drains the inbox and runs D6's archive check, and the watched Slack directory holds the relay's own records.
`recordingSlack.stamp` rewrites `relay.json` after every successful Slack call (`internal/slackengine/relay.go:125-133`).
`relayLog.fail` appends to `relay.log` on every failure (`internal/slackengine/relay.go:89-93`).
The walkthrough lens's case: a D6 archive Slack refuses appends to the log, which wakes the relay, which retries the archive.
That is a Slack-call loop with no one-second floor and no backoff, so the design breaks `PATTERN-daemon-wakeups`, which the record's Constraint claims to keep.
The existing-paths and interactions lenses raised the same wake source.

### [MEDIUM] A thread broadcast is forwarded twice

**Location:** `_lym/discussion/decision-record.md:198`, `:73-80`

**Issue:** D7 admits `thread_broadcast`, which sits both in the channel's history and in its thread's replies.
D2 reads those two routes with separate cursors, and neither route skips a message the other forwards.
The record breaks its own "forwarded once" rule and the D7 acceptance bullet.

### [MEDIUM] A hint's channel is never checked against the relay's own map

**Location:** `_lym/discussion/decision-record.md:41-43`, `:58`

**Issue:** D1's bound says a planted hint "can at most make a relay read its own channel".
No Decision says the relay checks the hint's name against its map, or drops a name that does not parse.
A channel outside the map has no cursor, which D5 treats as the channel's beginning.
The bot is a member of every hub's channels on the machine.
So a planted hint naming another hub's channel would make this relay read that channel from its start, and D7 does not say where the text goes.
This settles focus 1: D1's bound does not cover the hint name as written.

### [MEDIUM] An unreadable map of another live hub has no stated outcome

**Location:** `_lym/discussion/decision-record.md:39`, `:106`, `:152`

**Issue:** D1's routing and the D3 and D5 test "no other orch's entry in any live hub's map" both read every live hub's map.
Read as empty, an unreadable map lets the user token unarchive or adopt a channel that hub owns.
Read as an error, D1 does not say whether the envelope is left unacknowledged or dropped.

### [MEDIUM] A stalled holder and a hint lost after Slack's retries are unbounded

**Location:** `_lym/discussion/decision-record.md:44-45`, `:88-89`

**Issue:** The holder takes each live map's `internal/state` lock for each envelope.
A long-held lock or a stuck hint write stalls every hub's push while the holder keeps the machine lock, and the ping does not cover it.
"Slack retries it" holds only until Slack's retries run out.
After that the hint is lost until the next message or the next reconnect, which the Constraints put every few hours.
D2's list of gaps omits this case.
This settles focus 2: the acknowledgement rule needs its own bound, because Slack's docs name no deadline.

### [MEDIUM] Channel names collide across machines, and the user token reopens the other machine's channel

**Location:** `_lym/discussion/decision-record.md:123-125`, `:151-153`, `:106`

**Issue:** The no-other-map guard sees only this machine, since `hubregistry.Live` reads a per-machine record.
D4 names do not stay unique across machines.
`machinename.FromHostname` keeps only the part before the first dot, and many machines keep a default host name.
`-` separates segments and is also legal inside a machine name and a slug, so machine `neve` with slug `a-b` and machine `neve-a` with slug `b` both give `ly-neve-a-b`.
On `name_taken`, machine 2 reopens machine 1's archived beam channel with the operator's user token and invites its own bot.
Two machines then relay one channel, against the board's "two hubs or two machines never relay one channel".
D3's user-token bound is broken on that path.
This settles focus 4's adopt and reopen paths.

### [MEDIUM] Login never checks that the three tokens belong to one app

**Location:** `_lym/discussion/decision-record.md:98`, `:44`

**Issue:** Each token is confirmed alone.
An app-level token pasted from another machine's app makes this holder one of that app's connections.
Slack then hands it about half of that app's events, which no map here holds, so they are acknowledged and dropped.
A login is operator input at a trust boundary, and the cross-check costs one field per answer.

### [MEDIUM] Removing the poll strands the binary re-exec

**Location:** `_lym/discussion/decision-record.md:83`, `:370`

**Issue:** `runInbound` runs `deps.CheckBinary` after each poll, and that is the relay's only path onto a replaced binary (`internal/slackengine/relay.go:330-339`).
D2 removes the poll and names no loop that takes over the check.
The record is also silent on how the machine socket lock and the open connection behave across the exec.
The walkthrough lens raised the same gap.

### [MEDIUM] After an upgrade the relay exits and nobody is told

**Location:** `_lym/discussion/decision-record.md:101-103`

**Issue:** On a machine holding only today's bot token, a live relay re-execs onto the new binary and fails the three-token check in `RunRelay` (`internal/slackengine/relay.go:186`).
The refusal lands only in `relay.log`.
Dobby's `EnsureRelayWhileAway` then answers `OutcomeNotConfigured` every tick without a word (`internal/slackengine/start.go:180-181`).
Presence stays `slack`, and the operator who is away hears nothing.
The bound-breaker lens raised the same path.

### [MEDIUM] D3's "one check that refuses" misstates the start path

**Location:** `_lym/discussion/decision-record.md:101-103`, `:371`

**Issue:** `ensure` does its own token and setup reads, and answers `OutcomeNotConfigured` without an error (`internal/slackengine/start.go:172-182`).
Orch bring-up turns an `EnsureRelay` error into a warning (`internal/orchcli/start.go:549-551`), and the anchor change returns it as an error (`internal/fabriccli/anchordeps.go:282-283`).
If "refuses" means an error, every machine without Slack would warn or fail on those paths.
If it means the quiet outcome, the named missing kind reaches no one.
The bound-breaker lens raised the same misstatement.

### [MEDIUM] The way forward for a channel another hub holds cannot work

**Location:** `_lym/discussion/decision-record.md:156`

**Issue:** `lym fabric shortname` refuses to change a recorded shortname, "since that would orphan every name already in use" (`internal/fabriccli/shortname.go:42`).
`hubregistry`'s own refusal for duplicate shortnames names removing the extra clone (`internal/hubregistry/hubregistry.go:93`).

### [LOW] The stale-mail line has no single channel

**Location:** `_lym/discussion/decision-record.md:218`

**Issue:** `outboxRecord.Stale` is one hub-wide count, and `reportStale` posts one line (`internal/slackengine/outbound.go:43-47`, `:136-147`).
A line sent to "that message's channel" needs a count per channel, and the record states neither that change nor another choice.

### [LOW] A bot removed from its channel is never invited back

**Location:** `_lym/discussion/decision-record.md:219-220`

**Issue:** A post refused with `not_in_channel` is neither `is_archived` nor `channel_not_found`.
Each message in that channel is dropped after the cap, and the dropped line fails in the same channel (`internal/slackengine/outbound.go:196-199`).

### [LOW] Existing ways forward keep naming the removed `--channel` flag

**Location:** `_lym/discussion/decision-record.md:241`, `:284`

**Issue:** Today's setup-missing refusals offer `lym slack setup --channel <id> --operator <user-id>` (`internal/slackengine/inbound.go:228`, `internal/slackengine/outbound.go:81`, `internal/slackengine/credentials.go:87`).
D10 removes that flag, and the Constraint covers only new refusals.
This is a rule for one class of message, not a call-site list.

### [LOW] No Decision says how the holder learns other hubs' paths

**Location:** `_lym/discussion/decision-record.md:39-40`, `:280`

**Issue:** `internal/slackengine` "is told its `Geometry` and derives no path of its own" (`internal/slackengine/doc.go:17-18`), and `hubgeom` imports `slackengine`.
The holder needs every live hub's map and inbox paths, and no Decision says how it is told them.

### [LOW] D3 defers to a sibling task that has already landed

**Location:** `_lym/discussion/decision-record.md:96`; `_lym/discussion/support-log.md:8`

**Issue:** `slack-login-prompt` landed on `main` as `fd159ff2`, which is not yet on this branch.
Its `tokenPrompt` reads "Paste the Slack app's Bot User OAuth Token (xoxb-…) and press Enter: ", and its empty-input refusals name only `xoxb-…`.
D3 can cite that commit and state the rewording, so the question does not go to the parent.
The support log still records the task as running Darn-Write.
This settles focus 3: D3 neither contradicts nor duplicates that task, and the writer settles the overlap by citing the commit.

### [LOW] Two rules have no failing acceptance bullet

**Location:** `_lym/discussion/decision-record.md:312-324`

**Issue:** No acceptance bullet covers D1's once-per-episode notice for a connection Slack refuses, or D2's 24-hour thread cursor prune.
The other bullets are behaviour a test can check.
The `Review:` marks sit only on the two prose-only D10 bullets.
D11's seam and D2's cursor-forward rule each have a bullet.

### [NIT] Open risks holds a question the design already answers

**Location:** `_lym/discussion/decision-record.md:307-308`

**Issue:** This settles focus 5: the risk is no deferred research question.
D5 step 2 works whichever way Slack treats an archived channel's name: a map-held archived entry is reopened by id, and a create that collides is handled in both cases.
The line restates D5's rationale, and the question already sits in the ledger.

### [NIT] Two Constraints facts carry nothing the plan writer acts on

**Location:** `_lym/discussion/decision-record.md:287`, `:290`

**Issue:** With one connection per machine, the ten-connection cap never binds; it argues only against a rejected alternative.
The name grammar repeats D4.

### [NIT] D8's refusal cap is not named

**Location:** `_lym/discussion/decision-record.md:221`

**Issue:** This settles focus 6: the re-posts are bounded by the existing per-message `refusalCap` of five (`internal/slackengine/outbound.go:31-32`).
The record does not name that cap, nor how a failed reopen counts toward it.

## Focus 7 verdict

D7's operator-only and bot-skip guards hold in every channel.
The skip of other subtypes, bots and non-operators stays, and the woken relay applies its own hub's operator.
Dropping `@<slug>` replaces a beam-list check with the map lookup and weakens no guard.

## Rejected

- existing-paths F6, `contracts/orchrole/role-slack.md` still describing the `@<slug>` hint: the rubric's Do-not-flag list leaves cross-reference enumeration to Plan-Write's own lookups, and D7 and D8 already state the behaviour that file must follow.
- bound-breaker claim 18, a prime bring-up starting a relay between the stop and the archive: the lens itself judged it repaired by D8's `is_archived` reopen.
- interactions F3 and F6, bound-breaker D and E, walkthrough 1, 2, 3, 4 and 7, existing-paths F3 and F4, and bound-breaker F, I and J: merged into F1, F2, F3, F4, F10, F11 and F12 above, not rejected on their merits.
