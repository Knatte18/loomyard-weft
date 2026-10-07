---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md:D4 and D6
    summary: "An awaiting stop with no parent_notice (every PR-Gate hand-off, and a Bouncer escalation whose notice failed to render) reaches the orch through no channel: D4 excludes awaiting from batten notices and D6 relays only a non-empty parent_notice."
  - id: F2
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:D3 and D4
    summary: "D3 promises a pending notice is sent before the wait call returns for any reason, but the call returns the moment the state changes, before D4's report (up to 3 minutes) exists; and a Done return leaves a not-queued notice with no later check."
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:D6 and D4
    summary: "D6 silences the driver on proof of a live batten, not of a notice destination; with no orch strand recorded the stop is announced nowhere, the not-queued retry runs every 2 s with a log line per attempt, and the render rule is frozen for a driver's whole life."
  - id: F4
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:D7
    summary: "D7 addresses reed's strand window by id but does not say where the id comes from, and names only enumeration, layout and split although window-size, resize-hook, generation and attach-geometry ops also target the session's current window."
  - id: F5
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md:D7 and Open risks
    summary: "D7 leaves the batten window's lifecycle partly open: remain-on-exit off applied after creation can race a fast batten exit, a lock-refused second batten reports success then vanishes, and a reed session end kills battens that survived it before."
  - id: F6
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md:D3
    summary: "A failed stat of the child's status file reads as unchanged with no bound, so a missing or unreadable file waits silently forever, where today's per-bounce ReadStatus found it absent and respawned."
  - id: F7
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:D3 and Acceptance criteria
    summary: "The history entry cannot name the change as D3 and the acceptance criterion say: a history entry persists only the pointer's path, not its reason, and entries with equal output fold, so consecutive state changes would merge into one entry."
  - id: F8
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md:D3
    summary: "D3 puts driver liveness among the probes run at most once per notice_probe_s, yet the done arm's return on driver end and D4's driver-ended trigger need it on the 2 s check; the cadence of each read is not stated."
  - id: F9
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:D4
    summary: "A done notice carries a report path into the task worktree that Worktree-Teardown removes right after Done, so the orch cannot read it; the done-only newest-file lookup and its 3-minute wait serve a path that is dead on arrival."
  - id: F10
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:D2
    summary: "D2 says a request older than the cycle start is left for the idle phase as today, but the cycle's entry clears any pending request unconditionally, so no such request exists and the lower bound on RequestedAt is a no-op."
---

## Findings

### [BLOCKING] An awaiting stop with no `parent_notice` is announced nowhere

**Location:** `_lyx/discussion/decision-record.md`, D4 ("An `awaiting` child gets no state-changed notice") and D6 (the driver sends only an `awaiting` envelope's `parent_notice`, or a question).
**Issue:** D4's rationale and the support log's Q12 rest on the premise that `parent_notice` is the one channel for an awaiting stop.
Only the Bouncer sets it (`internal/shedadapters/bouncer.go` `escalate`), and a failed render there degrades to "the plain Reason with no notice".
The landing PR-Gate returns `shedengine.Awaiting` with only a `Reason` (`internal/landingshed/prgate.go` `awaiting`), so the common approve/reject hand-off carries no `parent_notice`.
Today the driver's generic escalation line covers that stop (`## Notifying the parent`: "At every escalation ... one short SendMessage").
Under D4 and D6 a watched driver stays silent on it (it is neither a `parent_notice` nor a question) and batten sends nothing, so the orch never hears that a pull request waits for its decision.
The Goal's "the driver messages the orch only with a question or the awaiting hand-off" is not what D6 specifies.
**Fix:** In D4, batten's state-changed notice covers an `awaiting` child whose status carries an empty `parent_notice` (batten already reads the child's status), with the stop report path as for any parked stop; an `awaiting` child whose status carries a non-empty `parent_notice` stays the driver's relay alone, so one message per event holds.
Say in D6 that the generic escalation line goes for a watched driver because batten's notice replaces it.
Add an acceptance criterion for an awaiting child without `parent_notice`, and correct Q12 and the Rejected alternatives entry for awaiting in the support log.

### [MEDIUM] "Sent before the call returns for any reason" cannot hold with the 3-minute report wait

**Location:** `_lyx/discussion/decision-record.md`, D3 ("sent at the latest before the call returns for any reason") against D4 ("once the driver's stop report for that stop exists, or the driver strand has ended, or 3 minutes after the change").
**Issue:** The wait call returns the moment the child's state differs from the state it began with.
At that moment the report does not exist, the driver is alive and less than 3 minutes have passed, so D4's send condition is not met.
Either the call holds its return for up to 3 minutes, which defeats the history entry and the pause response within one check, or it sends early without a report, which defeats the 3-minute wait, or the notice outlives the call, which the text does not allow.
Separately, when the done arm returns `Done` and batten ends, a notice `QueueNotice` did not queue is "retried on the next check", but no check follows.
**Fix:** State that the notice is pending across calls, derived from the child's status and the episode marker of sent notices, and sent by the wait of the call that begins in the new state, with D4's 3-minute clock running from `since`, not from the call's start.
Restrict the "before the call returns" rule to the returns the notice's own trigger has met, and say a pause or error return leaves the notice pending.
For the `Done` return, state one final send attempt and that a notice not queued then is logged once and lost, and add it to Open risks.

