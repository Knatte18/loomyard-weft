# Card 7 — slackengine-inbound

**Create:**
- `internal/slackengine#Clock`
- `internal/slackengine#Mailbox`
- `internal/slackengine#SlackClient`
- `internal/slackengine#Hub`
- `internal/slackengine#SlackProgram`
- `internal/slackengine#SetPresence`
- `internal/slackengine#inbound`
- `internal/slackengine#inbound.poll`
- `internal/slackengine#inboundText`
- `internal/slackengine#route`
- `internal/slackengine/inbound_test.go`
- `internal/slackengine_test#TestInboundPoll`
- `internal/slackengine_test#TestSetPresence`

**Extends:**
- `internal/slackengine/doc.go`

**Uses:**
- `internal/agentname#TaskSlug`
- `internal/creelengine#Message`

**Intent:** Add one inbound poll of the hub's channel: read the operator's top-level messages after the cursor and hand each, once, to the right orch as `[slack]` creel mail from `<shortname>:SLACK` (D1, D4), plus the presence commands and notices (D6).

The seams the relay is told, all declared here:
- `Clock` (`Now`, `After`), so no test waits out the 5-minute window.
- `Mailbox`: `Post(receiver, body string) (int64, error)`, `ReadAll(handle func([]creelengine.Message) error) error` and `ReadWait(timeout time.Duration, handle func([]creelengine.Message) error) error`, the calls of card 3's program client signing as `<shortname>:SLACK`.
- `SlackClient`: `History`, `Latest` and `PostMessage` as `*API` declares them.
- `Hub{Beams func() ([]string, error); Paused func(receiver string) (bool, error)}`: the hub's beam slugs and whether a pause covers a receiving orch, which `lym slack` fills (card 10).
- `SlackProgram` is the program name, the one place `SLACK` is spelled (`PATTERN-code-conventions`); receivers and the sender are formed through `internal/agentname` (`PATTERN-agent-addressing`).

`inbound.poll` does, in order:
1. Reads the setup file at every poll, so a `lym slack setup` reaches a live relay at its next poll (D2).
2. Reads the cursor `{channel, ts}` at `CursorPath`. With no cursor, or one for another channel than the configured one, it first writes the cursor as the configured channel and `Latest`'s `ts`, or the clock's now as a Slack `ts` on an empty channel, and only then reads, so no history is forwarded, a channel relayed before included (D4).
3. Reads `History(channel, cursor.ts)` and handles each message oldest first, writing the cursor past each once its handling ends.
   - A message with a `ThreadTS` other than its own `TS`, a `Subtype`, a `BotID`, or a `User` other than the configured operator is skipped and logged at Debug.
   - The text goes through `inboundText`: `<url|label>` and `<url>` whose value starts with `http://`, `https://` or `mailto:` become the URL, then `&amp;`, `&lt;` and `&gt;` are unescaped; `<#C…|name>`, `<!here>` and `<@U…>` stay as Slack delivers them.
   - A text that is exactly `away`, `back` or `status`, case and surrounding space ignored, goes to no orch: `away` and `back` call `SetPresence` with `slack` or `terminal`, `status` reads the presence, and each is answered in Slack with the resulting state (D6).
     `SetPresence` returning `changed` true with an error, the write stood and only the beams read or a notice failed, is logged at Warn once per episode, still answered with the resulting state and counted as handled, so the cursor moves: a retry would find no change and send no notice.
     `SetPresence` returning `changed` false with an error, the presence not written, makes the poll return the error with the cursor left before the message, and the relay retries with its backoff, which duplicates nothing since no notice or answer was sent; `status` does the same when `ReadPresence` errors.
     Once the first failure of that message is 5 minutes old, as for a failed post, the relay answers in Slack quoting the error, which names the presence file, and moves the cursor past it, so a stored value no retry can read holds later messages no longer than that.
   - `route` sends a text opening `@<slug> ` to `<shortname>:<slug>:orch` without the prefix when `slug` is one of `Hub.Beams`, and any other text to `<shortname>:orch`; a slug no beam has is answered in Slack listing the hub's beam slugs, and nothing is posted to creel.
     The poll calls `Hub.Beams` for a text opening `@<slug> ` and, as `SetPresence`'s `beams` argument, for an `away` or `back` that changes the presence; a `Hub.Beams` error of the routing call comes before any post: the poll returns it with the cursor left before the message, and the relay retries with its backoff, which forwards nothing twice.
   - The body is `[slack] <text>`; a body over 4096 bytes, creel's notice cap, is answered asking for a shorter message and not posted.
   - The post goes through `Mailbox.Post`. A post that fails or is refused leaves the cursor before that message, logs at Warn once per episode and returns, so later messages, presence commands included, wait; the relay retries with its backoff. Once the first failure of that message is 5 minutes old, the relay answers in Slack that the message was not delivered, quoting creel's error and asking for it to be sent again, and moves the cursor past it.
   - After a successful post, a receiver `Hub.Paused` reports covered is answered in Slack that the hub is paused and the message waits for the resume.
     A `Hub.Paused` error is logged at Warn once per episode, sends no paused answer, and still counts the message as handled, so the cursor moves past it: the post already reached creel, and a retry would forward it again.
   - Every answer goes through `PostMessage` with `&`, `<` and `>` escaped; an answer Slack refuses is logged at Warn and the message still counts as handled.
