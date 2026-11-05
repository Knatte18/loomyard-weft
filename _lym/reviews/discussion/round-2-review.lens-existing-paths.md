```yaml
lens: existing-paths
outcome: reported
```

Lens existing-paths: trace each existing code path that writes, resets or reads what the record introduces, and judge whether the decision's result survives it.

**Paths traced, per decision**

- **D3, relay process lifecycle:**
  - Prime anchor change, the `EndDobbies` call (`EndSparing` under `HubOwner`, sparing only push and sync): **breaks.** It ends the relay, but the restart misses several paths (F1).
  - Orch down, the orch's `endDobby` (`EndRole` of `RoleDobby` only): survives, and the relay keeps running.
  - Pair teardown (`EndSparing` under the slug): survives, since the relay is not registered under any slug.
  - Tender reap (`orphanOwner` never treats `HubOwner` as an orphan; unregistered orphans must be unregistered): survives.
  - `lym slack stop` (`EndRole` of the relay's role): survives as stated.
  - Re-exec on a replaced binary (same pid, `ReapInheritedChildren`): survives.
  - Hibernate engine and CLI: no `runproc` end path found, so the relay survives.
- **D4, inbound posts through creel:**
  - Creel sender resolution with a live strand whose pane pid is unknown: **breaks.** Creel refuses the program claim, and the record does not say what the cursor does then (F3).
  - Creel `retain`: survives. It carries the unread messages and the `SLACK` reader record forward.
- **D5, outbound read and acknowledgement:**
  - Creel's read reservation and its 30-second acknowledgement timeout: **breaks** under a long read (F2).
  - `retain` keeps the original post time through a carry (`OrigAt: msg.Posted`): survives, so the 24-hour skip reads the real stamp.
  - The program reader record carried forward: survives, and `SLACK` stays a known receiver after a long outage.
  - The notice cap: survives. Program mail of 4 KiB or less is typed in full, which matches D4's 4 KiB cap.
- **D6, presence file:**
  - Read by the prime dobby's start check, D8 and `resume-context`; written by the verbs and the relay. A missing file reads as `terminal`. Survives.
  - Orch down leaves the presence and the relay alone: survives.
- **D8, wait start in dobby's state record:**
  - A pause or resume rewrites status with history unchanged: **breaks** whenever dobby does not see the pause (F4).
  - A dobby restart through orch down or an anchor change: the stored start survives, as intended. The same path is part of F4.

**Findings**

**F1. MEDIUM, design, D3 (anchor-change restart) and the D3 acceptance criterion "the relay is live after a prime anchor change … whether the change lands or fails".**
- The relay is ended unconditionally: `EndDobbies` runs `runproc.EndSparing(runProcDir, dobbyOwner(prime)=HubOwner, …, RolePush, RoleSync)` for the prime even when no dobby is live.
- It is restarted only through `startDobbies(deps, ended)`. That call does nothing when `ended` is empty, meaning no prime dobby was live.
- On failure, the `fail` closure starts nothing when the change record stands or cannot be read.
- On the paths where dobbies do restart, `EnsureDobbies` goes through a `lym beam start` child, which is the prime's `BeamUp`. Per D3, `BeamUp` already starts the relay, so the separate anchor hook is redundant there and missing everywhere else.
- Result: with presence `terminal`, the relay stays down after a failed change whose record stands, and after any change made with no prime dobby live. The acceptance criterion cannot pass as worded.
- Evidence:
  - `internal/fabriccli/anchor.go:411` (the `EndDobbies` call)
  - `internal/fabriccli/anchor.go`, the `fail` closure's `standing` and `readErr` cases (about lines 360–368)
  - `internal/fabriccli/anchor.go:445-447` and `:472-477` (`startDobbies` returns nil when `ended` is empty)
  - `internal/fabriccli/anchordeps.go:85-89` (`dobbyOwner`), `:248-262`, `:263-269` (`EnsureDobbies` through beam start)

**F2. MEDIUM, design, D5 (outbound read and acknowledgement).**
- D5 posts each message inside the read's handler and acknowledges the read once every id is posted or skipped, "and then clears the record".
- The creel daemon releases a read that is not acknowledged within 30 seconds.
- A read returns every unread message, with no batch cap.
- So a backlog that takes Slack longer than 30 seconds to post releases the read before the acknowledgement arrives. Causes include `chat.postMessage`'s per-channel rate, or a `429` whose `Retry-After` is waited out inside the handler.
- If the record is then cleared, the next read posts every message again, and the loop repeats until the 24-hour skip drains it.
- The record states neither the 30-second bound on handling a read, nor that the record clears only on a confirmed acknowledgement.
- Evidence:
  - `internal/creelengine/server.go:25-27` (`ackReadTimeout = 30 * time.Second`)
  - `internal/creelengine/server.go:478-480` (`awaitAck` releases on no answer in time)
  - `internal/creelengine/doc.go:98` ("read returns the caller's unread messages oldest first: every one")
  - `internal/creelengine/client.go:136` (acknowledged only when the handler returns nil)

**F3. MEDIUM, design, D4 (cursor versus a refused creel post).**
- D4 moves the cursor past every message "once its handling ends, whether it was forwarded, answered or skipped".
- It never says what a creel post that fails or is refused does to the cursor.
- Creel refuses every program claim from outside a pane while any live strand's pane pid is unknown, and the daemon can also be down.
- If the cursor moves on, the operator's Slack message is lost silently. If it does not, the record does not state the retry and its bound.
- Evidence:
  - `internal/creelengine/identity.go:68-69` (the `unreadable` refusal comes before the program claim)
  - `internal/creelengine/identity.go:129-131` (a live strand with `PanePID &lt;= 0` sets `unreadable`)

**F4. LOW, design, D8 (wait start across a pause and resume) and the D8 acceptance criterion.**
- D8 clears the stored wait start only when dobby itself reads a pause covering the child.
- A pause and resume leave the child's status and history unchanged, per the focus statement about `persistTransient`.
- So a pause and resume that falls between two once-a-minute checks, or while the dobby is down, leaves the old start in the state record. The dobby can be down after an orch down or an anchor change, which ends dobbies and restarts them only on some paths (see F1).
- The resumed wait then reminds at once, which breaks the bound "a pause or a resume restarts the wait's clock" and the acceptance criterion "no reminder before a full interval after the resume".
- Evidence:
  - `internal/orchcli/cli.go:262-268` (orch down ends the dobby)
  - `internal/fabriccli/anchor.go:411` (an anchor change ends dobbies)
  - The record's D8 bullets 2 and 3

**Tool calls**
1. `internal/tenderengine/processes.go` and the `runproc.End*` callers: to find every other path that ends `HubOwner` processes, since the manifest held only the anchor path.
2. `internal/tendercli/deps.go` `EndOwner`, `internal/orchcli/cli.go` `endDobby`, and the hibernate functions: to see which owner and roles those end paths reach.
3. `internal/hibernateengine` and `internal/hibernatecli` `runproc` use, and the pause-gate installers: to check whether hibernate or the pause gate touch the relay.
4. `internal/tenderengine/reap.go` owner selection and the `InstallGate` callers: to confirm tender's `EndOwner` reaches only orphan owners.
