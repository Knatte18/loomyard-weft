```yaml
lens: code-correctness
outcome: reported
```

Task: run the code-correctness lens over the diff, tracing each changed function or branch with one concrete input and a failure of each call it makes.

**Per-function list (case run → verdict)**

- `fileTokens`, `readTokens`: two `xoxb-`, three tokens, an unknown prefix, empty piped input, and a terminal read error on the second prompt → all refuse and write nothing. Correct.
- `login`: a mismatching app id → refused before any write. An unchanged re-login → ends no relay. A first login with nothing stored → ends none. Correct apart from findings 1 and 2.
- `endLiveRelays`: `liveHubs` failing → reports the tokens as stored and gives a way forward. Correct. A `stop` failing partway → see finding 1.
- `RequireConfigured`, `Credentials.Missing`: bot only, app-level only, neither, and setup missing → each names the kind and the verb. Correct.
- `socketHolder.route`, `readLive`, `handleEnvelope`, `oweStalled`:
  - Event held by no map while one map is unreadable → left unacknowledged. Correct.
  - `hubregistry.Live` failing → every event left unacknowledged. Correct.
  - Hint write failing → acknowledged and owed a resync. Correct.
  - Route stalled past 10 seconds → acknowledged and owed a resync. Correct.
  - A hub newly listed, or any hub after a failed read → owed a resync. Correct.
- `serve`, `ping`, `run`:
  - Pong missing → returns `errPongTimeout`; the ping goroutine ends with `connCtx`. Correct.
  - `disconnect` with `link_disabled` → notice mailed, then reconnect. Correct.
  - Wait doubles to `waitCeiling` and resets only after a connection of at least one minute. Correct.
  - Stop during the lock wait → the late-acquired lock is released. Correct.
- `writeHint`: hint created in `geom.Dir` and renamed into the inbox; a failure removes the temp file. Correct.
- `takeHints`, `readsForHint`, `drain`:
  - Dot-prefixed file → left alone. Correct.
  - Unparsable name, or a channel not in the map or archived → removed unread. Correct.
  - Hint removed before the read. Correct.
  - Map unreadable → hints stay for the next wake. Correct.
  - Rate limit mid-batch → the rest owed with `notBefore`. Correct.
- `attempt`, `clearOwed`, `retryOwed`: transport error → stays owed. API refusal or `errChannelRefused` → not owed. A plain hint read does not clear an owed scan read. Correct.
- `readChannel`, `scanThreads`, `readThread`:
  - Empty cursor → reads from the channel's beginning. Correct.
  - Failure mid-batch → cursor stays before the failing message. Correct.
  - `thread_broadcast` → forwarded once by the top-level read; the thread read skips it and advances the thread cursor. Correct.
  - Thread with no cursor → starts at the later of the parent and the floor. Correct.
- `PruneThreads`, `ThreadStart`: the floor rises on every call even when nothing is pruned. For a thread with no cursor the result is the same as D2, because `ThreadStart` takes the later of the parent and the floor. Correct.
- `keeper.ensure`, `create`, `adopt`, `heldElsewhere`, `join`:
  - Archived entry → new channel replaces it. Correct.
  - `name_taken` on an archived channel, another creator's, or a channel held by another orch or hub → refused with one notice. Correct.
  - A live map or live-hub read failing → fails closed before any join. Correct.
  - Adoption → cursor and thread floor at the newest message, or at now for an empty channel. Correct.
  - Entry written before the invite. Correct.
- `inviteOperator`, `inviteOperators`: refused invite → one notice per episode, entry stays unrecorded and is retried. Correct.
- `retire`, `retireEntry`, `renameDated`: prime and archived entries skipped, a failed read of the beams list never reaches it, a refused orch skipped unless `retryRefused`, `name_taken` steps the numbered suffix, no unarchive. Correct.
- `ChannelName`, `ArchivedName`, `fitName`: an 80-character base plus a date → re-cut with a hash, 80 characters in all. Correct.
- `outboundRoute`, `channelFor`, `postTo`, `postRecovered`, `refused`:
  - Gone beam → posts to the prime's channel. Correct.
  - Beams read failing → open channel used, otherwise `errNoOpenChannel` and only that beam's mail waits. Correct.
  - `is_archived`, `not_in_channel` and `channel_not_found` each recover once, and the refusal counts once. Correct.
  - A rate limit joined with a refusal → the read still sees the rate limit. Correct.
- `RunRelay`, `missingTokenExit`, `wakeLoop.wake`, `run`, `watch`, `diskCheck`, `runBinaryCheck`:
  - Order is relay lock, first read, `RequireConfigured`.
  - The one-second floor holds on the file-event path.
  - On a re-exec the holder stops and the lock is released; a failed exec retakes the lock and restarts the holder.
  - Retire retries and the prime ensure retry run only at outbound wakes. Correct.
- `SetOperator`, `ReadChannelMap`, `UpdateChannelMap`, `OrchOf`: unknown keys kept, a missing file reads as an empty map. Correct.

**Findings**

1. LOW, design — `internal/slackcli/cli.go:303-308`. `endLiveRelays` returns at the first hub whose `stop` fails. Both tokens are already stored, so the relays of every later hub keep running on the replaced tokens. The error names only the failing hub, so those hubs are neither ended nor named. D3 says login ends the relay of each live hub and names each one. The loop should continue past a failure and name every hub not ended.

2. LOW, design — `internal/slackcli/cli.go:269`. The app match compares `helloAppID != botAppID` with no empty check. If `bots.info` and the `hello` message both return an empty app id, the two tokens pass as one app and are stored. D3 says the app-level token's app id must be the bot token's. Both ids should be required to be non-empty.

3. NIT, consistency — `internal/slackengine/socket.go:32`. The `Ack` comment says Slack asks for acknowledgement "within three seconds", but `holder.go:28` waits ten seconds before acknowledging a stalled route. The decision record says Slack names no acknowledgement deadline. A slow route can therefore get a Slack retry. That is harmless, because the reads are idempotent. The comment is wrong against the record.

**Tool calls**
1. Read `internal/slackengine/channelname.go`: it was in the diff but not in my context, and I needed it to check the name cut and the dated-name suffix.
