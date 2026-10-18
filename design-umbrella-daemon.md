# [WS 005] One daemon per umbrella owns its batten runs

`umbrella-daemon` · Note · enhancement, batten, orch, shed, workspaces

## Seen (2026-10-18)

- Every batten run is its own Go process in a tmux window, and its Run-Shed row waits on the child with no time limit (`internal/battenshed/doc.go`): it sends halt notices, retries the first verify failure, revives a dead driver's strands, and lands and tears down once the child is done.
- Nothing restarts those processes after a machine restart; `lyx batten run <slug>` resumes from disk, and each row is idempotent, but someone must run it.
- The orch already has a Go daemon of its own, the watcher (`lyx orch watch`).

## Operator view, not a decision (2026-10-18)

An orchestrator gets not only a long-lived thread but a daemon that looks after every batten run it owns (those whose parent is its worktree) and sets their next steps going; see `orch-split`.

## Agreed in discussion (2026-10-18)

- One daemon per umbrella (see `orch-any-worktree`), the orch watcher, replaces one waiting batten process per run.
- The daemon follows its runs' status files (fsnotify, with a periodic re-read as a fallback where file watching is unreliable, such as WSL and junctions); the child only writes its status, as today, and knows nothing of its owner.
- Creel only where an agent must be told: the daemon posts to its thread when something needs judgement.
- The same pass finds dead drivers, so after a restart only the daemons need starting, one per umbrella.

## Orch recommendation, not a decision

1. Step 1: the Run-Shed row checks the child once and exits while it still runs; the daemon re-runs `lyx batten run <slug>` for each run it owns, so notices, the verify retry, revival and landing stay batten's own code, run in short passes.
2. Step 2, optional and later: move logic from batten into the daemon.

Open: who owns a run whose parent has no running daemon, the same problem as a thread that is not running.
