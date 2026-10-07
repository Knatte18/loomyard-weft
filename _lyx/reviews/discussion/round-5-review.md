---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md:158
    summary: The watched marker tests where batten runs, not who the driver's parent is, so a pair created from another worktree and adopted or revived by a prime batten gets a silenced driver whose relay goes to a different orch than batten's notices
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:175
    summary: The strand-window seam claims to trust a recorded pane id only after the generation check accepted the state, but that check fails open on a probe failure and pane ids are server-wide, so a stale id can resolve to another session's window
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:106
    summary: The since value is held by the wait while the wait returns on every state change, pause and error, so its carrier across calls is unstated and a re-read can move it and the state-changed key
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:138
    summary: The quiet fallback counts from the child's newest history entry, a time D4 itself says is stale after a pause or producer failure, so a resumed child can get an immediate quiet notice
  - id: F5
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md:206
    summary: The Constraint pins exactly two refusals for the window verb, but the capability-probe refusal for new-window and a failed new-window name no way forward, and the read-back refusal's way forward leads to a lock refusal in the concurrent-call case it exists for
  - id: F6
    severity: NIT
    class: consistency
    location: _lyx/discussion/decision-record.md:227
    summary: Open risks restates the dead-batten, lost-destination and failed-relay bounds that D4 and D6 already state, in five entries for three facts
---

### [MEDIUM] The watched marker does not prove the driver's parent is the notice destination

**Location:** `_lyx/discussion/decision-record.md` D6 (lines 158-160), D4 (line 93), A7 (line 220).
**Issue:**
D4 and D6 rest on "a batten run from the prime: its notices and its driver's parent are the same orch".
The marker condition tests only where batten runs and whether the orch state records a strand.
A driver's parent is not where batten runs.
`hubgeom.ResolveParent` (`internal/hubgeom/parent.go`) names the orch of the worktree the pair was created from, read from the pair's origin record, and returns no parent when the origin or the shortname is missing.
The Run-Shed row creates, adopts and revives pairs, and a pair is hub-shared, so a prime batten can adopt or revive a pair that a batten started from a task worktree created.
A revive is a fresh driver launch, so `driverPrompt` renders again, finds the marker live and renders the silent rule, while the driver's parent is that other worktree's orch (or none).
Then a plain stop reaches the prime's orch through batten and the origin orch hears nothing, and an `awaiting` stop with a `parent_notice` is relayed to the origin orch while batten suppresses the prime's notice for up to 10 minutes.
D4's claim that the watched marker is "the only case where the driver's parent is the notice's destination" is false for that run.
The 10-minute fallback bounds the loss of the prime orch's notice only while the child is undecided, and nothing states that the silenced driver's other messages (the question, the friction-failed `done`) reach an orch that batten's notices never reach.
**Fix:**
State in D6 that the marker is also written only when the run's own parent, as `ResolveParent` returns it, is the prime's orch (the pair's origin is the prime), and removed otherwise.
Add the same condition to D4's suppression sentence and A7, and add an acceptance row for a run whose pair was created from a task worktree and is adopted by a prime batten (marker not written, driver keeps today's rule, awaiting notice sent at once).
Note: a pair with no resolvable parent has no relay at all, so it also gets the unwatched rule.

### [MEDIUM] The seam's trust premise for recorded pane ids is stronger than the generation check delivers

**Location:** `_lyx/discussion/decision-record.md` D7 (lines 173-178), Constraints and Acceptance (the seam rows).
**Issue:**
D7 says the seam "trusts a recorded id only after that check accepted the state", and that each op "resolves that pane's window id through the multiplexer", with "present in the multiplexer" as the test.
Two parts of that do not hold against `internal/reedengine/generation.go` and `spawn.go`.
`adoptPaneGenerationLocked` deliberately fails open: on a probe failure it logs and leaves the persisted ids as loaded, which its doc comment calls a two-sided trade it must not be tightened on.
A tmux pane id (`%n`) is server-wide and the hub's session shares one per-hub server, so an id resolved with a global target such as `display-message -t %5` can name a pane of another worktree's session; reed's own comments cite this as the R5-F4 failure.
The Decision names neither the fail-open case nor that the presence test and the window lookup must be scoped to the session.
The generation read itself does not depend on the ids it validates: its format is `#{session_id}|#{pid}|#{session_created}`, session fields only, so the focus question's second half is not a defect.
**Fix:**
State in D7 that a recorded id counts as present only when it appears in the session's own pane listing (a session-wide listing carrying each pane's window id), so a foreign or stale id never matches, and that an unvalidated state after a probe failure resolves the same way, with a miss falling to the next rule.
Replace "only after that check accepted the state" with that statement, and add an acceptance row for a recorded id that names a pane of another session on the same server.

### [LOW] `since` has no stated carrier across the calls of one batten process

