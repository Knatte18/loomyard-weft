```yaml
lens: guard-reach
outcome: reported
```

Guard-reach review of the Slack relay plan (cards 1–14). Every guard not listed below can be reached through a real entry point.

1. **MEDIUM** — Card 9, `RunRelay` step 4 (the relay's first read).
   - **Summary:** an accepted first read with unread mail falls into the retry branch, so the relay never polls while `<shortname>:SLACK` holds mail.
   - **Evidence:**
     - The card makes the first read `Mailbox.ReadAll` with "a handler that leaves any mail unread". The only way to leave mail unread is for the handler to return an error.
     - `internal/creelengine/client.go:266-268` (`Client.exchange`) returns that handler error unchanged, and `internal/creelcli#GoClient.ReadAll` passes it on.
     - The card's "Any other refusal or failure … is retried … and calls no Slack method" therefore catches the relay's own handler error.
     - Mail queued while the relay was down is the normal state, since D5 says mail waits in creel.
     - No `TestRunRelay` row puts unread mail in the mailbox at the first read.
   - **Fix:** the handler returns a package sentinel that the relay matches with `errors.Is` as "accepted". Add a row: a first read with unread SLACK mail writes `FirstReadAt`, polls, and leaves the mail for `outbound.read`.

2. **MEDIUM** — Card 8, `outbound.read` (its bounded `ReadWait`).
   - **Summary:** an idle wait that ends with no mail reaches the failure branch.
   - **Evidence:**
     - A `Client.Call` timeout returns `ErrWaitTimeout` (`internal/creelengine/client.go:174-175`, `:234-235`).
     - Card 8 says "any error keeps it". Card 9 step 7 appends "every refused or failed read" to the log and warns per episode.
     - The card never says that `ErrWaitTimeout` means an empty read: no failure log, the record kept as is, and the doubling wait rather than the floor.
     - No `TestOutboundRead` row covers a wait that expires with no mail.
   - **Fix:** state that branch and add the row.

3. **MEDIUM** — Card 10, `setup` (D1's private channel).
   - **Summary:** the "bot not a member" refusal, which names `/invite @<app>`, is reachable only for a public channel.
   - **Evidence:**
     - The card checks `ChannelInfo.IsMember` from `conversations.info`.
     - Slack answers a bot token that is not in a private channel with `channel_not_found`, which reaches card 6's `*APIError` branch, not `is_member: false`.
     - D1 defaults to one private channel per hub, so the operator's usual mistake gets an error that names no invite.
     - No `TestSlackCLI` row covers `channel_not_found`.
   - **Fix:** map `channel_not_found` from `ConversationInfo` to the same refusal naming the invite, and add that row.

4. **MEDIUM** — Card 11, the presence line (D6).
   - **Summary:** a fresh launch from a handoff gets a resume pointer that never passes through `WithPresence`.
   - **Evidence:**
     - `internal/orchcli/start.go:654` calls `orchengine.ChooseStartPrompt`, which at `internal/orchengine/launch.go:43-50` renders `RenderResumePrompt` from `LastHandoff` (or the start prompt).
     - That is the pointer handed to shuttle when a dead orch session is relaunched.
     - Card 11 wraps only `ResumeContext`, the three watcher sites and `launchOrch`'s reload pointer (`start.go:641`).
     - An orch relaunched while presence is `slack` therefore carries no presence line.
   - **Fix:** list the `ChooseStartPrompt` call site (or `ChooseStartPrompt` itself) on card 11, and add a row.

5. **MEDIUM** — Card 9, `RunRelay` step 2 against the `RelayLive` lock probe.
   - **Summary:** "a relay that cannot take the lock exits" can be reached by a liveness probe, not only by a second relay.
   - **Evidence:**
     - `RelayLive` probes by taking the lock.
     - Card 12's `EnsureRelayWhileAway` probes on every prime dobby tick, and `start`, `away` and `status` probe too.
     - A fresh relay that tries the lock during a probe exits, so no relay runs.
     - `away`'s `AwaitFirstRead` then refuses after 30 seconds, and nothing restarts the relay until the next tick.
     - Dobby's precedent waits `lock.ProbeGrace` on a normal start (`dobbyengine` `daemon.run`, `internal/dobbyengine/loop.go:214`, as the exploration reported).
   - **Fix:** a fresh start waits `lock.ProbeGrace`, as dobby's does; add a row with a lock held briefly by a probe.

6. **LOW** — Card 3, the `TestCreelCLI` row for a program claim from a non-orch pane.
   - **Summary:** the row can only pass on Linux, but the card does not limit it to Linux.
   - **Evidence:**
     - Off Linux, `resolveSender` takes `resolveFromEnvironment` (`internal/creelengine/identity.go:52-54`), which accepts a program claim whatever the pane.
     - `PATTERN-test-speed` ("No test skips by design") asks that a row bound to one OS is not generated on another.
   - **Fix:** say the row is generated on Linux only.

Tool calls beyond what my context held:
- I read `internal/orchengine/launch.go:30-60` and grepped the `Launch` callers, to see which typed or launch pointers card 11 leaves out; my context had only the caller list from grep.
- I grepped every `runproc.End`, `EndSparing` and `EndRole` site outside runproc, to check whether other paths end the `HubOwner` relay without restarting it; none was confirmed.
- I read `internal/tendercli/deps.go:95-120` and the `ChooseStartPrompt` callers, to judge the tender `EndOwner` scope (left unreported: its owner set is unclear) and to find the launch pointer's caller at `start.go:654`.