### [MEDIUM] D6 silences the driver without a notice destination; the retry has no rate and the render rule is frozen

**Location:** `_lyx/discussion/decision-record.md`, D6 (Bound), D4 (not-queued retry) and Open risks.
**Issue:** The watched marker proves a live batten, not that its notice has somewhere to go.
`QueueNotice` returns not-queued when the orch state records no strand (`internal/orchengine/notice.go`), and the driver's parent is the pair's recorded origin parent (`internal/loomcli/wiring.go` `parentName`), which need not be the prime's orch (A6 assumes it is).
A batten started with no orch strand recorded makes the driver silent and the notice a log line: the stop reaches nobody, with no bound stated.
D4 retries a not-queued notice "on the next check", every 2 s for as long as no orch strand exists, and `QueueNotice` logs an Info line on each attempt; the "one Warn per episode" does not cap those.
The render is also decided once: `resumeParkedDriver` (`internal/loomcli/driverresume.go`) types one line into the live driver and never re-renders, so a driver launched unwatched and later adopted by batten keeps today's rule (a duplicate message, not a loss), and one launched watched keeps the silent rule whatever batten does afterwards.
**Fix:** In D6, batten writes the watched marker only while its notices have a destination (the orch state records a strand when the marker is written or refreshed), and Open risks names the residue (the orch strand disappearing after the render).
In D4, a not-queued or failed notice is retried at most once per `notice_probe_s`, so the log line rate follows that interval.
In D6's Bound, state that the rule is fixed at the driver's fresh launch and a resumed or adopted driver keeps the rule it was launched with.

### [MEDIUM] Reed's strand-window addressing has no source for the id and an incomplete op set

**Location:** `_lyx/discussion/decision-record.md`, D7.
**Issue:** D7 makes reed address "the strand window by its id", but reed state records no window id and the engine creates its session with no id kept (`internal/reedengine/state.go` has no window field), so the plan writer has no source: a recorded id, or one derived from the Selvage pane or a bound strand pane, with a stated answer for a session that predates the change, a dead Selvage, or a recreated session.
D7 names enumeration, layout and split, but seventeen sites address the current window through `exactSessionWindowTarget` (`list-panes`, `select-layout`, the attach argv, and the `display-message` and `show-options` reads in `windowsize.go`, `reapply.go`, `generation.go` and `lifecycle.go`), including the resize-pin hook and the window-size readbacks.
A batten window selected by hand makes those read the wrong window.
`new-window` and a per-window `remain-on-exit` are new verbs outside `requiredSubcommands` (`internal/reedengine/probe.go`), and `doc.go` calls every named verb required, so a multiplexer without them fails deep inside the op, not at the capability probe.
**Fix:** In D7, name how the strand window id is obtained and what happens when none can be (for example derived from the Selvage pane's window, falling back to today's current-window target), state that every window-targeted op goes through that one addressing seam, not only the three named, and say whether `new-window` joins the required verbs or the op degrades with a refusal.

### [LOW] The batten window's lifecycle is partly open

**Location:** `_lyx/discussion/decision-record.md`, D7 and Open risks.
**Issue:** Reed sets `remain-on-exit on` globally (`internal/reedengine/lifecycle.go`), and D7 sets `off` for the window after creating it; a batten that exits at once (the run lock refused) can die before the option is set and leave a dead pane under the global `on`.
D7's second-batten bound says the window "closes", yet the verb has already answered success naming that window, and with `remain-on-exit off` the lock refusal's output vanishes with it.
The Open risks name only a tmux restart as killing battens, but ending the reed session kills the battens in its windows, which the `setsid nohup` start survived.
**Fix:** State that the window is created and its option set in one multiplexer invocation, state that the verb's envelope reports a started window only and that the lock refusal is read from batten's log, and add the session end to the Open risks entry.

### [MEDIUM] A failed stat of the child's status file waits forever

**Location:** `_lyx/discussion/decision-record.md`, D3 ("A failed stat reads as unchanged for that check").
**Issue:** The new wait compares the status file's modification time and decodes only on a change.
A stat that keeps failing (the file absent, as when the pair is removed or a checkout replaces it, or unreadable) reads as unchanged on every check, with no return, no notice and no bound.
Today every bounce calls `ReadStatus`, whose not-found result sends the row through its spawn arm.
A decode failure is a hard error while a missing file is a silent wait, with no stated reason.
**Fix:** State that a failed stat ends the wait: the call returns a budget-exempt `Stuck` and the next call's `ReadStatus` decides, so an absent file is handled as today and an unreadable one is the hard error a decode failure is.
Add the case to the acceptance criteria.

