# Decision record — viewer-off-shell

## Goal

A reed session's viewer follows the run by default: when a run starts it goes to the overview, and as the run spawns agents it moves to each newest agent window, from the shell too.
A per-session follow state, shown as one clickable button on the reed status bar, replaces today's viewer-position heuristics:
any window change the operator makes through reed's bindings turns follow off, and clicking the button turns it back on and jumps to the run.

## Scope

In:

- A per-session follow state with an on/off value, on by default.
- The focus rules in `internal/reedengine` rewritten around it: run start, new agent window, attach.
- Reed's manual window-change bindings turning follow off.
- One follow button on the reed status bar, and the verb its click runs to turn follow on and jump.
- A click on the button while follow is on turns it off: a deliberate addition to the board's "clicking turns follow back on", so the operator can stop following without changing window.

Out:

- The configurable button set, its config and the menu button: board note `reed-status-buttons`, which later takes the follow button into its set; this task does not edit that note.
- Batten: its Open-Window row and VS Code's folderOpen attach chain keep attaching as today, and batten's code needs no change; the fix lives in reed and in loom's run-start bootstrap.
- Detecting window changes made outside reed's bindings (D5's bound).
- Session switching (Alt+Left/Right, session buttons): it changes no window of a session and leaves every session's follow state as it is.

## Decisions

### D1 — Follow state is a tmux session user option `@lyx_follow`

Decision: the follow state is the session option `@lyx_follow`, with `off` meaning off and any other value, unset included, meaning on.
Reed's bindings write it inside the same tmux command list as their window change, and Go reads it at each focus decision.
Boot sets it on explicitly; a session booted by an older lyx reads unset, so it follows.
Rationale: the manual-switch bindings run inside tmux with no `lyx` process, so a tmux option is the one store they can write atomically with their `select-window`.
Bound: this is a Go decision reading a tmux user option, which the reed doc today says no Go decision does for `@strand`, `@strand_color`, `@lyx_unseen` and the wait marks;
the reed doc names `@lyx_follow` as the one decision-bearing session option, and those other options stay display only (`PATTERN-wait-mark-display-only` unchanged).
The rule is local to reed, so no PATTERN entry is added.
A failed read counts as off for that one decision and is logged at Warn, so a broken read never moves a viewer.
A failed write of the option or a failed select is logged and never fails the operation it rides on.

### D2 — A run start turns follow on: loom's start bootstrap

Decision: reed gains one engine operation that sets `@lyx_follow` on and selects the relevant window (`relevantWindow`: newest live agent window, else the overview, else none).
Loom's shared start bootstrap calls it once per `lyx loom start` entry, after `ensureStatusStrand`, whichever of keep, replace or add that took.
No strand add, replace or resume turns follow on by itself, and the orch's `runs` overview never does.
Rationale: a reused pair's next run keeps or replaces its `loom-status` strand rather than adding it, so only the bootstrap marks every run start; darn runs share that bootstrap.
Bound: every `lyx loom start` entry re-arms follow and may move a viewer who turned it off: a fresh run, a re-entrant start on a live run, and a resume of a parked run, which the orch, batten's Run-Shed or an approve or reject follow-up runs with no operator action.
The re-arm on a resume through `lyx loom start` is accepted, since it is the run's entry verb, and an operator who stopped following is moved back to it, as at its start.
The bound is the entry itself: no other loom verb, and no reed operation but D2's and D8's, turns follow on.
`lyx loom resume`, which wakes a parked driver without bringing reed up and never switches a client, leaves the follow state as it is; only a resume that enters through `lyx loom start` re-arms follow.

### D3 — A new agent window takes the viewer exactly when follow is on

