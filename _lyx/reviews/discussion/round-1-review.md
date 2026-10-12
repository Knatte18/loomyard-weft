---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md (D7, D3)
    summary: D7's PATTERN rule says an interrupt key goes only through the guarded interrupt, but Runner.Inject stays an exported unguarded key path that can carry Escape, and no Decision bounds it
    fix: In D7, state what becomes of `Runner.Inject`, which skips the ready-pane check by design and today delivers any key, Escape included. Either the PATTERN entry names Inject as the one exempt raw key path and forbids it the provider's interrupt key (so `creel`'s inject-with-interrupt reaches the Esc through the guarded `Interrupt` only, as D3 already says), or Inject refuses an input list equal to the engine's interrupt sequence. Add the chosen bound to D7's text and one Acceptance criterion pinning it.
  - id: F2
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md (D4, D5 still_busy and interrupted)
    summary: D5 defines still_busy by the pane (a turn still on screen) while D1/D4 classify by the lagging session state, and D4 settles only before the read after the first Esc, so a turn the second Esc stopped reads still_busy
    fix: In D4's pause-engine bullet, add that after the second Esc the engine again waits the settle time through `Deps.Sleep` before its final reading, and name which reading decides the list (the running-turn predicate on the state, or the pane through the guard's capability). Reword D5's `still_busy` and `interrupted` to name that same reading, and make Acceptance criterion 4 say the classification follows the settle after the second Esc.
  - id: F3
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md (D6, Acceptance criteria item 10)
    summary: D6 and the pre-change-entry criterion quote the guard's no-input-box reason as the pane shows no input box, but shuttle's readiness reason is no input box is on screen; a menu or picker may be open
    fix: In D6's example list and in the Acceptance criterion about a pane with the picker open, replace the quoted text with the guard's actual reason, "no input box is on screen; a menu or picker may be open", or drop the literal and say the guard's no-input-box reason; D6's scope does not include rewording shuttle's readiness reasons.
  - id: F4
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md (D4, orch watcher bullet)
    summary: The orch is recorded in the entry before its first Esc, and the watcher's existing entry-owns-the-orch short-circuit then settles the entry on the next tick, so D4's later-tick retry never runs unless D4 orders the first-Esc memory ahead of it; the restart bound is also unstated
    fix: In D4's orch-watcher bullet, add that the remembered first-Esc time is consulted before the check that settles an entry already owning the orch, and that the entry is marked handled only after the second Esc or the settled re-probe. Add the bound that a watcher restarted during the settle has lost that memory, finds the orch recorded, and sends no second Esc.
  - id: F5
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md (D3, D5 rationale, Open risks, Acceptance criterion 9)
    summary: The orch watcher treats ErrNoRunningTurn as an idle orch with no log, so a reworded running-turn hint silently withholds every orch Esc, against D5's rationale of keeping a guard that withholds every Esc visible
    fix: In D3 or D4's orch-watcher bullet, state that a guarded refusal is logged once per hub entry at Info (not Warn) naming the orch strand, and extend the Open risks item about the running-turn needle to say the orch's case shows in that log line rather than in the pause report. Adjust Acceptance criterion 9 to expect the Info line and no Warn.
  - id: F6
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md (D1)
    summary: D1 says the shared predicate keeps the pause's and send readiness's readings from drifting, but readiness applies it to a state adjusted by the unmatched turn start and its hold while the pause reads the raw folded state, so the two still differ
    fix: Reword D1's claim to say the two share the state rule, not the whole reading. Add to Open risks that a session whose folded state reads idle while its pane shows a running turn (an unmatched turn start) gets no Esc and lands in `already_idle`, and that a busy/turn state whose hold released reads running to the pause and reaches the guard, which withholds the Esc into `no_running_turn`.
---

The review judged `decision-record.md` and `support-log.md` against the Discussion-Review rubric, the writer stencil's gathering bounds, PATTERN, and the code the record describes: `internal/pauseengine` (`pause.go`, `resume.go`, `report.go`), `internal/orchengine/pause.go`, `internal/pausecli/deps.go`, `internal/shuttleengine` (`readiness.go`, `run.go`, the send path) and `pattern/PATTERN-pane-text-injection.md`.
No PATTERN entry is violated by the record; the Constraints section cites the right entries.

Focus verdicts not raised as findings:

- D3 bound on the first Esc: holds.
  The guard can only withhold; an Esc whose turn ends between capture and key lands alone, and the settle keeps any earlier Esc outside the double-Esc window.
- `ErrNoRunningTurn` as success: no session that was never interrupted is reported as stopped.
  A first-Esc refusal lands in `no_running_turn`, distinct from `interrupted`; a second-Esc refusal follows a first Esc the pane showed landing on a running turn.
  The orch's case is F5.
