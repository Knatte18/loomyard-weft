---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md:67-75 (Decision 6 confirmation list) and :146 (its acceptance criterion)
    summary: Decision 6 confirms over tender status session states but names still_busy and refused, which are pause-report categories no status reading returns, and leaves the unknown state (a failed or unsettled reading) neither withholding nor excluded, so a session mid-call that reads unknown passes confirmation; the orch's reads idle matches no state name either.
    fix: In Decision 6, replace the first two confirmation bullets with one rule over the shuttleengine state names tender's status reports for every pair and prime session, the orch included - confirmation holds only when each reads idle-done, idle-stalled, idle-fresh, asking or dead, so busy (any cause, background included) and unknown withhold it - and drop still_busy and refused from the re-read wording, saying the pause report's own still_busy and refused lists only feed the first reading. Rewrite the acceptance criterion at line 146 to the same list, adding a row that a session reading unknown withholds confirmation.
  - id: F2
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md:76 (Decision 6 pre-hibernate check) and :84 (Decision 7, no owned entry)
    summary: When a hub pause was already live the daemon owns no entry, and neither the confirmation reading nor the pre-hibernate check requires any live hub entry, so an operator lyx resume of that pause during the ten-minute window or after confirmation lets sessions start turns and the hibernate still fires.
    fix: In Decision 6, add to the confirmation conditions that a live hub-scope entry stands in the pause record at that reading, and change the pre-hibernate check to require a live hub-scope entry, owned or not, ending the request cancelled when none stands. Extend the acceptance criterion at line 150 so an operator lyx resume of a pre-existing hub pause before the hibernate call also ends the request cancelled with no hibernate.
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:101-103 (Decision 10) and :156 (its acceptance criterion)
    summary: Decision 10 rests on a false premise and misses a verb - loginctl has no sleep verbs (systemd 259 on the operator's machine lists none), while systemctl sleep, which puts the machine to sleep, is not among the four denied verbs.
    fix: In Decision 10, drop loginctl and deny systemctl with the verbs sleep, hibernate, suspend, hybrid-sleep and suspend-then-hibernate; in the acceptance criterion at line 156 replace loginctl hibernate with systemctl sleep.
  - id: F4
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:103 (Decision 10 bound)
    summary: The sleep deny's bound names what it refuses but not how it matches or what passes - whether flags between systemctl and the verb (systemctl --no-wall hibernate) match, which wrapping prefixes count, and that bash -c, eval, a variable, systemctl start or isolate of a sleep target and other sleep routes such as busctl or rtcwake pass.
    fix: Rewrite the Decision 10 bound to say the deny matches systemctl in command position through the deny table's shared command-position prefix (path-prefixed, after a separator, or after env, exec, xargs, sudo or time), with any flag words between systemctl and a denied verb, and that it is a guardrail, not a barrier - bash -c, eval, a shell variable, systemctl start or isolate of a sleep target, and other sleep routes such as busctl, dbus-send and rtcwake pass.
  - id: F5
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:47-50 (Decision 3) and :60-62 (Decision 5)
    summary: The hidden lyx hibernate watch verb, which pauses the hub and runs systemctl hibernate, has no caller check and no stated behavior when run directly, so an agent can reach the daemon's effects past the request refusal, or start a second daemon beside a live one.
    fix: In Decision 3, add that watch refuses an agent caller as request does. In Decision 5, add that watch acts only for a non-terminal request whose recorded daemon pid and start time are its own, and otherwise exits without pausing, hibernating or writing the request file; add an acceptance criterion that watch run with no request, or beside a live daemon, does nothing.
  - id: F6
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:56-57 (Decision 4) and :91 (Decision 8)
    summary: Decisions 4 and 8 branch on a live or dead daemon without naming the liveness test, and Decision 4 records only the pid in the locked hold although the request file holds pid and start time, so a reused pid could make a dead daemon read live.
    fix: In Decision 4, say request records the spawned daemon's pid and start time in the same lock hold, and define a live daemon as a running process with the recorded pid whose start time equals the recorded one; Decision 8 and status use that test.
  - id: F7
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md:68 (Decision 6 pause call)
    summary: Decision 6 runs tender's pause engine but does not name the grace it passes, which sets how long busy sessions get before the interrupt and so how much of the ten-minute confirmation window remains.
    fix: In Decision 6, state the grace the daemon passes to the pause engine (for example tender's default of ten seconds), and add it to the fixed-in-code line under Auto-mode assumptions.
  - id: F8
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:141-157 (Acceptance criteria)
    summary: Several refusals and terminal paths in the Decisions have no criterion a test could pin - --after with --after-all refused (D2), systemctl missing from PATH refused at request (D9), a batten status file that goes missing ending failed (D5), and a request over a terminal request replacing it (D4).
    fix: Add one acceptance criterion per item - --after with --after-all is refused; request refuses when systemctl is not on PATH; a waited-on status file that disappears ends the request failed with the reason; a request over a done, failed or cancelled request replaces it.
  - id: F9
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:58, :88, :104, :137
    summary: Deliberation narrative sits in decision-record.md - Decision 4's reasons for departing from the board, Decision 7's rationale (restating the support log's rejected alternative of leaving the hub paused), Decision 10's history of the 2026-10-13 ad-hoc hibernate, and the same date in the polkit open risk.
    fix: In Decision 4 keep only that the request is hub-scoped, departing from the board's flag on the machine, and move the since-clause reasons to support-log.md Rejected alternatives under the machine-level request entry; delete the Rationale lines of Decisions 7 and 10, which the support log already carries (add the 2026-10-13 incident to the support log's interview step 1 if wanted); in the polkit open risk drop the clause about 2026-10-13.
---

### [BLOCKING] Confirmation names states status never reports and lets `unknown` through

**Location:** `_lyx/discussion/decision-record.md:67-75`, `:146`
**Issue:** Decision 6 says the unconfirmed pause is re-read from tender's status session states.
`tenderengine.Status` reports each session's `shuttleengine.SessionStateName`: `busy`, `idle-done`, `idle-stalled`, `idle-fresh`, `asking`, `dead`, `unknown`.
`still_busy` and `refused` exist only in `PauseReport` (`internal/tenderengine/pause.go`), filled once by the pause sequence, so the re-read cannot evaluate them.
`unknown` is what a failed reading or unsettled evidence yields; the list neither withholds on it nor excludes it, so a session the reader cannot see into passes confirmation, the exact mid-call case the task exists to prevent.
"The orch reads idle" matches no state name; three idle states and `asking` exist.

### [BLOCKING] No live-pause requirement when the daemon owns no entry

**Location:** `_lyx/discussion/decision-record.md:76`, `:84`
**Issue:** Decision 7 has the daemon own no entry when a hub pause was already live.
Decision 6's pre-hibernate check tests only "its owned entry, when it has one", and confirmation never requires a live hub entry.
An operator `lyx resume` of the pre-existing pause at any point before the call lifts the only thing holding sessions still, and nothing in the sequence notices; the hibernate fires over running sessions.
The acceptance criterion at line 150 pins the cancel only for the owned case.

### [MEDIUM] `loginctl` has no sleep verbs; `systemctl sleep` is undenied

**Location:** `_lyx/discussion/decision-record.md:101-103`, `:156`
**Issue:** `loginctl --help` on this machine (systemd 259) lists no hibernate, suspend or sleep verb, so half the deny targets a command that does not exist and the acceptance criterion pins a non-command.
`systemctl --help` lists `sleep` ("Put the system to sleep") beside the four named verbs, and Decision 10 lets it pass.

### [MEDIUM] Sleep deny's bound omits its match shape and its bypasses

**Location:** `_lyx/discussion/decision-record.md:103`
**Issue:** The focus asks how a prefixed or wrapped command is treated.
The deny table's existing patterns build on `commandPositionPattern` in `internal/shuttleengine/claudeengine/settings.go`, which covers path prefixes, separators and the `env`, `exec`, `xargs`, `sudo`, `time` prefixes, and each existing deny's doc names what passes (`bash -c`, `eval`, a variable).
Decision 10 states none of this, nor whether flags may sit between `systemctl` and the verb, nor the other sleep routes it leaves open.
The rubric's attack-surface rule requires the bypassable guard's bound.

### [MEDIUM] Hidden `watch` verb bypasses the caller check

**Location:** `_lyx/discussion/decision-record.md:47-50`, `:60-62`
**Issue:** Decision 3 checks callers on `request` and `cancel` and exempts `status`; it says nothing of `watch`, the verb that carries every effect.
Decision 5 does not say what `watch` does when started outside `request`: with no request file, or beside a live daemon.
Answering the focus on Decision 3: yes, an agent can reach the daemon's effects through `lyx hibernate watch`.

### [LOW] Daemon liveness test unnamed

**Location:** `_lyx/discussion/decision-record.md:56-57`, `:91`
**Issue:** The request file holds pid and start time, and the codebase's liveness checks (`pauserecord`'s net-op markers, `parentlock`, `pairteardown`) compare both, but Decisions 4 and 8 never say "live" means that, and Decision 4's lock hold records the pid alone.
A replacing request that misreads a live daemon as dead spawns a second one.

### [LOW] Pause grace unstated

**Location:** `_lyx/discussion/decision-record.md:68`
**Issue:** `tenderengine.PauseRequest.Grace` is required input; `lyx pause` defaults to ten seconds and caps at sixty.
The plan writer has to pick it.

### [LOW] Acceptance criteria miss stated refusals and terminal paths

**Location:** `_lyx/discussion/decision-record.md:141-157`
**Issue:** The `--after`/`--after-all` exclusivity, the `systemctl`-on-`PATH` check, the vanishing status file and the replace-over-terminal path are each stated in a Decision with no criterion.
No criterion asserts behavior absent from the Decisions.

### [LOW] Deliberation narrative in the decision record

**Location:** `_lyx/discussion/decision-record.md:58`, `:88`, `:104`, `:137`
**Issue:** The requirement each carries is already stated in its Decision (hub scope, lift on wake, the deny itself); the remaining text is justification and history, which `Plan-Write` does not need and the support log already holds or can hold.

## Rejected

- Decision 2's "past Land" (focus): the batten recipe's rows are Worktree-Create, Seed-Child, Run-Shed, Land, Worktree-Teardown (`internal/battenrecipe/names.go`), so past Land is `current_producer` at Worktree-Teardown, derivable from the recipe order with no invented ordering. Not a defect.
- Decision 7's lift by identity (focus): an operator's re-added entry carries caller `EXTERNAL` or the orch's strand name, never `hibernate`, and `paused_at` is a nanosecond `time.Time`, so it cannot match the owned identity. Not a defect.
- Decision 7's five-minute wake window (focus): every path out of it lifts the owned entry and notifies the orch, and the cold-boot case is an open risk with `cancel` as its way out. Bounded; not a defect.
- Decision 8's cancel bound (focus): it states what a cancel can lift (only the owned entry, by identity) and the one-file-read miss before the call. Adequate once F6 names the liveness test.
- Decision 6's window between the last check and the call (focus): the pre-hibernate check and the call are back to back, and Decision 8 states the miss; the uncovered part is the missing live-entry requirement, raised as F2.
