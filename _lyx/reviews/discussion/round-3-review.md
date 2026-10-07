---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:89
    summary: An awaiting stop with a parent_notice is announced only by the driver relay and D4 suppresses batten's notice, so a dead driver or a failed relay send leaves that stop announced by no channel with no stated bound
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:95
    summary: since is the time of the child's newest history entry, but a paused or failed child appends no entry, so the 3-minute report wait and the report-time filter run from a stale time and are not a lower bound
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:71
    summary: D3 does not say the baseline modification time is taken before the call's status read, so a change written between the read and the stat is never seen and the row waits silently on a stale state
  - id: F4
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:162
    summary: D7 creates the window detached and sets remain-on-exit off in one invocation while addressing it by id, but the id is unknown inside the invocation and an untargeted set-option lands on the strands' window
  - id: F5
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md:158
    summary: D7's no-live-Selvage fallback to the current window voids the unconditional claim that a batten window is never read as the strands' panes, and the decision names no bound on that case
  - id: F6
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md:101
    summary: The not-queued retry never stops, so a stop with no orch strand retries every notice_probe_s with a QueueNotice Info line each time, and D4 states a bound only for the Warn
  - id: F7
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md:143
    summary: The watched marker proves a strand in the prime's orch state, but the driver's parent is the orch of the worktree the pair was created from; D6 does not say these are the same orch
  - id: F8
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md:76
    summary: D3 lists the driver liveness read among the cheap reads run on every 2 s check, but it is a reed status call, so a done child or a pending notice now runs it up to 15 times as often as the 30 s bounce did
  - id: F9
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:65
    summary: D3 puts a change text without a time in the entry output while the acceptance criteria require the time, and consecutive pause returns fold and a failed-stat return adds an entry, against the Goal's claim of state changes and a requested pause only
---

### [MEDIUM] An awaiting stop with a parent_notice can be announced by no channel

**Location:** `_lyx/discussion/decision-record.md` D4 (line 89), D6 (line 141), Open risks (line 208)
**Issue:** D4 sends no state-changed notice for an awaiting child whose status carries a `parent_notice`, and D6 makes the driver's relay the one channel.
The driver stencil says "record a failed send as a line in the report rather than retrying it", and the driver can be dead when the stop lands (batten revives it, but the revived driver renders no relay).
In either case an awaiting stop with a `parent_notice` reaches the orch by no channel, and the Goal's claim that the orch hears each run stop is false for it.
The Open risks name the reverse (a driver that still messages is noise) and a dead batten, not this case, and D4's rationale states the principle ("a notice lost ... would leave the run unannounced") for batten's own channel only.
This is a guard downgraded without a stated bound.
**Fix:** Give batten's notice a fallback for this stop with a stated bound, for example: an awaiting child with a `parent_notice` gets batten's state-changed notice only when its driver strand has ended or the 3 minutes passed without a stop report, since the relay follows the report.
Or state in D4 and the Open risks that the stop is unannounced when the relay fails, and name what bounds it.

### [MEDIUM] `since` is not a lower bound and mis-times the report wait for a paused or failed child

**Location:** `_lyx/discussion/decision-record.md` D4 (lines 88, 92, 95)
**Issue:** `since` is "the time of the child's newest history entry", described as "a lower bound on how long the child has been in its state".
`shedengine` appends no history entry for a pause exit or a hard producer error (`internal/shedengine/run.go` `appendHistory` skips an empty outcome, and the pause exits persist `st.History` unchanged), and `Status` carries no state timestamp.
For a `paused` or `failed` child the newest entry predates the change, possibly by hours.
Then `now - since` is an upper bound on the time in the state, the 3 minutes "from `since`" have already passed so the notice goes out at once with `report none yet`, and an earlier stop's report written after that old entry passes "at or after `since`".
**Fix:** Take the change time from what batten observed: the status file's modification time on the check that saw the state differ (the value D3 already stats), and use it for the 3-minute wait, the report-time filter and the notice's `since`.
Say in D4 what the notice's `since` means when the history gives a different time.

### [MEDIUM] D3's modification-time baseline can miss a change written during the call's start

**Location:** `_lyx/discussion/decision-record.md` D3 (lines 65, 71)
**Issue:** The wait compares the status file's modification time with the one it began with and decodes only on a change.
D3 does not say when the baseline is taken.
The call first reads the status (decoding it) and then waits; a child write landing between that read and the baseline stat is already in the baseline, so the wait sees no change.
The row then waits on a stale state: no history entry, no state-changed notice, and the 30 s probes do not re-read the status.
The old per-bounce read healed this within 30 s, and nothing in D3 does.
**Fix:** State that the baseline modification time is stat'ed before the call's status read and that "differs" (not "newer") is the test, so a write after the read always shows.

### [MEDIUM] D7's one-invocation `remain-on-exit off` has no target that works

**Location:** `_lyx/discussion/decision-record.md` D7 (lines 156, 162)
**Issue:** D7 says the op addresses the new window "by the window id tmux returns" and also sets `remain-on-exit off` "for it in one multiplexer invocation".
The id exists only in the invocation's output, so the chained `set-option` cannot use it.
Run against tmux 3.6 on an isolated socket, `new-window -d -P -F '#{window_id}' -n batten:x CMD \; set-option -w remain-on-exit off` set the option on the session's current window (`@0`, the strands' window) and left the new window (`@1`) unset; reed treats `remain-on-exit` as a correctness dependency for the strands' window.
The same chain with `-t '=<session>:=batten:x'` set it on the new window only.
**Fix:** State the target of the chained `set-option`: the exact window name (the verb has already refused a live window of that name, so the name is unique), and say that the id is returned for later ops only.
Add that a reed test reads the option back on both windows.