- Classification of state/pane combinations: running turn, `busy`/`background`, idle (including asking, dead and unknown states), a failed busy reading and a failed guard capture each land in one list.
  The state-idle, pane-running case is F6; the reading behind `still_busy` is F2.
- Two-Esc maximum: the pause engine's loop bound and claim-before-Esc order hold it, and the guard only withholds.
  The orch watcher's bound depends on F4.
- D6 for a recorded session whose pane shows the picker: the readiness path reads `busy`/`background` as ready by state, then refuses on the missing box, so nothing is typed into the picker, consistent with `PATTERN-pane-text-injection`.
  Only the quoted reason is wrong (F3).
- No relocation finding is raised, so the completeness-before-leanness test had nothing to protect.

### [MEDIUM] Runner.Inject left as an unbounded interrupt-key path

**Location:** `_lyx/discussion/decision-record.md` (D7, D3)

**Issue:** D7's new PATTERN entry says an interrupt key reaches an agent pane only through shuttle's guarded interrupt.
`Runner.Inject` in `internal/shuttleengine/run.go` is documented as deliberately skipping the ready-pane guard to deliver keys while the provider is busy, its tests deliver `{Key: "Escape"}` through it, and `pattern/PATTERN-pane-text-injection.md` lists it among the explicit key deliveries.
The record never says whether Inject may still carry the interrupt key, so the rule it adds has a bypass with no stated bound, which the rubric's attack-surface question flags.
`creel`'s planned `inject --interrupt` makes this the next caller to pick a path.

### [MEDIUM] still_busy is defined by the pane but read from the lagging state

**Location:** `_lyx/discussion/decision-record.md` (D4, D5)

**Issue:** D5 defines `still_busy` as "a turn still on screen after the second Esc", a pane reading.
D1 makes the pause's interrupt step read the state predicate, and D3's rationale says the state lags the transcript.
D4 requires the settle only between the first Esc and the next read; today's `interruptSession` reads `Busy` immediately after each Esc.
Without a settle and a named reading after the second Esc, a turn the second Esc stopped can still read running and land in `still_busy`, contradicting D5's definition and Acceptance criterion 4.

### [MEDIUM] Quoted no-input-box reason does not match the guard

**Location:** `_lyx/discussion/decision-record.md` (D6, Acceptance criteria)

**Issue:** D6 gives "the pane shows no input box" as an example of the guard's reason, and the Acceptance criterion for a pre-change entry over an open picker expects exactly that text.
`readinessReasonNoInputBox` in `internal/shuttleengine/readiness.go` reads "no input box is on screen; a menu or picker may be open".
D6's premise is that resume passes the guard's own reason through, so the plan writer would either write a criterion that cannot pass or reword a shuttle reason outside Scope.

### [LOW] Orch watcher retry is defeated by the entry-owns-the-orch short-circuit

**Location:** `_lyx/discussion/decision-record.md` (D4)

**Issue:** `handleHubEntry` records the orch in the entry before its first Esc, and `settledWithoutInterrupt` returns true for an entry that already lists the orch's session.
D4's later tick that re-probes after the settle runs through that same function, so as written it would settle the entry and never send the second Esc.
The same short-circuit is what bounds a watcher restarted during the settle (its in-memory first-Esc time is gone, so no second Esc), and D4 does not state that bound either.

### [LOW] Orch guard refusals are silent

**Location:** `_lyx/discussion/decision-record.md` (D3, D5 rationale, Open risks)

**Issue:** D5's rationale keeps `no_running_turn` so that a guard withholding every Esc is visible rather than hidden as idle, and Open risks relies on that list for a reworded running-turn hint.
The orch watcher, per D3 and Acceptance criterion 9, treats `ErrNoRunningTurn` as an idle orch with no Warn and nothing else, and the pause report shows the orch only as `delegated-to-the-orch-watcher`.
The same failure is therefore invisible for the orch.

### [LOW] D1 overstates that the readings cannot drift

**Location:** `_lyx/discussion/decision-record.md` (D1)

**Issue:** `sessionReadiness` adjusts the folded state before `judgeReadiness` sees it: an unmatched turn start whose hold still holds makes a send not ready, and a released hold turns `busy`/`turn` into `idle`/stalled.
`pausecli`'s `shuttlePair.Busy` reads the raw folded state from `ReadSessionStates`.
Sharing the predicate keeps the state rule in one place, but the two readings still differ, and the state-idle, pane-running case gets no Esc and lands in `already_idle` with no Open risk naming it.
