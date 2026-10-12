# Decision record — reed-multi-window

## Goal

Give every agent a run spawns a full-size tmux window of its own, stacked in depth like tabs in the run's terminal window, instead of a pane squeezed into the one strands' window.
The strands' window (window 0) keeps the driver, its `loom-status` strand, Selvage and every strand that does not ask for a window.
A producer with a few agents shows them together in one shared window; one with many gives each its own.
A new window never takes focus.
The same anchoring lets the orch open an extra named Claude session (such as one for exploring the repo) in its own window, through shuttle.

## Scope

In:

- reed: the `own-window` anchor, which is reserved and refused today, becomes a supported placement, covering add, launch, liveness, reconcile, removal, resume after a server restart, the status bar and the hub watchdog's name repair.
- reed: a shared window per declared group, split side by side or stacked, with the share cap and the split direction set in `reed.yaml`.
- shuttle: own-window becomes the default placement of a run's strand; the long-lived sessions that must stay in window 0 name their anchor explicitly.
- seatengine: a table's seats declare one group, so a small table shares a window and a large one gives each seat its own.
- shuttle: a new verb, `lyx shuttle open`, that opens a named LLM session with no prompt and no output files in its own window, with its session id recorded on the strand.
- `lyx reed add` and `lyx shuttle run` accept `own-window` for `--anchor`.
- The operator command index that lists `lyx` verbs gains `lyx shuttle open`.

Out:

- Cross-worktree columns and per-window attach through tmux session groups: dropped by the operator on 2026-10-12.
- The shuttle readiness probe fix: `run-window-bugs` owns it and re-checks it against this layout.
- New key bindings for stepping between windows: the status bar click and tmux's default prefix window keys, which the hermetic hub server keeps, cover it.
- Moving a live strand between window 0 and its own window.
- Zoom: the Alt+z zoom, the strand buttons and the zoom bracket stay as they are and act on window 0 only.

## Decisions

### D1 — `own-window` is a placement, declared on the strand's Display

Decision: `render.AnchorOwnWindow` stops being refused.
A strand anchored `own-window` gets no pane in window 0: reed opens it in a window of its own, or in its group's shared window, and the window 0 layout never places it.
`render.Display` gains two fields, persisted in the strand record: `Window`, the group name (empty for a strand that wants a window to itself), and `WindowSize`, the number of strands the spawner puts in that group.
Rationale: the board entry puts all of this on the strand's display anchoring setting, and reed already persists Display verbatim, so the spawner states intent and reed owns the mechanics.

### D2 — reed decides shared or per-strand from the declared group size

Decision: a strand with a non-empty `Window` and a `WindowSize` at most the `reed.yaml` key `window_share_max` (default 2) joins the group's shared window, created by the group's first strand and named by the group.
Otherwise the strand gets a window to itself, named by its strand role.
A `WindowSize` below 0 is refused at add, naming the field; 0 reads as 1.
A strand joining a live group whose recorded `WindowSize` differs from its own is refused, naming both sizes.
The shared-or-apart choice is recorded in reed's state on the group by its first strand, and later joiners follow that recorded choice rather than re-reading `window_share_max`, so a `reed.yaml` edit mid-group never splits a group across shared and own windows.
A join that would push a shared group's live count past its recorded `WindowSize`, such as a re-stepped seat added before its predecessor is removed, gives the joiner a window to itself, named by its strand role, and leaves the shared window as it is.
A group's live count is its strands bound to a present pane, alive or dead, since a dead pane D5 keeps still occupies the shared window.
The group's recorded choice and `WindowSize` are dropped when its last strand is removed, so a later instance under the same group name records afresh.
Rationale: every strand of a group declares the same size, so the outcome is the same whatever order the strands arrive in; the operator's line is two agents together, five or six apart.

### D3 — a shared window splits by the `reed.yaml` key `window_split`

