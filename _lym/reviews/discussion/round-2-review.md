---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: handler
    location: _lym/discussion/decision-record.md D3, second bullet (double fork)
    summary: D3's double fork has an intermediate that starts the relay and exits without waiting on it, which PATTERN-reap-children forbids, and the record neither cites the departure nor declares a change to the entry
    fix: "In decision-record.md either (a) add to `## PATTERN changes` a bullet `PATTERN-reap-children` — changes: <text that keeps the current rule and admits one exception, the intermediate of a double fork that exits at once so its child is reparented outside the starter's tree and reaped by the user's init>, keeping the rule text under the PATTERN cap, and name that exception in D3's double-fork bullet; or (b) replace the double fork in D3 with a start that hands the relay to a starter outside every pane that waits on it, naming that starter, and reword D3's identity bound to match."
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md D5, fifth bullet (record of posted ids, acknowledge, clear)
    summary: D5 posts a whole read inside creel's 30-second acknowledgement window and clears its posted-id record after acknowledging, so a backlog that takes longer than 30 seconds to post can be reposted on every read until the 24-hour skip drains it
    fix: "In D5, state that a read's handler stops posting before creel's 30-second acknowledgement deadline (`ackReadTimeout` in internal/creelengine/server.go) and releases the read with the rest unposted, waiting out any `Retry-After` outside the handler; state that the posted-id record is cleared only after the daemon confirms the acknowledgement and kept otherwise; extend D5's delivery bound to say a refused or lost acknowledgement reposts nothing the record holds; add an acceptance criterion: a backlog whose posting outlasts the acknowledgement window posts each message exactly once."
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md D4, second bullet (cursor moves once handling ends)
    summary: D4 says nothing about a forwarded message whose creel post fails or is refused, as creel does for a program claim from outside a pane while a live strand's pane pid is unknown, so the operator's text is either lost silently or retried without a bound
    fix: "In D4, add that a creel post that fails or is refused leaves the cursor before that message, logs at Warn and retries with the poll's backoff, and say whether, after a stated number of failures or time, the relay answers in Slack that the message could not be delivered (moving the cursor) or keeps retrying; state the same refusal as a cause of `lym slack away`'s 30-second refusal in D6, naming it in the refusal; add an acceptance criterion: a refused creel post is forwarded once creel accepts it, and is never skipped silently."
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D2, second and third bullets; D3 (`lym slack start`); D9
    summary: lym slack login, lym slack setup and lym slack start state no limit on who may run them, so any strand can repoint the token, channel or operator id and hand the operator's authority to a Slack user it names, or restart a relay the operator stopped
    fix: "In D2, add that `lym slack login` and `lym slack setup` refuse when run from any live strand's pane, an orch's included, by the same pane-ancestry check D6 uses, naming the operator's terminal as the way forward, with a Bound line saying what remains (a process of the operator's user outside every pane, as D6 and D7 already state); in D3, state who may run `lym slack start` (refused from a non-orch strand's pane like `stop`, or allowed with a Bound line saying it only restarts a relay that forwards the configured operator's text); add acceptance criteria for each refusal."
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D5, third bullet (sender filter)
    summary: D5 posts mail from a program of the hub, and creel's EXTERNAL sender parses as a program, so any process outside a pane can post to the operator's Slack, breaking the bound that only orchs and lym programs reach the channel
    fix: "In D5's third bullet, say that mail whose stamped sender is `<shortname>:EXTERNAL` is dropped and logged at Warn like a non-orch strand's; in D5's acceptance criterion for the sender filter, add that EXTERNAL mail is not posted."
  - id: F6
    severity: MEDIUM
    class: consistency
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md D3, third bullet (anchor-change start), and the D3 acceptance criterion
    summary: a prime anchor change always ends the relay, but restarts dobbies only when one was ended and never after a failure whose change record stands, so the criterion that the relay is live after the change, landed or failed, cannot pass, and on the landed path the prime BeamUp already starts it
    fix: "In D3's third bullet, replace the anchor-change start with one of: the anchor change starts the relay unconditionally once it ends, landed or failed, whenever the `slack` config is complete; or drop the anchor-change start and add a Bound line that the relay stays down after an anchor change until the prime bring-up, a start, `lym slack away` or, with presence `slack`, the next prime dobby tick. Make the D3 acceptance criterion match the chosen wording."
  - id: F7
    severity: LOW
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md D8, second bullet, its Bound, and the D8 acceptance criterion
    summary: the wait start is cleared only when dobby itself sees a pause, so a pause and resume between two once-a-minute checks or while dobby is down keep the old start and remind at once after the resume, against D8's bound and criterion
    fix: "In D8, make the wait start the later of the time dobby first read the child at the row and the time of the last resume that lifted a pause covering it, read from the pause record, or weaken D8's Bound to say a pause and resume dobby does not observe keep the old start; align the D8 acceptance criterion with the chosen wording."
  - id: F8
    severity: LOW
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md D2, third bullet, and D4, first bullet
    summary: the D4 cursor carries no channel, so a lym slack setup that changes the channel makes the relay read the new channel from the old one's ts and forward its history, and neither decision says whether a live relay picks up the new channel or operator
    fix: "In D4, key the persisted cursor by channel id so a changed channel takes the first-start path; in D2, state whether `lym slack setup` restarts a live relay or the relay reads the channel and operator at each poll."
  - id: F9
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D3, fifth bullet (re-exec)
    summary: the single-instance lock is released across the re-exec, as Go opens files close-on-exec, so a start in that window can run a second relay that forwards every message twice, and the record does not bound it
    fix: "In D3's re-exec bullet, state that the lock's descriptor is kept across the exec, or add a Bound line that a start racing the re-exec can run a second relay until the loser's lock attempt fails, and say the loser exits before its first poll."