**Location:** `_lyx/discussion/decision-record.md` D4 (line 106) against D3 (lines 60-84) and Q28.
**Issue:**
D4 says the wait holds `since` "for as long as it sees that state", so a rewrite in the same state does not move it, and Q28 says it is re-read from the file "when batten starts".
D3 makes the wait end on every state change, pause and error return, and leaves the notice pending across them.
The next call's wait begins in the same or the new state without having seen it at a check, so its `since` is either the call's baseline stat (the file's time at that instant) or a value carried between calls, and the Decision names neither.
If it is re-derived from the file, a same-state rewrite between two calls moves `since`, which moves the state-changed and driver-dead episode key (state plus `since`), so a notice already sent is sent again without a restart, and a report written before that rewrite stops qualifying (file time not at or after `since`).
The case D4 states as the only duplicate (a restart) is therefore not the only one.
**Fix:**
State where the episode's `since` lives while batten runs (in-process state that outlives a call and is dropped when the state changes), and that only a batten start re-reads it from the file.
Keep the acceptance row for a same-state rewrite and extend it to one that lands between two calls.

### [LOW] D5's quiet fallback uses a clock D4 declares unreliable

**Location:** `_lyx/discussion/decision-record.md` D5 (line 138) against D4's rationale (line 120) and Q19.
**Issue:**
D4 rejects the child's newest history entry as a time source because a pause or a producer failure appends no entry, so the time predates the change.
D5's no-live-run fallback counts `notice_quiet_min` from that same newest history entry and uses it as the episode key.
A child that was paused or failed and then resumed has a newest entry from before the stop; with no live shuttle run found yet (the driver is starting the next step), the 45 minutes may already have passed and a quiet notice fires at once.
**Fix:**
Count the fallback from the later of the newest history entry and `since`, and key the episode the same way, or state that the notice firing at once there is accepted.

### [LOW] The window verb's refusals: count pinned, one way forward misleading, probe refusal unnamed

**Location:** `_lyx/discussion/decision-record.md` Constraints (line 206), D7 (lines 179, 184-186).
**Issue:**
Verified against `contracts/specs/refusal-spec.md`: its `## Out of scope` section (line 248) lists batten refusals as out of scope and `PATTERN-refusal-way-forward` binds only modules with a section, so "no spec row lands" is correct.
Three parts do not hold up.
The Constraint says "both of the verb's refusals", a tally, yet D7 adds a third: `new-window` joins the capability probe "since the op cannot degrade", which is a refusal of this verb, and a failed `new-window` call is a fourth outcome; neither names a way forward or a test.
The read-back refusal names `lyx batten run <slug>` in a terminal, but the Decision's own cause for it is two calls sharing a name, in which case the first call's window holds the live run and a terminal run refuses on the run lock.
The Constraint's wording "a test reaches each" is met for the no-session refusal only by the acceptance row, and the read-back refusal's acceptance row says "leave one window and no dead one" without the refusal message.
**Fix:**
Reword the Constraint to cover every refusal of the verb, name the way forward of the probe and `new-window` failures (for example `lyx batten run <slug>` in a terminal), and make the read-back refusal name `lyx batten status <slug>` first, since the run may already be live.
Add the read-back refusal and its way forward to the acceptance row that covers two calls opening windows of one name.

### [NIT] Open risks repeats facts D4 and D6 already bound

**Location:** `_lyx/discussion/decision-record.md` Open risks (lines 227, 230-233).
**Issue:**
The dead-batten and lost-destination facts appear in D6's Bound (line 165) and again at lines 227 and 230, and the failed-relay delay appears in D4's Bound (line 100), at line 231 and at line 232.
Nothing in the repeats adds a requirement the plan writer needs, so deleting them loses nothing (completeness test passed: every requirement is stated in D4 and D6).
**Fix:**
Keep one Open-risks entry per fact: the dead batten with a lost destination (one entry), and the failed relay (one entry), and drop the others.

## Verdicts on the focus entries

1. Relay and routing (D4, D6): a failed or skipped relay from a live, reported driver cannot leave the orch never told while the child is undecided, because D4 states the 10-minute notice as its bound; that is not a defect.
   A batten whose destination differs from the driver's parent can, through F1.
2. tmux window name (D7): the claim matches tmux 3.6, which I ran on a private socket.
   Two `new-window -d -n batten:x` calls each chained with `set-option -w -t =sess:=batten:x remain-on-exit off` leave the older window `off` and the new one inheriting the global `on`, with no error.
   An untargeted chained option lands on the session's current window, as the support log says.
   A batten that exits at once under the chained option closes its window.
   The acceptance row for two calls in one moment covers the case; not a defect.
   Only the refusal text around it is F5.
3. Strand-window rule (D7): a present-but-dead Selvage pane is handled by the Decision (alive or corpse), and the generation read uses session fields only.
   The trust premise is F2.
4. Refusal spec: the out-of-scope claim is true; the refusal ways forward are F5.
5. `since`, episode keys and the baseline order: the baseline-before-read ordering plus the decode at every `notice_probe_s` check means a missed write is still read; that is not a defect.
   `since` has a stated source inside one wait but not across calls (F3), and D5's fallback uses another time source (F4).
