# Batten opens a terminal window per run instead of VS Code

`ide-opt-in` · Task · enhancement, ide, reed, batten · running · Finalize

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
## Done on 2026-10-08

Commit c0d604f12 removed the VS Code window batten opened per run (`battenshed/innerrun.go`, `ideengine/spawn.go`); VS Code now opens only on request.
What remains: batten opening the terminal window per worktree itself.
Today the hub orchestrator opens it by hand with `setsid -f konsole --workdir <pair> -p tabtitle=<slug> -e bash -lc "lyx reed attach; exec bash"` (CLAUDE.md, Watching runs), and there is no launcher for WSL (`wt.exe -w new …`) or other platforms.

## Task

`lyx batten run <slug> --window` opens one terminal window for the run's pair that runs `lyx reed attach` there, once the pair exists, titled with the slug.
The launcher is chosen per platform: Konsole on Linux (as in CLAUDE.md today), `wt.exe -w new` into `wsl.exe` on WSL; where no launcher is known the run goes on and the start output says no window was opened.
Re-running `lyx batten run` on an existing run opens no second window.
CLAUDE.md "Watching runs" and the orch role stencil then say batten opens the window, and drop the hand-written Konsole line.