---

The focus directives were each reached.

- **Relay sender identity:** the double fork holds. An orphan started from this pane was reparented to the user's systemd (pid 1885), outside the pane chain bash → claude → bash → tmux server. So a relay that `lym slack start` brings up from a non-orch strand's pane reads as no strand, and its acceptance criterion can pass. The exceptions are F3's refusal while a strand's pane pid is unknown, and F1's PATTERN departure.
- **Cursor durability:** holds. A first start writes the cursor before its first poll, so a relay that dies without handling a message forwards on restart what was written while it was down. F8 covers the one gap, a changed channel.
- **Relay lifecycle against its restarters:** see F6.
  `lym slack stop` sets `terminal`, then ends the relay, and no dobby tick restarts it in either presence state, as D3 states.
- **Mail retention and posting:**
  - `retain` carries unread mail and the program's read record forward with `OrigAt`, so D5's "no limit" and its 24-hour skip hold.
  - The retry does not drain without reposting against the 30-second acknowledgement window (F2).
- **Reminder wait start:** see F7.
- **Attack surface:**
  - `away`, `back` and `stop` are bounded by D6.
  - The Slack texts and `@<slug>` are bounded by D2's operator check and the hub's beam list.
  - `login`, `setup` and `start` are unbounded (F4), and the outbound filter lets EXTERNAL through (F5).

### [BLOCKING] Double fork departs from PATTERN-reap-children
**Location:** `_lym/discussion/decision-record.md` D3, second bullet
**Issue:** `PATTERN-reap-children` reads: "Starting a child process: its starter waits on it, by `cmd.Wait` or `proc.Reap`, never only releasing it".
D3's intermediate child starts the relay and exits at once, so the relay's starter only releases it. That is the point of the double fork, since the relay must be reparented out of the starter's tree.
No lym code has a double fork today. A search for double fork, intermediate and grandchild finds only test files and a reed comment.
The record does not cite the departure, and `## PATTERN changes` declares no change to the entry.

### [MEDIUM] D5's read handling outruns creel's acknowledgement window
**Location:** `_lym/discussion/decision-record.md` D5, fifth bullet
**Issue:** creel's daemon releases a read that is not acknowledged within `ackReadTimeout` (30 s, `internal/creelengine/server.go:26-27`, `awaitAck` at :478-480). A read hands out every unread message (`internal/creelengine/doc.go`, "read returns the caller's unread messages oldest first: every one").
After an outage the backlog can take longer than 30 s to post, at about one `chat.postMessage` a second per channel or with a `Retry-After` waited out inside the handler. The acknowledgement then fails.
D5 says the relay acknowledges "and then clears the record", without saying the clear waits for a confirmed acknowledgement. If it does not wait, every later read reposts the backlog.
The three lenses that found this agree on the mechanism.

