# reed: runs overview, gate checklist, Selvage in cycle, run windows close

The operator gets one place in the prime's reed session that shows every run in flight and its loom stage, without polling each run or reading the orch's chat.
Each task pair gets a gate checklist pane, Selvage joins the Alt+arrow cycle, and the `lyx batten run --window` terminal window closes when its run ends.

## What changes

- **Runs strand.**
  `lyx orch start` ensures a `runs` strand directly above the orch, running the new hidden `lyx batten overview` verb.
  It shows one line per run in flight: slug, batten state, loom stage, time in stage, run age and the pair's reed session name.
  A run leaves the list when it reaches `done` or its pair is gone.
  The pane reads status files without the status lock and redraws only when a line changes.
  `lyx orch stop` leaves the strand.
- **Strand sizing in reed.**
  `render.Display` gains an optional size declaration persisted in reed.json: a `top` anchor, a content kind and a sharing kind.
  A strand that declares nothing lays out exactly as before, and strands persisted by an older lyx stay positional.
  A content strand reports its line count through the new `Engine.SetContentRows`, which refuses an unknown strand, a strand without content sizing, a foreign pane id and a count below 1.
  The new `internal/statuspane` package holds the shared pane loop, frame fitting and report.
- **Seat tables.**
  The seats engine declares the chair and advisors as sharing strands, so side-by-side agents get equal heights.
- **Gate checklist.**
  A hidden `lyx loom` verb renders the run's recipe rows as a checklist in the existing `loom-status` strand, now kept on both child drivers.
  It shows the state line, then per row a box, the time spent summed over visits and the visit count above one.
- **Hub notice log.**
  Batten appends every notice it composes once per process to a hub log under the prime's `.lyx`, with UTC time, text and the run's attach command.
  The path comes through `internal/lyxdirs` and `internal/shedrun`, resolved in `battencli` and told to `battenshed`.
  A failed append is logged and never changes the notice.
  `lyx batten` help names the path.
- **Selvage in the cycle.**
  Alt+Up and Alt+Down step through the strand panes and Selvage as one wrapping cycle, and Alt+z zooms Selvage.
  Selvage is found through a display-only pane marker that no Go decision reads.
- **Run window.**
  The terminal window ends on a 0 attach.
  On any other status it prints the status, waits up to 30 seconds for the pair directory to disappear, and otherwise keeps a shell.

## Design decisions to know

- Content height is `min(content rows, max(min rows, floor(share × free height)))`.
  The runs strand uses a third share with 3 rows minimum, the checklist a quarter share with 3 rows minimum.
  The constants are in code, not reed.yaml.
- While any sharing strand is live, positional strands collapse; with none live, the bottom-most positional strand expands as before.
- A `top` anchor with a positional or sharing kind is refused by Rules.
- Window-resized pins now cover every content strand as they cover Selvage.
- A lone top status strand is pinned at the full box height, not its content height.
- The overview's hidden-runs summary counts every run not shown.
- The overview's attach command moved to the notice log.
- The checklist now shows on llm-driven runs, at the quarter share.
- `statuspane` took over loom's build identity and sidecar, so `statusbuild.go` is gone from `loomcli`.
- Docs updated: `reedengine`, `render`, `orchengine`, `battenshed`, `loomcli`, `termwindow`, `seatengine`, `docs/overview.md` and `PATTERN-daemon-wakeups`.

## To check by hand

- Run the `tmux`-tier tests: the new `reedengine` test and the other `tmux`-tier edits were compile-checked only.
- Verify on the installed tmux that a session killed while attached ends the client with status 0, and that a server shutdown ends it non-zero.
  With `detach-on-destroy off` in an operator `tmux_conf`, the run window moves onto another session and stays open.
- Check the overview's behaviour when a run's seed is undecodable: the spec does not settle whether such a run shows as in flight, so the plan's choice in `internal/battencli/overview.go` needs a look.
- Check the checklist's per-row time, which is wall time between history stamps and includes halt and wait time before the call that ends a visit.
  The wait before a `goto` entry is dropped.
- Check that a notice held back or dropped is logged again when a restarted batten composes it again, since the append is keyed on nothing persisted.
- The plan-level integration suite failed once on `internal/lyxtemp.TestSweep/two_concurrent_sweeps_both_succeed`, which passed on rerun; it is flaky and not attributed to this change.
- Look at the strand window on a small laptop screen: seat agents sharing rows may be cramped.
