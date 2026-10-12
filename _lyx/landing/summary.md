# Guard pause Esc by the pane and close an open rewind picker

`lyx pause` no longer opens Claude Code's rewind picker in idle drivers.
In the first live hub pause, both run drivers were idle at their prompt while a background `lyx shed step` ran, and the pause's Esc left each in the picker.
A later `lyx resume` would have typed its line into that dialog.

## What changes

- **Pause reads only a running turn as busy.**
  `shuttleengine` gains `SessionState.RunningTurn` (state `busy` with any cause but `background`), shared by send readiness and the pause.
  A session reading `busy`/`background` holds no grace and gets no Esc.
  It is listed under a new report `background` list with the labels of its outstanding tasks, and it is not recorded in the pause entry, so resume gives it no line.
- **Every interrupt key is guarded by the pane.**
  `Runner.Interrupt` and `Run.Interrupt` capture the pane and send the key only when it shows a running turn.
  Otherwise they send nothing and return the new `ErrNoRunningTurn`; a failed capture also sends nothing.
  `lyx shuttle interrupt` on a session with no running turn now fails, naming `lyx shuttle send`, instead of sending Esc.
  The pause, the orch watcher and the planned `creel` inject-with-interrupt treat `ErrNoRunningTurn` as success.
- **A second Esc waits out a provider settle time.**
  `claudeengine` exposes a 2 second settle, longer than Claude Code's double-Esc window.
  A retry waits at least that long after the first Esc and goes through the guard, so at most two Escs per session per pause, never close enough to open the picker.
- **An open rewind picker is closed once and fails the operation.**
  `claudeengine` reads the picker from its heading, pinned by the captured pane `testdata/pane-rewind-picker.txt`.
  In `Runner.Interrupt`, `Run.Interrupt` and the send path, a pane read as the picker gets exactly one Esc, a re-capture after the settle, and `ErrRewindPickerOpen`; nothing is typed.
  The pause lists such a session under `refused`, the orch watcher logs it at Warn and sends no further Esc, and resume withholds its line.
- **Report lists.**
  `interrupted`, `still_busy`, `background`, and `no_running_turn` (state read a running turn, the guarded Esc found none on the pane) are distinguished.
  `no_running_turn` sessions stay in the entry and get the resume line.
- **Resume** drops its separate busy pre-check and reports a send refused with `ErrSessionBusy` with the guard's own readiness reason (`SessionBusyError` now carries it) instead of a fixed "draft" text.
- **Orch watcher** never blocks a tick on the settle.
  It remembers per hub entry when its first Esc went and sends the guarded second Esc on a later tick past the settle, after its entry-live check.
  A refusal is logged once per hub entry at Info naming the orch strand.
- **`Runner.Inject` refuses any input list containing a key of the engine's interrupt sequence** and returns an error naming `lyx shuttle interrupt`, so no caller carries an Esc past the guard.
- `PATTERN-guarded-interrupt` is new in `PATTERN.md`'s "Agents and prompts" group, `PATTERN-pane-text-injection` points at it, and the docs of `pauseengine`, `orchengine`, `shuttleengine` and `claudeengine` and `docs/overview.md` are updated.

## Design decisions

- The running-turn and picker readings and the settle time stay in `internal/shuttleengine/claudeengine`; `shuttleengine`, `pauseengine` and `orchengine` see provider-neutral capabilities, and the settle reaches the pause engine through its `Pair` seam and the watcher through its `Session` seam.
- The guard can only withhold a key.
  It lets through an Esc whose turn ends between capture and key, and a capture whose transcript quotes the running-turn hint; either costs one Esc on an idle prompt, which opens nothing.
  Two operator-run `lyx shuttle interrupt` calls inside the settle time onto such a pane can still open the picker; the next send or interrupt closes it and fails.
- The pause and send readiness share the state rule, not the whole reading.
  A folded state reading idle while the pane shows a running turn lands in `already_idle`; a `busy`/`turn` state whose turn-start hold released reaches the guard and lands in `no_running_turn`.
- The pause entry's shape is unchanged; the new lists live in the report only.
- A watcher restarted during the settle loses the remembered first-Esc time and sends no second Esc.
- Out of scope: other dialogs, holding Go producers and `lyx shed step` processes under a pause (they keep running by design), and a turn that starts after the interrupt step, such as an agent woken when its background task ends.

## To check by hand

Not run:

- `lyx pause` against a driver idle at its prompt with a background `lyx shed step` sends no Esc and opens no picker; against a driver mid-turn it sends one Esc and leaves no picker.
- The two driver panes left in the picker on 2026-10-12 are closed by the operator, not by this change.

Findings left open by plan review:

- Pause settle waits run serially per session in `internal/pauseengine/pause.go`, so a pause over many busy sessions waits the settle once per session.
- The `shuttlePair` readings test in `internal/pausecli/deps_test.go` may pin a private helper; check it asserts behaviour through the seam.

Risks accepted:

- A session in `background` wakes when its task ends, and that turn runs under the pause uninterrupted.
- A Claude Code release that rewords the running-turn hint makes the guard withhold every Esc (visible under `no_running_turn`); rewording the picker heading leaves the picker undetected, and the send and interrupt paths fall back to their existing refusals.
- A resume over a pane with no input box waits shuttle's send-ready timeout per session before reporting the refusal.
- Assumes Claude Code's double-Esc window is under 2 seconds.