### [MEDIUM] D4 leaves a failed creel post undefined
**Location:** `_lym/discussion/decision-record.md` D4, second bullet
**Issue:** D4 gives every message three outcomes: forwarded, answered or skipped.
A `[slack]` post that creel refuses fits none of them. creel refuses a program claim from outside every pane while a live strand's pane pid is unknown (`internal/creelengine/identity.go:68-69`, :129-131), and it also refuses while the daemon is down.
D4 does not say whether the cursor moves past such a message, which loses the operator's words silently, or stays and retries.

### [MEDIUM] login, setup and start state no limit on who may run them
**Location:** `_lym/discussion/decision-record.md` D2, D3, D9
**Issue:** D2's guard, that only the configured operator's messages are accepted, holds only while that config holds.
D6 refuses `away`, `back` and `stop` from a non-orch strand's pane, but no decision limits `login`, `setup` or `start`.
A strand can run `lym slack setup` from its pane with a channel and user id it controls. That user's Slack text then reaches every orch as `[slack]` mail with the operator's authority.
The rubric's attack-surface rule requires each new verb's Decision to state what it lets through and what bounds it.

### [MEDIUM] EXTERNAL mail passes D5's sender filter
**Location:** `_lym/discussion/decision-record.md` D5, third bullet
**Issue:** `agentname.IsProgram` reports EXTERNAL as a program ("IsProgram reports whether full parses as a program sender, External included", `internal/agentname/agentname.go`).
creel stamps EXTERNAL on a sender it cannot place (`internal/creelengine/identity.go:113-115`), which is any plain CLI post from outside every pane.
D5 posts "an orch strand or a program of the hub", so EXTERNAL mail reaches the operator's phone. That contradicts D5's own bound, "only orchs and lym programs reach the channel".

### [MEDIUM] Anchor-change restart misses the paths the criterion names
**Location:** `_lym/discussion/decision-record.md` D3, third bullet, and the D3 acceptance criterion
**Issue:** For the prime, `EndDobbies` runs `runproc.EndSparing` under `runproc.HubOwner`, through `dobbyOwner` in `internal/fabriccli/anchordeps.go`. It spares only push and sync, so it ends the relay whether or not a dobby is live.
Dobbies start again only through `startDobbies(deps, ended)`, which returns nil when `ended` is empty (`internal/fabriccli/anchor.go:472-476`). The `fail` closure starts none when the change record stands or cannot be read.
The criterion "the relay is live after a prime anchor change with presence `terminal`, whether the change lands or fails" therefore cannot pass on those paths.
Where dobbies do start, `EnsureDobbies` goes through a `lym beam start` child, which is the prime's `BeamUp`. D3 already lists that as a relay start, so a separate hook there adds nothing.

### [LOW] An unobserved pause keeps the reminder's old wait start
**Location:** `_lym/discussion/decision-record.md` D8, second bullet and Bound
**Issue:** A pause and a resume rewrite the run's status without touching its history (`persistTransient` in `internal/shedengine/run.go`). So dobby learns of a pause only by reading one at its once-a-minute check.
A pause and resume that fall between two checks, or while dobby is down after an orch down or an anchor change, leave the stored start in place, and the next check reminds at once.
That contradicts D8's Bound, "a pause or a resume restarts the wait's clock", and its acceptance criterion.

### [LOW] Cursor is not keyed by channel
**Location:** `_lym/discussion/decision-record.md` D2, third bullet; D4, first bullet
**Issue:** `lym slack setup` can change the channel, but the cursor is one `ts` with no channel attached.
A relay then reads the new channel from the old channel's `ts` and forwards that channel's history, which defeats D4's first-start rule.
No decision says whether a live relay picks up the new keys.

### [LOW] Re-exec drops the single-instance lock for a moment
**Location:** `_lym/discussion/decision-record.md` D3, fifth bullet
**Issue:** D3 says the relay takes its lock "again" after the re-exec, so the lock is free between the exec and the new image's attempt.
A dobby tick or `lym slack start` in that window can start a second relay, which forwards each message twice.
The record states no bound for it.

## Rejected

- interactions F2 is kept as F8 at LOW, not MEDIUM: changing a hub's Slack channel is a rare operator step, and the effect is one replay of the new channel's history.
- walkthrough F4, existing-paths F4 and bound-breaker F6 describe the same defect as F7, and are merged into it.
- The three lenses' reports of the 30-second acknowledgement window, of the anchor-change restart and of the failed creel post are each merged into F2, F6 and F3.
