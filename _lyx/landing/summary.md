# reed: runs overview pane, gate checklist, Selvage keys, run window close
The prime's reed session gains a `runs` strand above the orch that lists every batten run in flight with its loom stage.
Every task pair's `loom-status` strand becomes a gate checklist of the run's recipe rows.
Selvage joins the Alt+Up/Alt+Down cycle, and the `lyx batten run --window` terminal window closes when its run ends.
Before this, the operator polled each run separately and saw run events only through the orch's chat.

No board issue is attached to this task, so there is no `Closes` line.

## What changes

- **Runs overview.**
  `lyx batten overview` is a new hidden pane verb that redraws one line per run in flight: slug, batten state, loom stage, time in that stage, run age and the pair's reed session name.
  `lyx orch start` ensures it in a strand named `runs` before the orch strand, and replaces a dead `runs` strand or one launched by another lyx build.
  `lyx orch stop` leaves it.
  A run shows until its batten state is `done` or its pair is gone.
  Halted and failed runs with a pair still show.
  With no run in flight the pane shows `no runs in flight`.
- **Gate checklist.**
  A new hidden `lyx loom` pane verb runs in the existing `loom-status` strand and shows the state line plus one line per recipe row in recipe order: a box, the row name, the time summed over its visits, and the visit count above one.
  The strand now stays on llm-driven runs too, which previously removed it.
- **Strand sizing in reed.**
  `render.Display` carries an optional size declaration persisted in reed.json, and the anchor vocabulary gains `top`.
  The kinds are positional (today's rule, still the default), content (height follows the pane's line count) and sharing (an even split of the rows the fixed panes leave).
  A strand persisted by an older lyx lays out positionally, and a window of positional strands lays out as before.
- **Content strands report their line count.**
  `reedengine.Engine.SetContentRows` takes a strand name, a pane id and a count, and re-applies the layout under the zoom bracket.
  It refuses an unknown strand, a strand without content sizing, a pane id other than the recorded one, and a count below 1, so a hand-run `lyx batten overview` resizes nothing.
- **Seat tables.**
  The seats engine declares its chair and advisor strands as sharing, so side-by-side agents get equal heights.
- **Hub notice log.**
  Each batten process appends every notice it composes once, at its first delivery attempt, to one hub log under the prime's `.lyx`: UTC time, text and the run's attach command.
  `lyx batten`'s help names the path.
- **Selvage keys.**
  Alt+Up and Alt+Down cycle through the strand panes and Selvage in physical order, wrapping, keeping the zoom.
  Alt+z zooms and unzooms Selvage.
  A hidden batten window's pane stays outside both.
- **Run window close.**
  The window ends on an attach that exits 0.
  On any other status it prints the status and waits up to 30 seconds for the pair directory to disappear, then keeps an interactive shell.
  Konsole and Windows Terminal under WSL share the one line, built through `internal/shell`.

The loom build identity and sidecar move from `loomcli` into the new `internal/statuspane` package, which also holds the pane loop, frame fitting, the content-rows report and the status cache shared by both pane verbs.
Docs for `reedengine`, `render`, `orchengine`, `battenshed`, `loomcli`, `seatengine` and `termwindow`, and `PATTERN-daemon-wakeups`, are updated.

## Design decisions

- Size math lives in `render`'s policy layer, which stays a pure leaf.
  Shares and minimums are constants in code, not reed.yaml keys: `runs` takes a third of the free height with at least 3 rows, the checklist a quarter with at least 3 rows.
- A top-anchored positional or sharing strand is refused by `render`'s rules, as `own-window` is.
- While a sharing strand is live, positional strands collapse; with none live, the bottom-most positional strand expands as today.
- The overview and checklist read status files without the run's status lock, relying on `internal/state` writing them atomically.
  A file that fails to read on one poll keeps its previous line, and on two polls shows `status unreadable`.
- A checklist row's time is wall time between history stamps, so it includes halts and waits before the call that ends the visit; a `goto` entry adds no time and starts the next visit.
- The notice log append is keyed on nothing persisted, so a notice held back or dropped and composed again by a restarted batten is logged again.
  The log is never rotated.
- Selvage is found by a display-only pane marker that no Go decision reads; it still gets no `@strand` or `@strand_color`.
- Two departures from the plan wording: a lone top status strand is pinned at the full box height rather than its content height, and the overview's hidden-runs line counts every run not shown (`… 2 more runs not shown`).

## To check by hand

- Run `lyx orch start` with two runs in flight and confirm the `runs` strand sits above the orch with two lines, grows and shrinks as runs start and land, and keeps a zoomed orch or zoomed Selvage zoomed.
- Run `lyx gate test --tags tmux` for `internal/reedengine` and `internal/termwindow`: the `tmux`-tier edits, including the new `reedengine` test, were only compile-checked.
- Close a run window after a normal landing on Konsole and on Windows Terminal under WSL.
  A non-zero attach status should wait for the worktree removal; the code expects a killed session to end the client with 0 and a server shutdown with non-zero, which was not measured on the installed tmux.
- An operator `tmux_conf` with `detach-on-destroy off` moves the window's client to another session when the pair session ends, so the window stays open.
- Seat agents sharing rows on a small screen may be cramped.
- The window-resized hook array gains one pin per content strand; the resize behaviour measured for Selvage is assumed to hold for more pinned panes.
- A status file that fails to decode while the run's seed is undecodable: the plan review left open what the overview shows for such a run.
  Check that it appears last, ordered by slug, and renders without hiding the other lines.
- The checklist's wall-time attribution across a halt (the wait before a resume counts in the row; the wait before a `goto` does not) is a design choice to confirm against real halted runs.
- The plan-level integration suite failed once on `github.com/Knatte18/loomyard/internal/lyxtemp.TestSweep/two_concurrent_sweeps_both_succeed`, which passed on rerun and was not attributed to this change.