Decision: `window_split` takes `width` (default: side by side) or `height`; reed lays the shared window out with tmux's even layout in that direction after every join and leave.
Any other value fails up, add and resume, naming the key, so a `reed.yaml` edit on a live session surfaces at the next join.
`window_share_max` is an integer at least 0, where 0 gives every grouped strand a window to itself; a negative value fails the same ops the same way.
Remove never fails on either key: it kills the strand's pane and ends its window as D5 and D6 say, skips only the even layout of a shared window it leaves, and reports the invalid key in its result, so a `reed.yaml` typo never blocks shuttle's cleanup, a seat stop or D12's self-removal.
A window to itself has one pane and no layout.
No size pins, resize hooks or Selvage live in an own window.
`window_share_max` and `window_split` are ordinary per-worktree `reed.yaml` keys resolved through the usual config layers, so a hub-layer value reaches every worktree; `reed` stays a non-hub-wide module.
Rationale: the operator asked for both directions as a setting, and a laptop and a desktop differ.

### D4 — a new window never takes focus, and announces itself on the status bar

Decision: reed opens every own window detached and never selects it.
The window carries the user options `@lyx_own` (the marker), `@strand_color` (its segment's color) and `@lyx_unseen`, set at creation.
The status bar gives it a window button labelled by the window name in its segment color, carrying its strands' wait marks, and styled as unseen while `@lyx_unseen` is set.
A session-scoped `session-window-changed` hook clears `@lyx_unseen` on the window the client moves to; it is pinned at boot and re-pinned by every attach pre-flight, like the navigation bindings.
Where the multiplexer lacks `set-hook` (psmux), `@lyx_unseen` is never set, so no window reads unseen forever.
All of these are display only: no Go decision reads them.
Bound: the hook writes one window option on the window the operator selected and nothing else.
Rationale: the operator rules out any automatic switch; a marked button is visible without moving the operator.

### D5 — the window closes when its last strand goes, and the view falls back to window 0

Decision: removing an own-window strand kills its pane, and with the last pane of the window the window ends.
Before the kill, a client whose current window is that window is moved to the strands' window.
Own windows keep `remain-on-exit` on, as window 0 does, so an agent that dies leaves its window showing the dead pane until its strand is removed.
Reconcile's dead-pane kill does not reach own windows: a dead own-window pane keeps its strand binding and is killed only by the strand's removal, so D5's client select always runs before its window ends.
In window 0 the dead-pane kill is unchanged, and so a dead agent's window 0 pane still goes at the next reed op.
Shuttle's stop path removes a tracked own-window strand whose pane is present but dead, not only a live one: the stop verb, `StopStrand`, the step stop and supersession on respawn all go through it, and so does a seat table's stop.
It removes only the strand a run record names, never an untracked pane, and a window 0 strand keeps today's live-only removal.
A seat table's advisors keep their panes after they finish, and the table removes every advisor strand, live or dead, before its step returns, so a shared seat window ends with its table.
Shuttle's `Wait`, on finalizing a run `OutcomeDied` whose strand is own-window, saves the pane's last capture into the run directory and removes the strand, so the run directory keeps the failure evidence and no dead window outlives its run, in the prime as in a pair.
A run that ended `OutcomeTimeout` keeps its strand, since its agent may still be live and a later attach may take it up; if that agent dies with no later stop or respawn, its window stays until `lyx reed remove`, a pair's teardown or, in the prime, the operator's removal, which is accepted, as a timed-out agent's window 0 strand stays today.
Bound: the select runs only for a client on a window that is about to end, so it never moves an operator who is elsewhere.
Rationale: the board asks for the window to close when the agent ends and for the fall back to the driver; a dead agent's window is its failure evidence and goes with its strand.

### D6 — the strands' window stays defined by Selvage, and own windows are never read as it

Decision: the strands' window seam resolves through the Selvage pane, and its fallback through a recorded strand pane considers only strands placed in window 0.
Liveness, reconcile and name repair read own-window strands through a session-scoped pane listing, so a pane in another window counts as present; a pane the operator killed by hand reads absent, as a killed window 0 pane does today.
Reconcile's untracked-pane reap, its dead-pane kill and its keep-one-dead-pane survival rule all stay limited to window 0's panes: the survival rule asks whether any window 0 pane is alive, never a pane in an own window.
In own windows reed kills a pane only when it is a recorded strand's pane being removed, and ends a window only when it carries `@lyx_own` and none of its panes is bound to a strand in reed's state, read under the op lock after any add in progress has persisted its binding.
`OpenWindow` never adopts, kills or replaces a window carrying `@lyx_own`, and a group window is found by its window option, never by name.
Windows opened by `OpenWindow` (`@lyx_hidden`) are never own windows and are left alone.
Rationale: today every op assumes one strands' window; an own-window strand must neither be counted dead nor have its window read as window 0, and the new window end must not reach a window reed did not open as an own window.

### D7 — every launch goes through the one strand launch chokepoint

Decision: the pane of an own-window strand is created by `new-window -d` (or a split of the group window) inside the same launch path every strand uses, so the pane-binary prelude, `LYX_STRAND_NAME`, the pane title lock, `@strand` and `@strand_color` apply as in window 0, and the split guard that refuses a silently reused pane applies to the group-window split.
Rationale: `PATTERN-pane-binary-resolution` and `PATTERN-agent-name` hold by construction only through that one path.

### D8 — resume after a server restart reopens own windows detached

Decision: `Resume` relaunches a live-recorded own-window strand into a fresh detached window (or its group's window), in strand-table order, applying the done-when drop and the pause hold exactly as for window 0 strands.
Rationale: a restarted server must bring an agent back where it lived, without taking focus.

### D9 — a change of anchor into or out of `own-window` is refused

Decision: `UpdateStrand` refuses a Display whose anchor moves a strand between `own-window` and any other anchor, naming the strand and the way forward: remove the strand and add it again.
A change of `Window` or `WindowSize` on an own-window strand is refused the same way.
Rationale: moving a live pane between windows is a separate mechanism nobody asked for; refusing keeps every recorded strand's window stable.

### D10 — shuttle places a run's strand in its own window by default

Decision: shuttle's default anchor for an empty `Display.Anchor` becomes `own-window`, in run and in attach alike, and `lyx shuttle run --anchor` defaults to it.
A session that lives for the whole run or the whole hub names its anchor explicitly and stays in window 0: today the loom driver (already `below-parent`) and the orch (`internal/orchcli`, made `below-parent`); Plan checks every other `shuttleengine.Spec` site against that rule.
Attach to a strand an older binary recorded in window 0 leaves it where it is; only a fresh spawn takes the new default.
Rationale: every agent a driver spawns gets its own window by the operator's 2026-10-05 and 2026-10-12 notes, and a default reaches every producer, present and future, in one place.

### D11 — a seat table declares one window group

Decision: `seatengine` sets every seat's Display to `own-window`, with `Window` naming the table instance (its role prefix and, when set, its round) and `WindowSize` the table's seat count, replacing the sharing size it sets today.
A one-seat table gets a window to itself, named by its seat's role.
Rationale: the producer, not the strand, is the unit; reed's cap (D2) then shows a two-seat table together and a five-seat table apart, and a later round never joins an earlier round's window.

### D12 — `lyx shuttle open` opens a named session with no turn

Decision: a new shuttle verb, `lyx shuttle open --role <role> [--model <spec>] [--segment <segment>]`, adds a strand in its own window running the provider with the strand name as its session name and no prompt, records the session id on the strand, and returns the strand's name and guid without waiting.
It writes no run record and no output files, so `lyx shuttle state` and the hold notices do not cover it.
It refuses while a pause covers its worktree, naming the pause.
Its launch line chains, after the provider exits, a write of an exit marker under the worktree's `.lyx/` and then reed's detached self-removal, so the strand and its window go when the operator ends the session; the marker path is the strand's done-when path.
When the self-removal fails, the pane goes dead and stays with its strand bound until `lyx reed remove` or a pair's teardown, and a server restart's `Resume` drops the strand through its done-when path instead of relaunching it; a pair teardown ends the reed session first, which ends the pane's shell with it, so teardown never triggers the self-removal.
`Resume` relaunches it through its resume command, which resumes the recorded session under the same name and keeps the self-removal chain.
The watchdog's name repair types a rename into its pane through the guarded paste-submit when its session name drifts, as for any strand with a session id.
Provider specifics (the launch line, the session name flag, the resume of a session with no turns) live in `claudeengine`.
Bound: one interactive session per call, in the caller's own worktree under the caller's settings, never a turn; no cap on the count: in a pair worktree the pair's teardown ends every opened session, and in the prime, which has no pair teardown, an opened session lives until its provider exits or the operator runs `lyx reed remove` on it, and a server restart's `Resume` brings back one whose provider had not exited, which is accepted; the role follows reed's name grammar and a held role is numbered as any add.
Rationale: the board's former `extra-agent-strand` needs a named session the watchdog's name repair covers, and the operator rules that every agent goes through shuttle.

## Constraints

- `PATTERN-pane-binary-resolution` and `PATTERN-agent-name`: through D7's chokepoint; the open verb's `Spec` takes its role from the flag, never a string literal.
- `PATTERN-wait-mark-display-only`: the window button reads `@lyx_wait` for display only.
- `PATTERN-spawn-color`: the spawner names the segment; reed colors the window button from it.
- `PATTERN-shuttle-provider-seam`: D12's provider specifics.
- `PATTERN-shell-mechanics-seam`: D12's chained self-removal is composed through `internal/shell`.
- `PATTERN-spawn-observability`: D12's session start is logged through the same launch path shuttle runs use.
- `PATTERN-cli-cobra` and `PATTERN-cliwire-sole-wiring`: `lyx shuttle open` is a cobra verb with `audience`, `Short` and JSON errors, wired through `internal/cliwire`.
- `PATTERN-tolerant-records`: the new Display keys are added with `omitempty`, and reed.json stays tolerant of unknown keys.
- `PATTERN-config-layers` and `PATTERN-config-strictness`: `window_share_max` and `window_split` enter `reed.yaml`'s templates (POSIX and Windows) with their defaults and go through `configengine`.
- `PATTERN-test-speed` and `PATTERN-test-isolation`: tmux-backed behavior is tested at the tmux tier through `tmuxkit.Main`; the decision logic of D2, D3 and D9 is tested as pure functions.
- Session targeting stays exact-match (`=<session>:`).
- No binary change restarts the hub watchdog daemon today: an older daemon left running after a deploy lists window 0 panes only.
  An older daemon only re-applies window 0's layout, which persists nothing and places no own-window strand, renames sessions only for strands with a present alive pane, and reaps only sessions of gone worktrees, so it leaves own-window strands untouched and no daemon restart is needed.

## Auto-mode assumptions

- An unseen-window marker is how a new window announces itself.
- The orch, not the operator, is the expected caller of `lyx shuttle open`, in the prime or any worktree.
- `lyx shuttle open` takes a role, model and segment and nothing else.

## Open risks

- A detached window created while no client is attached takes tmux's default size until a client views it, and an agent starting there renders at that size; `run-window-bugs`, which re-checks the readiness probe against this layout, owns any fix.

<!-- lyx:carry-over Discussion-Review -->
- Discussion-Review, round 2: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lyx/reviews/discussion/round-2-review.md; fixer report: _lyx/reviews/discussion/round-2-fixer-report.md.
  - d3-window-split-validation-and-share-max-range (class design, severity MEDIUM, open in rounds 1, 2)
  - d5-dead-pane-premise-vs-d6-reconcile (class design, severity BLOCKING, open in rounds 1, 2)
<!-- /lyx:carry-over Discussion-Review -->

<!-- lyx:carry-over Plan-Review -->
- Plan-Review, round 2: converged; each finding below was fixed or disputed by the round's fixer, and no fresh reviewer has seen the result; the fixer report says which.
  Review: _lyx/reviews/plan/round-2-review.md; fixer report: _lyx/reviews/plan/round-2-fixer-report.md.
  - card4-relaunch-kill-unlicensed-by-d6-stale-listing (class decision, severity MEDIUM, open in rounds 1, 2)
<!-- /lyx:carry-over Plan-Review -->

<!-- lyx:carry-over Webster-Review -->
- Webster-Review, round 1: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lyx/reviews/webster/round-1-review.md; fixer report: _lyx/reviews/webster/round-1-fixer-report.md.
  - own-window-relaunch-leaves-dead-pane (class design, severity MEDIUM, open in rounds 1)
<!-- /lyx:carry-over Webster-Review -->

## Acceptance criteria

- A strand added with `own-window` and no group gets its own detached window named by its role; the client's current window and active pane do not change.
- Two strands declaring the same group with `WindowSize` 2 share one window split side by side; with `window_split: height` they stack; a group declaring `WindowSize` 5 puts each strand in its own window.
- Strands of one group arriving in either order end in the same arrangement, and a join declaring a different size than the live group's is refused.
- A join that would push a shared group past its recorded `WindowSize` gets a window to itself and leaves the shared window unchanged; a `window_share_max` edit between two joins of one group does not change where the second joiner lands.
- Removing an own-window strand ends its window when it was the last pane, and a client viewing it lands on window 0; a client on another window does not move.
- An agent that dies in an own window leaves the window showing its dead pane, reported dead, until its strand is removed, including across later reed ops in the worktree; a later op still kills a dead window 0 pane as today.
- A step stop of a run whose own-window agent died ends its window, as does a respawn superseding it; a run that `Wait` ends `OutcomeDied` removes its own-window strand and leaves the pane's last capture in its run directory; a timed-out run keeps its strand.
- An own-window strand's pane is reported live by status, never reaped as untracked, and never makes its window read as the strands' window, including when Selvage is dead.
- An own window whose panes are bound to no strand is ended; a window without `@lyx_own`, and a group window whose first strand is mid-add, are not.
- `OpenWindow` with a name equal to an own window's opens its own window and leaves the own window alone.
- A window 0 layout apply, the resize hook and the watchdog's resize loop ignore own-window strands.
- After a server restart, `Resume` reopens each own-window strand in a detached window, drops one whose done-when paths all exist, and holds one under a live pause.
- `UpdateStrand` refuses a move into or out of `own-window`, and a change of its group, naming the way forward.
- A `WindowSize` below 0 is refused at add; an invalid `window_split` or a negative `window_share_max` fails `lyx reed up`, add and resume naming the key, including when `reed.yaml` is edited on a live session; a `window_share_max` of 0 gives each grouped strand its own window.
- Remove under an invalid `window_split` or a negative `window_share_max` still removes the strand, ends its window when it was the last pane, and reports the invalid key.
- A group whose last strand is removed records its choice and size afresh at its next join; a dead pane still bound in a shared window counts toward the group's live count.
- The status bar shows a button for each own window in its segment color, marked unseen until the operator first selects it, and carrying its strands' wait marks; `@lyx_hidden` windows still get none.
- A shuttle run with no anchor lands in its own window; the loom driver and the orch stay in window 0; attach to a strand recorded in window 0 neither moves nor refuses it.
- A seat table of two shares one window, which ends when the table returns, including when one of its advisors died; a table above the cap gives each seat its own; two rounds of one table never share a window.
- `lyx shuttle open --role explore` adds a strand named `<shortname>:<slug>:explore` in its own window, with its session id recorded so name repair restores a drifted session name, writes no run record, and the window closes when the session exits; a second call gives `explore-2`; a call under a live pause is refused; after a server restart `Resume` reopens the opened session under its recorded session name in its own window, and its window still closes when the session exits; a pair teardown with an opened session live does not run its self-removal; when the self-removal fails, the dead pane stays bound until `lyx reed remove`, and a later `Resume` drops the strand rather than relaunching it.
- The Selvage, zoom and navigation-binding behavior of window 0 is unchanged.

## Notes for the plan writer

- `internal/reedengine/windowtarget.go`, `overlay.go` (`listPanes`), `reconcile.go`, `strand.go` (anchor validation), `spawn.go` (`launchStrandLocked`), `revive.go`/`lifecycle.go` (Resume), `namerepair.go`, `bar.go` and `bindings.go` are where window 0 is assumed.
- `internal/reedengine/window.go` (`OpenWindow`) shows the detached `new-window` invocation and its exact-name option chaining.
- `internal/shuttleengine/spec.go` and `attach.go` hold the anchor default; `internal/seatengine/prompt.go` (`seatSpec`) holds the seat Display, and `wait.go` the done-only strand removal.
- D5's dead-strand removal reaches `removeStrandIfLive` in `internal/shuttleengine/stop.go`, the live skip in `stepstop.go` (`StopStepRuns`) and `removeSupersededStrands` in `attach.go`.
