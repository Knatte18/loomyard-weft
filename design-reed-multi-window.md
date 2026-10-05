# reed: cross-worktree columns, own-window anchoring, per-window attach

`reed-multi-window` · Note · feature, reed

From `manifest/roadmap.md` Someday, three items kept together:
- **reed: cross-worktree columns.** All worktrees in one tmux window, a column per worktree; needs a name for the new per-worktree grouping layer this introduces and a column-count/fallback policy.
- **reed: own-window strand anchoring.** A `display` anchor that spawns a strand into its own switchable tmux window instead of a pane.
- **reed: independent per-window attach via tmux session groups.** tmux's session-groups feature would let multiple `lyx reed attach` invocations show different windows independently, instead of sharing one today.

## Design

> Three related, not-yet-scoped ideas about reed outgrowing "one pane per strand, one window per worktree." Grouped here because each is a candidate building block for the others.

## Cross-worktree columns

All worktrees in one tmux window, a column per worktree.

Needs its own name for the per-worktree grouping layer this introduces — it cannot be called "session": that's already tmux's own term, and already 1:1 with a worktree in reed's plumbing today. Also needs a decision on how many columns fit before falling back to tmux windows-as-pages.

Candidate group-layer names surveyed so far and still free: **Heddle, Batten, Bobbin, Sley**. Already taken elsewhere in this codebase: Warp, Weft, Shuttle, Treadle, Shed, Loom, Reed, Strand, Fabric, Quarry, Crucible, and Selvage (claimed by the shipped header-replacement work — see `internal/reedengine`'s `selvagepane.go`).

## Own-window strand anchoring

A `display` anchor that spawns a strand into its own switchable tmux window instead of a pane.

## Independent per-window attach via tmux session groups

Today two `lyx reed attach` invocations join the same tmux session object — a shared "current window" pointer, and mismatched terminal sizes fighting over layout.

tmux's session-groups feature (`new-session -t <existing-session>`) would let each invocation show a different window independently, which reed's `attach` verb does not use today. This is a candidate building block for both ideas above, likely exposed as something like an `--window <n>` flag, or hidden entirely behind one verb that spawns the needed loose terminal windows automatically.

## Operator, 2026-10-05: strands in depth, not stacked in height

Two strands stacked as panes in one window are unreadable on a small laptop screen.
Stack them in depth instead: one tmux window per strand, behaving like tabs within the run's terminal window.

- The driver always lives in window 0, with its `loom-status` strand beside it.
- Each agent the driver spawns (Discussion-Write, a Burler round, Webster's Master) gets its own window, named by its role, so the tmux status line lists what runs.
- The window closes when the agent ends, and the view falls back to the driver.
- **No focus stealing.** A new window must never pull focus away from a window the operator is typing in; an automatic switch on spawn is unwanted for that reason.
  Open: how a new window announces itself instead (status-line highlight, tmux activity flag, a switch only when the current window is idle), and whether tmux session groups are needed so each attached terminal keeps its own current window.

## Related

- `internal/reedengine`'s package documentation — the shipped work that claims the "Selvage" name; reed still has no window support today.
