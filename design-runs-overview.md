# One-screen overview of every run in the hub, in its own tmux window

`runs-overview` · Notes · feature

From `manifest/HANDOFF.md`: one screen listing every run in the hub, one line each with state, producer and age.
`lyx shed status [run-id]` is per run and `lyx orch status` covers only the orchestrator, so the hub today polls each run separately.


## Where it shows


- Operator 2026-10-02: a status strand under orch would need its height to follow the number of runs, which reed cannot do; an own tmux window may fit better.
- reed today lays out one strand per pane in one window per worktree; `manifest/designs/reed-multi-window.md` sketches own-window strand anchoring, independent per-window attach via tmux session groups, and cross-worktree columns.
- Deep enough to stand alone, separate from `orch-owns-the-loop`.
