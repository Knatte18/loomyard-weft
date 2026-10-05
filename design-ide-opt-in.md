# VS Code only on request; other ways to watch a run

`ide-opt-in` · Note · enhancement, ide, reed, batten

From `manifest/roadmap.md` Someday, kept by the operator on 2026-10-03.
The common case is reviewing the final PR rather than watching an agent edit live, so VS Code per worktree should be opt-in.
Alternatives the operator has considered: separate terminals that pop up instead of VS Code, or tmux windows.
Check what batten already opens first.
Related: `batten-windows`, `ide-open-verb`, `runs-overview`.

## Operator, 2026-10-05

On a small laptop screen a reed session inside a VS Code window is hard to read.
Instead of VS Code, a run opens one terminal **window** per worktree that runs `lyx reed attach` there.
A window, not a tab: the operator tiles several runs side by side to watch them at once, which tabs cannot do.
On WSL that is a new Windows Terminal window (`wt.exe -w new …`) running `wsl.exe` into the worktree; other platforms need their own launcher.
Tabs fit the agents inside one run instead (see `reed-multi-window`).
VS Code opens only on request.