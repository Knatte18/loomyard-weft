# Support log — viewer-off-shell

## Interview

Autonomous mode: no operator answers; every pick below is an auto-pick recorded in the question ledger.
No advisors were configured.

### Turn 1 — exploration

Read the board entry, notes `reed-status-buttons` and `reed-status-buttons-prototype`, the focus-rule, binding and status-bar paragraphs of `internal/reedengine/doc.go`, `ownwindow.go`'s focus decisions, `bar.go`'s formats, `lyx reed switch`, and the two overview spawners (`loomcli/bootstrap.go`, `orchcli/runs.go`).
Findings: attach selects `relevantWindow`, which is empty before the overview opens; the overview add selects nothing; a new agent window follows only from overview, dead-agent, relevant or driver windows; `KeepViewer` is set only by `lyx shuttle run` without `--focus`.

### Turn 2 — design batch (auto-picked)

1. Where does follow state live? Recommended: tmux session option, written by bindings atomically with their select (D1). Alternative: reed.json via a `run-shell` verb per keystroke.
2. What is "a run starts"? Recommended: fresh add of an overview strand (D2). Alternatives: session boot only; a new reed verb callers invoke at run start.
3. Do viewer-position heuristics survive? Recommended: no, follow state alone decides, `Focus`/`KeepViewer` keep overriding (D3).
4. Attach with another client attached? Recommended: select when follow is on (D4).
5. Which bindings are manual switches? Recommended: Alt+Up/Down and window, shell and strand-button clicks (D5).
6. Button click semantics and place? Recommended: toggle; session line before the time (D7).
7. How does the click jump? Recommended: a told-geometry reed verb like `switch` (D8).

### Turn 3 — lens review

Every fork returned a report.

- premises 1, goal-scope 1, scenarios 1 and 2 (merged, MEDIUM): an overview add misses runs that keep or replace `loom-status`, and a re-add mid-run would re-arm — fixed: D2 now keys on each `lyx loom start` entry through one reed operation, with the re-entry bound stated and an acceptance row for the reused-pair run.
- premises 2 (LOW): "every run opens its overview by a fresh add" false — fixed with D2's rewrite.
- premises 3 (LOW): click branches are `run-shell -C` format strings — fixed: noted in Notes for the plan writer.
- premises 4, 6 (LOW): premises confirmed — no change.
- premises 5 (LOW): `attachWindowSelectionLocked` lives in `ownwindow.go` — fixed in Notes.
- goal-scope 2 (LOW): toggle-off click is an unlicensed addition — fixed: named in Scope In as deliberate.
- goal-scope 3 (LOW): `Focus` override unbounded — fixed: D3 names its users.
- goal-scope 4 (LOW): default-on vs the shell diagnosis unstated — fixed: D3's rationale ties them.
- board-fit 1 (MEDIUM): placement cuts across the prototype note — fixed: D7 says the notes may move the button.
- board-fit 2 (LOW): `batten` label with no batten change — fixed: Scope Out says batten needs no change.
- board-fit 3 (LOW): `spawn-audit-log` not named — rejected: no conflict, and that note covers every spawn generically.
- board-fit 4, 5 (LOW): `engine-config-seams` and `orch-split` — rejected: lens found no overlap.
- pattern-fit 1 (MEDIUM): verb misread as exempt from `PATTERN-spawn-observability` — fixed: constraint now says its tmux calls go through the logging seam.
- pattern-fit 2 (LOW): background list of `lyx`-running bindings and `#` doubling — fixed in D8 and Constraints.
- pattern-fit 3 (LOW): say no PATTERN entry is added — fixed in D1.
- pattern-fit 4 (LOW): `PATTERN-test-isolation` uncited — fixed in Constraints.
- leanness 1 (LOW): auto-mode bullets restate decisions — fixed: bare pointers.
- leanness 2 (LOW): open risk restates D5's bound — fixed: risk removed, D5 keeps it.
- leanness 3 (LOW): note-edit clause is scope — fixed: moved to Scope Out.
- leanness 4 (LOW): annotation and function names are plan detail — fixed: D8 states the requirement only; D1's read mechanics cut.
- leanness 5 (LOW): `driverWindows` dead-code call — fixed: cut.
- leanness 6 (LOW): Scope In indexes the decisions — rejected: the lens judged it acceptable as an index.
- leanness 7 (LOW): Notes repeat D2's rationale — fixed with the Notes rewrite.
- scenarios 3 (LOW): write/select failure paths unstated — fixed in D1.
- scenarios 4 (LOW): spawn burst hops — fixed: stated in D3.
- scenarios 5 (LOW): click on current window — fixed: D5 and an acceptance row.
- scenarios 6 (LOW): no Windows scenario — rejected: D9 changes no Windows code path, and the tmux-tier tests do not run psmux.
- scenarios 7 (LOW): session-name quoting — fixed: D8 bound and an acceptance row.

