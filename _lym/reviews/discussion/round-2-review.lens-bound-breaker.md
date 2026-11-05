```yaml
lens: bound-breaker
outcome: reported
```

**Lens: bound-breaker.** I tested every bound, guard and claim about existing code in `decision-record.md` and tried to break each one.

**Claims attacked**
- D3, double fork keeps the relay out of every pane tree: **held**. An orphan from this pane was reparented to `systemd --user` (pid 1885), which is outside the pane chain bash → claude → bash → tmux server.
- D3, "creel reads it as no strand" at the first read: **broken** (F3).
- D3, the relay is restarted after a prime anchor change, "after a change that lands and after one that fails": **broken** (F1).
- D3, a stopped relay stays down until a start, `away`, a prime bring-up or an anchor change: **held**.
- D3, re-exec takes the lock again: **broken, narrowly** (F7).
- D4, a first start writes the cursor so a restart reads from there: **held**.
- D4, every message is handled exactly once and the cursor moves only once handling ends: **broken**, because the record never says what a failed creel post does (F3).
- D4, a hold delays at most an hour past its last renewal: **held**. `maxHoldDuration = time.Hour` in `internal/creelengine/pool.go:23`.
- D4, the 4 KiB notice cap: **held**. `maxNoticeBytes = 4096` and `isNotice` in `internal/creelengine/reminder.go:23-30`.
- D5, mail waits for a relay that stays down, with no limit: **held**. `carriedMessage` carries unread mail (`internal/creelengine/fold.go:499-515`), and `OrigAt` keeps the original stamp for the 24-hour skip.
- D5, at-least-once delivery that drains without reposting: **broken** against creel's 30-second acknowledgement window (F2).
- D5, "only orchs and lym programs reach the channel": **broken** through EXTERNAL (F4).
- D2, the relay accepts only the configured operator's messages: **bypassable** through `lym slack setup` (F5).
- D6, no process inside a non-orch strand's pane changes presence: **held as stated**. The bound already admits a deliberate escape out of the pane.
- D7, authority rests on creel's program trust: **held as stated**.
- D8, a pause or a resume restarts the wait's clock: **broken** (F6).
- D8, a restarted dobby may remind once early: **held**.
- Constraints, four hubs fit Tier 3 at 48 requests a minute: **held** for `conversations.history`.

**Findings**

F1 — MEDIUM, consistency, D3 and the D3 acceptance criterion: the relay is not restarted after a failed prime anchor change, or when no dobby was ended.
- **Issue:** `EndDobbies` calls `runproc.EndSparing` under `HubOwner` for the prime whether or not a dobby is live, so the relay always ends.
  Restart goes only through `startDobbies(deps, ended)`, which does nothing when `ended` is empty.
  `fail()` starts no dobby when the change record stands or cannot be read.
  So the criterion "the relay is live after a prime anchor change with presence `terminal`, whether the change lands or fails" cannot pass on those paths.
  With presence `slack`, the relay also stays down until a dobby comes back, and nothing restarts the dobby.
- **Evidence:** `internal/fabriccli/anchordeps.go:248-262`, `internal/fabriccli/anchor.go:350-366`, `internal/fabriccli/anchor.go:472-476`.

F2 — MEDIUM, design, D5 retry and record: creel's 30-second acknowledgement window is not bounded.
- **Issue:** creel releases a read whose acknowledgement does not arrive within `ackReadTimeout = 30s`.
  A read returns every unread message, so a backlog that takes Slack longer than 30 seconds to post, or a 429 `Retry-After` hit inside the handler, outlives the window.
  The relay's acknowledgement then fails, and "acknowledges the read … and then clears the record" does not say the record is cleared only on a confirmed acknowledgement.
  If it is cleared anyway, every reread reposts the whole backlog, until the 24-hour skip ends it.
- **Evidence:** `internal/creelengine/server.go:27` and `:478-480`; `internal/creelengine/doc.go:98` ("read returns … every one").

F3 — MEDIUM, design, D4 and D3: a failed creel post has no defined outcome.
- **Issue:** D4 moves the cursor past a message once its handling ends, but never says what happens when the `[slack]` post fails: creel down, or the program claim refused.
  creel refuses a claim from a process in no pane whenever any live strand's pane pid is unknown, so "creel reads it as no strand" does not always hold.
  The operator's message is then either silently lost or retried, and the record does not say which.
  The same refusal makes `lym slack away` time out after 30 seconds with nothing to tell the operator why.
- **Evidence:** `internal/creelengine/identity.go:68-69` and `:129-131`.

F4 — MEDIUM, design, D5 sender filter: EXTERNAL mail can reach the operator's Slack.
- **Issue:** D5 posts mail from "an orch strand or a program of the hub".
  `agentname.IsProgram` accepts `&lt;shortname&gt;:EXTERNAL`, because it is a well-formed upper-case name.
  Any process outside a pane gets the EXTERNAL sender through the plain CLI, for example an agent's `(setsid lym creel post --to &lt;sn&gt;:SLACK …&amp;)`.
  An agent can therefore get text onto the operator's phone, which breaks the bound "only orchs and lym programs reach the channel".
  The record never says EXTERNAL is dropped.
- **Evidence:** `internal/agentname/agentname.go:141-149` and `:209-212`; `internal/creelengine/identity.go:113-115`.

F5 — MEDIUM, design, D2 and D9: `lym slack login` and `lym slack setup` have no stated limit on who may run them.
- **Issue:** The guard "accepts a message only when its `user` equals the configured operator id" holds only while the config does.
  D6's non-orch-pane refusal covers `away`, `back` and `stop` only.
  Any strand can run `lym slack setup` or `lym slack login` from its pane and point the channel and operator id at a user it controls.
  That user's Slack text then reaches the orchs as `[slack]` mail with the operator's authority.
- **Evidence:** D2 and D6 of `decision-record.md`; `internal/creelengine/identity.go:55-80`.

F6 — LOW, design, D8 bound and acceptance criterion: a pause dobby never sees does not restart the wait's clock.
- **Issue:** The start is cleared only when dobby reads a pause covering the child.
  A pause and resume that fall between two once-a-minute checks, or while dobby is down (a hibernate, an anchor change), leave the old start in the state record.
  The reminder then fires at once after the resume, which the D8 criterion forbids.
- **Evidence:** D8 of `decision-record.md`; `persistTransient` leaves history unchanged (focus file, `internal/shedengine/run.go`).

F7 — LOW, design, D3 re-exec: two relays can run during a re-exec.
- **Issue:** Go opens files close-on-exec, so a lock held through a file descriptor is dropped at the re-exec until the new image takes it again.
  A dobby tick or `lym slack start` that runs in that window starts a second relay, and every message is then forwarded twice.
  The record does not bound this.
- **Evidence:** D3 of `decision-record.md`, "taking its lock again".

**Tool calls**
- `internal/pauserecord/hold.go` and the creel read verb sites: to check what the pause gate covers and where a read enters, which the manifest lacked.
- `internal/agentname/agentname.go` `IsProgram`, `External`: to check whether EXTERNAL counts as a program name for D5's sender filter.