### [LOW] D7's fallback voids the "never read as the strands' panes" claim

**Location:** `_lyx/discussion/decision-record.md` D7 (lines 157-159)
**Issue:** D7 says "A batten window, current or not, is therefore never read as the strands' panes", and one line earlier a session with no live Selvage pane falls back to the current window.
The Selvage pane is recreated through that fallback target, so with the batten window current the new Selvage lands in it and defines the strand window from then on.
The decision also leaves open that "the id reed state already records" is a pane id, so the window id is queried per op, and says nothing of the cost or of the ops that run outside an `Engine` (`TmuxCmd.listPanes` and `ListSessions` are engine-less).
**Fix:** Make the claim conditional ("while the Selvage pane is live") and state the bound for the fallback, for example falling back to the window of any recorded strand pane before the current window, or creating the Selvage pane in the session's first window.
State that the window id is resolved from the pane id at each op and that the seam is the one place `exactSessionWindowTarget` is replaced.

### [LOW] The not-queued retry has no end and its log lines are only half bounded

**Location:** `_lyx/discussion/decision-record.md` D4 (line 101), D6 (line 149)
**Issue:** A notice `QueueNotice` did not queue is retried "at most once per `notice_probe_s`" with one Warn per episode.
`QueueNotice` logs an Info line carrying the notice on every attempt with no strand recorded (`internal/orchengine/notice.go`), so a stop nobody can receive retries and logs every 30 s for as long as the child stays in its state.
D4's stated bound covers the Warn only.
**Fix:** Bound the retry: retry only after the orch state records a strand (the fact D6's marker already tests), or stop after a fixed number of attempts, and say which.

### [LOW] The watched marker proves the prime's orch, not the driver's parent

**Location:** `_lyx/discussion/decision-record.md` D6 (lines 143-144), A6 (line 197)
**Issue:** Batten notices go to the prime's orch (`orchengine.QueueNotice(orchcli.PrimePaths(location), ...)` in `internal/battencli/wire.go`), while the driver's parent is `ResolveParent`, the orch of the worktree the pair was created from.
For a batten started from the prime these are the same session; for one started from another worktree they need not be.
D6 silences the driver on a marker that proves a strand in the prime's orch state only.
A6 states the same-session assumption for D7 and not for D6.
**Fix:** Decide and state it: the marker is written only when batten runs in the prime (the case the Goal describes), or add A7 stating that the driver's parent is the prime's orch whenever the marker exists.

### [LOW] Driver liveness is not a cheap read

**Location:** `_lyx/discussion/decision-record.md` D3 (lines 76-77), D4 (line 88)
**Issue:** D3 runs "the driver liveness read while a done child or a pending notice waits" on every 2 s check, among the cheap reads.
`DriverAlive` in `internal/battencli/wire.go` loads reed's config and geometry and calls `reedengine.Status`, which lists panes through tmux; the done arm called it once per 30 s bounce.
D4's driver-ended trigger and the done arm's `Done` both need it, so a pending notice (up to 3 minutes) and a done child (up to the 900 s grace) now make a tmux round trip every 2 s.
The same applies to the not-parked resume retry, which D3 does not give a cadence.
**Fix:** Give driver liveness its own cadence in D3 (`notice_probe_s`, or a shorter named interval) and name the cadence of each read, the resume retry included.

### [LOW] History entries: the stated text, the acceptance criteria and the Goal disagree

**Location:** `_lyx/discussion/decision-record.md` D3 (lines 65-68), Acceptance criteria (lines 218-220), Goal (line 10)
**Issue:** D3's example output is `child running → blocked at Plan-Review`, with no time; the acceptance criteria require an entry "whose output names the change, with its time".
Consecutive state changes never fold, since each starts from the previous end state, so the time is unnecessary, but the two parts disagree.
A requested pause returns an exempt Stuck with a reason only: two pauses with no state change between them are equal consecutive entries and fold into one (`Repeats`), against "adding one history entry".
The failed-stat return also adds an entry that is neither a state change nor a pause, against the Goal's "nothing else".
**Fix:** Drop "with its time" from the acceptance criterion or add the time to D3's text, and give the pause return a distinct output or say a repeat folds.
Either count the failed-stat return in the Goal or say it is the one other entry.

## Focus verdicts

- Stops with no parent_notice: the PR-Gate and a Bouncer escalation without a notice get batten's notice (D4), and the Goal matches D6; the only unannounced awaiting stop is the one in F1.
- Notice timing: D3 now leaves a notice pending (derived from the status and the episode marker) and sends it in the wait of the call that begins in the new state, so the old "sent before the call returns for any reason" rule is gone; the `Done` return makes one last attempt and loses a notice not queued, stated in Open risks. Not a defect except F2's timing source.
- Notice destination: F6 and F7. Resumed and adopted drivers are stated in D6.
- Reed addressing: F4 and F5; `new-window` as a required verb is consistent with the probe (`requiredSubcommands`, `internal/reedengine/probe.go`), and `set-option` is already in it.
- Done notice: carries no report, so the teardown ordering is settled and the newest-file lookup is gone. Not a defect.
- Shed history: F9.
- Failed stat: ends the wait and the next read decides; an absent file reaches the spawn arm and an unreadable one is the hard error, as today. Not a defect beyond F9's entry count.
- Read cadence: F8.
- Cycle start in D2: `tickIdle` enters the handoff phase and then clears the pending request (`internal/orchengine/watcher.go`), so every request found later is newer, and D2 states no lower bound or older-request clause. Not a defect.
