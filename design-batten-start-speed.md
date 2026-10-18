# Fast batten start and local-only fabric pairs

`batten-start-speed` · Task · enhancement, batten, fabric, shed

## Seen (2026-10-17/18)

- Batten's start of `creel-notices` (row times from `lyx batten status creel-notices`): Worktree-Create done 13:15:13, Seed-Child about 1 s, Open-Window about 8 s, Seed-IDE about 2 s, Open-IDE about 2 s; about 10 to 12 s before Run-Shed, as the operator sees it.
- Open-IDE is off in the hub and does nothing, yet its row took about 2 s, as did Seed-IDE: each batten row seems to cost about 2 s of overhead on its own.
- Open-Window does not wait for the window: `OpenAttach` in `internal/termwindow/termwindow.go` returns after `cmd.Start()`, and opening Konsole takes milliseconds; the other 6 s are unexplained (`termwindow.Resolve`, the config read from the pair's stack in `internal/battencli/wire.go` `switchedActions`, or the row's own status and history writes).
- `lyx fabric add` in the sandbox hub (5-file repo) took about 1.1 s, 92% of it two serial `git ls-remote` origin probes; `lyx fabric remove --remote` took 6 to 7 s, almost all network (force-with-lease push, archive-tag push, weft branch delete). `internal/fabricengine/add.go` has `SkipGit`/`SkipPush` for tests only; `internal/fabricengine/remove.go` always pushes an archive tag when the weft has an origin.

## Scope

1. Measure where batten's start goes, row by row and inside each row, and cut the overhead so a row that does nothing costs milliseconds and Open-Window costs no more than starting the window.
2. Local-only pairs in fabric: an add with no origin probes and a remove with no push, archive tag or remote delete, for pairs that never reach origin (the batch pairs `weaver` and `parallel-batch-parts` describe).

Out: Weaver itself; the overlap with `command-producer` (one generic producer for the flag-gated side-action rows) is left to the operator to bundle or not.