Decision: `newAgentWindowSelected` becomes: a `Focus` strand always takes the viewer; a strand whose display sets only `KeepViewer` never does; any other takes it when `@lyx_follow` is on, from any window, the shell included.
The viewer-position branches (overview, dead agent window, relevant window, driver window) go.
Opening a window never changes the follow state; the selected window still carries no `@lyx_unseen`.
Agent windows opening back to back each take the viewer in turn, so it ends on the last one.
Rationale: follow is on by default and only a manual binding turns it off, so a viewer resting on the shell by default now follows, while one the operator moved by hand does not; where the viewer happens to be no longer decides.
Bound: with follow on, a viewer on the shell or an older agent window is moved, which the current doc forbids; the bound is the follow state itself.
The `Focus` override ignores follow: a strand spawned with an explicit focus flag (`lyx orch start`'s orch strand, `lyx reed add --focus`, `lyx shuttle run --focus`, which an agent may run inside a task pair) takes the viewer whatever the follow state.

### D4 — Attach selects the relevant window by follow state

Decision: with follow on, attach appends `; select-window -t <relevant window>` also when another client is attached; with follow off, attach keeps today's rule (select only when no other client is attached).
Attach never changes the follow state.
With no relevant window yet, attach selects nothing and follow stays on, so D2 or D3 moves the viewer once the run opens its overview or an agent.
Rationale: this is the reported bug's cause: Open-Window attaches before the overview exists, and a second attached client (VS Code's attach chain) suppressed the select.
Bound: every client of a session shares its current window, so with follow on an attach moves every attached client to the relevant window, including one that reached its window by a change outside reed's bindings (D5's bound); with follow off it moves none while another client is attached.

### D5 — Reed's manual window-change bindings turn follow off

Decision: Alt+Up, Alt+Down, and a status-bar click on a window button, the shell button or a strand button set `@lyx_follow` to `off` in the same command list as their window change, also when the click lands on the current window.
Alt+z, Alt+Left/Right and session-button clicks leave it as it is.
Rationale: reed owns these bindings, so it can tell a manual switch from its own `select-window` calls, which never touch the option.
Bound: a window change made outside reed's bindings (tmux prefix keys, an operator's own `tmux_conf` bindings) leaves follow on, and the next agent window then moves the viewer; the reed doc states this.

### D6 — Reed's own automatic moves leave follow as it is

Decision: the existing moves of a viewer off an ending window (`selectRelevantWindowIfCurrentLocked`, `RemoveStrand`, `ReplaceStrand`, reconcile) stay as they are, with or without follow, and never change the follow state.
Rationale: a window that ends must move its viewer somewhere, and that is not an operator choice.

### D7 — One follow button on the session line

Decision: `status-format[1]`, the session-buttons line, gets one `range=user|follow` button right of the session buttons and left of the time, labelled `follow`, lit while `@lyx_follow` is not `off` and dim while it is.
A left click on it toggles: from on it sets off and changes no window; from off it sets on and runs, through `run-shell -b`, the D8 verb.
When the D8 verb's command cannot be composed (the socket path, the tmux path or the `lyx` binary unresolved), a click from off only sets `@lyx_follow` on in tmux and selects no window, and the failure is logged at Warn as for the session-switch keys; the next agent window then takes the viewer.
Rationale: the board keeps action buttons apart from the window buttons on line 0, and the session line needs no new status row.
The placement is the one this task ships; note `reed-status-buttons-prototype` and note `reed-status-buttons` may move the button when they settle the button set's layout.

### D8 — `lyx reed follow`: the jump verb the button runs

Decision: a reed subcommand `lyx reed follow`, told the socket, the session and the tmux path like `lyx reed switch`, and like it run from a binding: no stencil-seed pass, silent on success.
It sets `@lyx_follow` on and selects the session's relevant window by `relevantWindow`'s rule, else changes no window.
Rationale: the relevant-window rule needs pane liveness and window ordering a tmux format cannot express; a verb told its geometry needs no cwd, config or state lookup.
The plan writer decides how the verb identifies agent and overview windows from tmux facts (`@lyx_own` and pane liveness today; an overview marker if none exists) and how it shares the rule with `relevantWindow` and D2's operation.
Bound: the verb only sets one session option and selects one window of the told session; an unknown session or a failed tmux call prints an error envelope and changes nothing else.
The binding names `lyx` by absolute path per `PATTERN-pane-binary-resolution`, doubling the path's `#` for run-shell while the session stays a `#{session_name}` format, and the session reaches the verb as one quoted argument.

### D9 — Windows and psmux

Decision: no Windows-specific behavior: a psmux without `bind-key` or `run-shell` never sets follow off, so its sessions always follow, and every binding stays non-fatal as today.

## Constraints