## Rejected alternatives

- Follow state in reed.json: a manual-switch binding would need a `lyx` spawn per keystroke and could race its own `select-window`.
- Turning follow on only at session boot: a pair session reused across runs, or one the operator stopped following, would never follow the next run.
- An overview strand add as the run-start signal (the first pick): a reused pair's next run keeps or replaces `loom-status`, so it would not re-arm follow; the lens review replaced it with loom's start bootstrap.
- The orch's `runs` overview as a run start: the orch session's agent is a `Focus` strand and its overview is no task run.
- Keeping the viewer-position heuristics beside follow: a viewer the operator moved to the overview by hand would still be moved, against the operator's "never disturbed".
- Fixing Open-Window to attach later or to pass a select: the attach is not the only route (VS Code's chain attaches too), and follow fixes every route in reed.
- A button that only turns follow on: leaves no way to stop following without changing window.
- The button on line 0 among the window buttons: the board keeps action buttons apart from window buttons.
- Computing the jump target in a pure tmux format: pane liveness and highest-index selection over agent windows are not expressible there.

## Review rounds

### Round 1 — APPROVED, six findings, all fixed

- F1 (MEDIUM): D2's bound treated every `lyx loom start` as an explicit run start, missing the agent-driven resume — fixed: D2's bound names fresh run, re-entry and resume, accepts the re-arm on resume, and an acceptance row pins it.
  Narrowing to a start that seeds a new run was rejected: a resume is the same run continuing, and following it is the default the goal sets.
- F2 (MEDIUM): no acceptance row failed without D4 — fixed: a second-attach row with an overview present, follow on and off.
- F3 (LOW): the overview-replace row contradicted D6 — fixed: the row excepts D6's move.
- F4 (LOW): D4's bound ignored D5's outside-binding changes — fixed: D4 states what the select moves.
- F5 (LOW): D3 missed `lyx shuttle run --focus` — fixed: D3 bounds the override by the explicit focus flag.
- F6 (LOW): D7 left the unresolvable-path click open — fixed: a click from off then only sets the option and logs at Warn.

## Question ledger

- Q1 follow-state store — resolved, auto-pick: tmux session option `@lyx_follow` (D1).
- Q2 run-start signal — resolved, auto-pick: each `lyx loom start` entry through one reed operation (D2), revised from the overview add after the lens review.
- Q3 heuristics vs follow — resolved, auto-pick: follow alone, `Focus`/`KeepViewer` override (D3).
- Q4 attach with other clients — resolved, auto-pick: select when follow is on (D4).
- Q5 manual-switch bindings — resolved, auto-pick: Alt+Up/Down, window/shell/strand clicks (D5).
- Q6 button semantics and placement — resolved, auto-pick: toggle on the session line (D7).
- Q7 jump mechanism — resolved, auto-pick: `lyx reed follow` told socket, session and tmux path (D8).
