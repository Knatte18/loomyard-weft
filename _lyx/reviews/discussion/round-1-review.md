---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:46
    summary: D2's bound rests on the false premise that every lyx loom start is an explicit start of the run, while lyx loom start is also the resume verb agents run after a park, an approve or a reject, so follow is re-armed and the viewer moved with no operator action
    fix: In D2, replace the Bound sentence with one that names every entry kind that re-arms follow, a fresh run, a re-entrant start on a live run, and a resume of a parked run run by the orch, batten or an approve/reject follow-up, and states what bounds it. Either accept the re-arm on resume and say so (the operator turned follow off and an agent-driven resume moves them back to the run), or narrow the call to a start that seeds a new run and add that case to the Acceptance criteria. Add one acceptance row for the chosen resume behavior (a parked run resumed by `lyx loom start` with follow off).
  - id: F2
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:124
    summary: No acceptance criterion fails without D4; the second-client row attaches before any relevant window exists, where attach selects nothing either way, and D2 alone then moves both clients
    fix: In Acceptance criteria, add a row that pins D4 itself, for example, with follow on, an overview present and a first client on the shell, a second attach selects the overview and moves the first client there; with follow off, the same second attach selects nothing and leaves the first client on the shell.
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:133
    summary: The acceptance row saying a replace of an overview strand outside the bootstrap never changes the viewer's window contradicts D6, which keeps ReplaceStrand moving a viewer off the window it ends onto the new overview
    fix: Reword the row to exclude D6's move, for example to read, Adding, replacing or resuming an overview strand outside the start bootstrap, the orch's runs included, changes no follow state and moves no viewer except D6's move off a window that ends.
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:64
    summary: D4 removes the guard that a second attach never moves a viewer, and its stated bound, that a select with follow on moves only clients following anyway, is false under D5's bound, where a viewer moved by tmux prefix keys or tmux_conf bindings still has follow on
    fix: In D4, replace the last Rationale sentence with a Bound line saying that every client of a session shares its current window, so with follow on an attach moves every attached client to the relevant window, including one that reached its window by a change outside reed's bindings (D5's bound); with follow off it moves none while another client is attached.
  - id: F5
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:56
    summary: D3 says the Focus override's only users are the orch strand and lyx reed add --focus, but lyx shuttle run --focus also sets Focus, and an agent can run it inside a task pair
    fix: In D3, name `lyx shuttle run --focus` among the Focus users and drop the claim that none is a run in a task pair, or state the bound as that a strand spawned with an explicit focus flag takes the viewer whatever the follow state.
  - id: F6
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md:81
    summary: D7 leaves open what the follow button does when the lyx path, socket path or tmux path cannot be resolved at pin time, the case in which reed today leaves M-Left and M-Right unbound
    fix: In D7, add one sentence deciding the degraded click from off, for example that when the D8 verb's command cannot be composed, a click from off only sets `@lyx_follow` on in tmux and selects no window, and the failure is logged at Warn as for the session-switch keys.
---

### [MEDIUM] D2's re-arm bound misreads what `lyx loom start` is

**Location:** `_lyx/discussion/decision-record.md:46`

**Issue:** `lyx loom start` is the resume verb as well as the run-start verb.
`internal/loomcli/approve.go` and `reject.go` name it as the `resume` step, `driverprompt.go` describes a parked driver resumed by `lyx loom start` typing one line, `battenshed/innerrun.go` routes a halted child to it, and batten's Run-Shed spawns it (`internal/battencli/wire.go:1191`).
D2's bound covers only a re-entrant start on a live run and calls every start explicit, so the decision does not state that an agent-driven resume after a park re-arms follow and moves a viewer who turned it off.
The rubric's attack-surface question requires the bound to name what the new routing edge lets through.
The darn premise holds: battencli spawns the one bootstrap verb for loom and darn alike.

### [MEDIUM] D4 has no acceptance criterion of its own

**Location:** `_lyx/discussion/decision-record.md:124`

**Issue:** In the second-client row, both clients attach before the overview exists, so `relevantWindow` is empty and attach appends no select with or without D4.
Both clients then reach the overview through D2's select, which moves every client of the session.
The row passes with D4 unimplemented, and no other row covers an attach while another client is attached and a relevant window exists.

### [LOW] Acceptance row on overview replaces contradicts D6

**Location:** `_lyx/discussion/decision-record.md:133`

**Issue:** `ReplaceStrand` selects the replacement overview's window when the replaced strand's window was current (`internal/reedengine/replace.go:94`), and D6 keeps that move.
The row says a replace outside the bootstrap changes the viewer's window in no case, so a test written from it fails against D6's behavior.

### [LOW] D4's stated bound is incomplete

**Location:** `_lyx/discussion/decision-record.md:64`

**Issue:** The premise holds: reed creates no grouped sessions (`lifecycle.go`'s one `new-session`), so every client of a session shares its current window.
But D5's bound leaves follow on after a window change made outside reed's bindings, so "moves only clients that are following anyway" is false for that client.
D4 removes the current doc's guard that a second attach never moves a viewer, so the rubric requires it to state what the select can now disturb.

### [LOW] D3 miscounts the `Focus` users

**Location:** `_lyx/discussion/decision-record.md:56`

**Issue:** `internal/shuttlecli/run.go` sets `Focus: focus` from its `--focus` flag, a third user beside `orchcli/start.go` and `reedcli/add.go`.
D3 bounds the override by naming its users, and the list it gives is short one.

### [LOW] D7 leaves the degraded click undecided

**Location:** `_lyx/discussion/decision-record.md:81`

**Issue:** `switchBindingCommandsLocked` leaves the session-switch keys unbound when the socket path, tmux path or binary cannot be resolved.
The D8 binding needs the same resolution, and D7 does not say whether the button's from-off branch is then unbound, sets the option alone, or is pinned anyway.
That is a choice the record must make, not the plan writer.

## Verdicts on the focus entries

- D2 and D3 overrides: F1 and F5; D3's own bound is the follow state and holds otherwise.
- D4 premise: holds, since tmux keeps the current window per session and reed groups no sessions; F4 covers the unstated disturbance.
- D5 and D6: consistent with Scope Out.
  Every reed-owned window-change path is accounted for: the bindings (D5), spawn's new-window select (D3), `RemoveStrand`, `ReplaceStrand` and reconcile (D6), attach (D4), and D2's operation and D8's verb.
  `lyx reed switch` issues only `switch-client`, and `select-pane` in `apply.go` does not change the current window (checked against tmux 3.6 on a scratch server).
- D8 and D7: D8's bound covers a click from off (one option, one window of the told session); D7's toggle-off sets one option and changes no window, so it is bounded. F6 covers the unresolvable-path case.
- Acceptance criteria: F2 and F3; every other behavior-changing decision has a row that would fail without it, and D9 changes no behavior.
- Relocation: none raised.
  The deliberation in the record (D4's bug diagnosis, D7's placement note) carries premises the plan writer needs.
