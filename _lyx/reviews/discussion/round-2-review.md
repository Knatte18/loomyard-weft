---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:97
    summary: A relaunch withdraws only the old cycle's injects, so its unread note-request mail is reminded to the relaunched session, contradicting the at-most-one-line bound
    fix: In D5's Fresh launch bullet, make `withdraw` remove the caller's own queued injects and its own unread mail to the receiver, each journaled with reason `withdrawn` (expired for an inject, undelivered for mail); change the Scope bullet at decision-record.md:16 and D5's sentence at line 99 to say so; extend the Bound at line 101 to say a message the orch already read stays read; and extend the acceptance criterion at line 215 to say the relaunched orch is reminded of no note request from the old cycle.
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:102
    summary: A failed withdraw at relaunch leaves old /clear and /compact command injects queued for the fixed orch name, and nothing retries the withdraw
    fix: "In D5's Fresh launch bullet, replace the sentence at line 102 with: when the withdraw or the release fails, `lyx orch start` records that in the orch state, and the watcher it spawns retries both, signing as `ORCHWATCH`, on every tick before it queues any step, clearing the record once both succeed; until then it starts no cycle and shows the failure as State.Stuck. Add the bound: until a retry succeeds, an old inject can still reach the new session, a `/clear` or `/compact` included, for at most the inject TTL the watcher's step injects carry, and D5 names that TTL. Add an acceptance criterion: a relaunch whose withdraw is refused gets the withdraw retried by the watcher, and no old `/clear` reaches the session once a retry succeeds."
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:131
    summary: D6 makes NotifyPrime decide whether it runs in its strand's pane, but no caller-side check exists and a mismatch with creel's ancestry walk loses the notice
    fix: "In D6's Sender paragraph, replace the in-pane test with a rule that needs no caller-side ancestry walk: when `$LYX_STRAND_NAME` names a non-orch strand, `NotifyPrime` posts with that strand claim; when creel refuses it because the process runs in no strand pane, it posts once more as its program with no strand claim; in every other case it posts as its program. State the bound: the program retry happens only on that refusal, so a caller in a non-orch pane never signs as a program. Change line 132 to say a detached caller whose ancestry no longer reaches the pane gets that refusal and signs as its program, and one whose spawning parent still lives signs as the strand."
  - id: F4
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:110
    summary: D5 filters turn ends by comparison with the sending time, but shuttle's parsed events carry no time and the hook stamps only whole seconds
    fix: "In D5's transcript bullet, rename the source to the run's events file, and state the mechanism: the claude engine's event parser returns each Stop's time from the stamp line before it, which needs a shuttle provider-seam change listed under Scope In; a turn end counts for a step only when its stamp's second is after the second of creel's `sending` record, and a Stop with no readable stamp never counts. Bound: a step whose own turn ends within the second it was typed is missed and times out, as an unconfirmed step does. Change the acceptance criterion at line 219 to pin that a turn end stamped in the sending second confirms nothing."
---

### [MEDIUM] A relaunch leaves the old cycle's note request reminded to the new session

**Location:** `_lyx/discussion/decision-record.md:97`

**Issue:** D5's Fresh launch bullet withdraws only ORCHWATCH's queued injects and releases its hold, and `withdraw` (line 99, Scope line 16) is defined over injects alone.
The note request is mail (D4), and creel addresses the orch by its fixed name (`agentname.RoleOrch`), so a request the old session left unread survives a relaunch, `--continue` or `--adopt` and is reminded to the relaunched session at once.
The acceptance criterion at line 215 states that outcome ("its mail is reminded at once").
The relaunched orch then reads a handoff request whose reply id the reset state no longer tracks, spends a turn writing a note, and the watcher drops the reply.
That makes line 101's Bound, "at most one line of the old cycle reaches the relaunched session", false.
`withdraw` is a verb this task adds, so it can cover the caller's own unread mail without a second verb.

### [MEDIUM] A failed withdraw at relaunch lets old command injects reach the new session

**Location:** `_lyx/discussion/decision-record.md:102`

