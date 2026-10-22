---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md D1 (Bound of condition 3, Consequences), D6, Open risks
    summary: No Decision bounds what the removal edges (D6 reclaim, start's relaunch RemoveStrand, Resume's relaunch kill, shuttle stop) may kill when condition 3 misreads a live engine as dead after the grace, as a non-exec wrapper script set as the claude binary or a Ctrl-Z suspended engine does
    fix: In D1, add to Bound of condition 3 the misjudged-live cases and their bound. State that tmux reports the foreground process group leader's argv[0], so an engine started through a shell wrapper that does not exec reads as the shell, and so does an engine suspended with Ctrl-Z. Then pick and record one bound, either a Constraint that the engine binary (claude.yaml `binary`, template comment in internal/shuttleengine/claudeengine/template.yaml) must be the engine or exec it, documented on that key, or a rule that condition 3 is skipped when the strand's launch command does not start a known non-shell program. In D6 and in D1's Consequences, state that every removal reading D1 liveness (D6 reclaim, start's StartRelaunch RemoveStrand, Resume, shuttle stop) kills a present pane that holds a bare shell, and that a pane misread under the residual cases is killed with its engine. Move the wrapper and suspended cases from an unbounded open risk into that stated bound.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lyx/discussion/decision-record.md D2 (Decision, Rationale), D6 (with D2's guard)
    summary: D2 rests on a false premise, because reed's strand removal already kills only panes the reed session lists, so a pane in another session is never killed today
    fix: Rewrite D2's Rationale to state today's guard, which is that killStrandPanes filters the kill list through paneIDsInSession over the reed session's listing (internal/reedengine/strand.go) and killReplacedOwnWindowPaneLocked kills only a pane the session lists. Keep as new only the server-wide lookup, the `session:window` placement on the status row and `strand_elsewhere`. Drop the claim that removal's kill-pane by id would end a pane in another session. In D6, replace "with D2's guard" by "with removal's existing session-membership guard".
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md D1 condition 2 and its second departure, D2
    summary: D1 says today's listings are scoped to the reed session, but a shared-window strand is listed only from the strands' window, so a pane moved to another window of the session reads absent and D2's server-wide lookup names its own session as elsewhere
    fix: In D1 condition 2, state that liveness lists every strand's pane across the whole reed session (the session-wide listWindowPanes listing presentPanes already takes), not only own-window strands' panes, so that a shared-window pane moved within the session reads present. In D2, define elsewhere as any session other than the reed session. Add an acceptance criterion that a shared-window strand whose pane was moved to another window of the reed session reads live and is not relaunched.
  - id: F4
    severity: MEDIUM
    class: decision
    origin: lens:walkthrough
    location: _lyx/discussion/decision-record.md D5 (withdraw refusal) and its acceptance criterion
    summary: Withdraw refuses on phase not idle with cycle_trigger requested instead of on whether the pending request has started, so a request made during a requested cycle cannot be withdrawn, and a reload after an earlier requested cycle is refused because CycleTrigger is never reset
    fix: In D5, key the refusal to the marker. Withdraw removes any pending marker, in any phase, because a cycle start clears the marker it consumes (internal/orchengine/watcher.go tickIdle) and a marker present during a cycle has not started. It answers withdrawn false with phase and cycle_trigger when no marker is pending. State that it never refuses for a stale CycleTrigger, since toIdle does not reset that field and startReload enters PhaseResuming without setting it. Update the acceptance criterion to cover a refresh written during a running requested cycle, which is withdrawn, and a withdraw during an auto-compaction reload.
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md D5 (race bullets)
    summary: D5's locked mode-and-time compare covers only the requested-cycle start, while the hard or soft start's ClearCycleRequest, the stale-request removal and removeSatisfiedRequest still read and then delete the marker unlocked, so a refresh answered requested true can vanish unrun
    fix: In D5, state that every watcher removal of the marker goes through one locked compare-and-remove of the mode and request time it read, under the orch state lock withdraw takes. That covers the cycle start of every trigger, the stale-request removal and removeSatisfiedRequest in internal/orchengine/watcher.go. State whether a hard or soft cycle start consumes a request pending at that moment. If it does, say so in place of "a request made during a hard or soft cycle is pending". If it does not, leave the marker for the next idle. Add an acceptance criterion for a refresh written between a hard start's read and its clear.
  - id: F6
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md D5 (distill and refresh note, held_by_pause)
    summary: The note that a request held by a hub pause waits for the resume is false for any pause longer than handoff_timeout_s (600 by default), because the first idle tick after the resume removes the marker as a stale cycle request
    fix: In D5, decide and record the request's lifetime under a pause. Either measure the stale age excluding time a live hub pause entry held the cycle machine, so the note stays true, or keep the timeout and change the note to name the expiry (that the request is dropped when handoff_timeout_s passes before the resume) and show it in cycle_request. Add the matching acceptance criterion for a pause longer than the timeout.
  - id: F7
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md D1 (Bound of condition 3, "a strand with none was launched before this change")
    summary: A strand with no launch time is not only one launched before this change, since old-binary processes, pinned by the pane prelude, keep launching strands after it, and an old binary that updates a strand in place drops the unknown key, so such strands get condition 3 with no grace from their first second
    fix: In D1, replace the premise. Either derive the grace from a fact every binary leaves, such as a launch timestamp reed already writes, or tmux's own pane start time if the installed tmux exposes one, verified against its manual. Or state the bound that a strand without a launch time is judged without grace, and say what an old-binary launch can then lose. Cite internal/state's rule that an element updated in place loses its unknown keys (internal/state/doc.go).
  - id: F8
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md D1 (A backend that answers an empty foreground command skips condition 3), Constraints (tmux fact)
    summary: The empty-answer bound is unverified for tmux, which by the reviewer's reading of its format code falls back to the pane's argv and then its shell when the foreground read fails, so a failed read would answer the shell and read dead
    fix: Research tmux's format_cb_current_command and osdep_get_name at the versions in use (3.4 on the host of issue 746, 3.6 on this host) and record in the Constraints fact what tmux answers when the foreground lookup fails. If it answers the pane's shell, replace the empty-answer bound with the bound that holds, or name the residual.
  - id: F9
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md D1 condition 3 (compared against the shell reed launched the pane with)
    summary: Two cases leave the issue 746 fix silently false-live, an rc file that execs another shell and a changed shell key while the attach pre-flight keeps the session's earlier pins, and neither is named
    fix: In D1, say which shell condition 3 compares against (the session's pinned default-shell read from tmux, or cfg.Shell), so a changed shell key does not misjudge old panes. Add the rc-exec case to Open risks as a stated false-live residual.
  - id: F10
    severity: LOW
    class: design
    origin: lens:existing-paths
    location: _lyx/discussion/decision-record.md D1 (Readers, Consequences)
    summary: Landing's conflictSessionStopper stops only live strands, so under D1 a conflict strand whose engine exited to the shell is skipped and its present pane and record stay, the same gap D1 closes for shuttle's stop
    fix: Add to D1's Consequences that landing's conflict-session stop (internal/loomcli/landingdeps.go conflictSessionStopper) removes a present shell-only strand, as shuttle's stop does, and add it to the acceptance criterion that names shuttle's stop.
  - id: F11
    severity: LOW
    class: design
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md D6 against the D1 grace
    summary: A holder whose engine exited at once still reads live inside the 30-second grace, so a fast retry of the same name is numbered and refused by seatengine again, which the record does not state
    fix: In D6's Bound, state that a holder inside its launch grace keeps the name, so a retry within 30 seconds of the holder's launch is numbered and refused as today.
  - id: F12
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md D7 (Creel's Strands skip)
    summary: The skip covers only a failed resolve, so a worktree removed after ResolveWorktree succeeds still fails the call at the wire or reed status step, short of the board's expectation that a mid-teardown lookup skips the worktree
    fix: In D7, extend the skip to a reed state read, wire or reed status failure of a worktree whose directory no longer exists when checked after the failure, and add that case to the acceptance criterion.
  - id: F13
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md Acceptance criteria (shell-only strand rows)
    summary: The acceptance criteria assert a shell-only strand reads dead with no Windows exception, while D1 skips condition 3 on Windows
    fix: Qualify the shell-only rows of Acceptance criteria as POSIX, and add that on Windows such a strand reads live as D1 states.
  - id: F14
    severity: LOW
    class: consistency
    origin: handler
    location: _lyx/discussion/decision-record.md Open risks (child-shell foreground group)
    summary: The open risk rests on the unevidenced claim that Claude Code's tool commands stay in its own process group, which the reviewer checked on this host
    fix: Record the evidence in the risk. On Claude Code 2.1.308 under tmux 3.6, ps over an agent pane's tty lists only the pane's bash and claude, with claude as the tty's foreground process group, and a running tool command is not on the tty. Name the source and versions.
  - id: F15
    severity: NIT
    class: consistency
    origin: handler
    location: _lyx/discussion/decision-record.md D1 (Departures from the operator decision, first bullet)
    summary: The departure's reasons restate the rejected engine-name alternative that support-log.md's Rejected alternatives already carries
    fix: Keep in D1 the departure itself and its requirement (condition 3 tests not the shell, and a pane running another program reads live), and drop the sentence about a Node-based or version-named binary, which stays in support-log.md's Rejected alternatives.
---

# Discussion review — round 1

### [BLOCKING] Removal edges are unbounded against a condition 3 misjudgment

**Location:** decision-record.md D1 Bound of condition 3 and Consequences; D6; Open risks

**Issue:** D1 makes four existing or new removal paths act on its liveness: D6's reclaim, start's StartRelaunch RemoveStrand, Resume's relaunch with killReplacedOwnWindowPaneLocked, and shuttle stop's widened removability.
The only bounds stated are the 30-second grace and an empty answer.
tmux reports the foreground process group leader, so a live engine reads as the shell in two cases.
One is an engine run through a non-exec `#!/bin/bash` wrapper, which the claude.yaml `binary` key allows.
The other is an engine suspended with Ctrl-Z.
In both, every one of those paths kills a live session after the grace.
Open risks names only the test-helper shell case, and no Decision states what the kills can hit.

### [MEDIUM] D2's kill guard already exists

**Location:** decision-record.md D2, D6

**Issue:** `killStrandPanes` narrows its kill list with `paneIDsInSession` over the reed session's listing (internal/reedengine/strand.go), and `killReplacedOwnWindowPaneLocked` kills only a listed own-window pane.
The Rationale's claim that removal would end a pane in another session is false, so a plan writer may build a second guard.

### [MEDIUM] Session-level placement does not hold for shared-window strands

**Location:** decision-record.md D1 condition 2, D2

**Issue:** `listStrandPanes` lists only the strands' window, and `presentPanes` adds session-wide panes only for own-window strands (internal/reedengine/ownwindow.go).
A shared-window pane moved within the session reads absent, so D2 sends it to the lookup, which finds it in the reed session itself.
Resume or `--if-absent` then launches a second engine, the outcome the departure says session-level placement avoids.

### [MEDIUM] Withdraw cannot remove a pending request during a requested cycle

**Location:** decision-record.md D5

**Issue:** A requested start clears the marker, so a refresh during that cycle is a fresh pending marker, yet withdraw refuses it.
Its way forward leads to the request running at the first turn end after idle, because a requested trigger waits for no quiet time.
`CycleTrigger` is also never reset by `toIdle`, so an auto-compaction reload after a requested cycle is refused as a requested cycle.

### [MEDIUM] Other marker removals escape D5's lock

**Location:** decision-record.md D5

**Issue:** `tickIdle` clears the marker for every trigger at cycle start, the stale sweep deletes after a read, and `removeSatisfiedRequest` reads then deletes during a cycle (internal/orchengine/watcher.go).
None of these is under the compare.
A refresh written in that gap answers `requested: true` and is dropped, which contradicts "a request made during a hard or soft cycle is pending".

### [MEDIUM] A request held by a pause expires at the resume

**Location:** decision-record.md D5

**Issue:** `pauseTick` holds every tick before `tickIdle`, and `tickIdle` removes a marker aged past `handoff_timeout_s` (600 by default in internal/orchengine/template.yaml).
A distill under an hour-long pause is told it waits for the resume, and is then dropped at the resume.

### [MEDIUM] A missing launch time does not mean an old launch

**Location:** decision-record.md D1 Bound of condition 3

**Issue:** The pane prelude pins `lyx` to the binary that spawned the pane, so old-binary runs keep adding strands with no launch time after this change lands.
internal/state drops unknown keys from a slice element updated in place, so an old binary's update strips the key from a new strand.
Such strands are judged without grace from their first second.

### [LOW] The empty-answer bound is unverified on tmux

**Location:** decision-record.md D1, Constraints

**Issue:** tmux's manual says only "Current command if available".
The reviewer's reading of tmux's format code, not checked against local source, is that a failed foreground read falls back to the pane's argv and then its shell.
That would make the answer never empty, and a failed read would count as dead.

### [LOW] Two false-live cases are unnamed

**Location:** decision-record.md D1 condition 3

**Issue:** An rc file that execs another shell leaves that shell in the foreground after the engine exits.
A changed `shell` key compares old panes against a shell they do not run, because the attach pre-flight keeps the session's earlier pins.

### [LOW] Landing's conflict stop skips a shell-only strand

**Location:** decision-record.md D1

**Issue:** `conflictSessionStopper` in internal/loomcli/landingdeps.go skips a strand that is not live, so under D1 a shell-only conflict strand keeps its pane and record.

### [LOW] D6 cannot reclaim inside the grace

**Location:** decision-record.md D6

**Issue:** A holder younger than 30 seconds reads live, so a fast retry of that name is numbered and refused by seatengine, and the record does not say so.

### [LOW] The worktree skip misses a teardown after the resolve

**Location:** decision-record.md D7

**Issue:** After a successful `ResolveWorktree`, a removed directory fails the call at the wire or status step, because D7 skips only a failed resolve.

### [LOW] Acceptance criteria omit the Windows skip

**Location:** decision-record.md Acceptance criteria

**Issue:** The shell-only rows hold only where condition 3 runs, and D1 skips it on Windows.

### [LOW] The child-shell risk lacks its evidence

**Location:** decision-record.md Open risks

**Issue:** The claim about Claude Code's process group cites no source or version.
It holds on this host: on Claude Code 2.1.308 and tmux 3.6, `ps -t` over an agent pane lists only bash and claude, with claude as the tty's foreground group.

### [NIT] A departure restates a rejected alternative

**Location:** decision-record.md D1 Departures

**Issue:** The reason about Node-based or version-named binaries is the engine-name alternative's rejection, which support-log.md already holds.

## Rejected

- bound-breaker's D4 finding, that `EventWaiting` counting as a turn end lets the gate pass mid-turn: `EventWaiting` is a turn end with background work outstanding (internal/shuttleengine/engine.go), not a point inside a turn. Today's `written` reply also moves the phase on with forks running, so D4 adds no new exposure.
- bound-breaker's note that D2's lookup repeats on every poll for a dead but bound strand: a cost the record bounds to strands absent from the session, not a defect.
- shuttle run's wording of a dead pane for a present shell-only pane (existing-paths, folded into its finding 4): it is message text only and changes no decision.