4. Returns whether anything was forwarded, answered or the presence changed, which resets the relay's wait to its floor (card 9).
   A presence change counts whoever made it: `inbound` keeps the presence the previous poll read, and a poll that reads another value, as after `lym slack away` or `back` in a terminal, counts it as activity.

An `*APIError` from `Latest` or `History` (such as `invalid_auth`, `token_revoked`, `channel_not_found` or `not_in_channel`) is a state the operator must fix: the poll returns it, and on the first such error of an episode it posts, from the mailbox to `<shortname>:orch`, one `[slack relay]` notice naming Slack's error code, with the way forward "run `lym slack login` for a refused token or `lym slack setup` for a channel the bot cannot read, from the operator's terminal"; the relay keeps polling with its backoff and the episode ends at the first successful read.
The notice opens `[slack relay]`, not `[slack]`, so it carries no operator authority.

`SetPresence` writes the presence and, only when it changed, reads `beams` and posts `[slack presence] presence is now <state>` from the mailbox to `<shortname>:orch` and to `<shortname>:<slug>:orch` for every beam `beams` lists; the notice asks for no reply (D6).
It returns `changed` false with an error when the presence was not written, and `changed` true with an error when the write stood and the beams read or a notice failed, every failed notice joined into that error.

`internal/slackengine/inbound_test.go` is tier0, with an in-memory `SlackClient`, `Mailbox`, `Hub` and `Clock` declared in the file; the `Mailbox` hands out scripted `ReadWait` answers and records each acknowledgement from the start, so cards 8 and 9 use it unchanged:
`TestInboundPoll` rows — a top-level operator message reaches `<shortname>:orch` as `[slack] <text>` from the mailbox; a forwarded message whose text holds `<https://x.io/a|label>`, `<https://x.io/a>`, `<#C1|general>`, `<!here>`, `<@U1>` and `&amp; &lt;c&gt;` reaches the mailbox with the URL for each of the first two, the next three unchanged and `& <c>`; another user's, a bot's (`BotID`), a thread reply and a subtype message reach no receiver and still move the cursor; `@payments do x` reaches `<shortname>:payments:orch` as `[slack] do x`; `@nope x` gets one Slack answer listing the beams and no post; a `@payments x` whose `Hub.Beams` errors posts nothing to the mailbox, sends no Slack answer, returns the error and leaves the cursor before the message, and the poll after `Hub.Beams` recovers forwards it once as `[slack] x` to `<shortname>:payments:orch`; a first poll with no cursor forwards none of the existing history, writes the cursor at the newest `ts`, and a message written after it is forwarded once on the next poll; an empty channel's first poll writes the clock's now; a poll after the setup's channel changed, to a new channel or back to one relayed before, forwards none of that channel's history; a second poll after a forwarded message forwards nothing again; a refused post keeps the cursor before it and holds a later `away`, is forwarded on the poll after the mailbox accepts within 5 minutes, and with the clock moved past 5 minutes is answered once as not delivered and passed; a 4097-byte body gets one answer and no post; a paused receiver gets the post and one paused answer; a `Hub.Paused` that errors after an accepted post forwards the message once, logs the error, sends no answer and moves the cursor, and a second poll forwards nothing; `away`, `back` and `status` set or read the presence and are answered with the state; an `away` whose `Hub.Beams` errors after the presence write leaves presence `slack`, gets one answer and moves the cursor, and a second poll forwards nothing; an `away` whose presence write fails, with a directory at the presence path, gets no answer, returns the error and leaves the cursor, and is handled once on the poll after the write works; a `status` over a presence file holding an unknown value gets no answer, returns the error and leaves the cursor, and with the clock moved past 5 minutes is answered once quoting the error and passed; an answer quoting `a & <b>` reaches the fake Slack as `a &amp; &lt;b&gt;`; an answer Slack refuses still moves the cursor past its message; a presence file changed between two polls by another writer makes the second poll report activity, and an unchanged one does not; a `History` `*APIError` `invalid_auth` posts one `[slack relay]` notice to `<shortname>:orch` naming `lym slack login`, a second failing poll posts none, and after a successful poll the next failure posts one again.
`TestSetPresence` rows — a change mails the prime orch and each beam's orch one `[slack presence]` notice; a write of the same state mails nothing; a failed notice returns an error while the presence is written; a `beams` that errors after the write returns `changed` true and the error, with the presence written.

**ImpactSummary:** New unexported poll and exported seams inside the new package; nothing outside it calls them until cards 9 and 10.