**Issue:** Line 102 has a failed withdraw or release logged and the start going on, with the old injects lapsing "within their TTL".
D3 and D5 name no TTL for the watcher's step injects, so they take creel's default of ten minutes (`defaultInjectTTL`, `internal/creelengine/pool.go:19`).
D3 keeps an inject for a dormant receiver, and the relaunched session has the same orch name, so a queued `/clear` or `/compact` from the old cycle is typed into the fresh session and wipes or compacts it.
This failure path is reachable: D1 states creel refuses every sender outside a pane while any live strand's pane process is unknown, and `lyx orch start` signs as a program, so it is refused whenever it runs from a task pane.
The old hold also stays for up to an hour (`maxHoldDuration`), and the new watcher, which is the same owner, releases a hold only on a return to idle, so the new session's mail stays unreminded.
The Bound states the TTL but not that a destructive command can land, and nothing retries the withdraw.
The new watcher is the natural retrier: it signs as `ORCHWATCH` from a detached process and has queued nothing yet on its first tick, so a withdraw it runs then removes only old injects.

### [MEDIUM] D6's in-pane test has no mechanism, and a mismatch loses the notice

**Location:** `_lyx/discussion/decision-record.md:131`

**Issue:** D6 has a `NotifyPrime` caller sign as its strand only when "the calling process runs in that strand's pane".
Creel decides the same fact from the peer's process ancestry in `resolveSender` (`internal/creelengine/identity.go`), and no package exposes that walk to a caller; the only pane-PID users are creel, reed and the creel fake.
A caller-side guess that differs from creel's walk is refused either way: a strand claim from outside the pane gets "runs in no strand pane", and a program claim from a non-orch pane gets "only an orch strand's pane may claim a program name".
`proc` detaches with `Setsid` (`internal/proc/proc_linux.go:212`), which keeps the parent: a detached loom driver's ancestry still reaches its task pane while the spawning process lives, and stops reaching it once that process exits.
So line 132's "runs in no pane, so it signs as its program" holds only after the parent exits, and a program claim made before then is refused.
A refused notice is warned and dropped (line 129), so the plan writer needs a rule that cannot disagree with creel.
Settling the focus question for the named callers: `hibernate watch` and a detached loom driver sign as their program once their ancestry leaves the pane, and tender's revive signs as `TENDER`, which D7 bounds correctly to the orch's pane or no pane.

### [MEDIUM] D5's sending-time filter rests on event times shuttle does not provide

**Location:** `_lyx/discussion/decision-record.md:110`

**Issue:** D5 has the skills, retry and pointer steps read "only transcript events after creel's `sending` record", comparing the journal's time with event times.
Those steps read the run's events file, not the transcript, through `ParseEvents` (`internal/shuttleengine/claudeengine/events.go`), and `shuttleengine.Event` has no time field.
The only time in that file is the stamp line each hook writes before its payload, which `ParseEvents` skips, written by `date -u +%Y-%m-%dT%H:%M:%SZ` at whole-second precision (`internal/shuttleengine/claudeengine/settings.go:257`), and empty when `date` fails.
The watcher reads outcomes without blocking (line 116), so it cannot record the events-file offset at the moment of `sending`, and a time comparison is the only route.
That comparison therefore needs a shuttle provider-seam change, which Scope does not list, and a rule for a Stop stamped in the same second as `sending`.
Without that rule, the acceptance criterion at line 219 cannot be pinned deterministically.

## Rejected

- Focus 1, adopting an earlier reload step's inject on restart: not a defect.
  Each step is persisted pending with its own persist time before it is queued (line 111), and the journal lookup reads only ORCHWATCH's events since that time.
  Every earlier step's inject is posted and settled before the watcher reads its outcome and persists the next step, and line 114 orders the retry step's persist after the skills inject's outcome.
  `Store.Log` returns `Subject` as the body's first line, which for a text inject is the whole line, so matching the pointer by text works.
- Focus 2, the hold at relaunch: `release` signed as `ORCHWATCH` clears it, since creel's `Release` checks the owner by name and works after the strand is removed. Only the failure path is a defect, raised in F2. The note-request mail is raised in F1.
- Focus 3, a timeout on every reload step: the skills, retry and pointer steps already time out (`tickResuming`), and D5 line 107 gives the color and plugins steps the handoff timeout, counted from first queueing.
  Clearing is the one untimed phase, and line 96 states it with the `lyx orch status` stall, so the hold bound is accurate as written.
- Focus 4, a reply lost between the creel read and the save: not a defect. `Client.Call` runs the handler before it writes the ack, and a handler error leaves the messages unread (`internal/creelengine/client.go`), so D4 line 75 holds. A save that succeeds before the ack fails leaves the reply unread, but the restarted watcher no longer has a pending request, so it drops the reply as D4 says.
- Focus 5, holding reminders between the auto-compaction reload's steps: D5 line 92 takes the hold on entering the resuming phase and keeps it until idle, and the acceptance criterion at line 216 pins it. The offset part is raised in F4.
