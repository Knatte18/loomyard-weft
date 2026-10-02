# See every run at once: an own tmux window for the hub runs

`runs-window` · Someday · feature

## Goal

The operator sees many runs at once without squeezing the orch pane.

## Notes

- Operator 2026-10-02: a status strand under orch would need its height to follow the number of runs, which reed cannot do; an own tmux window may fit better.
- reed today lays out one strand per pane in one window per worktree; `manifest/designs/reed-multi-window.md` sketches own-window strand anchoring, independent per-window attach via tmux session groups, and cross-worktree columns.
- `runs-overview` (one line per run with state, producer and age) is the content such a window would show.
- Deep enough to stand alone, separate from `orch-owns-the-loop`.
