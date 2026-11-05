```yaml
lens: walkthrough
outcome: reported
```

Walkthrough lens: each decision run end to end on one concrete case, end state checked against its purpose.

**Decisions walked**
- D1, one channel per hub. Case: the `ly` hub's relay reads only its own channel id. Purpose met.
- D2, credentials. Case: `lym slack login` reads the token from stdin, `auth.test` answers ok, the file is written; `setup` then checks the channel with `conversations.info` and the user with `users.info`, and the scopes allow both calls. Purpose met.
- D3, relay lifecycle. Case 1: `lym slack start` from the operator's terminal. The orphan is reparented to the systemd user manager, outside every pane, as checked live in this pane. Its first read is accepted as a program. Purpose met.
  Case 2: a prime anchor change while the prime dobby is not live. Purpose not met (F3).
- D4, inbound. Case 1: `@foo fix x` from the operator reaches `ly:foo:orch` as `[slack] fix x`. Purpose met.
  Case 2: the creel post of a message is refused. What happens next is undefined (F2).
- D5, outbound. Case 1: a failed post on message 2, then success. The record skips id 1, so nothing is posted twice. Purpose met.
  Case 2: 50 queued messages after a relay outage. Purpose not met (F1).
  Case 3: a beam dobby's reminder from `ly:foo:DOBBY` gets the `@foo` hint, since scoped program names exist. Purpose met.
- D6, presence. Case: `lym slack away` from the terminal starts the relay, waits for its first read, sets `slack`, and mails notices as the program `SLACK`. A process outside every pane may claim a program name. The Slack text `away` takes the same path through the relay. Purpose met.
- D7, orch rules. Case: the orch replies to SLACK mail with `--reply`, which lands in the relay's mailbox and is posted. Purpose met.
- D8, reminders. Case: a child at PR-Review is paused at 29.5 min and resumed at 29.8 min, between two once-a-minute checks. The wait's start is never cleared, so a reminder fires at 30 min. Purpose not met (F4).
- D9, CLI module. Case: the cobra subtree is mounted. Purpose met.

**Findings**

F1. MEDIUM, design, D5 (read, acknowledge, clear record).
D5 does not bound how long one read may take to handle, or when the record is cleared, against creel's 30-second acknowledgement deadline. A backlog can be reposted over and over.
- Case: the relay comes back to 50 queued messages. `read` hands all of them over in one response.
- Slack's `chat.postMessage` allows about one post per second per channel, so posting all 50 takes at least 50 s.
- The daemon releases the read unacknowledged after `ackReadTimeout`, which is 30 s, so the relay's acknowledgement fails.
- D5 says the relay "acknowledges the read … and then clears the record". If the record is cleared after that failed acknowledgement, the next read hands out all 50 again with no record of what was posted, and they are posted again. This repeats until the mail is more than 24 hours old.
- D5's own bound, "a crash between a post and its record write posting that message once more", misses this case.
- Evidence: `internal/creelengine/server.go:26-27`, `:478-480`; `internal/creelengine/client.go:136`; `internal/creelengine/doc.go` (`read returns the caller's unread messages oldest first: every one`).
- Fix: clear the record only after the daemon confirms the acknowledgement, and bound each read's handling below `ackReadTimeout`, or post in batches that fit inside it.

F2. MEDIUM, design, D4 (forwarding versus the cursor).
D4 gives each message three outcomes: forwarded, answered or skipped. A creel post that fails or is refused fits none of them.
- Example: creel refuses a program claim from outside every pane while any live strand's pane pid is unknown.
- The record never says whether the cursor moves past such a message, which drops the operator's text silently, or holds it and retries.
- Evidence: `internal/creelengine/identity.go:68-69`; D4 bullets 2 and 9.

F3. MEDIUM, consistency, D3 (restart after an anchor change) and the D3 acceptance criterion.
- `EndDobbies` ends every `HubOwner` process except pushes and syncs, so the relay goes down with the dobbies.
- The restart runs only through `startDobbies(ended)`. `ended` lists only worktrees whose dobby process was live, and `startDobbies` does nothing when the list is empty. A failure that leaves the change record standing starts nothing at all.
- So with the prime dobby down, or after that kind of failure, the relay stays down.
- This breaks the criterion "the relay is live after a prime anchor change with presence terminal, whether the change lands or fails".
- On the path where the change lands, `EnsureDobbies` goes through `lym beam start`, which runs the prime's `BeamUp` and so already starts the relay (D3 bullet 3). The separate anchor hook is either redundant or has to run unconditionally.
- Evidence: `internal/fabriccli/anchordeps.go:85-88`, `:248-257`; `internal/fabriccli/anchor.go:366-381`, `:472-476`.

F4. LOW, consistency, D8 (wait start versus pause).
- The wait's start is cleared only when a once-a-minute dobby check sees a pause covering the child.
- A pause and resume that both fall between two checks, or while the dobby is down, keep the old start. A reminder can then fire right after the resume.
- That breaks D8's bound "a pause or a resume restarts the wait's clock" and its criterion "no reminder before a full interval after the resume".
- Evidence: `internal/dobbyengine/loop.go:27` (`idleCeiling = time.Minute`).

**Tool calls**
1. `internal/agentname` `ParseProgram` and dobby's program name: to confirm a slug-scoped program sender exists for D5's answer hint.
2. `internal/dobbyengine` loop interval and state file: to confirm D8's once-a-minute check and the state record it would use.
3. `internal/pauserecord/hold.go` and where the gate is installed: to check whether a pause's network gate would hold the relay's Slack calls in D4's pause case. It is installed only in `tendercli`, so it does not.