- `PATTERN-pane-binary-resolution`: the button's `run-shell` names `lyx` through `panebin.go`'s composition, never PATH; the background file's list of bindings that run `lyx` gains the follow button.
- `PATTERN-cli-cobra`: the verb is a cobra subcommand in reed's subtree.
- `PATTERN-shell-mechanics-seam`: any pane-shell command string goes through `internal/shell`.
- `PATTERN-spawn-observability`: the verb's tmux calls go through reed's existing tmux seam, which logs them.
- `PATTERN-wait-mark-display-only`: wait marks stay display only; D1 adds a decision-bearing option of its own and touches no wait mark.
- `PATTERN-test-speed`, `PATTERN-test-economy` and `PATTERN-test-isolation`: pure decisions (`newAgentWindowSelected`, the status format, the binding command lists) are Tier 1 table rows extending the existing tests; tmux behavior goes in the existing tmux-tier integration tests, which run through `tmuxkit.Main`.
- `PATTERN-documentation-lifecycle`: the focus-rule, binding and status-bar paragraphs of `internal/reedengine/doc.go`, the reedcli doc for the verb and loom's bootstrap doc are rewritten in the same commit.
- A package pushed past its row in `cmd/lyx/testdata/test-budget.yaml` by new top-level tests lists that file under `Edit:`.

## Auto-mode assumptions

- Run start is each `lyx loom start` entry (D2).
- The button toggles (D7).
- `Focus` and `KeepViewer` override follow (D3).
- With follow off, attach keeps today's rule (D4).
- The button sits on the session line (D7).

## Open risks

- An automatic switch races an operator's manual one by at most one switch; a binding's off write landing after reed's read of on lets one more automatic move through.

<!-- lyx:carry-over Discussion-Review -->
- Discussion-Review, round 2: converged; each finding below was fixed or disputed by the round's fixer, and no fresh reviewer has seen the result; the fixer report says which.
  Review: _lyx/reviews/discussion/round-2-review.md; fixer report: _lyx/reviews/discussion/round-2-fixer-report.md.
  - loom-resume-verb-not-in-or-out-of-d2-rearm (class decision, severity MEDIUM, open in rounds 2)
<!-- /lyx:carry-over Discussion-Review -->

## Acceptance criteria

- A session attached before any overview or agent exists, viewer on the shell: the run's start selects the overview, and each later agent window selects that window.
- The same with a second client attached before the run starts: both clients end on the overview, then on each newest agent.
- With follow on, an overview present and a first client on the shell, a second attach selects the overview and moves the first client there; with follow off, the same second attach selects nothing and leaves the first client on the shell.
- A second run in a reused pair, whose `loom-status` strand is kept or replaced, with follow off: the start turns follow on and selects the relevant window.
- A parked run resumed by `lyx loom start` with follow off: the resume turns follow on and selects the relevant window.
- A halted run woken by `lyx loom resume` with follow off leaves follow off and moves no viewer.
- Alt+Up, Alt+Down and a click on a window, shell or strand button, the current window's included, set `@lyx_follow` off; a later agent window then leaves the viewer where it is, the shell included.
- Alt+z, Alt+Left/Right and a session-button click leave `@lyx_follow` unchanged.
- The follow button renders lit with follow on and dim with it off; a click from on sets off and changes no window; a click from off sets on and selects the newest live agent window, else the overview, else no window.
- `lyx reed follow` against a session with no agent and no overview window sets follow on and selects nothing; against an unknown session it fails with an error envelope; a session name that needs quoting reaches it intact.
- A `Focus` strand takes the viewer with follow off; a `KeepViewer` strand never takes it with follow on.
- A session booted by an older lyx (`@lyx_follow` unset) follows; a failed read of the option moves no viewer.
- An ending viewed window still moves its viewer as today, with follow on and off - to the relevant window, or to the new overview window when the overview strand is replaced - and leaves the follow state unchanged.
- Adding, replacing or resuming an overview strand outside the start bootstrap, the orch's `runs` included, changes no follow state and moves no viewer except D6's move off a window that ends.

## Notes for the plan writer

- Focus code lives in `internal/reedengine/ownwindow.go` (`relevantWindow`, `newAgentWindowSelected`, `newAgentWindowTakesViewerLocked`, `attachWindowSelectionLocked`); `AttachArgv` is in `attach.go`.
- Bindings are pinned in `bindings.go`, where the shell and strand click branches are `run-shell -C` format strings; the status formats are in `bar.go`; `lyx reed switch` in `internal/reedcli/switch.go` is the template for the verb.
- Loom's run-start bootstrap is `ensureStatusStrand` in `internal/loomcli/sharedbootstrap.go`, called from `start.go`.