### [MEDIUM] The history entry cannot "name the change" as written

**Location:** `_lyx/discussion/decision-record.md`, D3 ("the returned budget-exempt `Stuck`'s output names the change") and the acceptance criterion "adds exactly one budget-exempt entry naming the change, with its time".
**Issue:** Shed persists only `Producer`, `Outcome`, `Output` (the pointer's path), `At` and the gate attempts in a history entry (`internal/shedengine/run.go` `appendHistory`); the `Reason` is not stored, and D3 holds `shedengine` unchanged.
Entries fold into the previous one when both are budget-exempt `Stuck` with equal producer and output (`foldsInto`), so with an empty output two consecutive changes (running to blocked, then blocked to running) become one entry with `Repeats` 2.
"The history records the child's state changes" would then fail, and the entry would not name the change.
The pause-request return also adds an entry, which the Goal's "nothing else" omits.
**Fix:** State that the change text rides in the pointer's output field, so each distinct change is its own entry and the entry names it, that the reason mirrors it for `lyx batten status`, and that the Goal and acceptance criterion count a requested pause as the one other entry.

### [LOW] The cadence of each read in the wait is not stated

**Location:** `_lyx/discussion/decision-record.md`, D3 ("The probes of D5 (driver liveness, agent activity, wait markers) run at most once per `notice_probe_s`").
**Issue:** The done arm returns `Done` once the driver strand ends, and D4's trigger "the driver strand has ended" needs the same liveness read; at one read per 30 s the first lags the acceptance criterion, and at one per 2 s the D3 rule is wrong.
The decision-file read and the report-file stat have no stated cadence either.
**Fix:** State that the cheap file reads (status stat, pause flag, decision record, report stat) and driver liveness while a done child or a pending notice waits run on every check, and that only the running child's liveness, activity and wait-marker probes follow `notice_probe_s`.

### [MEDIUM] A done notice's report path is dead on arrival

**Location:** `_lyx/discussion/decision-record.md`, D4 (the done-child "newest file in the run's drive-reports directory" and `report <path>`).
**Issue:** After the done notice, batten's next row is `Worktree-Teardown`, which removes the task worktree (`internal/battenshed/teardown.go`), and the drive-reports directory lives under that worktree's `_lyx`.
The orch reads the notice after the path is gone.
The done-only lookup, its file-time rule and the 3-minute wait for a done child's report serve a path nobody can open.
**Fix:** Drop `report` from a done notice, along with the newest-file lookup and the 3-minute report wait for a done child, and keep `report` for the parked stops (blocked, paused, failed, awaiting).
A done notice then goes out on the done call's first check, which also removes the done arm's part of the ordering problem.

### [LOW] D2's "older than the cycle start" clause describes a state that cannot occur

**Location:** `_lyx/discussion/decision-record.md`, D2 ("one older than `CycleStartedAt` ... left for the idle phase as today").
**Issue:** `tickIdle` calls `ClearCycleRequest` right after entering the handoff phase, so any request pending at the cycle's start is gone before the cycle runs, whichever trigger started it.
Every request that exists later was written after that clear, which makes the `RequestedAt >= CycleStartedAt` bound a no-op guard and the "as today" clause false; a plan writer may preserve older requests across the entry to match it.
**Fix:** Reword D2 so the lower bound is stated as an invariant of the entry's own clear, drop the "older than `CycleStartedAt`" clause, and keep the upper bound and the mode test.

## Questions the directive named, settled without a finding

- D2 (the pending-request test, directive's D3): `CycleStartedAt` is separate from `PhaseEnteredAt`, which `enter` overwrites on each phase, and the compact test (at or after the start, not after the boundary) removes exactly the requests the finished compaction covered; not a defect beyond F10.
- D6 marker versus the seed (directive's D7): the marker is a separate file under the run's scratch directory, so `WriteSeed`'s write-once equality check is untouched; not a defect.
- D5 provider text in the notice (directive's D6): `oneLine` replaces control and line-separator characters and the record cuts the text to 200 characters under the fixed prefix, which `QueueNotice` accepts; not a defect.
- D5 stalls (directive's D6): unreadable transcripts, several live runs, dead pids and a busy or hung run each have a stated limit; not a defect.
- Awaiting producing one message or two (directive's D7 versus D5): D4 excludes awaiting, so never two; the zero-message case is F1.

## Focus departures

- The directive's entries name Decisions by the numbering before the burler round 1 withdrawal (its D3 to D8 are the record's D2 to D7).
  I matched each entry to its subject, not to the label, and judged the record as it now stands; I departed from no directive's substance.
- The directive's D5 entry on "the stop report path and the since value tied to the stop": the record's file-time rule ties both to the stop, and I record the one gap I found there (the done path, F9) beside it.
