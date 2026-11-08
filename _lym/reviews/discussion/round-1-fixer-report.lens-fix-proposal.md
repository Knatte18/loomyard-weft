```yaml
lens: fix-proposal
outcome: reported
```

Lens: fix-proposal. For each finding fixed as proposed, I looked for a concrete case where the proposal itself is wrong.

Proposals challenged: F1, F2 (adopted part), F3, F4, F5, F6, F7, F8, F10, F11, F12, F13, F14, F15, F16, F17, F18, F19, F20, F21, F22.

1. **BLOCKING** — D2: "A `thread_broadcast` is forwarded only by the top-level read; a thread read skips it and moves the thread cursor past it." (F4)
   - A broadcast reply is a thread reply, so under D1 the holder names its hint `<channel id>_<thread ts>` and writes no channel hint.
   - The thread read skips it, and no top-level read is triggered.
   - The broadcast then waits for the next top-level message, resync or start, which can be hours away. That breaks the Goal's "within about a second" and the D7 bullet that a `thread_broadcast` is forwarded.
   - Evidence: D1, "A hint is a file named … `<channel id>_<thread ts>` for a thread reply". D2's list of top-level read triggers has no thread hint in it.
   - A sound change: the holder also writes the channel hint for a broadcast, or the thread read forwards a broadcast and the top-level read skips it.

2. **MEDIUM** — D3: "whose creator is this machine's bot user (D5)", on every unarchive and invite; D5 step 2's creator test. (F8)
   - D3 itself lets a machine move to a new app: "a machine moving to a new app logs in its bot token first". The new app has a new bot user.
   - From then on, every channel in the map fails the creator test. Every archived beam channel is refused its reopen.
   - Every post is refused with `not_in_channel`, because the new bot is in no old channel. Its invite is refused by the same test, so the post is dropped after the cap.
   - Every ensure that collides on the name with `name_taken` is refused too, including the prime's channel.
   - The only way forward offered is "renaming that channel in Slack", given only for step 2. So the machine silently loses all its channels.
   - Evidence: D3's bound covers every user-token call, D8's `not_in_channel` path goes "under D3's bound", and the D3 rationale sentence quoted above.

3. **MEDIUM** — D2: the relay watches "the inbox directory, which holds only hints"; "When `fswatch.Watch` fails at open, the relay checks … on disk instead". (F1)
   - `fswatch.Watch` returns the error of `inner.Add(dir)` (`internal/fswatch/fswatch.go:41-44`), and fsnotify fails `Add` on a directory that does not exist.
   - Nothing in the record says the relay creates `.lym/slack/inbox/` before it watches it.
   - On a fresh hub, every relay therefore lands on the on-disk fallback, which grows to sixty seconds while it finds nothing. The first message after an idle stretch then misses the Goal's second.
   - The record should say the relay creates the inbox directory before it opens the watch.

4. **MEDIUM** — D2: "A file-event wake never starts within a second of the previous wake's start." (F3)
   - The record does not say what happens to an event that arrives inside that second.
   - Take two operator messages 0.5 s apart. The second hint's event falls inside the floor.
   - If that event is dropped rather than held for the floor's end, the hint waits for the outbound read's wait, up to 60 s. That breaks the new D2 bullet that a hint wakes the relay "through its file event alone".
   - The record should say an event inside the floor is deferred to the floor's end.

5. **MEDIUM** — D1: "One envelope's map reads and hint write have ten seconds … the holder drops the connection and reconnects." (F7)
   - D2 makes every map write "a read-modify-write under the map's `internal/state` lock", and the record never says that lock is released before a Slack call.
   - If hub B's relay updates a cursor while it holds the lock across a `conversations.history` call that is slowed by a rate limit, the holder passes ten seconds.
   - The holder then reconnects, Slack redelivers the envelope, and the same stall can repeat. Every hub's push churns because of one hub's read.
   - The record should say no Slack call is made while a map lock is held.

6. **LOW** — D8: "A reopen, invite or ensure that itself fails counts as one refusal of the message." (F22)
   - A reopen that fails on a transport error or a rate limit would count toward the cap. That contradicts the existing rule that such failures count toward no cap (`internal/slackengine/outbound.go:182-186`).
   - The text also leaves open whether an `is_archived` refusal followed by a failed reopen counts as one refusal or two.

Further tool calls: none.
